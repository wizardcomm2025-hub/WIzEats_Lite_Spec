# 03 — Waiter dashboard — developer cases

Use case: [03-waiter-dashboard.md](../03-waiter-dashboard.md). Contract: [api-contract.md](../api-contract.md#post-apiwaiterdashboard). Fixtures: [README.md](./README.md#fixtures).

`POST /api/waiter/dashboard`, no body, WTR-A's token unless stated. All 200 replies carry `message: "Dashboard data fetched successfully"`. `averageOrderValue` and `tableTurnoverRate` come out of PHP `round()`. Cases tagged *Fails until … is fixed* assert the target; today's result is in brackets.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| WAIT-03-D01 | — (happy path) — *Fails until WAIT-D3 is fixed* | DASH-A | `totalOrders` 2, `totalRevenue` `"525.00"`, `averageOrderValue` 262.5, `tableTurnoverRate` 0.25, `topSellingItems` `[ { menuItemName: "Paneer Tikka", quantity: "2", revenue: "500.00" } ]` (today: `totalOrders` 5, `averageOrderValue` 105.0) |
| WAIT-03-D02 | Business rule — *Fails until WAIT-D3 is fixed* | Set ORD-Y1 to status 2, `grandTotal` 315.00 (SQL) | same as D01 — yesterday's order counts nowhere (today: `totalOrders` 5, `totalRevenue` `"840.00"`, `averageOrderValue` 168.0) |
| WAIT-03-D03 | Restaurant isolation | Call as a REST-B waiter | their own counts; nothing from REST-A |
| WAIT-03-D04 | Role — *Fails until WAIT-D2 is fixed* | Call as CHF-A; as OWN-A | 403 `{ status: false, message: "You do not have permission to perform this action", data: null }` both (today: CHF-A all zeros; OWN-A `totalOrders` 1) |
| WAIT-03-D05 | Business rule | WTR-A creates and pays today an order with ITEM-1 sent as `Paneer Tikka` × 1 and as `PT` × 1 | Today two rows `Paneer Tikka` (`"3"` with ORD-T1) and `PT` (`"1"`) (ORD-D1); after ORD-D1 is fixed one row `Paneer Tikka` `"4"` |
| WAIT-03-D06 | Authentication / Account state / Super-admin / Error hygiene — switched-off part *Fails until AUTH-D2 is fixed* | no token → 401; panel cookie only → 401; `GET` → 405; NOREST → 403; WTR-A-OFF old token → 403 `Your account is inactive.` (today 200) | as listed; no exception text in any body |
| WAIT-03-D07 | Over-posting / Injection | body `{ "userId": "<OWN-A hex>", "date": "2020-01-01", "restaurantId": "' OR 1=1 --" }` | same as D01 (body ignored) |
| WAIT-03-D08 | Rate limiting | 60 calls in a minute | all 200; no 429 |
