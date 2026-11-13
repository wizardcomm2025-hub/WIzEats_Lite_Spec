# Reports — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-12.

| Endpoint | Middleware | Rate limit | Use case |
|---|---|---|---|
| `POST /api/reports/dashboard-summary` | `auth:sanctum`, `restaurant.tenant` | none | [01](./01-view-dashboard.md) |
| `POST /api/reports/top-selling-items` | same | none | [01](./01-view-dashboard.md) |
| `POST /api/reports/order-count-by-channel` | same | none | [01](./01-view-dashboard.md) |
| `POST /api/reports/revenue-and-order-trend` | same | none | [02](./02-see-trends.md) |
| `POST /api/reports/get-all-items` | same | none | [03](./03-export-report.md) |
| `POST /api/reports/get-all-orders` | same | none | [03](./03-export-report.md) |

No role check (RPT-D3). The common failures: no or invalid token → 401 `{ "message": "Unauthenticated." }`; no `restaurantId` → 403 `Restaurant context missing`; `GET` → 405. AUTH-D2 applies. Validation failures: 422 `{ "status": false, "message": "<first error>", "errors": { … } }`. Figures: [data-model.md](./data-model.md).

---

## Period endpoints (`dashboard-summary`, `top-selling-items`, `order-count-by-channel`)

| Field | Rule |
|---|---|
| `filter_type` | required; `today`, `yesterday`, `last_7_days`, `last_15_days` or `last_30_days` |

| Endpoint | 200 body | Error |
|---|---|---|
| `dashboard-summary` | `{ "status": true, "message": "Dashboard summary fetched successfully", "data": { "total_revenue", "total_orders", "avg_order_value", "table_turnover_rate", "revenue_breakdown": { "cash", "upi" } } }` | 500 `{ status: false, message: "Error fetching dashboard summary", data: null }` |
| `top-selling-items` | `{ "status": true, "message": null, "data": [ { "menuItemName", "total_quantity", "total_amount" } ] }` | 500 `Failed to fetch top selling items` |
| `order-count-by-channel` | `{ "status": true, "message": "Order count by channel fetched successfully", "data": [ { "orderChannelId", "orderChannel", "orderChannelName", "total_orders", "orderChannelpercentage" } ] }` | 500 `Failed to fetch order count by channel` |

## `POST /api/reports/revenue-and-order-trend`

| Field | Rule |
|---|---|
| `trend_type` | required; `daily`, `weekly` or `monthly` |

200 `{ "status": true, "message": "Revenue & order trend fetched successfully", "data": { "revenue": [ … ], "orders": [ … ] } }`. Error: 500 `Failed to fetch revenue trend`.

## `POST /api/reports/get-all-items`

| Field | Rule |
|---|---|
| `startDate` | nullable date (default today, UTC) |
| `endDate` | nullable date, ≥ `startDate` (default today) |

200 `{ "status": true, "message": "items fetched successfully", "data": [ { "menuItemName", "quantity", "total", "last_created" } ] }`. Error: 500 `Failed to fetch items`.

## `POST /api/reports/get-all-orders`

Same fields as `get-all-items`.

| Situation | HTTP | Body |
|---|---|---|
| No orders in the range | 200 | `{ "status": true, "message": "Orders fetched successfully", "data": [] }` |
| Any order in the range (RPT-D3) | 500 | `{ "message": "Server Error" }` (with `APP_DEBUG=true`: the debug JSON for `Malformed UTF-8 characters, possibly incorrectly encoded`) |
| Database error | 500 | `{ "status": false, "message": "Failed to fetch orders", "data": null }` |
