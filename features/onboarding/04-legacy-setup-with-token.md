# 04 — Legacy setup with a token

## Summary

Older app versions created the restaurant first (through the legacy register endpoint) and then sent the profile with the owner's token. `/api/restaurants/setup` still accepts that call. Today it accepts any valid token — any role, any restaurant state — and overwrites the caller's restaurant profile, settings and channel switches each time. The current app never sends a token here. *Target (ONB-D1, To fix):* only an active owner of an active restaurant whose onboarding is **not** done may use it, once.

## Prerequisites

- **Backend:** `POST /api/restaurants/setup` with `Authorization: Bearer` — [api-contract.md](./api-contract.md#post-apirestaurantssetup)
- **Data:** as [01](./01-register-restaurant.md)
- **Depends on:** [AUTH 07](../auth/07-legacy-otp-and-register.md) (legacy register), [AUTH 02](../auth/02-sign-in-with-pin.md)

## Actors & preconditions

- **Actor:** any caller holding a valid token (old app, or anyone).
- **Trigger:** a direct API call.
- **Preconditions:** a valid token. The tenant middleware is **not** applied.

## Main flow

1. The server takes the user from the token; `phone` in the body is ignored.
2. It finds that user's restaurant. *Target (ONB-D1):* it refuses, writing nothing, if the user is switched off (403 `Your account is inactive.`), the restaurant is missing (404), switched off (403 `Your restaurant is inactive.`), the caller is not an owner (403 `You do not have permission to perform this action`) or onboarding is already done (409 `Restaurant is already onboarded`). Approval is not checked.
3. In one transaction it overwrites the restaurant's name, address, owner name and e-mail; sets the contact phone to the caller's phone, `createdBy` to the caller's role and onboarding done; renames the caller to the owner name given.
4. It overwrites the five settings flags (the restaurant code is kept), makes the first zone `Main Hall` and active, adds Table 1–4 only if that zone has none, and makes sure the four order channels exist, setting each one's `isActive` from the flags (Dine In always on).
5. It issues a new token for the caller and answers 200 with the restaurant and the caller's `mustChangePin`.
6. After the commit it sends the admin notice again. No PIN e-mail. Approval is unchanged.

## Alternative flows

- **Caller is a waiter or chef.** Today allowed; the effects above apply to their restaurant, and the waiter's / chef's own name becomes the owner name given. *Target:* 403, nothing written.
- **Onboarding already done (any approval state).** Today allowed; everything is overwritten, approval unchanged. *Target:* 409 `Restaurant is already onboarded`, nothing written.
- **Repeated calls.** Today each one overwrites again and issues another token. *Target:* the first succeeds; every later one (and the loser of two parallel calls) gets 409.
- **User or restaurant switched off.** Today allowed. *Target:* 403, nothing written.
- **Caller has no restaurant.** 404 `Restaurant not found`; nothing written.
- **Invalid / revoked token.** Treated as no token: `phone` is then required and must be new — a new restaurant may be created ([01](./01-register-restaurant.md)).
- **Validation.** As [01](./01-register-restaurant.md), except `phone` is not required.

## Acceptance criteria

1. Given an owner token of a restaurant with onboarding not done, when setup is called with a full body, then 200, onboarding is done, the details are saved, and the starter data exists.
2. *(Target, ONB-D1)* Given a waiter token of an Accepted restaurant, when setup is called, then 403 `You do not have permission to perform this action` and nothing changes. (Today: 200 and the restaurant's details, settings, channel switches and contact phone are replaced, and the waiter is renamed.)
3. Given any token call, then no new restaurant or user is created and `phone` in the body has no effect.
4. Given a token whose user has no restaurant, then 404 `Restaurant not found` and nothing changes.
5. Each successful call issues one more token for the caller.
6. *(Target, ONB-D1)* Given an owner token of a restaurant whose onboarding is done, then 409 `Restaurant is already onboarded`, nothing changes and no token is issued.
7. *(Target, ONB-D1)* Given the caller or the restaurant is switched off, then 403 with `Your account is inactive.` / `Your restaurant is inactive.` and nothing changes.

## Platform notes

### POS API

Same endpoint as [01](./01-register-restaurant.md); token via `auth('sanctum')`; no tenant middleware, so the X-D2 middleware checks do not reach it — ONB-D1's target puts them in the service. Today no role or approval check; same 10 / min limit.

### Data — MySQL

As 01, but updates the existing rows; touches the caller's `users.name`.

### POS app

Not used by the current app.

### Super-admin panel

The restaurant's row shows the new details; status and approval unchanged.

### Integrations

Admin notice e-mail on each call.

## Configuration

None beyond 01.

## Out of scope

- Removing this path — not planned; ONB-D1 restricts it instead.
- Sending legacy owners to this form from the app — not done (ONB-D4).

## Open questions

- None open. Related decisions: ONB-D1 (To fix), ONB-D4.
