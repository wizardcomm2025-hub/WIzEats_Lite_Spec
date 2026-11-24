# Public QR ordering — data model

Sources:

- `db_script/pos_stage.sql` (re-checked 2026-11-24: unchanged)
- `app/Services/Public/PublicService.php`, `app/Http/Controllers/Public/PublicController.php`

## Menu reads (use case 01)

| Table | Filter today | Filter target | Columns returned |
|---|---|---|---|
| `restaurants` | `restaurantId` = as sent | `hex2bin`, `isApproved` 1, `status` 1 | `restaurantName` |
| `restaurant_configs` | `restaurantId` = as sent | `hex2bin` | `taxPercent` → `restaurantConfig` |
| `order_channels` | `restaurantId` = as sent, `isActive` = 1 | `hex2bin`, `isActive` 1, `isDelete` 0 | `channelCode`, joined with `,` → `orderChannel` |
| `menu_items` ⟕ `item_categories` | `mi.restaurantId` = as sent; no `isActive` / `isDelete` filter | `hex2bin`; item and category `isActive` 1 and `isDelete` 0 | item fields, `itemCategoryName`, `itemCategoryDescription` |

"As sent": today the request string is compared directly with the `binary(16)` column, so only the 16 raw bytes match.

## Create writes (use case 02)

| Table | Today (never reached — fails on `orderChannel`) | Target |
|---|---|---|
| `orders` | `orderNumber` `DDMMYY/<restaurantId as sent>/OD-<n>`, `orderChannelId` = 3, `orderChannel` (missing column → 1054), totals from the body, `createdBy` `customer`, customer fields; no `orderId` | new GUID; `orderNumber` `<restaurantCode>/<n>` as ORD; `orderChannelId` from `channelCode`; `tableId` / `roomId` (hex → binary); `orderStatus` 0; `taxPercent` from the config; `subTotal` from menu prices; `createdBy` `customer`; customer fields |
| `kots` | `kotNumber` `<orderNumber>/KO-1`; no `kotId` → 1364 | new GUID; numeric `kotNumber` as ORD; `kotNote`; `createdBy` `customer` |
| `order_items` | names / prices from the body; no `orderItemId` | new GUIDs; the menu's name and price; tax at the order's % |
| `tables` / `rooms` | `status` 2 where the ID = as sent | `status` 2 for the order's table / room |

## Add writes (use case 03)

| Step | Today | Target |
|---|---|---|
| Lookup | `WHERE orderId = <int> AND restaurantId = <as sent> FOR UPDATE` — binary IDs read as numbers (most are 0) | `hex2bin(orderId)`, the restaurant, `createdBy` `customer`, `FOR UPDATE` |
| KOT and lines | no `kotId` → 1364 | as ORD 03 |
| Totals | `UPDATE … WHERE orderId = <int>` (1292 in strict mode) | `subTotal` raised by the new lines |

All writes run in one transaction, so every failure leaves the database unchanged.
