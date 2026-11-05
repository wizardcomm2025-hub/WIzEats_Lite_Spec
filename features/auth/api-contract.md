# Sign-in & PIN — API contract

Shared shapes (401 `Unauthenticated.`, 403 `Restaurant context missing`, 429 `Too Many Attempts.`) are in [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts below are from the code on 2026-11-05.

**Envelope note.** PIN and OTP endpoints answer with `success`; `register` and `logout` answer with `status` (AUTH-D10). Every validation failure is HTTP 422 with `errors` holding all field errors.

## Endpoints

| Endpoint | Middleware | Rate limit | Used by | Use case |
|---|---|---|---|---|
| `POST /api/auth/checkPhone` | public | 20 / min per IP | App | [01](./01-check-phone.md) |
| `POST /api/auth/loginPin` | public | 5 / min per phone + 30 / min per IP | App | [02](./02-sign-in-with-pin.md) |
| `POST /api/auth/forgotPin` | public | 3 / 10 min per phone + 10 / min per IP | App | [03](./03-reset-forgotten-pin.md) |
| `POST /api/auth/changePin` | `auth:sanctum`, `restaurant.tenant` | none (AUTH-D7) | App | [04](./04-change-pin.md) |
| `POST /api/auth/logout` | `auth:sanctum`, `restaurant.tenant` | none | App | [05](./05-log-out.md) |
| `POST /api/auth/sendOtp` | public | none | Old app versions | [07](./07-legacy-otp-and-register.md) |
| `POST /api/auth/verifyOtp` | public | none | Old app versions | [07](./07-legacy-otp-and-register.md) |
| `POST /api/auth/register` | public | none | Old app versions | [07](./07-legacy-otp-and-register.md) |

No AUTH endpoint checks the role.

---

## `POST /api/auth/checkPhone`

**Purpose:** say whether a phone number has an account.

| Field | Type | Required | Rule |
|---|---|---|---|
| `phone` | string | yes | `^[6-9][0-9]{9}$` — 10 digits, first digit 6–9, no `+91`, spaces or dashes |

| Situation | HTTP | Body |
|---|---|---|
| Number has an account (any status) | 200 | `{ "success": true, "message": "Phone checked", "data": { "exists": true } }` |
| No account | 200 | `{ "success": true, "message": "Phone checked", "data": { "exists": false } }` |
| `phone` missing / empty | 422 | `{ "success": false, "message": "Phone number is required", "errors": {…} }` |
| `phone` not matching the rule | 422 | `{ "success": false, "message": "Please enter a valid mobile number", "errors": {…} }` |
| Over the limit | 429 | `{ "message": "Too Many Attempts." }` |
| Unexpected error | 500 | `{ "success": false, "message": "Something went wrong", "error": "<exception text>" }` (AUTH-D5) |

Writes nothing.

---

## `POST /api/auth/loginPin`

**Purpose:** sign in; issues a new token.

| Field | Type | Required | Rule |
|---|---|---|---|
| `phone` | string | yes | as `checkPhone` |
| `pin` | string or number | yes | exactly 6 digits |

Checks, in this order: validation → phone exists → `users.status = 1` → PIN matches. Restaurant approval and status are **not** checked (AUTH-D2); the response reports them.

| Situation | HTTP | Body |
|---|---|---|
| Success | 200 | `{ "success": true, "message": "Login successful", "data": { …payload } }` |
| `phone` missing | 422 | `success: false`, `message: "Phone number is required"` |
| `phone` invalid | 422 | `message: "Please enter a valid mobile number"` |
| `pin` missing | 422 | `message: "PIN is required"` |
| `pin` not 6 digits | 422 | `message: "PIN must be 6 digits"` |
| Phone not registered | 404 | `{ "success": false, "message": "Phone number not found in our records", "data": null }` |
| User switched off | 403 | `{ "success": false, "message": "Your account is inactive.", "data": null }` |
| Wrong PIN, or user has no PIN | 401 | `{ "success": false, "message": "Invalid PIN", "data": null }` |
| Over the limit | 429 | `{ "message": "Too Many Attempts." }` |
| Unexpected error | 500 | `{ "success": false, "message": "Something went wrong", "error": "<exception text>" }` (AUTH-D5) |

When both fields fail validation, `message` is the `phone` error.

**Success payload (`data`):**

| Field | Value |
|---|---|
| `token` | `<id>|<secret>` — send as `Authorization: Bearer <token>` |
| `restaurantId` | 32-char hex |
| `isOnboardingDone` | true / false / null |
| `userId` | 32-char hex |
| `name`, `phone`, `email`, `role`, `createdBy` | from `users` |
| `phone_verified_at` | timestamp or null |
| `status` | user status (true) |
| `isApproved` | `"Pending"` / `"Accepted"` / `"Rejected"` |
| `mustChangePin` | true / false |

Never contains `pin`.

**Writes:** one `personal_access_tokens` row. Nothing else.

---

## `POST /api/auth/forgotPin`

**Purpose:** e-mail a new PIN to the user's restaurant.

| Field | Type | Required | Rule |
|---|---|---|---|
| `phone` | string | yes | as `checkPhone` |

| Situation | HTTP | Body |
|---|---|---|
| Sent | 200 | `{ "success": true, "message": "A new PIN has been sent to <masked e-mail>", "data": null }` |
| `phone` missing / invalid | 422 | as `checkPhone` |
| Phone not registered | 404 | `{ "success": false, "message": "Phone number not found in our records", "data": null }` |
| Restaurant missing or has no e-mail | 422 | `{ "success": false, "message": "No email is registered for this account. Please contact support.", "data": null }` |
| Mail could not be sent | 500 | `{ "success": false, "message": "Could not send the PIN email. Please try again later.", "data": null }` |
| Over the limit | 429 | `{ "message": "Too Many Attempts." }` |
| Unexpected error | 500 | `{ "success": false, "message": "Something went wrong", "error": "<exception text>" }` (AUTH-D5) |

Works for switched-off users too (AUTH-D6). Masking: see [README → Key concepts](./README.md#key-concepts).

**Writes (on 200 only):** `users.pin` = new random 6-digit PIN (hashed), `users.mustChangePin = 1`. Tokens untouched (AUTH-D4). On 404 / 422 / 500 nothing changes.

---

## `POST /api/auth/changePin`

**Purpose:** replace the signed-in user's PIN.

| Field | Type | Required | Rule |
|---|---|---|---|
| `currentPin` | string | yes | 6 digits |
| `newPin` | string | yes | 6 digits; equal to `newPin_confirmation`; different from `currentPin` |
| `newPin_confirmation` | string | yes (via `confirmed`) | must equal `newPin` |

| Situation | HTTP | Body |
|---|---|---|
| Changed | 200 | `{ "success": true, "message": "PIN changed successfully", "data": null }` |
| `currentPin` missing | 422 | `message: "Current PIN is required"` |
| `currentPin` not 6 digits | 422 | `message: "Current PIN must be 6 digits"` |
| `newPin` missing | 422 | `message: "New PIN is required"` |
| `newPin` not 6 digits | 422 | `message: "New PIN must be 6 digits"` |
| Confirmation missing or different | 422 | `message: "New PIN and confirmation do not match"` |
| `newPin` = `currentPin` | 422 | `message: "New PIN must be different from the current PIN"` |
| Current PIN wrong | 401 | `{ "success": false, "message": "Current PIN is incorrect", "data": null }` |
| No / invalid token | 401 | `{ "message": "Unauthenticated." }` |
| Token's user has no restaurant | 403 | `{ "status": false, "message": "Restaurant context missing" }` |
| Unexpected error | 500 | `{ "success": false, "message": "Something went wrong", "error": "<exception text>" }` (AUTH-D5) |

Allowed whether or not `mustChangePin` is set. The user is always the token's user; any `phone`, `userId` or `restaurantId` in the body is ignored.

**Writes (on 200 only):** `users.pin` = hash of `newPin`, `users.mustChangePin = 0`. Tokens untouched (AUTH-D4).

---

## `POST /api/auth/logout`

**Purpose:** end the session of the token sent.

No body.

| Situation | HTTP | Body |
|---|---|---|
| Logged out | 200 | `{ "status": true, "message": "Logged out successfully" }` |
| No / invalid / already logged-out token | 401 | `{ "message": "Unauthenticated." }` |
| Token's user has no restaurant | 403 | `{ "status": false, "message": "Restaurant context missing" }` |
| Unexpected error | 500 | `{ "status": false, "message": "Something went wrong", "error": "<exception text>" }` (AUTH-D5) |

**Writes:** deletes the `personal_access_tokens` row of the token sent. The user's other tokens stay.

---

## `POST /api/auth/sendOtp` (legacy)

| Field | Type | Required | Rule |
|---|---|---|---|
| `phone` | string | yes | `digits:10` (any 10 digits) |

| Situation | HTTP | Body |
|---|---|---|
| Known phone | 200 | `{ "success": true, "message": "OTP sent successfully", "data": { "otp": 123456 } }` |
| `phone` missing / not 10 digits | 422 | `{ "success": false, "message": "Validation failed", "errors": {…} }` |
| Unknown phone | 404 | `{ "success": false, "message": "Phone number not found in our records", "data": null }` |
| Unexpected error | 500 | `{ "success": false, "message": "Something went wrong!", "error": "<exception text>" }` |

No SMS is sent. Writes nothing. (AUTH-D1)

## `POST /api/auth/verifyOtp` (legacy)

| Field | Type | Required | Rule |
|---|---|---|---|
| `phone` | string | yes | `digits:10` |
| `otp` | string | yes | `digits:6` |

| Situation | HTTP | Body |
|---|---|---|
| OTP is `123456` and phone known | 200 | `{ "success": true, "message": "OTP verified successfully", "data": { …payload } }` — same as `loginPin` payload **without** `mustChangePin` |
| Validation failure | 422 | `{ "success": false, "message": "Validation failed", "errors": {…} }` |
| Any other OTP | 400 | `{ "success": false, "message": "Invalid OTP", "data": null }` |
| OTP `123456`, phone unknown | 404 | `{ "success": false, "message": "User not found", "data": null }` |
| Unexpected error | 500 | `{ "success": false, "message": "Something went wrong", "error": "<exception text>" }` |

User status is **not** checked. **Writes:** `users.phone_verified_at = now`, one token. (AUTH-D1)

## `POST /api/auth/register` (legacy)

| Field | Type | Required | Rule |
|---|---|---|---|
| `phone` | string | yes | as `checkPhone` |

| Situation | HTTP | Body |
|---|---|---|
| Phone exists and verified | 200 | `{ "status": true, "message": "Login successful", "type": "login", "data": { user } }` |
| Phone exists, not verified | 200 | `{ "status": true, "message": "Phone registered but not verified. Please verify OTP.", "type": "verify", "data": { user } }` |
| New phone | 200 | `{ "status": true, "message": "Registered successfully", "type": "register", "data": { user } }` |
| Validation failure | 422 | `{ "status": false, "message": "<Phone number is required / Please enter a valid mobile number>", "errors": {…} }` |
| Unexpected error | 500 | `{ "status": false, "message": "Something went wrong", "error": "<exception text>" }` |

`data` is the user row (IDs as hex, no `pin`). **No token is issued.** **Writes on a new phone:** a `restaurants` row (`isOnboardingDone = 0`) and an owner `users` row (`status = 1`, PIN `123456` hashed, `mustChangePin = 1`). (AUTH-D1)

**Parallel calls with the same new phone (AUTH-D11 — change from today):** both rows are written together or not at all. One call answers `type: "register"`; the other answers 200 `type: "verify"` with the same user. Never a 500, never a restaurant without a user.
