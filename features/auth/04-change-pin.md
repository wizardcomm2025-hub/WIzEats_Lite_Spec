# 04 — Change PIN

## Summary

A signed-in user replaces their PIN by entering the current PIN and a new one twice. The app forces this right after sign-in when the PIN was e-mailed (reset) or is a default, and shows nothing else until it is done. The API also accepts a change at any other time, though the app has no entry for that today.

## Prerequisites

- **Backend:** `POST /api/auth/changePin` — [api-contract.md](./api-contract.md#post-apiauthchangepin)
- **Data:** `users.pin`, `users.mustChangePin` — [data-model.md](./data-model.md)
- **Depends on:** [02](./02-sign-in-with-pin.md), [03](./03-reset-forgotten-pin.md)

## Actors & preconditions

- **Actor:** signed-in owner, waiter or chef.
- **Trigger:** the app opens Change PIN after sign-in.
- **Preconditions:** signed in; restaurant Accepted; `mustChangePin = 1` (for the app screen). The API needs only a valid token of a user with a restaurant.

## Main flow

1. The app shows `Set a new PIN` with three PIN fields.
2. The user enters the current PIN, a new PIN and the new PIN again.
3. `Change PIN` becomes active once all three are complete, confirm matches new, and new differs from current.
4. The server validates the fields and checks the current PIN against the stored hash.
5. The server stores the new PIN (hashed) and sets `mustChangePin = 0`.
6. The app opens: chef → KOT, owner / waiter → Home.

## Alternative flows

- **Confirm differs.** `New PIN and confirmation do not match` shows as soon as confirm is complete; button stays disabled.
- **New equals current.** `New PIN must be different from the current PIN`; button stays disabled.
- **Wrong current PIN.** 401 `Current PIN is incorrect`; the user stays on the screen and stays signed in.
- **No limit on tries.** Wrong current PINs are not rate-limited (AUTH-D7).
- **Logout instead.** `Logout` signs out at once, no confirmation; `mustChangePin` stays 1.
- **App killed before changing.** On restart the app shows Change PIN again.
- **Other sessions.** Other devices stay signed in (AUTH-D4).
- **No network.** `Could not change PIN`.
- **Double tap.** Button disabled while the request runs.

## Acceptance criteria

1. Given `mustChangePin = 1` and an Accepted restaurant, when the user signs in, then only `Set a new PIN` is shown — no drawer, no other screens.
2. Given correct current PIN and matching new PINs, when `Change PIN` is tapped, then the app opens (Home, or KOT for a chef).
3. Given a successful change, when the user signs in again, then the old PIN gives `Invalid PIN` and the new PIN works and does not ask to change again.
4. Given a wrong current PIN, when `Change PIN` is tapped, then `Current PIN is incorrect` shows and the user remains signed in on this screen.
5. Given confirm differs from new, then `New PIN and confirmation do not match` shows and the button is disabled.
6. Given new equals current, then `New PIN must be different from the current PIN` shows and the button is disabled.
7. Given the user taps `Logout`, then the Login phone step shows and the next sign-in shows Change PIN again.
8. Given the app is closed on this screen, when reopened, then Change PIN shows again.

## Platform notes

### POS API

`POST /api/auth/changePin`, `auth:sanctum` + `restaurant.tenant`, no rate limit. FormRequest `ChangePinRequest`. The user is always the token's user. Messages in [api-contract.md](./api-contract.md#post-apiauthchangepin).

### Data — MySQL

Writes `users.pin` (hashed), `users.mustChangePin = 0` for the token's user only. Tokens untouched.

### POS app

`ChangePinScreen`, thunk `changePin` (sends `newPin_confirmation` = `newPin`). Shown by `RootNavigator` only when Accepted and `mustChangePin`. See [ui-ux.md](./ui-ux.md#change-pin).

### Super-admin panel

Not touched.

### Integrations

Not touched.

## Configuration

PIN length (6) is fixed in the API rules and the app.

## Out of scope

- Changing PIN by choice from Profile — not in the app (AUTH-D9).
- Owner setting a staff member's PIN — STAFF.

## Open questions

- None open. Related decisions: AUTH-D7, AUTH-D9 in the [README](./README.md#decisions).
