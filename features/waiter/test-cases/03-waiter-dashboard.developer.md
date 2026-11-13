# 03 — Waiter dashboard — developer cases

Use case: [03-waiter-dashboard.md](../03-waiter-dashboard.md). Contract: [api-contract.md](../api-contract.md#post-apiwaiterdashboard). Fixtures: [README.md](./README.md#fixtures) (DASH-A).

`POST /api/waiter/dashboard`, no body, WTR-A's token unless stated. All 200 replies carry `message: "Dashboard data fetched successfully"`. `averageOrderValue` and `tableTurnoverRate` come out of PHP `round()`, so JSON shows whole values with a decimal point (`105.0`, `0.0`).

| ID | Category | Request / manipulation | Expected `data` |
|---|---|---|---|
| WAIT-03-D01 | — (happy path; accepted WAIT-D3) | DASH-A | `totalOrders` 5, `totalRevenue` `"525.00"`, `averageOrderValue` 105.0, `tableTurnoverRate` 0.25, `topSellingItems` `[ { menuItemName: "Paneer Tikka", quantity: "2", revenue: "500.00" } ]` |
| WAIT-03-D02 | Business rule (accepted WAIT-D3) | Set ORD-Y1 to status 2, `grandTotal` 315.00 (SQL) | `totalOrders` 5, `totalRevenue` `"840.00"`, `averageOrderValue` 168.0, `tableTurnoverRate` 0.25, `topSellingItems` unchanged (yesterday's order not counted) |
| WAIT-03-D03 | Restaurant isolation | Call as OWN-B | OWN-B's own counts; nothing from REST-A |
| WAIT-03-D04 | Role | Call as CHF-A (no orders); as OWN-A | CHF-A: `totalOrders` 0, `totalRevenue` 0, `averageOrderValue` 0.0, `tableTurnoverRate` 0.0, `topSellingItems` []; OWN-A: `totalOrders` 1 |
| WAIT-03-D05 | Business-rule bypass (ORD-D1) | WTR-A creates and pays today an order with ITEM-1 sent as `Paneer Tikka` × 1 and as `PT` × 1 | `topSellingItems` lists `Paneer Tikka` (quantity `"3"`, with ORD-T1) and `PT` (`"1"`) as separate rows |
| WAIT-03-D06 | Authentication / Account state / Super-admin / Error hygiene | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403; WTR-A-OFF old token → 200 | as listed; no exception text in any body |
| WAIT-03-D07 | Over-posting / Injection | body `{ "userId": "<OWN-A hex>", "date": "2020-01-01", "restaurantId": "' OR 1=1 --" }` | same as D01 (body ignored) |
| WAIT-03-D08 | Rate limiting | 60 calls in a minute | all 200; no 429 |
