# 01 — View the dashboard — developer cases

Use case: [01-view-dashboard.md](../01-view-dashboard.md). Contract: [api-contract.md](../api-contract.md#period-endpoints-dashboard-summary-top-selling-items-order-count-by-channel). Fixtures: [README.md](./README.md#fixtures).

`POST /api/reports/<endpoint>` with `{ "filter_type": "today" }`, OWN-A's token unless stated. "Summary" = `dashboard-summary` `data`; "Top" = `top-selling-items` `data`; "Channels" = `order-count-by-channel` `data`. Values that pass through PHP `round()` (`avg_order_value`, `table_turnover_rate`, `cash`, `upi` when there are paid orders) appear in JSON with a decimal point (`525.0`, `0.0`); compare them as numbers. Cases tagged *Fails until … is fixed* assert the target; today's result is in brackets.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| RPT-01-D01 | — (happy path) | ORDER-LIVE and ORDER-PAID | Summary `{ total_revenue: 525, total_orders: 1, avg_order_value: 525, table_turnover_rate: 0, revenue_breakdown: { cash: 525, upi: 0 } }` — revenue = collected (600 − 75); Top `[ { menuItemName: "Paneer Tikka", total_quantity: "2", total_amount: "500.00" } ]` (the running order not counted — since 2026-11-18); Channels 1 row: Takeaway, `total_orders` 1, `orderChannelpercentage` 100 |
| RPT-01-D02 | Business rule | ORDER-LIVE only | Summary all 0; Channels `[]`; Top `[]` |
| RPT-01-D03 | Business rule | ORDER-PAID and ORDER-PAID-T1 | Summary `total_orders` 2, `total_revenue` 1050 ((600 − 75) + 525), `table_turnover_rate` 0.25 (1 table order ÷ 4 tables), cash 1050 |
| RPT-01-D04 | Business-rule bypass (possible until BILL-D1 is fixed) | (a) ORDER-PAID paid with a UPI entry **without** `receivedAmount` (b) an order paid with `paymentType: "CARD"`, `receivedAmount` 525 | (a) `total_orders` 1 but `total_revenue` 0, upi 0, and not in Top (nothing collected) (b) revenue 525, cash 0, upi 0 (other types count in revenue only) |
| RPT-01-D05 | Business rule — *Fails until RPT-D2 is fixed* | PAID-AT(yesterday 14:00 IST), PAID-AT(1 Nov 14:00 IST), PAID-AT(28 Oct 14:00 IST); run on 24 Nov | `today` 0 · `yesterday` 1 · `last_7_days` 1 · `last_15_days` 1 (from 10 Nov) · `last_30_days` 3 (from 26 Oct) (today: `last_30_days` 2 — calendar month, 28 Oct excluded) |
| RPT-01-D06 | Business rule — *Fails until RPT-D2 is fixed* | PAID-AT(today 00:30 IST = yesterday 19:00 UTC) | `today`: 1 order; `yesterday`: 0 (today: 0 and 1 — UTC days) |
| RPT-01-D07 | Restaurant isolation | ORDER-B-PAID only | REST-A: Summary all 0; Top `[]`; Channels `[]` |
| RPT-01-D08 | Input validation / Error hygiene | (a) `{}` (b) `{ "filter_type": "last_year" }` on each endpoint | 422 (a) `The filter type field is required.` (b) `The selected filter type is invalid.` |
| RPT-01-D09 | Role — *Fails until RPT-D3 is fixed* | D01 with WTR-A's token, then CHF-A's | 403 `{ status: false, message: "You do not have permission to perform this action", data: null }` each, on all three endpoints (today: same figures as D01) |
| RPT-01-D10 | Authentication / Account state / Super-admin — switched-off part *Fails until AUTH-D2 is fixed* | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403; an owner token taken before that owner was switched off → 403 `Your account is inactive.` (today 200) | as listed |
| RPT-01-D11 | Rate limiting | 60 calls a minute to each endpoint | all 200; no 429 |
| RPT-01-D12 | Injection / Over-posting | `{ "filter_type": "today' OR 1=1 --" }`; `{ "filter_type": "today", "restaurantId": "<REST-B hex>" }` | 422 `The selected filter type is invalid.`; REST-A's figures (field ignored) |
| RPT-01-D13 | Error hygiene — *Fails until RPT-D3 is fixed* | force a failure (e.g. drop DB read rights) on `dashboard-summary` and `top-selling-items` | 500 with the fixed message and no `error` key (today: `dashboard-summary` puts the exception text in `error`) |
| RPT-01-D14 | Business rule (since 2026-11-18) | An order billed at 525.00 but paid with one CASH leg `receivedAmount` 100 (possible until BILL-D1 is fixed) | `total_orders` 1, `total_revenue` 100; not in Top (payments don't cover the total) |
