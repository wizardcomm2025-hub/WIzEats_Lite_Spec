# 03 — Chef dashboard — developer cases

Use case: [03-chef-dashboard.md](../03-chef-dashboard.md). Contract: [api-contract.md](../api-contract.md#post-apichefdashboard). Fixtures: [README.md](./README.md#fixtures).

`POST /api/chef/dashboard`, no body, CHF-A's token unless stated. "Counts(r, a, j)" = 200 `{ status: true, message: "Chef KOT stats fetched successfully", data: { total_received: r, total_accepted: a, total_rejected: j } }`. Status changes use RAW IDs as in KITCH-02-D02.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| KITCH-03-D01 | — (happy path; accepted KITCH-D1) | KOT-P and KOT-R exist; nobody has changed them | Counts(0, 0, 0) |
| KITCH-03-D02 | Business rule (accepted KITCH-D3) | CHF-A sets KOT-P to 1 → call; to 3 → call; to 2 → call | Counts(1, 1, 0); Counts(1, 0, 0); Counts(1, 0, 1) |
| KITCH-03-D03 | Business rule (accepted KITCH-D3) | After D02, OWN-A sets KOT-P to 1; call as CHF-A, then as OWN-A | Counts(0, 0, 0); Counts(1, 1, 0) |
| KITCH-03-D04 | Business rule | CHF-A sets KOT-P to 2; set KOT-P's `created_at` to yesterday by SQL; call | Counts(0, 0, 0) |
| KITCH-03-D05 | Restaurant isolation (accepted KITCH-D2/D3) | CHF-A sets KOT-B (REST-B) to 1; call | Counts(1, 1, 0) — another restaurant's KOT is counted |
| KITCH-03-D06 | Role (accepted KITCH-D2) | WTR-A calls; OWN-A calls (no changes made by them) | Counts(0, 0, 0) both; no 403 |
| KITCH-03-D07 | Authentication / Account state / Super-admin / Error hygiene | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403; WTR-A-OFF old token → 200 | as listed; no exception text in any body |
| KITCH-03-D08 | Over-posting / Injection | body `{ "chefId": "<CHF-B hex>", "date": "2020-01-01", "status": "1 OR 1=1" }` after D02's first step | Counts(1, 1, 0) — body ignored |
| KITCH-03-D09 | Rate limiting | 60 calls in a minute | all 200; no 429 |
