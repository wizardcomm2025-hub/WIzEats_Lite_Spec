# Delivery board — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-11.

| Endpoint | Middleware | Rate limit | Use case |
|---|---|---|---|
| `POST /api/orders/channel/{orderChannelId}` | `auth:sanctum`, `restaurant.tenant` | none | [01](./01-open-delivery-board.md) |
| `POST /api/orders/update-status/{kotId}` | same | none | [02](./02-move-delivery-order.md) |

No role check. The common failures: no or invalid token → 401 `{ "message": "Unauthenticated." }`; no `restaurantId` → 403 `Restaurant context missing`; `GET` → 405. AUTH-D2 applies.

---

## `POST /api/orders/channel/{orderChannelId}`

| Field | Rule |
|---|---|
| `{orderChannelId}` (path) | dashes removed, then `hex2bin` |

| Situation | HTTP | Body |
|---|---|---|
| Odd-length path (e.g. `3`, `NaN`) | 404 | `{ "status": false, "message": "hex2bin(): Hexadecimal input must have an even length" }` |
| Even length, not hex | 404 | `{ "status": false, "message": "hex2bin(): Input string must be hexadecimal string" }` |
| Valid hex, any channel (DLV-D1) | 200 | `{ "status": true, "data": { "status": false, "message": "Failed to retrieve delivery orders", "code": 500 } }` |
| Intended (not reachable) | 200 | `{ "status": true, "data": { "status": true, "message": "Delivery orders retrieved successfully", "data": [ { orderId, orderNumber, restaurantId, orderChannelId, customerName, customerPhone, customerAddress, grandTotal, kotId, kotStatus, kotNumber, items: [ { menuItemName, quantity } ] } ] } }` |

## `POST /api/orders/update-status/{kotId}`

| Field | Rule |
|---|---|
| `{kotId}` (path) | cast with `(int)` (DLV-D2) |
| `status` | required integer; no range (DLV-D2) |

| Situation | HTTP | Body |
|---|---|---|
| `status` missing / not an integer | 422 | `{ "status": false, "message": "Validation error", "errors": { "status": [ … ] } }` |
| Cast to a number no KOT reads as | 400 | `{ "status": false, "message": "KOT not found" }` |
| Cast to 0 (or a number some KOT reads as) | 400 / 500 | 400 `{ "status": false, "message": "SQLSTATE[22007]: … 1292 Truncated incorrect DOUBLE value: … (Connection: mysql, SQL: update `kots` …)" }`; or 500 `{ "message": "Server Error" }` when that text isn't valid UTF-8 |
| Intended (not reachable) | 200 | `{ "status": true, "message": "KOT status updated successfully", "data": { "kotId": <int>, "status": <int> } }` + WhatsApp for 1 / 3 / 4 (DLV-D3) |

With `APP_DEBUG=true`, the 500 case returns the debug JSON (`message`, `exception`, `trace`) instead of `Server Error`.
