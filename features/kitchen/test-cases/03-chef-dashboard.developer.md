# 03 — Chef dashboard — developer cases

Use case: [03-chef-dashboard.md](../03-chef-dashboard.md). Contract: [api-contract.md](../api-contract.md#post-apichefdashboard). Fixtures: [README.md](./README.md#fixtures).

`POST /api/chef/dashboard`, no body, CHF-A's token unless stated. "Counts(r, a, x)" = 200 `{ status: true, message: "Chef KOT stats fetched successfully", data: { total_received: r, total_accepted: a, total_rejected: x } }`. Status changes are made through `update-status` (with KITCH-D1 fixed) or by SQL. Cases tagged *Fails until … is fixed* assert the target; today's result is in brackets.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| KITCH-03-D01 | — (happy path) — *Fails until KITCH-D3 is fixed* | KOT-P and KOT-R exist (today); nobody changed them through the API | Counts(2, 1, 0) — both received, KOT-R (Ready) counts as accepted (today: Counts(0, 0, 0)) |
| KITCH-03-D02 | Business rule — *Fails until KITCH-D3 is fixed* | KOT-P only; CHF-A sets it to 1 → call; to 3 → call | Counts(1, 1, 0); Counts(1, 1, 0) (today, by RAW id: Counts(1, 1, 0); then Counts(1, 0, 0)) |
| KITCH-03-D03 | Business rule — *Fails until KITCH-D3 is fixed* | KOT-P set to 1 by SQL with `chefId` = OWN-A; call as CHF-A | Counts(1, 1, 0) — the restaurant's KOTs, whoever changed them (today: Counts(0, 0, 0)) |
| KITCH-03-D04 | Business rule | KOT-X (Rejected) only; set its `created_at` to yesterday by SQL; call | Counts(0, 0, 0) |
| KITCH-03-D05 | Restaurant isolation — *Fails until KITCH-D3 is fixed* | KOT-B (REST-B) set to 1 by SQL with `chefId` = CHF-A; no REST-A KOTs; call | Counts(0, 0, 0) — another restaurant's KOT is not counted (today: Counts(1, 1, 0)) |
| KITCH-03-D06 | Role — *Fails until KITCH-D2 is fixed* | WTR-A calls; OWN-A calls | 403 `{ status: false, message: "You do not have permission to perform this action", data: null }` both (today: Counts(0, 0, 0)) |
| KITCH-03-D07 | Authentication / Account state / Super-admin / Error hygiene — switched-off part *Fails until AUTH-D2 is fixed* | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403; a switched-off chef's old token → 403 `Your account is inactive.` (today 200) | as listed; no exception text in any body |
| KITCH-03-D08 | Over-posting / Injection | KOT-P at status 1; body `{ "chefId": "<CHF-B hex>", "date": "2020-01-01", "status": "1 OR 1=1" }` | Counts(1, 1, 0) — body ignored (target; today the same only if CHF-A made the change) |
| KITCH-03-D09 | Rate limiting | 60 calls in a minute | all 200; no 429 |
