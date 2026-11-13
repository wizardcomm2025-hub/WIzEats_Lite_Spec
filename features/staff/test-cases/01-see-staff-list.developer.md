# 01 — See the staff list — developer cases

Use case: [01-see-staff-list.md](../01-see-staff-list.md). Contract: [api-contract.md](../api-contract.md#post-apistaffgetstaffdetails). Fixtures: [README.md](./README.md#fixtures).

All requests: `POST /api/staff/getStaffDetails`, JSON, OWN-A's bearer token unless stated.

| ID | Category | Request / manipulation | Expected | Database after |
|---|---|---|---|---|
| STAFF-01-D01 | — (happy path) | `{}` | 200 `{ status: true, message: "Staff members fetched successfully", data: [WTR-A, CHF-A] }`; OWN-A not included; IDs are 32-char hex | unchanged |
| STAFF-01-D02 | — (happy path) | `{ "userId": "<WTR-A hex>" }` | 200, `data` = [WTR-A] only | unchanged |
| STAFF-01-D03 | Role (accepted STAFF-D3) | WTR-A's token, `{}` | 200 with both WTR-A and CHF-A (the full list) | unchanged |
| STAFF-01-D04 | Restaurant isolation | `{ "userId": "<WTR-B hex>" }` (REST-B's waiter) | 200, `data: []` | unchanged |
| STAFF-01-D05 | Restaurant isolation | `{ "userId": "<OWN-A hex>" }`; `{ "userId": "<OWN-B hex>" }` | 200, `data: []` both | unchanged |
| STAFF-01-D06 | Input validation / Injection / Error hygiene | `userId` = (a) `"xyz"` (b) 31 hex chars (c) `"' OR 1=1 --"` (d) `123` (number) | (a)–(c) 500 `{ status: false, message: "Failed to fetch staff members", data: null }` — no `error` key, no SQL (d) 422 `The user id field must be a string.` | unchanged |
| STAFF-01-D07 | Authentication | (a) no token (b) `Bearer garbage` (c) `GET` | (a)(b) 401 `{ "message": "Unauthenticated." }` (c) 405 | unchanged |
| STAFF-01-D08 | Account state (AUTH-D2) | (a) OWN-P (Pending restaurant) (b) WTR-A-OFF token taken before the switch-off (c) NOREST | (a)(b) 200 (c) 403 `Restaurant context missing` | unchanged |
| STAFF-01-D09 | Error hygiene | D01's response | No `pin`, `password` or `remember_token` in any row | — |
| STAFF-01-D10 | — (role matching) | Set WTR-A's role to `WAITER`, then to `manager`; call `{}` each time | `WAITER`: still listed. `manager`: not listed | — |
| STAFF-01-D11 | Super-admin boundary | No bearer; ADMIN-1's panel session cookie | 401 `{ "message": "Unauthenticated." }` | unchanged |
| STAFF-01-D12 | Rate limiting | 60 calls in one minute | All 200; no 429 | unchanged |
