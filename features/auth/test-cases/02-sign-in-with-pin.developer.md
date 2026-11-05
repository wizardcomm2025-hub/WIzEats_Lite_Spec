# 02 — Sign in with PIN — developer cases

Use case: [02-sign-in-with-pin.md](../02-sign-in-with-pin.md). Contract: [api-contract.md](../api-contract.md#post-apiauthloginpin). Fixtures and standard headers: [README.md](./README.md).

All requests: `POST /api/auth/loginPin`. "Probe" = `POST /api/restaurants/showDetails` with the returned token.

| ID | Category | Caller | Request body / manipulation | Expected | Database after |
|---|---|---|---|---|---|
| AUTH-02-D01 | — (happy path) | none | phone + PIN of OWN-A; repeat for CHF-A and MCP-A | 200 · `success: true` · `message: "Login successful"` · `data` has `token`, `restaurantId` (32 hex), `userId` (32 hex), `name`, `phone`, `email`, `role`, `status`, `isOnboardingDone`, `isApproved: "Accepted"`, `createdBy`, `phone_verified_at`, `mustChangePin` (false for OWN-A / CHF-A, true for MCP-A). `role` is `owner` / `chef` / `waiter`. **No `pin` key anywhere.** Probe → 200 with REST-A. | one new `personal_access_tokens` row per call: `tokenable_id` = the user, `name = 'pos-token'`, `expires_at` null |
| AUTH-02-D02 | Business-rule bypass | none | OWN-A's phone + a wrong 6-digit PIN | 401 · `{ "success": false, "message": "Invalid PIN", "data": null }` | no token row added; `users` unchanged |
| AUTH-02-D03 | Business-rule bypass | none | NOPIN-A's phone + `000000`; then + `123456` | 401 · `Invalid PIN` (both) | no token row |
| AUTH-02-D04 | Account state | none | WTR-A-OFF's phone + correct PIN | 403 · `{ "success": false, "message": "Your account is inactive.", "data": null }` | no token row |
| AUTH-02-D05 | Input validation | none | PH-NEW + `123456` | 404 · `{ "success": false, "message": "Phone number not found in our records", "data": null }` | nothing created |
| AUTH-02-D06 | Input validation | none | (a) `{ "pin": "123456" }` (b) `{ "phone": "<OWN-A>" }` (c) `{}` | (a) 422 · `Phone number is required` (b) 422 · `PIN is required` (c) 422 · `Phone number is required`, `errors` has both fields | no token row |
| AUTH-02-D07 | Input validation | none | OWN-A's phone + pin (a) `12345` (b) `1234567` (c) `12a456` (d) `-12345` (e) `12.345` | 422 · `PIN must be 6 digits` (all) | no token row |
| AUTH-02-D08 | Input validation | none | correct PIN of OWN-A + phone (a) with `+91` (b) starting `5` (c) as an array | 422 · `Please enter a valid mobile number` (all); never 500 | no token row |
| AUTH-02-D09 | Rate limiting | none, one IP | OWN-A's phone: 5 wrong PINs, then the correct PIN, all within one minute | tries 1–5: 401 `Invalid PIN` · try 6: 429 `{ "message": "Too Many Attempts." }` even though the PIN is right. After the minute: correct PIN → 200. | no token row until the post-minute call |
| AUTH-02-D10 | Rate limiting | none, one IP | 31 calls within one minute, each with a **different** phone (wrong PINs) | calls 1–30: 401 / 404 · call 31: 429 | no token row |
| AUTH-02-D11 | Over-posting / Restaurant isolation | none | WTR-A's phone + PIN, plus `"userId": "<OWN-B id>", "restaurantId": "<REST-B id>", "role": "owner"` | 200 · `data.userId` = WTR-A, `data.restaurantId` = REST-A, `data.role: "waiter"`. Probe → REST-A, never REST-B. | token row belongs to WTR-A |
| AUTH-02-D12 | Account state | none | correct PIN of OWN-P, OWN-R, OWN-OFF | 200 for all three (restaurant state is not checked — AUTH-D2) · `isApproved`: `"Pending"`, `"Rejected"`, `"Accepted"` respectively | one token row each |
| AUTH-02-D13 | Concurrency | none | two identical correct calls for OWN-A sent in parallel | both 200 · two **different** tokens · both work on the probe | two new token rows |
| AUTH-02-D14 | Injection | none | phone `9876543210 OR 1=1`; pin `1' OR '1'='1` | 422 (phone message; then pin message) | no token row |
| AUTH-02-D15 | Business-rule bypass | none | a user whose PIN starts with `0` (set it via 04), sent as the string `"0xxxxx"`; then as a JSON number without the leading zero | string: 200 · number: 422 `PIN must be 6 digits` | one token row (string call) |
| AUTH-02-D16 | Error hygiene | none | OWN-A's credentials while the database is unreachable | 500 · nothing written. The body may include exception text (an `error` field, or Laravel's error output) — accepted risk (AUTH-D5). | unchanged |
| AUTH-02-D17 | Authentication | none | inspect D01's token and its row | token is `<id>|<40-char secret>`; `personal_access_tokens.token` holds the SHA-256 of the secret, **not** the secret; `users.pin` holds a bcrypt hash (`$2y$…`), not the PIN | — |
