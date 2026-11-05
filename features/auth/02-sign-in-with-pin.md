# 02 — Sign in with PIN

## Summary

An owner, waiter or chef with a known number enters their 6-digit PIN. The server checks the account and the PIN and issues a session token. The app stores the session and opens the screen that fits the restaurant's approval, the PIN state and the user's role. This is the only sign-in the current app offers.

## Prerequisites

- **Backend:** `POST /api/auth/loginPin` — [api-contract.md](./api-contract.md#post-apiauthloginpin)
- **Data:** `users`, `personal_access_tokens`, `restaurants` (read) — [data-model.md](./data-model.md)
- **Depends on:** [01](./01-check-phone.md) (PIN step), ONB (Pending-approval screen), STAFF (accounts and first PIN)

## Actors & preconditions

- **Actor:** owner, waiter or chef.
- **Trigger:** enters 6 digits on the PIN step and taps `Login`.
- **Preconditions:** the number has an account ([01](./01-check-phone.md)).

## Main flow

1. The user enters 6 digits; `Login` becomes active.
2. The app sends phone and PIN.
3. The server finds the user by phone, checks `status = 1` and compares the PIN with the stored hash.
4. The server creates a new token and returns it with the user's role, restaurant, onboarding flag, approval and `mustChangePin`.
5. The app stores the token and the user summary on the device.
6. The app routes: not Accepted → Pending-approval (ONB); Accepted and `mustChangePin` → Change PIN ([04](./04-change-pin.md)); otherwise chef → KOT, owner / waiter → Home.

## Alternative flows

- **Wrong PIN.** 401 `Invalid PIN`; the user stays on the PIN step; nothing written.
- **User has no PIN set.** Same as wrong PIN.
- **Switched-off user.** 403 `Your account is inactive.` — even with the right PIN.
- **Restaurant pending, rejected or switched off.** Sign-in succeeds; the app shows the Pending-approval / Rejected screen (ONB). A switched-off restaurant is not checked (AUTH-D2).
- **Too many attempts.** The 6th try within a minute for one phone (or the 31st from one IP) gets `Too Many Attempts.`; no token, even with the right PIN, until the minute passes.
- **Number deleted between steps.** 404 `Phone number not found in our records`.
- **Already signed in elsewhere.** Allowed; each device gets its own token. Earlier tokens keep working (AUTH-D3).
- **No network / server error.** The app shows the server message or `Login failed`.
- **Double tap.** `Login` is disabled while the request runs.
- **Change number.** `Change Phone Number` returns to an empty phone step.

## Acceptance criteria

1. Given an active owner of an Accepted restaurant with `mustChangePin = 0`, when the correct PIN is entered, then Home opens.
2. Given an active chef of an Accepted restaurant, when the correct PIN is entered, then the KOT screen opens.
3. Given a user with `mustChangePin = 1` in an Accepted restaurant, when the correct PIN is entered, then only the Change PIN screen is shown.
4. Given an owner of a Pending restaurant, when the correct PIN is entered, then the Pending-approval screen is shown.
5. Given a wrong PIN, when `Login` is tapped, then `Invalid PIN` shows and no session is created.
6. Given a switched-off user, when the correct PIN is entered, then `Your account is inactive.` shows and no session is created.
7. Given 5 wrong tries within a minute, when a 6th try is made (even with the right PIN), then `Too Many Attempts.` shows and no session is created.
8. `Login` stays disabled until 6 digits are entered; letters can't be entered.
9. The PIN is never shown in clear on screen and never returned by the server.
10. Each successful sign-in creates exactly one new session; existing sessions on other devices keep working.

## Platform notes

### POS API

`POST /api/auth/loginPin`, public, throttle `loginPin` (5 / min per phone, 30 / min per IP). FormRequest `LoginPinRequest`. Order of checks and all messages in [api-contract.md](./api-contract.md#post-apiauthloginpin). Restaurant status and approval are reported, not enforced (AUTH-D2). Token name `pos-token`, abilities `*`, no expiry (AUTH-D3).

### Data — MySQL

Reads `users` by `phone`, then `restaurants` (`isOnboardingDone`, `isApproved`). Writes one `personal_access_tokens` row.

### POS app

`PinStep`, thunk `loginPin`. Stores `AUTH_TOKEN` and `USER` in device storage. Routing in `RootNavigator`. See [ui-ux.md](./ui-ux.md#1-screens-and-entry-points).

### Super-admin panel

Not touched. (The super admin's approve / reject and user on / off decide what this flow shows — ONB, ADM.)

### Integrations

Not touched.

## Configuration

Rate limits in `AppServiceProvider`. Token expiry `sanctum.expiration` (null — AUTH-D3).

## Out of scope

- Pending-approval and Rejected screens — ONB.
- Biometric sign-in — not offered.

## Open questions

- None open. Related decisions: AUTH-D2, AUTH-D3, AUTH-D8 in the [README](./README.md#decisions).
