# 01 — Check a phone number

## Summary

Anyone opening the app enters a mobile number. The app asks the server whether the number has an account. A known number moves to the PIN step; a new number goes to restaurant onboarding. This single entry step serves both existing staff and new restaurant owners.

## Prerequisites

- **Backend:** `POST /api/auth/checkPhone` — [api-contract.md](./api-contract.md#post-apiauthcheckphone)
- **Data:** `users.phone` — [data-model.md](./data-model.md)
- **Depends on:** onboarding screen (ONB) for new numbers

## Actors & preconditions

- **Actor:** anyone, not signed in.
- **Trigger:** enters a number and taps `Continue`.
- **Preconditions:** the app shows the Login phone step (no stored session).

## Main flow

1. The user enters a 10-digit mobile number after the fixed `+91` prefix.
2. The app checks the format and sends the number to the server.
3. The server validates it and looks it up in `users`, regardless of the user's status or restaurant.
4. The server answers whether an account exists.
5. If it exists, the app shows the PIN step for that number ([02](./02-sign-in-with-pin.md)).
6. If not, the app opens the onboarding screen with the number filled in (ONB).

## Alternative flows

- **Empty or short number.** The app shows `Mobile number is required` / `Enter a valid 10-digit number` and sends nothing.
- **Number starting 0–5.** The server refuses it; the app shows `Please enter a valid mobile number`.
- **Switched-off user or unapproved restaurant.** Still reported as existing; the PIN step decides ([02](./02-sign-in-with-pin.md)).
- **Too many checks.** After 20 checks a minute from one IP the server refuses; the app shows `Too Many Attempts.`
- **No network / server error.** The app shows the server message or `Could not check this number`; the user stays on the phone step.
- **Double tap.** `Continue` is disabled while the check runs.

## Acceptance criteria

1. Given a registered number, when the user taps `Continue`, then the PIN step opens with `Enter the 6-digit PIN for +91 <number>`.
2. Given an unregistered valid number, when the user taps `Continue`, then the onboarding screen opens with that number.
3. Given an empty field, when `Continue` is tapped, then `Mobile number is required` shows and no request is sent.
4. Given 9 digits, when `Continue` is tapped, then `Enter a valid 10-digit number` shows.
5. Given `5123456789`, when `Continue` is tapped, then `Please enter a valid mobile number` shows.
6. Given a registered but switched-off user's number, when checked, then the PIN step opens.
7. Given 21 checks in one minute from one IP, then the 21st is refused with `Too Many Attempts.` and nothing changes.
8. The response never contains anything about the user beyond `exists`.
9. Editing the number clears any error shown.

## Platform notes

### POS API

`POST /api/auth/checkPhone`, public, throttle `checkPhone` (20 / min per IP). FormRequest `PhoneRequest`. Envelope `success`. Messages in [api-contract.md](./api-contract.md#post-apiauthcheckphone).

### Data — MySQL

Reads `users.phone` only. Writes nothing.

### POS app

`PhoneLoginScreen` → `PhoneStep`. Thunk `checkPhone`. See [ui-ux.md §2](./ui-ux.md#login--phone-step).

### Super-admin panel

Not touched.

### Integrations

Not touched.

## Configuration

Rate limit is set in `AppServiceProvider`. Country code `+91` is fixed in the app.

## Out of scope

- Onboarding a new restaurant — ONB.
- Numbers outside India — not supported.

## Open questions

- None open. Related decision: AUTH-D8 (phone enumeration accepted) in the [README](./README.md#decisions).
