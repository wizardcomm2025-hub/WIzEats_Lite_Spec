# Reports — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-12, re-checked 2026-11-24 (API commit d54d6c8; `ReportService` changed 2026-11-18 / 19). Rows marked **Target** describe agreed fixes (RPT-D1 to RPT-D3) and are not in the code yet.

| Endpoint | Middleware | Rate limit | Roles — target (X-D1) | Use case |
|---|---|---|---|---|
| `POST /api/reports/dashboard-summary` | `auth:sanctum`, `restaurant.tenant` | none | owner | [01](./01-view-dashboard.md) |
| `POST /api/reports/top-selling-items` | same | none | owner | [01](./01-view-dashboard.md) |
| `POST /api/reports/order-count-by-channel` | same | none | owner | [01](./01-view-dashboard.md) |
| `POST /api/reports/revenue-and-order-trend` | same | none | owner | [02](./02-see-trends.md) |
| `POST /api/reports/get-all-items` | same | none | owner | [03](./03-export-report.md) |
| `POST /api/reports/get-all-orders` | same | none | owner | [03](./03-export-report.md) |

Today no role check (RPT-D3). **Target:** others → 403 `{ "status": false, "message": "You do not have permission to perform this action", "data": null }`. The common failures: no or invalid token → 401 `{ "message": "Unauthenticated." }`; no `restaurantId` → 403 `Restaurant context missing`; `GET` → 405. Today AUTH-D2 applies; **target (X-D2)** 403 `Your account is inactive.` / `Your restaurant is not approved.` / `Your restaurant is inactive.` Validation failures: 422 `{ "status": false, "message": "<first error>", "errors": { … } }`. Figures: [data-model.md](./data-model.md).

---

## Period endpoints (`dashboard-summary`, `top-selling-items`, `order-count-by-channel`)

| Field | Rule |
|---|---|
| `filter_type` | required; `today`, `yesterday`, `last_7_days`, `last_15_days` or `last_30_days` |

| Endpoint | 200 body | Error |
|---|---|---|
| `dashboard-summary` | `{ "status": true, "message": "Dashboard summary fetched successfully", "data": { "total_revenue", "total_orders", "avg_order_value", "table_turnover_rate", "revenue_breakdown": { "cash", "upi" } } }` | 500 `{ status: false, message: "Error fetching dashboard summary", error: "<exception text>", data: null }` — **target** no `error` key |
| `top-selling-items` | `{ "status": true, "message": "Top selling items fetched successfully", "data": [ { "menuItemName", "total_quantity", "total_amount" } ] }` | 500 `Failed to fetch top selling items` (target: no `error` key) |
| `order-count-by-channel` | `{ "status": true, "message": "Order count by channel fetched successfully", "data": [ { "orderChannelId", "orderChannel", "orderChannelName", "total_orders", "orderChannelpercentage" } ] }` | 500 `Failed to fetch order count by channel` |

## `POST /api/reports/revenue-and-order-trend`

| Field | Rule |
|---|---|
| `trend_type` | required; `daily`, `weekly` or `monthly` |

200 `{ "status": true, "message": "Revenue & order trend fetched successfully", "data": { "revenue": [ … ], "orders": [ … ] } }`. Error: 500 `Failed to fetch revenue trend`.

## `POST /api/reports/get-all-items`

| Field | Rule |
|---|---|
| `startDate` | nullable date (default today) — today a UTC date; **target** an IST date |
| `endDate` | nullable date, ≥ `startDate` (default today) — same |

200 `{ "status": true, "message": "items fetched successfully", "data": [ { "menuItemName", "quantity", "total", "last_created" } ] }` — today every order line of the range; **target** only lines of fully paid orders (RPT-D1). Error: 500 `Failed to fetch items`.

## `POST /api/reports/get-all-orders`

Same fields as `get-all-items`; IST dates since 2026-11-19.

| Situation | HTTP | Body |
|---|---|---|
| OK | 200 | `{ "status": true, "message": "Orders fetched successfully", "data": [ { orderId (hex), orderChannel, tableId (hex or null), roomId (hex or null), grandTotal, orderStatus, paymentStatus ("Paid" / "Unpaid"), created_at (IST) } ] }` — newest first. `grandTotal` is `en_IN`-formatted today (`"1,050.00"`); **target** a plain decimal (`"1050.00"`) |
| Database error | 500 | `{ "status": false, "message": "Failed to fetch orders", "data": null }` |
