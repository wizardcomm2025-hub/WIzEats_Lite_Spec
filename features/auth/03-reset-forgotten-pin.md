# 03 — Reset a forgotten PIN

## Summary

A user who has forgotten their PIN taps `Forgot PIN?` on the PIN step. The server creates a new random PIN, e-mails it to the restaurant's e-mail address and marks the account so the user must choose their own PIN after signing in. The PIN goes to the restaurant, not to the person, so a waiter or chef gets it from the owner.

## Prerequisites

- **Backend:** `POST /api/auth/forgotPin` — [api-contract.md](./api-contract.md#post-apiauthforgotpin)
- **Data:** `users.pin`, `users.mustChangePin`, `restaurants.restaurantEmail` — [data-model.md](./data-model.md)
- **Integrations:** outgoing mail (`PinMail`)
- **Depends on:** [01](./01-check-phone.md), [04](./04-change-pin.md)

## Actors & preconditions

- **Actor:** owner, waiter or chef, not signed in.
- **Trigger:** taps `Forgot PIN?` on the PIN step.
- **Preconditions:** the number has an account.

## Main flow

1. The app clears the PIN boxes and sends the phone number.
2. The server finds the user and their restaurant.
3. The server generates a new random 6-digit PIN.
4. The server e-mails the PIN to the restaurant's e-mail address.
5. Only after the mail is sent, the server stores the new PIN (hashed) and sets `mustChangePin = 1`.
6. The app shows `A new PIN has been sent to <masked e-mail>` in green.
7. The user signs in with the new PIN ([02](./02-sign-in-with-pin.md)) and is taken to Change PIN ([04](./04-change-pin.md)).

## Alternative flows

- **Restaurant has no e-mail.** 422 `No email is registered for this account. Please contact support.`; PIN unchanged.
- **Mail fails.** 500 `Could not send the PIN email. Please try again later.`; the old PIN still works.
- **Too many resets.** A 4th reset within 10 minutes for one phone (or the 11th in a minute from one IP) gets `Too Many Attempts.`; nothing changes.
- **Switched-off user.** The reset still happens (AUTH-D6); sign-in is then refused with `Your account is inactive.`
- **Reset by someone else.** Anyone who knows the number can trigger it; the old PIN stops working at once (AUTH-D6).
- **Other sessions.** Devices already signed in stay signed in (AUTH-D4).
- **No network.** `Could not send a new PIN`.
- **Double tap.** `Forgot PIN?` is disabled while the request runs.

## Acceptance criteria

1. Given a user whose restaurant has an e-mail, when `Forgot PIN?` is tapped, then `A new PIN has been sent to <masked e-mail>` shows and the restaurant receives an e-mail with a 6-digit PIN.
2. Given a reset was sent, when the user signs in with the old PIN, then `Invalid PIN` shows.
3. Given a reset was sent, when the user signs in with the e-mailed PIN in an Accepted restaurant, then the Change PIN screen opens.
4. Given a restaurant with no e-mail, when `Forgot PIN?` is tapped, then `No email is registered for this account. Please contact support.` shows and the old PIN still works.
5. Given the mail server is down, when `Forgot PIN?` is tapped, then `Could not send the PIN email. Please try again later.` shows and the old PIN still works.
6. Given 3 resets within 10 minutes, when a 4th is tried, then `Too Many Attempts.` shows and the last e-mailed PIN still works.
7. The masked e-mail shows the first 2 characters of the name part, stars for the rest, and the full domain.
8. The new PIN never appears in the app or in any response.

## Platform notes

### POS API

`POST /api/auth/forgotPin`, public, throttle `forgotPin` (3 / 10 min per phone, 10 / min per IP). FormRequest `PhoneRequest`. Messages in [api-contract.md](./api-contract.md#post-apiauthforgotpin).

### Data — MySQL

Reads `users`, `restaurants`. On success writes `users.pin` (hashed) and `users.mustChangePin = 1`. Tokens untouched.

### POS app

`PinStep` → `Forgot PIN?`, thunk `forgotPin`; success text in `notice` (green). See [ui-ux.md §4](./ui-ux.md#4-feedback-states).

### Super-admin panel

Not touched. (The restaurant e-mail is edited in RST / ADM.)

### Integrations

Mail `PinMail` with reason `reset`, to `restaurants.restaurantEmail`. Mail failures are logged to the Google Sheets error log and the server log.

## Configuration

Mail transport in `.env` / `config/mail.php`. Rate limits in `AppServiceProvider`.

## Out of scope

- Resetting a PIN by SMS — not offered.
- Owner resetting a staff member's PIN from the app — STAFF.

## Open questions

- None open. Related decisions: AUTH-D4, AUTH-D6 in the [README](./README.md#decisions).
