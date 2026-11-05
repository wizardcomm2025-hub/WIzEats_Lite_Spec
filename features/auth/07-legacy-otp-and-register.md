# 07 — Legacy OTP sign-in and register

## Summary

Before PIN sign-in, the app signed users in with a phone OTP and created new restaurants through a phone-only register call. The current app no longer calls these endpoints, but they stay live and unchanged for old app versions (AUTH-D1). This file records exactly what they do so developers can test them and see the accepted risk: the OTP is fixed at `123456` and is not rate-limited.

## Prerequisites

- **Backend:** `POST /api/auth/sendOtp`, `/verifyOtp`, `/register` — [api-contract.md](./api-contract.md#post-apiauthsendotp-legacy)
- **Data:** `users`, `restaurants`, `personal_access_tokens` — [data-model.md](./data-model.md)
- **Depends on:** none

## Actors & preconditions

- **Actor:** any caller (old app version or anyone on the internet). No UI in the current app.
- **Trigger:** a direct API call.
- **Preconditions:** none.

## Main flow

**Register** — `POST /api/auth/register` with a phone:
1. Existing verified phone → returns the user with `type: "login"`. No token.
2. Existing unverified phone → returns the user with `type: "verify"`. No token.
3. New phone → creates a restaurant (onboarding not done) and an active owner with PIN `123456` and `mustChangePin = 1`; returns the user with `type: "register"`.

**OTP sign-in:**
4. `sendOtp` with a registered phone returns `otp: 123456`. No SMS is sent.
5. `verifyOtp` with that phone and `123456` sets `phone_verified_at` and returns a token and the sign-in payload (without `mustChangePin`).

## Alternative flows

- **Wrong OTP.** 400 `Invalid OTP`. The check runs before the phone lookup, so a wrong OTP gives the same answer for known and unknown phones.
- **Unknown phone on sendOtp.** 404 `Phone number not found in our records`.
- **Right OTP, unknown phone.** 404 `User not found`.
- **Switched-off user.** `verifyOtp` still issues a token (no status check). Accepted under AUTH-D1.
- **Validation.** `sendOtp` / `verifyOtp` accept any 10 digits (no 6–9 rule) and answer `Validation failed`; `register` uses the 6–9 rule and its messages.
- **No rate limit** on any of the three.
- **Register floods.** Each new phone creates a restaurant and an owner; nothing limits it.
- **Same new phone twice at once.** Exactly one restaurant and one owner are created; the other call returns 200 `type: "verify"` with that same user. No restaurant is left without an owner and no 500 is returned (AUTH-D11 — a change from today, where the losing call can fail and leave an empty restaurant).

## Acceptance criteria

1. Given a registered phone, when `sendOtp` is called, then 200 with `data.otp = 123456`.
2. Given a registered phone, when `verifyOtp` is called with `123456`, then 200 with a working token.
3. Given any OTP other than `123456`, then 400 `Invalid OTP` and nothing is written.
4. Given a switched-off user, when `verifyOtp` is called with `123456`, then 200 with a working token.
5. Given a new phone, when `register` is called, then one restaurant and one owner are created, and that owner can sign in with PIN `123456` and is forced to change it.
6. Given an existing phone, when `register` is called, then nothing is created and the existing user is returned without `pin`.
7. None of the three endpoints is rate-limited.
8. Given two parallel `register` calls with the same new phone, then exactly one restaurant and one owner exist for it, one call answers `type: "register"` and the other `type: "verify"` with the same user, and no restaurant row exists without a user (AUTH-D11).

## Platform notes

### POS API

Public; no throttle. `OtpController` / `OtpService`; `AuthController::register` / `AuthService::loginOrRegister`. Messages in [api-contract.md](./api-contract.md#post-apiauthsendotp-legacy).

### Data — MySQL

`verifyOtp` writes `users.phone_verified_at` and a token. `register` may create `restaurants` + `users` rows; the two rows are created together or not at all (AUTH-D11).

### POS app

Not used by the current app.

### Super-admin panel

Restaurants created by `register` appear as Pending in the panel (ONB / ADM).

### Integrations

None — no SMS provider.

## Configuration

The OTP value is hard-coded.

## Out of scope

- Securing or removing these endpoints — deferred by AUTH-D1. Revisit with a new decision.

## Open questions

- None; behaviour fixed by AUTH-D1 and AUTH-D11.
