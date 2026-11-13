# Public QR ordering — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-12.

| Endpoint | Middleware | Rate limit | Use case |
|---|---|---|---|
| `POST /api/public/menu` | none (`api` group only) | none (PUB-D3) | [01](./01-load-public-menu.md) |
| `POST /api/public/order` | none | none | [02](./02-place-public-order.md) (no `orderId`), [03](./03-add-to-public-order.md) (with `orderId`) |

No token is needed or read. `GET` → 405. A validation failure on either endpoint → 422 `{ "status": false, "message": "<first error>", "errors": { … } }`.

---

## `POST /api/public/menu`

| Field | Rule |
|---|---|
| `restaurantId` | required, string. Matches only as 16 raw bytes |
| `tableId`, `roomId` | nullable, string; ignored |

| Situation | HTTP | Body |
|---|---|---|
| Any restaurant ID that matches nothing (hex, unknown) | 200 | `{ "status": true, "message": "Menu items retrieved successfully", "data": { "restaurantName": null, "restaurantConfig": null, "orderChannel": "", "items": [] } }` |
| Raw ID of a restaurant | 200 | `data`: `{ restaurantName: "Rest A", restaurantConfig: { taxPercent: "5.00" }, orderChannel: "DINE_IN,TAKEAWAY,ROOM_SERVICE,DELIVERY"` (any order), `items: [ { menuItemId, restaurantId, categoryId` (hex), `name, description, price: "250.00", isVeg: true, isActive: true, imageUrl, version, createdBy, isDelete: 0, created_at, updated_at, itemCategoryName, itemCategoryDescription }, … ] }`. All items, including inactive and deleted ones (PUB-D2) |
| Exception | 500 | `{ "status": false, "message": "Failed to retrieve public menu items", "data": null }` |

## `POST /api/public/order`

| Field | Rule |
|---|---|
| `orderId` | **not validated**; `isset` decides between create and add. Passed to an `int` parameter |
| `restaurantId` | required, string |
| `tableId`, `roomId` | nullable, string. **Both keys must be present** on create (PUB-D1) |
| `orderChannelId` | required, integer; ignored (create uses 3) |
| `kotNote` | nullable, string. On add, the key must be present |
| `items` | required, array, min 1 |
| `items.*.menuItemId` | required, string |
| `items.*.menuItemName` | nullable, string |
| `items.*.quantity` | required, integer, min 1 |
| `items.*.price` | required, numeric, min 0 |
| `customerName`, `customerPhone`, `customerAddress` | nullable, string |
| `paymentStatus` | nullable, integer; ignored |

All 500s below have the shape `{ "status": false, "message": "Something went wrong", "error": "<text>" }` unless stated.

| Situation | HTTP | `error` |
|---|---|---|
| Create, valid body (PUB-D1) | 500 | `Failed to create order with KOT: SQLSTATE[42S22]: Column not found: 1054 Unknown column 'orderChannel' in 'field list' (Connection: mysql, SQL: insert into `orders` (…) values (…))`. The values include the customer fields (PUB-D3) |
| Create, raw `restaurantId` | 500 | body `{ "message": "Server Error" }` (the error text isn't valid UTF-8) |
| Create, `tableId` / `roomId` key missing | 500 | `Undefined array key "tableId"` / `"roomId"` |
| Add, hex `orderId` starting with a letter | 500 | `App\Services\Public\PublicService::addKotToPublicOrder(): Argument #1 ($orderId) must be of type int, string given, called in <path>\app\Http\Controllers\Public\PublicController.php on line 42` |
| Add, hex `orderId` starting with a digit | 500 | `A non-numeric value encountered` |
| Add, `orderId` an array | 500 | `…Argument #1 ($orderId) must be of type int, array given…` |
| Add, no matching order | 500 | `Order not found` |
| Add, matched order billed | 500 | `Bill already generated, cannot add items to this order` |
| Add, matched in-progress order, no `kotNote` key | 500 | `Undefined array key "kotNote"` |
| Add, matched in-progress order | 500 | `SQLSTATE[HY000]: General error: 1364 Field 'kotId' doesn't have a default value (Connection: mysql, SQL: insert into `kots` …)` |
| Intended create (not reachable) | 201 | body `{ status: true, message: "Order created successfully", data: { orderId, orderNumber, kotId, kotNumber, subTotal, taxAmount, grandTotal, roundOff } }` |
| Intended add (not reachable) | 201 | body `{ status: true, message: "Order created successfully", data: { orderId, kotId, subTotal, taxAmount, grandTotal } }` |

With `APP_DEBUG=true`, the `Server Error` case returns the debug JSON instead. The other 500s are built by the controller and look the same either way.
