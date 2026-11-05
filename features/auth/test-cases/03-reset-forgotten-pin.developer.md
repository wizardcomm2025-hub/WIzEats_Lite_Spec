# 03 — Reset a forgotten PIN — developer cases

Use case: [03-reset-forgotten-pin.md](../03-reset-forgotten-pin.md). Contract: [api-contract.md](../api-contract.md#post-apiauthforgotpin). Fixtures and standard headers: [README.md](./README.md).

All requests: `POST /api/auth/forgotPin`. Mail transport MAIL-OK unless stated. Before each case, note the user's `pin` hash and `mustChangePin`.

| ID | Category | Caller | Request body / manipulation | Expected | Database after |
|---|---|---|---|---|---|
| AUTH-03-D01 | — (happy path) | none | phone of WTR-A | 200 · `{ "success": true, "message": "A new PIN has been sent to <masked REST-A e-mail>", "data": null }` — mask = first 2 chars + one `*` per remaining char of the name part + `@domain`. One e-mail to REST-A's address containing a 6-digit PIN; the PIN appears nowhere in the response. loginPin with the old PIN → 401; with the e-mailed PIN → 200 with `mustChangePin: true`. | WTR-A: `pin` hash changed, `mustChangePin = 1`; WTR-A's tokens unchanged |
| AUTH-03-D02 | Input validation | none | phone PH-NEW | 404 · `{ "success": false, "message": "Phone number not found in our records", "data": null }` · no e-mail | nothing written |
| AUTH-03-D03 | Business-rule bypass | none | phone of OWN-NM | 422 · `{ "success": false, "message": "No email is registered for this account. Please contact support.", "data": null }` · no e-mail | OWN-NM unchanged; old PIN still works |
| AUTH-03-D04 | Integration failure | none | phone of OWN-A with MAIL-DOWN | 500 · `{ "success": false, "message": "Could not send the PIN email. Please try again later.", "data": null }` — no `error` field | OWN-A `pin` and `mustChangePin` unchanged; old PIN still works |
| AUTH-03-D05 | Input validation | none | `{}`; `{ "phone": "" }`; `+91` prefix; starts with `5`; array | 422 · `Phone number is required` (first two) · `Please enter a valid mobile number` (rest) | nothing written |
| AUTH-03-D06 | Rate limiting | none | phone of CHF-A 4 times within 10 minutes | calls 1–3: 200 (3 e-mails) · call 4: 429 `{ "message": "Too Many Attempts." }` · the PIN in the **3rd** e-mail works | CHF-A `pin` = the 3rd PIN, `mustChangePin = 1` |
| AUTH-03-D07 | Rate limiting | none, one IP | 11 calls within one minute, each with a different registered phone | calls 1–10: 200 · call 11: 429 | only the first 10 users changed |
| AUTH-03-D08 | Account state | none | phone of WTR-A-OFF | 200 · reset e-mailed (AUTH-D6). loginPin with the new PIN → 403 `Your account is inactive.` | WTR-A-OFF `pin` changed, `mustChangePin = 1`, `status` still 0 |
| AUTH-03-D09 | Over-posting / Restaurant isolation | none | phone of WTR-A plus `"email": "attacker@example.test", "restaurantId": "<REST-B id>"` | 200 · message masks **REST-A's** e-mail · e-mail goes to REST-A's address only; nothing to `attacker@example.test` or REST-B | only WTR-A changed |
| AUTH-03-D10 | Authentication | `Bearer <OWN-B token>` | phone of WTR-A | 200 · WTR-A is reset (the public endpoint ignores the token; the body phone decides) | WTR-A changed; OWN-B unchanged |
| AUTH-03-D11 | Business-rule bypass | `Bearer <OWN-A token>` obtained before the reset | reset OWN-A, then call the probe `POST /api/restaurants/showDetails` with the old token | probe → 200 (AUTH-D4: tokens survive a reset) | OWN-A tokens unchanged |
| AUTH-03-D12 | Concurrency | none | two resets for CHF-A sent in parallel | both 200 (or second 429 if the window is used up) · exactly one of the e-mailed PINs works; no 500 | CHF-A `mustChangePin = 1` |
| AUTH-03-D13 | Injection | none | phone `9876543210'; DROP TABLE users;--` | 422 · `Please enter a valid mobile number` | `users` intact |
| AUTH-03-D14 | Error hygiene | none | phone of OWN-A while the database is unreachable | 500 · nothing written. The body may include exception text (an `error` field, or Laravel's error output) — accepted risk (AUTH-D5). | unchanged |
