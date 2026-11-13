# Waiter — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-11.

| Endpoint | Middleware | Rate limit | Use case |
|---|---|---|---|
| `POST /api/waiter/ready-kots` | `auth:sanctum`, `restaurant.tenant` | none | [01](./01-see-ready-orders.md) |
| `POST /api/waiter/updatePickedUpStatus/{kotId}` | same | none | [02](./02-mark-kot-delivered.md) |
| `POST /api/waiter/dashboard` | same | none | [03](./03-waiter-dashboard.md) |

No role check and no request body is read (WAIT-D2). The common failures: no or invalid token → 401 `{ "message": "Unauthenticated." }`; no `restaurantId` → 403 `Restaurant context missing`; `GET` → 405. AUTH-D2 applies.

---

## `POST /api/waiter/ready-kots`

| Situation | HTTP | Body |
|---|---|---|
| No ready, un-picked KOT in the restaurant | 200 | `{ "status": true, "message": "No ready KOTs found", "data": [] }` |
| At least one (WAIT-D1) | 500 | `{ "status": false, "message": "An error occurred while fetching pending kots", "error": "Malformed UTF-8 characters, possibly incorrectly encoded" }` |
| Every listed ID happens to be valid UTF-8 (very rare) | 200 | `{ "status": true, "message": "Pending KOTs fetched successfully", "data": [ …KOTs ] }` ([row shape](./data-model.md#ready-list-row-per-kot)) |

## `POST /api/waiter/updatePickedUpStatus/{kotId}`

| Field | Rule |
|---|---|
| `{kotId}` (path) | used as is: hex text never matches (WAIT-D1); the 16 raw bytes, percent-encoded, do match any KOT (WAIT-D2) |

| Situation | HTTP | Body |
|---|---|---|
| No KOT in the whole database has `isPickedUp` 1 after the update | 200 | `{ "status": true, "message": "KOT picked up status updated successfully", "data": [] }` |
| At least one does (WAIT-D2) | 500 | `{ "status": false, "message": "An error occurred while update kot picked up status ", "error": "Malformed UTF-8 characters, possibly incorrectly encoded" }` — the update has been made |
| Database failure | 500 | `{ "status": false, "message": "Picked up status update failed", "data": null }` |

## `POST /api/waiter/dashboard`

| Situation | HTTP | Body |
|---|---|---|
| OK | 200 | `{ "status": true, "message": "Dashboard data fetched successfully", "data": { "totalOrders": <int>, "totalRevenue": <0 or decimal string>, "averageOrderValue": <number>, "tableTurnoverRate": <number>, "topSellingItems": [ { "menuItemName", "quantity" (string), "revenue" (decimal string) } ] } }` |
| Error | 500 | `{ "status": false, "message": "Failed to fetch dashboard data", "data": null }` |

Counts as in [data-model.md](./data-model.md#dashboard-sources) (WAIT-D3).
