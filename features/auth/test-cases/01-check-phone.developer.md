# 01 — Check a phone number — developer cases

Use case: [01-check-phone.md](../01-check-phone.md). Contract: [api-contract.md](../api-contract.md#post-apiauthcheckphone). Fixtures and standard headers: [README.md](./README.md).

All requests: `POST /api/auth/checkPhone`, no token unless stated. 422 bodies are `{ "success": false, "message": "<text>", "errors": { "phone": […] } }`.

| ID | Category | Caller | Request body / manipulation | Expected | Database after |
|---|---|---|---|---|---|
| AUTH-01-D01 | Input validation | none | `{}` | 422 · `Phone number is required` | unchanged |
| AUTH-01-D02 | Input validation | none | `{ "phone": "" }` | 422 · `Phone number is required` | unchanged |
| AUTH-01-D03 | Input validation | none | `{ "phone": "          " }` (spaces only) | 422 · `Phone number is required` | unchanged |
| AUTH-01-D04 | Input validation | none | phone of OWN-A with `+91` prefix | 422 · `Please enter a valid mobile number` | unchanged |
| AUTH-01-D05 | Input validation | none | phone of OWN-A with a space after 5 digits | 422 · `Please enter a valid mobile number` | unchanged |
| AUTH-01-D06 | Input validation | none | 10 digits starting with `5` | 422 · `Please enter a valid mobile number` | unchanged |
| AUTH-01-D07 | Input validation | none | 11 digits; then 9 digits | 422 · `Please enter a valid mobile number` (both) | unchanged |
| AUTH-01-D08 | Input validation | none | phone of OWN-A as a JSON **number**, not a string | 200 · `success: true` · `data.exists: true` | unchanged |
| AUTH-01-D09 | Input validation | none | `{ "phone": ["<OWN-A phone>"] }` (array); then `{ "phone": { "a": 1 } }` | 422 · `Please enter a valid mobile number`; never 500 | unchanged |
| AUTH-01-D10 | Injection | none | `{ "phone": "9876543210' OR '1'='1" }`; then `"<script>alert(1)</script>"` | 422 · `Please enter a valid mobile number` | unchanged; no `users` row read beyond normal |
| AUTH-01-D11 | Account state | none | phone of OWN-A; then WTR-A-OFF; then OWN-P | 200 · `{ "success": true, "message": "Phone checked", "data": { "exists": true } }` for all three. `data` has **only** `exists` — no name, role, status or restaurant. | unchanged |
| AUTH-01-D12 | Input validation | none | phone PH-NEW | 200 · `data.exists: false` | no `users` or `restaurants` row created for PH-NEW |
| AUTH-01-D13 | Over-posting | none | phone PH-NEW plus `"restaurantId": "<REST-A id>", "role": "owner", "status": 1` | 200 · `data.exists: false`; response identical to D12 | nothing created |
| AUTH-01-D14 | Rate limiting | none, one IP | 21 requests within one minute (any valid phones) | requests 1–20: 200 · request 21: 429 · `{ "message": "Too Many Attempts." }` | unchanged |
| AUTH-01-D15 | Authentication | `Authorization: Bearer garbage` | phone of OWN-A | 200 · `data.exists: true` (public endpoint ignores the header; no 401) | unchanged |
| AUTH-01-D16 | Error hygiene | none | phone of OWN-A while the database is unreachable | 500 · nothing written. The body may include exception text (an `error` field, or Laravel's error output) — accepted risk (AUTH-D5). | unchanged |
