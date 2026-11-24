# Take an order & send KOTs — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-11, re-checked 2026-11-24 (API commit d54d6c8; `KotService` changed 2026-11-16 / 17). Rows marked **Target** describe agreed fixes and are not in the code yet.

| Endpoint | Middleware | Rate limit | Roles — target (X-D1) | Use case |
|---|---|---|---|---|
| `POST /api/orders/getOrderChannels` | `auth:sanctum`, `restaurant.tenant` | none | owner, waiter | [01](./01-choose-channel-and-table.md) |
| `POST /api/orders/createOrder` | same | none | owner, waiter | [02](./02-send-first-kot.md) |
| `POST /api/orders/kots/{orderId}` | same | none | owner, waiter | [03](./03-add-kot.md) |
| `POST /api/orders/{orderId}` | same | none | owner, waiter | [04](./04-view-running-order.md) |

Today there is no role check, and `Idempotency-Key` is ignored (ORD-D4). **Target:** chefs get 403 `{ "status": false, "message": "You do not have permission to perform this action", "data": null }`; `Idempotency-Key` honoured (below). The common failures: no or invalid token → 401 `{ "message": "Unauthenticated." }`; no `restaurantId` → 403 `Restaurant context missing`; `GET` → 405. Today AUTH-D2 applies; **target (X-D2)** 403 `Your account is inactive.` / `Your restaurant is not approved.` / `Your restaurant is inactive.`

Route order matters: `/orders/orderHistory`, `/orderHistoryLive`, `/getOrderChannels`, `/createOrder`, `/kots/{id}`, `/generateBill/{id}` and `/update-status/{id}` are matched before `/orders/{orderId}`.

**Target `Idempotency-Key` (ORD-D4)** on `createOrder` and `kots/{orderId}`: kept 24 hours per user. The same key with the same body → the first answer (status and body) again, nothing written. The same key with a different body → 422 `{ "status": false, "message": "Idempotency-Key reused with a different request", "data": null }`. No header → no protection.

---

## `POST /api/orders/getOrderChannels`

| Field | Rule |
|---|---|
| `channelId` | optional string — validated but never used (the service reads `orderChannelId`) |

200 `{ "status": true, "message": "Order channels retrieved successfully", "data": [ { orderChannelId, restaurantId, channelCode, channelName, description, isActive, createdBy, isDelete, created_at, updated_at } ] }` — **active** and not deleted, so only the channels switched on in the settings (RST-D1 fixed 2026-11-16). On error: 500 `{ status: false, message: "Failed to retrieve tables" }` (sic).

## `POST /api/orders/createOrder`

| Field | Rule today | Target |
|---|---|---|
| `orderChannelId` | required string (hex; not checked against the restaurant) | an active channel of the restaurant → else 422 `Invalid channel` (ORD-D1 / X-D8) |
| `tableId`, `roomId` | nullable string (hex; not checked) | an active, not-deleted table / room of the restaurant → else 422 `Invalid table` / `Invalid room`; one with a running order → 409 (ORD-D4) |
| `kotNote` | nullable string | same |
| `items` | required array, ≥ 1 | same |
| `items.*.menuItemId` | required string (hex; not checked) | an active, not-deleted item of the restaurant in an active category → else 422 `Menu item not available` (ORD-D1) |
| `items.*.menuItemName` | nullable string — stored as sent | ignored; the menu's name is stored |
| `items.*.quantity` | required integer ≥ 1 | same |
| `items.*.price` | required numeric ≥ 0 — stored as sent | ignored; the menu's price is stored (the field may be left out) |
| `customerName`, `customerPhone` | nullable string; `required_if:orderChannelId,2` (never true for a GUID); never saved | saved when sent, any channel; required for the `DELIVERY` channel (ORD-D3) |

| Situation | HTTP | Body |
|---|---|---|
| Created | 201 | `{ "status": true, "message": "Order created successfully", "data": { "orderId": "<hex>", "orderNumber": "<code>/<n>", "kotId": "<hex>", "kotNumber": <n>, "subTotal": <number> } }` |
| Validation | 422 | first error, e.g. `The items field is required.`, `The items.0.quantity field must be at least 1.`, `The items.0.price field must be at least 0.`, `The order channel id field is required.` |
| **Target (ORD-D1)** item, channel, table or room not the restaurant's / not active / malformed | 422 | `{ "status": false, "message": "Menu item not available" \| "Invalid channel" \| "Invalid table" \| "Invalid room", "data": null }` |
| **Target (ORD-D3)** Delivery without customer name / phone | 422 | `Customer name is required for delivery orders` / `Customer phone is required for delivery orders` |
| **Target (ORD-D4)** table / room already has a running order (status 0 or 1) | 409 | `{ "status": false, "message": "This table already has a running order", "data": { "orderId": "<hex>" } }` (`This room …`) |
| A bad hex ID, a total out of range, any other error | 500 | `{ "status": false, "message": "Something went wrong", "error": "<exception text>" }` (ORD-D4). **Target:** no `error` key (X-D5); bad IDs → the 422s above |

## `POST /api/orders/kots/{orderId}`

| Field | Rule |
|---|---|
| `{orderId}` | 32-char hex (dashes removed) |
| `kotNote` | nullable string — today **must be present** (key), else 400; target optional |
| `items…` | as in createOrder (target: menu name and price, ORD-D1) |

| Situation | HTTP | Body |
|---|---|---|
| Added | 201 | `{ "status": true, "message": "KOT added successfully", "data": { orderId, orderNumber, kotId, kotNumber, subTotal } }` |
| Order not in the caller's restaurant / unknown | 400 | `{ "status": false, "message": "Order not found" }`. **Target (X-D4):** 404 `{ "status": false, "message": "Order not found", "data": null }` |
| Status ≠ 0 | 400 | `message`: `Bill already generated, cannot add items to this order` |
| `kotNote` key missing | 400 | `message`: `Undefined array key "kotNote"`. **Target:** added with no note |
| Bad hex | 400 | `message`: the PHP `hex2bin()` warning text. **Target:** 404 `Order not found` |
| Validation | 422 | as createOrder |
| **Target** unexpected error | 500 | `{ "status": false, "message": "Something went wrong", "data": null }` (today 400 with the exception text) |

## `POST /api/orders/{orderId}`

| Situation | HTTP | Body |
|---|---|---|
| Status 0 | 200 | `{ "status": true, "data": { "source": "live", orderId, orderNumber, orderStatus: 0, subTotal, taxPercent, "kots": [ { kotId, kotNumber, items: [ { menuItemId, menuItemName (current menu name), totalQuantity, price, total, taxPercent, taxAmount } ] } ] } }` |
| Status ≥ 1 with a bill | 200 | `data: { source: "bill", orderId, orderNumber, orderStatus, subTotal, taxPercent, billId, billNumber, billStatus, grandTotal, items: [ { menuItemName, quantity, price, total, taxPercent, taxAmount } ] }` |
| Not found / another restaurant | 404 | `{ "status": false, "message": "Order not found" }` |
| Bad hex | 404 | `message`: the PHP warning text. **Target (X-D5):** `Order not found` |
