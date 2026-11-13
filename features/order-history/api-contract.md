# Order history — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-11.

| Endpoint | Middleware | Rate limit | Use case |
|---|---|---|---|
| `POST /api/orders/orderHistoryLive` | `auth:sanctum`, `restaurant.tenant` | none | [01](./01-see-live-orders.md), [03](./03-filter-and-search.md) |
| `POST /api/orders/orderHistory` | same | none | [02](./02-see-past-orders.md), [03](./03-filter-and-search.md) |

No role check (HIST-D3). The common failures: no or invalid token → 401 `{ "message": "Unauthenticated." }`; no `restaurantId` → 403 `Restaurant context missing`; `GET` → 405. AUTH-D2 applies.

## Request fields (both endpoints)

| Field | Rule |
|---|---|
| `startDate` | nullable date |
| `endDate` | nullable date, after or equal to `startDate` |
| `orderChannelId` | nullable integer, must exist in `order_channels.orderChannelId` (HIST-D1) |
| `page` | nullable integer ≥ 1 |
| `perPage` | nullable integer 1–100 |
| `search` | nullable string ≤ 255 |
| `paymentType` | nullable, `CASH` or `UPI` |

Validation failure: 422 `{ "status": false, "message": "<first error>", "errors": { … } }`.

## `POST /api/orders/orderHistoryLive`

| Situation | HTTP | Body |
|---|---|---|
| OK | 200 | `{ "status": true, "message": "Live order history retrieved successfully", "data": [ <live row> ] }` — [row](./data-model.md#live-status-0-1) |
| Error (e.g. channel code 0) | 500 | `{ "status": false, "message": "Failed to retrieve live order history", "data": null }` |

## `POST /api/orders/orderHistory`

| Situation | HTTP | Body |
|---|---|---|
| OK | 200 | `{ "status": true, "message": "Order history retrieved successfully", "data": [ <history row> ], "pagination": { currentPage, perPage, total, lastPage, from, to } }` |
| Page with no rows | 200 | same, `data: []`, `pagination: { currentPage, perPage, total, lastPage }` (`lastPage` 0 when `total` is 0) |
| Error | 500 | `{ "status": false, "message": "Failed to retrieve order history", "data": null, "pagination": null }` |

## Channel filter outcomes (HIST-D1)

| `orderChannelId` sent | Reply |
|---|---|
| `0` | 500 (as above) |
| `1`, `2`, `3`, any integer no channel reads as | 422 `The selected order channel id is invalid.` |
| hex string | 422 `The order channel id field must be an integer.` |
| `null` / absent | no channel filter |
