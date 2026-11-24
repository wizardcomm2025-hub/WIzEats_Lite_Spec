# Order history — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-11, re-checked 2026-11-24 (API commit d54d6c8; history changed 2026-11-18). Rows marked **Target** describe agreed fixes (HIST-D2, HIST-D3) and are not in the code yet.

| Endpoint | Middleware | Rate limit | Roles — target (X-D1) | Use case |
|---|---|---|---|---|
| `POST /api/orders/orderHistoryLive` | `auth:sanctum`, `restaurant.tenant` | none | owner, waiter | [01](./01-see-live-orders.md), [03](./03-filter-and-search.md) |
| `POST /api/orders/orderHistory` | same | none | owner, waiter | [02](./02-see-past-orders.md), [03](./03-filter-and-search.md) |

Today no role check (HIST-D3). **Target:** chefs → 403 `{ "status": false, "message": "You do not have permission to perform this action", "data": null }`. The common failures: no or invalid token → 401 `{ "message": "Unauthenticated." }`; no `restaurantId` → 403 `Restaurant context missing`; `GET` → 405. Today AUTH-D2 applies; **target (X-D2)** 403 `Your account is inactive.` / `Your restaurant is not approved.` / `Your restaurant is inactive.`

## Request fields (both endpoints)

| Field | Rule |
|---|---|
| `startDate` | nullable date |
| `endDate` | nullable date, after or equal to `startDate` |
| `channelCode` | nullable string, must exist in `order_channels.channelCode` (since 2026-11-18; replaces `orderChannelId`) |
| `page` | nullable integer ≥ 1 |
| `perPage` | nullable integer 1–100 |
| `search` | nullable string ≤ 255 |
| `paymentType` | nullable, `CASH` or `UPI` |

Validation failure: 422 `{ "status": false, "message": "<first error>", "errors": { … } }`, e.g. `The selected channel code is invalid.` A field not in the rules (e.g. the old `orderChannelId`) is ignored.

## `POST /api/orders/orderHistoryLive`

| Situation | HTTP | Body |
|---|---|---|
| OK | 200 | `{ "status": true, "message": "Live order history retrieved successfully", "data": [ <live row> ] }` — [row](./data-model.md#live-status-0-1); newest first |
| Error | 500 | `{ "status": false, "message": "Failed to retrieve live order history", "data": null }` |

Filters today: channel, dates (UTC days). **Target (HIST-D2):** channel and `search`; dates, payment mode and paging ignored; rows add `orderDateTime`.

## `POST /api/orders/orderHistory`

| Situation | HTTP | Body |
|---|---|---|
| OK | 200 | `{ "status": true, "message": "Order history retrieved successfully", "data": [ <history row> ], "pagination": { currentPage, perPage, total, lastPage, from, to } }` — newest first |
| Page with no rows | 200 | same, `data: []`, `pagination: { currentPage, perPage, total, lastPage, from: 0, to: 0 }` (`lastPage` 0 when `total` is 0) |
| Error | 500 | `{ "status": false, "message": "Failed to retrieve order history", "data": null, "pagination": null }` |
