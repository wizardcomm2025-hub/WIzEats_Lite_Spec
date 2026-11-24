# Public QR ordering — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-12, re-checked 2026-11-24 (API commit d54d6c8 — no change). The **Target** sections describe the agreed working API (PUB-D1 to PUB-D3); none of it is in the code yet.

| Endpoint | Middleware | Rate limit today | Rate limit target | Use case |
|---|---|---|---|---|
| `POST /api/public/menu` | none (`api` group only) | none | 60 / min per IP → 429 `{ "message": "Too Many Attempts." }` | [01](./01-load-public-menu.md) |
| `POST /api/public/order` | none | none | 10 / min per IP | [02](./02-place-public-order.md) (no `orderId`), [03](./03-add-to-public-order.md) (with `orderId`) |

No token is needed or read. `GET` → 405. A validation failure on either endpoint → 422 `{ "status": false, "message": "<first error>", "errors": { … } }`.

---

## `POST /api/public/menu`

### Today

| Field | Rule |
|---|---|
| `restaurantId` | required, string. Matches only as 16 raw bytes |
| `tableId`, `roomId` | nullable, string; ignored |

| Situation | HTTP | Body |
|---|---|---|
| Any restaurant ID that matches nothing (hex, unknown) | 200 | `{ "status": true, "message": "Menu items retrieved successfully", "data": { "restaurantName": null, "restaurantConfig": null, "orderChannel": "", "items": [] } }` |
| Raw ID of a restaurant | 200 | `data`: `{ restaurantName, restaurantConfig: { taxPercent }, orderChannel: "<active channel codes, comma-separated>", items: [ { menuItemId, restaurantId, categoryId` (hex), `name, description, price, isVeg, isActive, imageUrl, version, createdBy, isDelete, created_at, updated_at, itemCategoryName, itemCategoryDescription }, … ] }` — all items, including inactive and deleted ones |
| Exception | 500 | `{ "status": false, "message": "Failed to retrieve public menu items", "data": null }` |

### Target (PUB-D1, PUB-D2)

| Field | Rule |
|---|---|
| `restaurantId` | required, 32-char hex (dashes allowed) |
| `tableId`, `roomId` | nullable, string; ignored |

| Situation | HTTP | Body |
|---|---|---|
| Approved, active restaurant | 200 | `{ "status": true, "message": "Menu items retrieved successfully", "data": { "restaurantName", "restaurantConfig": { "taxPercent" }, "orderChannel": "<active channel codes, comma-separated>", "items": [ { menuItemId, categoryId, name, description, price, isVeg, imageUrl, itemCategoryName, itemCategoryDescription } ] } }` — only active, not-deleted items in active, not-deleted categories; IDs hex |
| Unknown, malformed, not approved or switched off | 404 | `{ "status": false, "message": "Restaurant not found", "data": null }` |
| Exception | 500 | `{ "status": false, "message": "Failed to retrieve public menu items", "data": null }` |

## `POST /api/public/order`

### Today

| Field | Rule |
|---|---|
| `orderId` | **not validated**; `isset` decides between create and add. Passed to an `int` parameter |
| `restaurantId` | required, string (raw bytes to match) |
| `tableId`, `roomId` | nullable, string. **Both keys must be present** on create |
| `orderChannelId` | required, integer; ignored (create uses 3) |
| `kotNote` | nullable, string. On add, the key must be present |
| `items` | required, array, min 1; `items.*.menuItemId` required string; `menuItemName` nullable string; `quantity` integer ≥ 1; `price` numeric ≥ 0 |
| `customerName`, `customerPhone`, `customerAddress` | nullable, string |
| `paymentStatus` | nullable, integer; ignored |

Every create and every add ends in 500 `{ "status": false, "message": "Something went wrong", "error": "<exception text>" }` (or `{ "message": "Server Error" }` when the text holds raw bytes). The `error` texts: `Failed to create order with KOT: SQLSTATE[42S22] … Unknown column 'orderChannel' …` (the full SQL, customer fields included); `Undefined array key "tableId"` / `"roomId"`; the `TypeError` for a hex `orderId` (with the server path); `A non-numeric value encountered`; `Order not found`; `Bill already generated, cannot add items to this order`; `Undefined array key "kotNote"`; `SQLSTATE[HY000]: … 1364 Field 'kotId' doesn't have a default value …`. Nothing is written.

### Target (PUB-D1 to PUB-D3)

| Field | Rule |
|---|---|
| `orderId` | absent → create; present → add; 32-char hex (dashes allowed) |
| `restaurantId` | required, 32-char hex; an approved, active restaurant → else 404 `Restaurant not found` |
| `channelCode` | create: required; an active channel of the restaurant → else 422 `Invalid channel`. Add: ignored |
| `tableId` | required for `DINE_IN`: an active table of the restaurant → else 422 `Invalid table` |
| `roomId` | required for `ROOM_SERVICE`: an active room of the restaurant → else 422 `Invalid room` |
| `kotNote` | nullable string (the key may be left out) |
| `items` | required array ≥ 1; `items.*.menuItemId` an active item of the restaurant in an active category → else 422 `Menu item not available`; `quantity` integer ≥ 1; `menuItemName` / `price` ignored (taken from the menu) |
| `customerName`, `customerAddress` | nullable string ≤ 255; required for `DELIVERY` (`Customer name is required for delivery orders` / `Customer address is required for delivery orders`) |
| `customerPhone` | nullable, `^[6-9][0-9]{9}$` (`The customer phone field format is invalid.`); required for `DELIVERY` (`Customer phone is required for delivery orders`) |
| `orderChannelId`, `paymentStatus` | ignored |
| header `Idempotency-Key` | honoured like ORD-D4 |

| Situation | HTTP | Body |
|---|---|---|
| Created | 201 | `{ "status": true, "message": "Order created successfully", "data": { "orderId": "<hex>", "orderNumber": "<restaurantCode>/<n>", "kotId": "<hex>", "kotNumber": <n>, "subTotal": <number> } }` |
| KOT added | 201 | `{ "status": true, "message": "KOT added successfully", "data": { orderId, orderNumber, kotId, kotNumber, subTotal } }` |
| Add: no such public in-progress order of the restaurant | 404 | `{ "status": false, "message": "Order not found", "data": null }` |
| Add: order billed or paid | 409 | `{ "status": false, "message": "Bill already generated, cannot add items to this order", "data": null }` |
| Create: table / room has a running order | 409 | `{ "status": false, "message": "This table already has a running order", "data": null }` (`This room …`) — no `orderId` given out |
| Business checks above | 422 / 404 | as listed |
| Over the limit | 429 | `{ "message": "Too Many Attempts." }` |
| Unexpected | 500 | `{ "status": false, "message": "Something went wrong", "data": null }` — no exception text |
