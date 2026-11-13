# 04 — Legacy setup with a token

## Summary

Older app versions created the restaurant first (through the legacy register endpoint) and then sent the profile with the owner's token. `/api/restaurants/setup` still accepts that call. Today it accepts any valid token — any role, any restaurant state — and overwrites the caller's restaurant profile and settings each time. The current app never sends a token here; this file records the behaviour so developers can test it (ONB-D1, accepted risk).

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
2. It finds that user's restaurant.
3. In one transaction it overwrites the restaurant's name, address, owner name and e-mail; sets the contact phone to the caller's phone, `createdBy` to the caller's role and onboarding done; renames the caller to the owner name given.
4. It overwrites the five settings flags (the restaurant code is kept), makes the first zone `Main Hall` and active, adds Table 1–4 only if that zone has none, and makes sure the four order channels exist.
5. It issues a new token for the caller and answers 200 with the restaurant and the caller's `mustChangePin`.
6. After the commit it sends the admin notice again. No PIN e-mail. Approval is unchanged.

## Alternative flows

- **Caller is a waiter or chef.** Allowed; the effects above apply to their restaurant, and the waiter's / chef's own name becomes the owner name given (ONB-D1).
- **Restaurant already Accepted.** Allowed; stays Accepted with the new details (ONB-D1).
- **Repeated calls.** Each one overwrites again and issues another token.
- **Caller has no restaurant.** 404 `Restaurant not found`; nothing written.
- **Invalid / revoked token.** Treated as no token: `phone` is then required and must be new — a new restaurant may be created ([01](./01-register-restaurant.md)).
- **Validation.** As [01](./01-register-restaurant.md), except `phone` is not required.

## Acceptance criteria

1. Given an owner token of a restaurant with onboarding not done, when setup is called with a full body, then 200, onboarding is done, the details are saved, and the starter data exists.
2. Given a waiter token of an Accepted restaurant, when setup is called, then 200 and the restaurant's name, address, owner name, e-mail, settings and contact phone are replaced; the first zone is named `Main Hall`; the waiter's name becomes the owner name given; the restaurant stays Accepted.
3. Given any token call, then no new restaurant or user is created and `phone` in the body has no effect.
4. Given a token whose user has no restaurant, then 404 `Restaurant not found` and nothing changes.
5. Each successful call issues one more token for the caller.

## Platform notes

### POS API

Same endpoint as [01](./01-register-restaurant.md); token via `auth('sanctum')`; no role or approval check; same 10 / min limit.

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

- Restricting this path — deferred by ONB-D1.
- Sending legacy owners to this form from the app — not done (ONB-D4).

## Open questions

- None open. Related decisions: ONB-D1, ONB-D4.
