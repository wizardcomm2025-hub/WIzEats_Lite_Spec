# Take an order & send KOTs — data model

Sources:

- `db_script/tables/tables.sql`
- `app/Services/Order/KotService.php` (`createOrderWithKot`, `addKotToOrder`, `getMergedBill`)
- `app/Models/Order.php`, `Kot.php`, `OrderItem.php`

## `orders` (columns ORD writes)

| Column | Value on create | Changed by ORD later |
|---|---|---|
| `orderId` | new GUID | — |
| `orderNumber` | `<restaurantCode>/<n>` (ORD-D3) | — |
| `restaurantId` | the caller's restaurant | — |
| `tableId`, `roomId` | `hex2bin` of what was sent; the app sends `null` (ORD-D2) | — |
| `orderChannelId` | `hex2bin` of the posted ID — **not checked** to be the restaurant's channel | — |
| `orderStatus` | 0 | — (BILL sets 1 / 2) |
| `taxPercent` | `restaurant_configs.taxPercent` or 0 | — |
| `subTotal` | Σ quantity × price as sent | + each added KOT |
| `taxAmount`, `discount…`, `roundOff`, `grandTotal` | 0 (column defaults) | BILL |
| `customerName`, `customerPhone` | never set (ORD-D3) | — |
| `createdBy`, `createdByUserId` | caller's role and user ID | — |

## `kots`

`kotId` (GUID), `kotNumber` (text holding a number, ORD-D3), `kotNote`, `createdBy`, `status` 0 = pending (KITCH moves it). There is no `orderId` or `restaurantId` column: a KOT is tied to its order only through its `order_items`.

## `order_items`

| Column | Value |
|---|---|
| `orderItemId` | GUID |
| `restaurantId`, `orderId`, `kotId` | the order's |
| `menuItemId` | `hex2bin` of what was sent — not checked (ORD-D1) |
| `menuItemName` | as sent (may be null) |
| `quantity` | int ≥ 1 |
| `price` | decimal(10,2), as sent ≥ 0 |
| `total` | quantity × price |
| `taxPercent` | the order's |
| `taxAmount` | round(total × taxPercent / 100, 2) |

## Numbering (ORD-D3)

- **Order:** take today's orders of the restaurant ordered by `orderId` DESC, which is random, and keep the first. If its number is `<code>/<k>`, the new number is `<code>/<k+1>`, else `<code>/1`.
- **KOT:** take today's KOTs of the restaurant (through `order_items`) ordered by `kotId` DESC, which is random, and keep the first. If its number is numeric `k`, the new number is `k+1`, else `1`.
- There is no unique key on either, so duplicates are possible.

## Status side effects

- `tables.status = 2` / `rooms.status = 2` is attempted with `WHERE tableId = '<hex text>'`, which never matches (ORD-D2).
- The occupy / leave of a table (status 1 / 0) goes through TBL's `tableSave`.

## Transactions

- Create: one transaction; any failure leaves nothing.
- Add KOT: one transaction with `SELECT … FOR UPDATE` on the order.
