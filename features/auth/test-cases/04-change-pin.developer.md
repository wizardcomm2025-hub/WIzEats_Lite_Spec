# 04 — Change PIN — developer cases

Use case: [04-change-pin.md](../04-change-pin.md). Contract: [api-contract.md](../api-contract.md#post-apiauthchangepin). Fixtures and standard headers: [README.md](./README.md).

All requests: `POST /api/auth/changePin`. "Valid body" = `{ "currentPin": "<current>", "newPin": "<new>", "newPin_confirmation": "<new>" }` with `<new>` ≠ `<current>`.

| ID | Category | Caller | Request body / manipulation | Expected | Database after |
|---|---|---|---|---|---|
| AUTH-04-D01 | — (happy path) | MCP-A token (MCP-A also signed in on a second token) | valid body | 200 · `{ "success": true, "message": "PIN changed successfully", "data": null }`. loginPin with the old PIN → 401; with the new → 200, `mustChangePin: false`. Both MCP-A tokens still work on the probe (AUTH-D4). | MCP-A: `pin` = hash of new, `mustChangePin = 0`; token rows unchanged |
| AUTH-04-D02 | Authentication | (a) none (b) `Bearer abc` (c) `Bearer 999999|<40 random chars>` (d) a token after `logout` | valid body | 401 · `{ "message": "Unauthenticated." }` (all four) | no `users` change |
| AUTH-04-D03 | Account state | NOREST token | valid body | 403 · `{ "status": false, "message": "Restaurant context missing" }` | NOREST unchanged |
| AUTH-04-D04 | Business-rule bypass | OWN-A token | `currentPin` wrong, rest valid | 401 · `{ "success": false, "message": "Current PIN is incorrect", "data": null }`. The token still works on the probe afterwards (the app must not treat this 401 as a dead session). | OWN-A unchanged |
| AUTH-04-D05 | Input validation | OWN-A token | (a) no `currentPin` (b) `currentPin` 5 digits (c) no `newPin` (d) `newPin` `12ab56` (e) no `newPin_confirmation` (f) confirmation differs (g) `newPin` = `currentPin` | 422 with `success: false` and message: (a) `Current PIN is required` (b) `Current PIN must be 6 digits` (c) `New PIN is required` (d) `New PIN must be 6 digits` (e) `New PIN and confirmation do not match` (f) same as (e) (g) `New PIN must be different from the current PIN` | OWN-A unchanged |
| AUTH-04-D06 | Restaurant isolation / Over-posting | WTR-A token | valid body for WTR-A plus `"phone": "<OWN-B phone>", "userId": "<OWN-B id>", "restaurantId": "<REST-B id>"` | 200 | only WTR-A changed; OWN-B's PIN still works |
| AUTH-04-D07 | Business-rule bypass | OWN-A token (`mustChangePin = 0`) | valid body | 200 (the API allows a change at any time) | OWN-A `pin` changed, `mustChangePin = 0` |
| AUTH-04-D08 | Rate limiting | OWN-A token | 20 calls in a row with a wrong `currentPin` | all 401 `Current PIN is incorrect`; **no** 429 (no limit — AUTH-D7) | OWN-A unchanged |
| AUTH-04-D09 | Account state | WTR-A token, then WTR-A switched off (`status = 0`) | valid body | 200 (the token keeps working — AUTH-D2) | WTR-A `pin` changed |
| AUTH-04-D10 | Concurrency | OWN-A token | two valid calls in parallel with **different** new PINs | both 200 · exactly one new PIN works afterwards; no 500 | `mustChangePin = 0` |
| AUTH-04-D11 | Injection | OWN-A token | `newPin` `12345'` and `newPin_confirmation` the same | 422 · `New PIN must be 6 digits` | unchanged |
| AUTH-04-D12 | Error hygiene | OWN-A token | valid body while the database is unreachable | 500 · nothing written. The body may include exception text (an `error` field, or Laravel's error output) — accepted risk (AUTH-D5). | unchanged |
| AUTH-04-D13 | Super-admin boundary | no bearer; the cookies of a signed-in super-admin panel session | valid body | 401 · `{ "message": "Unauthenticated." }` — the panel session does not authenticate `/api/*` | unchanged |
