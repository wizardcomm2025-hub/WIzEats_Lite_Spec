# Take an order & send KOTs — data model

Sources:

- `db_script/pos_stage.sql` (re-checked 2026-11-24: no ORD table changed)
- `app/Services/Order/KotService.php` (`createOrderWithKot`, `addKotToOrder`, `getMergedBill`)
- `app/Models/Order.php`, `Kot.php`, `OrderItem.php`

## `orders` (columns ORD writes)

| Column | Value on create | Changed by ORD later |
|---|---|---|
| `orderId` | new GUID | — |
| `orderNumber` | `<restaurantCode>/<n>` (ORD-D3) | — |
| `restaurantId` | the caller's restaurant | — |
| `tableId`, `roomId` | `hex2bin` of what was sent (the app sends the real IDs since the ORD-D2 fix); not checked today (target ORD-D1) | — |
| `orderChannelId` | `hex2bin` of the posted ID — **not checked** to be the restaurant's channel today (target ORD-D1) | — |
| `orderStatus` | 0 | — (BILL sets 1 / 2) |
| `taxPercent` | `restaurant_configs.taxPercent` or 0 | — |
| `subTotal` | Σ quantity × price as sent (target: menu prices) | + each added KOT |
| `taxAmount`, `discount…`, `roundOff`, `grandTotal` | 0 (column defaults) | BILL |
| `customerName`, `customerPhone` | never set today; target: as sent (ORD-D3) | — |
| `createdBy`, `createdByUserId` | caller's role and user ID | — |

## `kots`

`kotId` (GUID), `kotNumber` (text holding a number, ORD-D3), `kotNote`, `createdBy`, `status` 0 = pending (KITCH moves it). There is no `orderId` or `restaurantId` column: a KOT is tied to its order only through its `order_items`.

## `order_items`

| Column | Value |
|---|---|
| `orderItemId` | GUID |
| `restaurantId`, `orderId`, `kotId` | the order's |
| `menuItemId` | `hex2bin` of what was sent — not checked today (target: an active item of the restaurant — ORD-D1) |
| `menuItemName` | as sent (may be null); target: the menu's name |
| `quantity` | int ≥ 1 |
| `price` | decimal(10,2), as sent ≥ 0; target: the menu's price |
| `total` | quantity × price |
| `taxPercent` | the order's |
| `taxAmount` | round(total × taxPercent / 100, 2) |

## Numbering (ORD-D3)

- **Order:** take today's orders of the restaurant ordered by `created_at` DESC (since 2026-11-16; before, by the random `orderId`) and keep the first. If its number is `<code>/<k>`, the new number is `<code>/<k+1>`, else `<code>/1`.
- **KOT:** take today's KOTs of the restaurant (through `order_items`) ordered by `created_at` DESC and keep the first. If its number is numeric `k`, the new number is `k+1`, else `1`.
- There is no lock and no unique key on either, so parallel requests can get the same number. Target: allocated under a lock, unique per restaurant per day.

## Status side effects

- `tables.status = 2` / `rooms.status = 2` with the converted ID (since 2026-11-16; before, it never matched).
- The occupy / leave of a table (status 1 / 0) goes through TBL's `tableSave`, of a room through ROOM's `roomSave`.

## Transactions

- Create: one transaction; any failure leaves nothing.
- Add KOT: one transaction with `SELECT … FOR UPDATE` on the order.
