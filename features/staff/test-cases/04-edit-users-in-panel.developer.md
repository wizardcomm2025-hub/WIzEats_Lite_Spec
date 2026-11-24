# 04 — Edit users in the panel — developer cases

Use case: [04-edit-users-in-panel.md](../04-edit-users-in-panel.md). Contract: [api-contract.md → Panel routes](../api-contract.md#panel-routes-web). Fixtures and how to call the panel: [README.md](./README.md#panel-requests-use-case-04).

All requests: `POST /admin/user/update`, JSON with `Accept: application/json`, ADMIN-1's session cookies and `X-CSRF-TOKEN`, unless stated. Reset users afterwards. "422 `<msg>`" = Laravel's `{ "message": "<msg>", "errors": {…} }`. Cases tagged *Fails until … is fixed* assert the target; today's result is in brackets.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| STAFF-04-D01 | — (happy path) | `{ "id": "<WTR-A hex>", "name": "Waiter Two", "role": "waiter", "status": "0" }` | 200 `{ "success": true }` | WTR-A name `Waiter Two`, status 0; `updated_at` **unchanged** |
| STAFF-04-D02 | Input validation / Error hygiene — *Fails until STAFF-D6 is fixed* | `status` = (a) `""` (b) missing (c) `"abc"` (d) `"5"` | (a)(b) 422 `The status field is required.` (c)(d) 422 `The selected status is invalid.` (today: (a)–(c) 500 error page, (d) 200 and status 5 stored) | unchanged |
| STAFF-04-D03 | Business rule (role `owner` is valid in the panel) | WTR-A `role: "owner"`; then WTR-A `loginPin` | 200; loginPin payload `role: "owner"` | role `owner` |
| STAFF-04-D04 | Input validation — *Fails until STAFF-D6 is fixed* | WTR-A (a) `name: ""` (b) `role: ""` (c) `role: "manager"` (d) `role: "CHEF"` | (a) 422 `The name field is required.` (b) 422 `The role field is required.` (c) 422 `The selected role is invalid.` (d) 200 (today: all 200) | (a)–(c) unchanged (today: NULL / `manager`) (d) role `CHEF` |
| STAFF-04-D05 | Input validation — *Fails until STAFF-D6 is fixed* | `id` = 32 hex chars of no user | 404 (today: 200 `{ "success": true }`) | nothing changed |
| STAFF-04-D06 | Input validation / Error hygiene — *Fails until STAFF-D6 is fixed* | (a) no `id` (b) `id: "xyz"` | (a) 422 `The id field is required.` (b) 404 (today: (a) 500 (b) 200, nothing changed) | unchanged |
| STAFF-04-D07 | Over-posting | WTR-A body plus `phone: "<PH-NEW>"`, `pin: "000000"`, `restaurantId: "<REST-B hex>"`, `mustChangePin: 0` | 200 | only name / role / status change |
| STAFF-04-D08 | Authentication / Super-admin boundary | (a) no panel session (guest cookie + its CSRF token) (b) session, no `X-CSRF-TOKEN` (c) no session, `Authorization: Bearer <OWN-A token>` | (a)(c) 302 to `/`; with `Accept: application/json` 401 `{ "message": "Unauthenticated." }` (b) 419; JSON `{ "message": "CSRF token mismatch." }` | unchanged |
| STAFF-04-D09 | Restaurant isolation (by design) | `id` = WTR-B (REST-B) while on REST-A's page | 200 — the super admin may edit any restaurant's user | WTR-B changed |
| STAFF-04-D10 | — (list); unknown restaurant *Fails until STAFF-D6 is fixed* | `GET /admin/user-management/<REST-A hex>`; `?page=2`; an unknown 32-hex restaurant | 200 pages: 3 rows each, all roles incl. owner; unknown → 404 (today: 200 with an empty table) | — |
| STAFF-04-D11 | Injection | WTR-A `name: "\"><script>alert(1)</script>"`; then `GET` the page and open WTR-A's edit dialog | 200; page source shows the name HTML-escaped in the cell and in `data-name`; no script runs; the dialog shows the text | stored exactly |
| STAFF-04-D12 | Rate limiting | 30 updates in one minute | All 200; no 429 | last values |
