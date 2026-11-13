# 03 — Switch a staff member on or off — developer cases

Use case: [03-switch-staff-on-off.md](../03-switch-staff-on-off.md). Contract: [api-contract.md](../api-contract.md#post-apistaffupdate). Fixtures: [README.md](./README.md#fixtures).

All requests: `POST /api/staff/update`, JSON, OWN-A's bearer token unless stated. Reset every user touched afterwards. "Updated" = 200 `{ status: true, message: "Staff member updated successfully", data: null }`.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| STAFF-03-D01 | — (happy path) | `{ "userId": "<WTR-A hex>", "name": "Waiter Two", "status": false }`; then WTR-A loginPin; then WTR-A's **old** token on `POST /api/restaurants/showDetails` | Updated (`data` is null); loginPin refused (switched-off response, AUTH 02); the old token → 200 (AUTH-D2) | WTR-A `name` `Waiter Two`, `status` 0, `updated_at` now |
| STAFF-03-D02 | Business rule (accepted STAFF-D1) | What the app sends: `{ "userId": "<WTR-A hex>", "name": "<name>", "isActive": false }` | 422 `{ status: false, message: "The status field is required.", errors: { status: [...] } }` | unchanged |
| STAFF-03-D03 | Restaurant isolation (accepted STAFF-D2) | `{ "userId": "<OWN-B hex>", "name": "Hacked", "status": false }` | Updated | **OWN-B** (another restaurant's owner) renamed `Hacked`, status 0 |
| STAFF-03-D04 | Role (accepted STAFF-D2) | WTR-A's token: `{ "userId": "<OWN-A hex>", "name": "Owner A", "status": false }` | Updated | OWN-A status 0 — the owner's next sign-in is refused |
| STAFF-03-D05 | Business-rule bypass (accepted STAFF-D2) | OWN-A switches **itself** off | Updated; OWN-A's current token still works | OWN-A status 0 |
| STAFF-03-D06 | Input validation | `userId` = 32 hex chars of no user | 404 `{ status: false, message: "Staff member not found", data: null }` | unchanged |
| STAFF-03-D07 | Input validation / Injection / Error hygiene | `userId` = (a) `"xyz"` (b) 31 hex chars (c) WTR-A's dashed UUID (d) `"' OR 1=1 --"` | 500 `{ status: false, message: "Failed to update staff member", data: null }` — no `error` key, no SQL | unchanged |
| STAFF-03-D08 | Input validation | (a) no `name` (b) `status: "yes"` (c) `status: 2` (d) no `userId` (e) `name` 256 chars | 422: (a) `The name field is required.` (b)(c) `The status field must be true or false.` (d) `The user id field is required.` (e) `The name field must not be greater than 255 characters.` | unchanged |
| STAFF-03-D09 | Over-posting | valid body for WTR-A plus `role: "owner"`, `phone: "<PH-NEW>"`, `restaurantId: "<REST-B hex>"`, `pin: "000000"`, `mustChangePin: true` | Updated | only `name` / `status` change; role, phone, restaurant, PIN, `mustChangePin` unchanged |
| STAFF-03-D10 | Authentication | (a) no token (b) `Bearer garbage` (c) `GET` | (a)(b) 401 `{ "message": "Unauthenticated." }` (c) 405 | unchanged |
| STAFF-03-D11 | Account state (AUTH-D2) | WTR-A-OFF's token taken before the switch-off: set CHF-A `status: false` | Updated | CHF-A status 0 |
| STAFF-03-D12 | Concurrency | Two parallel updates for WTR-A: `status: true` and `status: false` | Both Updated | `status` ∈ {0, 1}; no error |
| STAFF-03-D13 | Injection | `name` = `'); DROP TABLE users; --` | Updated | stored exactly; `users` intact |
| STAFF-03-D14 | Error hygiene | The 500 bodies of D07 | As in D07 — nothing beyond the three keys | — |
| STAFF-03-D15 | Rate limiting | 30 updates in one minute | All Updated; no 429 | last values |
| STAFF-03-D16 | Super-admin boundary | No bearer; ADMIN-1's panel session cookie | 401 `{ "message": "Unauthenticated." }` | unchanged |
