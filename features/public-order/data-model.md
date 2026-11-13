# Public QR ordering — data model

Sources:

- `db_script/tables/tables.sql`
- `app/Services/Public/PublicService.php`, `app/Http/Controllers/Public/PublicController.php`

## Menu reads (use case 01)

| Table | Filter | Columns returned |
|---|---|---|
| `restaurants` | `restaurantId` = as sent | `restaurantName` |
| `restaurant_configs` | `restaurantId` = as sent | `taxPercent` → `restaurantConfig` |
| `order_channels` | `restaurantId` = as sent, `isActive` = 1 | `channelCode`, joined with `,` → `orderChannel` |
| `menu_items` ⟕ `item_categories` | `mi.restaurantId` = as sent; **no** `isActive` / `isDelete` filter (PUB-D2) | `mi.*`, `itemCategoryName`, `itemCategoryDescription` |

"As sent": the request string is compared directly with the `binary(16)` column, so only the 16 raw bytes match.

## Create writes (use case 02) — intended, never reached (PUB-D1)

| Table | Columns written |
|---|---|
| `orders` | `restaurantId`, `orderNumber` (`DDMMYY/<restaurantId as sent>/OD-<n>`), `tableId`, `roomId`, `orderChannelId` = **3**, `orderStatus` 0, **`orderChannel`** (missing column → error 1054), `taxPercent`, `taxAmount`, `subTotal` (from the body), `roundOff`, `grandTotal`, `createdBy` `customer`, `customerName`, `customerPhone`, `customerAddress`, timestamps. No `orderId` is supplied, and `paymentStatus` is not written (column default 0). |
| `kots` | `kotNumber` (`<orderNumber>/KO-1`), `kotNote`, `createdBy` `customer`, timestamps. No `kotId` → error 1364 |
| `order_items` | `restaurantId`, `orderId` (from `insertGetId`), `kotId`, `menuItemId`, `menuItemName`, `quantity`, `price`, `total` (all from the body). No `orderItemId` |
| `tables` / `rooms` | `status` = 2 where the ID = as sent |

The order sequence `n` comes from today's order (by `created_at` date, UTC) with the highest `orderId`. That is a random GUID, so `n` is not reliably the next number (same as ORD-D3).

## Add writes (use case 03) — intended, never reached (PUB-D1)

| Step | SQL |
|---|---|
| Lookup | `SELECT * FROM orders WHERE orderId = <int> AND restaurantId = <as sent> LIMIT 1 FOR UPDATE`. Binary IDs are read as numbers (most are 0) |
| KOT | `INSERT INTO kots (kotNote, createdBy, created_at, updated_at)`: no `kotId` → 1364; no `kotNumber` |
| Lines | as for create |
| Totals | `UPDATE orders SET subTotal, taxAmount, grandTotal, roundOff, updated_at WHERE orderId = <int>`: would fail with 1292 in strict mode |

All writes run in one transaction, so every failure leaves the database unchanged.
