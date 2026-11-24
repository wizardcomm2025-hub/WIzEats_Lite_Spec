# Waiter — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-11, re-checked 2026-11-24 (API commit d54d6c8 — no change). Rows marked **Target** describe agreed fixes (WAIT-D1 to WAIT-D3) and are not in the code yet.

| Endpoint | Middleware | Rate limit | Roles — target (X-D1) | Use case |
|---|---|---|---|---|
| `POST /api/waiter/ready-kots` | `auth:sanctum`, `restaurant.tenant` | none | waiter | [01](./01-see-ready-orders.md), [02](./02-mark-kot-delivered.md) (Delivered tab) |
| `POST /api/waiter/updatePickedUpStatus/{kotId}` | same | none | waiter | [02](./02-mark-kot-delivered.md) |
| `POST /api/waiter/dashboard` | same | none | waiter | [03](./03-waiter-dashboard.md) |

Today there is no role check and no request body is read (WAIT-D2). **Target:** others → 403 `{ "status": false, "message": "You do not have permission to perform this action", "data": null }`. The common failures: no or invalid token → 401 `{ "message": "Unauthenticated." }`; no `restaurantId` → 403 `Restaurant context missing`; `GET` → 405. Today AUTH-D2 applies; **target (X-D2)** 403 `Your account is inactive.` / `Your restaurant is not approved.` / `Your restaurant is inactive.`

---

## `POST /api/waiter/ready-kots`

| Field | Rule |
|---|---|
| `pickedUp` | today ignored. **Target (WAIT-D2):** optional boolean; `true` lists today's picked-up KOTs instead |

| Situation | HTTP | Body |
|---|---|---|
| No ready, un-picked KOT in the restaurant | 200 | `{ "status": true, "message": "No ready KOTs found", "data": [] }` |
| At least one (WAIT-D1) | 500 | `{ "status": false, "message": "An error occurred while fetching pending kots", "error": "Malformed UTF-8 characters, possibly incorrectly encoded" }`. **Target:** 200 as the next row |
| **Target** KOTs found | 200 | `{ "status": true, "message": "Pending KOTs fetched successfully", "data": [ { kotId: "<hex>", kotNumber, orderNumber, kotNote, status: 4, tableName, zoneName, roomName, floorName, created_at, isPickedUp: 0, items: [ { menuItemName, quantity } ] } ] }` — newest first, no binary IDs |
| **Target (WAIT-D2)** `{ "pickedUp": true }` | 200 | `{ "status": true, "message": "Picked up KOTs fetched successfully", "data": [ …same shape, isPickedUp: 1, pickedUp_at ] }` — the restaurant's KOTs picked up today, newest pick-up first; none → `message: "No picked up KOTs found"`, `data: []` |
| **Target** unexpected error | 500 | `{ "status": false, "message": "Something went wrong", "data": null }` — no `error` key |

## `POST /api/waiter/updatePickedUpStatus/{kotId}`

| Field | Rule |
|---|---|
| `{kotId}` (path) | today used as is: hex text never matches (WAIT-D1); the 16 raw bytes, percent-encoded, do match any KOT (WAIT-D2). **Target:** 32-char hex, dashes allowed, a KOT of the caller's restaurant |

| Situation | HTTP | Body |
|---|---|---|
| No KOT in the whole database has `isPickedUp` 1 after the update | 200 | `{ "status": true, "message": "KOT picked up status updated successfully", "data": [] }` |
| At least one does (WAIT-D2) | 500 | `{ "status": false, "message": "An error occurred while update kot picked up status ", "error": "Malformed UTF-8 characters, possibly incorrectly encoded" }` — the update has been made |
| Database failure | 500 | `{ "status": false, "message": "Picked up status update failed", "data": null }` |
| **Target** picked up | 200 | `{ "status": true, "message": "KOT picked up status updated successfully", "data": { "kotId": "<hex>", "kotNumber": <n>, "pickedUp_at": "<time>" } }` — this KOT only |
| **Target (WAIT-D1)** unknown / other restaurant's / malformed `kotId` | 404 | `{ "status": false, "message": "KOT not found", "data": null }` |
| **Target (WAIT-D2)** KOT not in Ready | 409 | `{ "status": false, "message": "KOT is not ready for pickup", "data": null }` |
| **Target (WAIT-D2)** already picked up | 409 | `{ "status": false, "message": "KOT already picked up", "data": null }` |

## `POST /api/waiter/dashboard`

| Situation | HTTP | Body |
|---|---|---|
| OK | 200 | `{ "status": true, "message": "Dashboard data fetched successfully", "data": { "totalOrders": <int>, "totalRevenue": <0 or decimal string>, "averageOrderValue": <number>, "tableTurnoverRate": <number>, "topSellingItems": [ { "menuItemName", "quantity" (string), "revenue" (decimal string) } ] } }` |
| Error | 500 | `{ "status": false, "message": "Failed to fetch dashboard data", "data": null }` |

Counts as in [data-model.md](./data-model.md#dashboard-sources) — today all-time for orders and revenue (WAIT-D3); **target** today only.
