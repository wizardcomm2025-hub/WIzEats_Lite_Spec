# Delivery board — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-11, re-checked 2026-11-24 (API commit d54d6c8 — no change). Rows marked **Target** describe agreed fixes (DLV-D1 to DLV-D3) and are not in the code yet.

| Endpoint | Middleware | Rate limit | Roles — target (X-D1) | Use case |
|---|---|---|---|---|
| `POST /api/orders/channel/{orderChannelId}` | `auth:sanctum`, `restaurant.tenant` | none | owner, waiter | [01](./01-open-delivery-board.md) |
| `POST /api/orders/update-status/{kotId}` | same | none | owner, waiter | [02](./02-move-delivery-order.md) |

Today no role check. **Target:** chefs → 403 `{ "status": false, "message": "You do not have permission to perform this action", "data": null }`. The common failures: no or invalid token → 401 `{ "message": "Unauthenticated." }`; no `restaurantId` → 403 `Restaurant context missing`; `GET` → 405. Today AUTH-D2 applies; **target (X-D2)** 403 `Your account is inactive.` / `Your restaurant is not approved.` / `Your restaurant is inactive.`

---

## `POST /api/orders/channel/{orderChannelId}`

| Field | Rule |
|---|---|
| `{orderChannelId}` (path) | dashes removed, then `hex2bin`. **Target:** the restaurant's `DELIVERY` channel |

| Situation | HTTP | Body |
|---|---|---|
| Odd-length path (e.g. `3`, `NaN`) | 404 | `{ "status": false, "message": "hex2bin(): Hexadecimal input must have an even length" }`. **Target:** `Channel not found` |
| Even length, not hex | 404 | `{ "status": false, "message": "hex2bin(): Input string must be hexadecimal string" }`. **Target:** `Channel not found` |
| Valid hex, any channel (DLV-D1) | 200 | `{ "status": true, "data": { "status": false, "message": "Failed to retrieve delivery orders", "code": 500 } }` (what the app gets today; shown as an empty board) |
| **Target** another restaurant's channel, or not the `DELIVERY` channel | 404 | `{ "status": false, "message": "Channel not found", "data": null }` |
| **Target** listed | 200 | `{ "status": true, "message": "Delivery orders retrieved successfully", "data": [ { orderId: "<hex>", orderNumber, orderChannelId: "<hex>", customerName, customerPhone, customerAddress, grandTotal, kotId: "<hex>", kotStatus, kotNumber, items: [ { menuItemName, quantity } ] } ] }` — newest first by `created_at`; orders whose KOT is Completed (3) left out |
| **Target** unexpected error | 500 | `{ "status": false, "message": "Something went wrong", "data": null }` |

## `POST /api/orders/update-status/{kotId}`

| Field | Rule |
|---|---|
| `{kotId}` (path) | today cast with `(int)` (DLV-D2). **Target:** 32-char hex, dashes allowed, a KOT of the caller's restaurant on a Delivery-channel order |
| `status` | required integer; today no range (DLV-D2). **Target:** 0–4 and an allowed change |

| Situation | HTTP | Body |
|---|---|---|
| `status` missing / not an integer | 422 | `{ "status": false, "message": "Validation error", "errors": { "status": [ … ] } }` |
| Cast to a number no KOT reads as | 400 | `{ "status": false, "message": "KOT not found" }` |
| Cast to 0 (or a number some KOT reads as) | 400 / 500 | 400 `{ "status": false, "message": "SQLSTATE[22007]: … 1292 Truncated incorrect DOUBLE value: … (Connection: mysql, SQL: update `kots` …)" }`; or 500 `{ "message": "Server Error" }` when that text isn't valid UTF-8 |
| **Target** updated | 200 | `{ "status": true, "message": "KOT status updated successfully", "data": { "kotId": "<hex>", "status": <int> } }` + WhatsApp to the order's `customerPhone` for 1 / 3 / 4 (DLV-D3) |
| **Target** unknown / other restaurant's / non-delivery / malformed `kotId` | 404 | `{ "status": false, "message": "KOT not found", "data": null }` |
| **Target** status outside 0–4 | 422 | `The selected status is invalid.` |
| **Target** change not in Pending→Accepted, Pending→Rejected, Accepted→Out for Delivery, Out for Delivery→Completed | 409 | `{ "status": false, "message": "Invalid status change", "data": null }` |

With `APP_DEBUG=true`, today's 500 case returns the debug JSON (`message`, `exception`, `trace`) instead of `Server Error`.
