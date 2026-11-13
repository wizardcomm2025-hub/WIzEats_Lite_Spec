# 01 — View the dashboard — developer cases

Use case: [01-view-dashboard.md](../01-view-dashboard.md). Contract: [api-contract.md](../api-contract.md#period-endpoints-dashboard-summary-top-selling-items-order-count-by-channel). Fixtures: [README.md](./README.md#fixtures).

`POST /api/reports/<endpoint>` with `{ "filter_type": "today" }`, OWN-A's token unless stated. "Summary" = `dashboard-summary` `data`; "Top" = `top-selling-items` `data`; "Channels" = `order-count-by-channel` `data`. Values that pass through PHP `round()` (`avg_order_value`, `table_turnover_rate`, `cash`, `upi` when there are paid orders) appear in JSON with a decimal point (`525.0`, `0.0`); compare them as numbers.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| RPT-01-D01 | — (happy path) | ORDER-LIVE and ORDER-PAID | Summary `{ total_revenue: "525.00", total_orders: 1, avg_order_value: 525, table_turnover_rate: 0, revenue_breakdown: { cash: 525, upi: 0 } }`; Top `[ { menuItemName: "Paneer Tikka", total_quantity: "4", total_amount: "1000.00" } ]` (running order included — RPT-D1); Channels 1 row: Takeaway, `total_orders` 1, `orderChannelpercentage` 100 |
| RPT-01-D02 | Business rule (accepted RPT-D1) | ORDER-LIVE only | Summary `total_revenue` 0, `total_orders` 0, `avg_order_value` 0, `table_turnover_rate` 0, cash 0, upi 0; Channels `[]`; Top still lists `Paneer Tikka` `"2"` |
| RPT-01-D03 | Business rule | ORDER-PAID and ORDER-PAID-T1 | Summary `total_orders` 2, `total_revenue` `"1050.00"`, `table_turnover_rate` 0.25 (1 table order ÷ 4 tables), cash 1050 ((600 − 75) + 525) |
| RPT-01-D04 | Business-rule bypass (accepted RPT-D1) | (a) ORDER-PAID paid with a UPI entry **without** `receivedAmount` (b) an order paid with `paymentType: "CARD"`, `receivedAmount` 525 | (a) `upi` 0 although revenue 525.00 (b) revenue counts it; cash 0, upi 0 |
| RPT-01-D05 | Business rule (accepted RPT-D2) | PAID-AT(yesterday 14:00 IST), PAID-AT(1 Nov 14:00), PAID-AT(25 Oct 14:00) | `today`: 0 orders; `yesterday`: 1; `last_7_days`: 1; `last_15_days`: 2 (yesterday, 1 Nov); `last_30_days`: 2 (yesterday, 1 Nov — not 25 Oct) |
| RPT-01-D06 | Business rule (accepted RPT-D2) | PAID-AT(today 00:30 IST = yesterday 19:00 UTC) | `today`: 0 orders; `yesterday`: 1 |
| RPT-01-D07 | Restaurant isolation | ORDER-B-PAID only | REST-A: Summary all 0; Top `[]`; Channels `[]` |
| RPT-01-D08 | Input validation / Error hygiene | (a) `{}` (b) `{ "filter_type": "last_year" }` on each endpoint | 422 (a) `The filter type field is required.` (b) `The selected filter type is invalid.` |
| RPT-01-D09 | Role (accepted RPT-D3) | D01 with WTR-A's token, then CHF-A's | Same figures as D01 (no 403) |
| RPT-01-D10 | Authentication / Account state / Super-admin | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403; WTR-A-OFF old token → 200 (AUTH-D2) | as listed |
| RPT-01-D11 | Rate limiting | 60 calls a minute to each endpoint | all 200; no 429 |
| RPT-01-D12 | Injection / Over-posting | `{ "filter_type": "today' OR 1=1 --" }`; `{ "filter_type": "today", "restaurantId": "<REST-B hex>" }` | 422 `The selected filter type is invalid.`; REST-A's figures (field ignored) |
