# 07 — Legacy OTP sign-in and register — developer cases

Use case: [07-legacy-otp-and-register.md](../07-legacy-otp-and-register.md). Contract: [api-contract.md](../api-contract.md#post-apiauthsendotp-legacy). Fixtures and standard headers: [README.md](./README.md).

These endpoints stay as they are (AUTH-D1). Cases marked **accepted risk** pass when the risky behaviour is observed; they exist so a later change is noticed. No tester file — no UI.

## `POST /api/auth/sendOtp`

| ID | Category | Caller | Request body / manipulation | Expected | Database after |
|---|---|---|---|---|---|
| AUTH-07-D01 | Business-rule bypass (**accepted risk**) | none | phone of OWN-A | 200 · `{ "success": true, "message": "OTP sent successfully", "data": { "otp": 123456 } }` · no SMS | unchanged |
| AUTH-07-D02 | Input validation | none | phone PH-NEW | 404 · `{ "success": false, "message": "Phone number not found in our records", "data": null }` | unchanged |
| AUTH-07-D03 | Input validation / Injection | none | `{}`; 9 digits; `abcdefghij`; `98765' OR 1` | 422 · `{ "success": false, "message": "Validation failed", "errors": {…} }` | unchanged |
| AUTH-07-D04 | Input validation | none | `0123456789` (not a valid mobile, but 10 digits) | 404 `Phone number not found in our records` (passes validation — looser than PIN endpoints) | unchanged |

## `POST /api/auth/verifyOtp`

| ID | Category | Caller | Request body / manipulation | Expected | Database after |
|---|---|---|---|---|---|
| AUTH-07-D05 | — (happy path, **accepted risk**) | none | phone of UNV-A, `otp: "123456"` | 200 · `success: true` · `message: "OTP verified successfully"` · `data` like loginPin's payload without `mustChangePin`; token works on the probe | UNV-A `phone_verified_at` = now; one token row |
| AUTH-07-D06 | Business-rule bypass | none | phone of OWN-A, `otp: "654321"`; then phone PH-NEW, `otp: "654321"` | 400 · `{ "success": false, "message": "Invalid OTP", "data": null }` for both (no hint whether the phone exists) | unchanged |
| AUTH-07-D07 | Business-rule bypass | none | phone PH-NEW, `otp: "123456"` | 404 · `{ "success": false, "message": "User not found", "data": null }` | nothing created |
| AUTH-07-D08 | Account state (**accepted risk**) | none | phone of WTR-A-OFF, `otp: "123456"` | 200 · token works on the probe (no status check) | `phone_verified_at` set; one token row |
| AUTH-07-D09 | Rate limiting (**accepted risk**) | none, one IP | 50 calls to `sendOtp` and 50 to `verifyOtp` (wrong OTP) within one minute | no 429 on either | unchanged |
| AUTH-07-D10 | Authentication (**accepted risk**) | none | knowing only WTR-A's phone: `verifyOtp` with `123456`, then `POST /api/waiter/dashboard` with the token | both 200 — full sign-in without the PIN | one token row for WTR-A |
| AUTH-07-D11 | Input validation / Injection | none | `otp` (a) missing (b) `12345` (c) `12a456` (d) `123456' OR 1` | 422 · `Validation failed` | unchanged |

## `POST /api/auth/register`

| ID | Category | Caller | Request body / manipulation | Expected | Database after |
|---|---|---|---|---|---|
| AUTH-07-D12 | — (happy path) | none | phone PH-NEW | 200 · `{ "status": true, "message": "Registered successfully", "type": "register", "data": { user } }` · no token. loginPin with PH-NEW + `123456` → 200 with `mustChangePin: true`, `isApproved: "Pending"`, `isOnboardingDone: false`. | one `restaurants` row (`isOnboardingDone = 0`, `isApproved = 0`); one `users` row (`role = 'owner'`, `status = 1`, `mustChangePin = 1`, PIN hash of `123456`) |
| AUTH-07-D13 | Account state | none | phone of OWN-A (verified) | 200 · `message: "Login successful"` · `type: "login"` · `data` = OWN-A's user row without `pin` · no token | nothing created |
| AUTH-07-D14 | Account state | none | phone of UNV-A | 200 · `message: "Phone registered but not verified. Please verify OTP."` · `type: "verify"` | nothing created |
| AUTH-07-D15 | Input validation / Injection | none | `{}`; `+91…`; starts with `5`; `9876543210'--` | 422 · `{ "status": false, "message": "<Phone number is required / Please enter a valid mobile number>", "errors": {…} }` | nothing created |
| AUTH-07-D16 | Over-posting / Restaurant isolation | none | another new phone plus `"role": "chef", "restaurantId": "<REST-A id>", "status": 0, "pin": "111111"` | 200 · `type: "register"` | a **new** restaurant; the user is `owner`, `status = 1`, PIN `123456`; REST-A gains no user |
| AUTH-07-D17 | Rate limiting (**accepted risk**) | none, one IP | 20 calls with 20 different new phones within one minute | all 200 | 20 new restaurants and 20 owners |
| AUTH-07-D18 | Concurrency | none | two calls with the same new phone in parallel (repeat 10 times with fresh phones) | every pair: one 200 `type: "register"` and one 200 `type: "verify"` with the **same** `userId`; never 500 (AUTH-D11 — fails until built) | per phone: exactly one `users` row and one `restaurants` row; **no** `restaurants` row without a user |
| AUTH-07-D19 | Error hygiene | none | any of the three endpoints while the database is unreachable | 500 · nothing written. The body may include exception text (an `error` field, or Laravel's error output) — accepted risk (AUTH-D5). | unchanged |
| AUTH-07-D20 | Business-rule bypass (**accepted risk**) | none | phone of CHF-A | 200 · `data` exposes CHF-A's `name`, `email`, `role`, `restaurantId`, `status` to an unauthenticated caller | nothing created |
