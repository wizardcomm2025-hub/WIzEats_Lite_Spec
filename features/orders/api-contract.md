# Take an order & send KOTs — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-11.

| Endpoint | Middleware | Rate limit | Use case |
|---|---|---|---|
| `POST /api/orders/getOrderChannels` | `auth:sanctum`, `restaurant.tenant` | none | [01](./01-choose-channel-and-table.md) |
| `POST /api/orders/createOrder` | same | none | [02](./02-send-first-kot.md) |
| `POST /api/orders/kots/{orderId}` | same | none | [03](./03-add-kot.md) |
| `POST /api/orders/{orderId}` | same | none | [04](./04-view-running-order.md) |

There is no role check, and `Idempotency-Key` is ignored (ORD-D4). The common failures: no or invalid token → 401 `{ "message": "Unauthenticated." }`; no `restaurantId` → 403 `Restaurant context missing`; `GET` → 405. AUTH-D2 applies.

Route order matters: `/orders/orderHistory`, `/orderHistoryLive`, `/getOrderChannels`, `/createOrder`, `/kots/{id}`, `/generateBill/{id}` and `/update-status/{id}` are matched before `/orders/{orderId}`.

---

## `POST /api/orders/getOrderChannels`

| Field | Rule |
|---|---|
| `channelId` | optional string — validated but never used |

200 `{ "status": true, "message": "Order channels retrieved successfully", "data": [ { orderChannelId, restaurantId, channelCode, channelName, description, isActive, createdBy, isDelete, created_at, updated_at } ] }`, active and not deleted. On error: 500 `{ status: false, message: "Failed to retrieve tables" }` (sic).

## `POST /api/orders/createOrder`

| Field | Rule |
|---|---|
| `orderChannelId` | required string (32-char hex expected; not checked against the restaurant) |
| `tableId`, `roomId` | nullable string (hex) |
| `kotNote` | nullable string |
| `items` | required array, ≥ 1 |
| `items.*.menuItemId` | required string (hex) |
| `items.*.menuItemName` | nullable string |
| `items.*.quantity` | required integer ≥ 1 |
| `items.*.price` | required numeric ≥ 0 |
| `customerName`, `customerPhone` | nullable string; `required_if:orderChannelId,2` (never true for a GUID) |

| Situation | HTTP | Body |
|---|---|---|
| Created | 201 | `{ "status": true, "message": "Order created successfully", "data": { "orderId": "<hex>", "orderNumber": "<code>/<n>", "kotId": "<hex>", "kotNumber": <n>, "subTotal": <number> } }` |
| Validation | 422 | first error, e.g. `The items field is required.`, `The items.0.quantity field must be at least 1.`, `The items.0.price field must be at least 0.`, `The order channel id field is required.` |
| A bad hex ID (odd length / not hex), a total out of range, any other error | 500 | `{ "status": false, "message": "Something went wrong", "error": "<exception text>" }` (ORD-D4) |

## `POST /api/orders/kots/{orderId}`

| Field | Rule |
|---|---|
| `{orderId}` | 32-char hex (dashes removed) |
| `kotNote` | nullable string — **must be present** (key), else 400 |
| `items…` | as in createOrder |

| Situation | HTTP | Body |
|---|---|---|
| Added | 201 | `{ "status": true, "message": "KOT added successfully", "data": { orderId, orderNumber, kotId, kotNumber, subTotal } }` |
| Order not in the caller's restaurant / unknown | 400 | `{ "status": false, "message": "Order not found" }` |
| Status ≠ 0 | 400 | `message`: `Bill already generated, cannot add items to this order` |
| `kotNote` key missing | 400 | `message`: `Undefined array key "kotNote"` |
| Bad hex | 400 | `message`: the PHP `hex2bin()` warning text |
| Validation | 422 | as createOrder |

## `POST /api/orders/{orderId}`

| Situation | HTTP | Body |
|---|---|---|
| Status 0 | 200 | `{ "status": true, "data": { "source": "live", orderId, orderNumber, orderStatus: 0, subTotal, taxPercent, "kots": [ { kotId, kotNumber, items: [ { menuItemId, menuItemName (current menu name), totalQuantity, price, total, taxPercent, taxAmount } ] } ] } }` |
| Status ≥ 1 with a bill | 200 | `data: { source: "bill", orderId, orderNumber, orderStatus, subTotal, taxPercent, billId, billNumber, billStatus, grandTotal, items: [ { menuItemName, quantity, price, total, taxPercent, taxAmount } ] }` |
| Not found / another restaurant | 404 | `{ "status": false, "message": "Order not found" }` |
| Bad hex (e.g. `NaN`) | 404 | `message`: the PHP warning text |
