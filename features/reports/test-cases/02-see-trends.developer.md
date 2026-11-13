# 02 — See revenue and order trends — developer cases

Use case: [02-see-trends.md](../02-see-trends.md). Contract: [api-contract.md](../api-contract.md#post-apireportsrevenue-and-order-trend). Fixtures: [README.md](./README.md#fixtures).

`POST /api/reports/revenue-and-order-trend`, OWN-A's token unless stated. "Trend-OK" = 200 `{ status: true, message: "Revenue & order trend fetched successfully", data: { revenue: [...], orders: [...] } }`.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| RPT-02-D01 | — (happy path; accepted RPT-D2) | PAID-AT(Mon 9 Nov 14:00) × 2 and PAID-AT(Sun 15 Nov 14:00) × 1; `daily` | Trend-OK; `orders` = `[ { label: "Sunday", day_number: 1, total_orders: 1 }, { label: "Monday", day_number: 2, total_orders: 2 } ]`; `revenue` in the same order with `total_revenue` `"525.00"` / `"1050.00"` |
| RPT-02-D02 | Business rule (accepted RPT-D2) | PAID-AT(30 Nov 14:00) and PAID-AT(3 Nov 14:00); `weekly` | `orders` = `[ { label: "Week 1", total_orders: 1 }, { label: "Week 5", total_orders: 1 } ]` |
| RPT-02-D03 | Business rule | PAID-AT(12 Nov), PAID-AT(5 Jan 2026), PAID-AT(20 Dec 2025); `monthly` | `orders` labels `January`, `November` (1 each); 2025 not counted |
| RPT-02-D04 | Business rule (accepted RPT-D1) | ORDER-LIVE and a billed but unpaid order; `daily` | `revenue: []`, `orders: []` |
| RPT-02-D05 | Input validation / Injection / Over-posting / Error hygiene | (a) `{}` (b) `{ "trend_type": "yearly" }` (c) `{ "trend_type": "daily' --" }` (d) `daily` + `restaurantId: "<REST-B hex>"` | (a) 422 `The trend type field is required.` (b)(c) 422 `The selected trend type is invalid.` (d) REST-A's trend |
| RPT-02-D06 | Restaurant isolation | ORDER-B-PAID only; `daily` | empty lists |
| RPT-02-D07 | Role / Authentication / Account state / Super-admin / Rate limiting | WTR-A, CHF-A → Trend-OK; no token → 401; panel cookie → 401; `GET` → 405; NOREST → 403; 60 calls a minute → no 429 | as listed |
