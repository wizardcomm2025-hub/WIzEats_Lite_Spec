# Delivery board — data model

Sources:

- `db_script/tables/tables.sql` (`orders`, `order_items`, `kots`, `order_channels`)
- `app/Services/Order/OrderService.php` (`getOrdersList`), `app/Services/Order/KotService.php` (`updateOrderKOTStatus`)

## Columns the board reads

| Table | Columns |
|---|---|
| `orders` | `orderId`, `orderNumber`, `restaurantId`, `orderChannelId`, **`orderChannel`** (not in the GUID schema — DLV-D1), `customerName`, `customerPhone`, `customerAddress`, `grandTotal` |
| `order_items` | `kotId`, `menuItemName`, `quantity` |
| `kots` | `status` (as `kotStatus`), `kotNumber` |

## Intended list query (DLV-D3)

`orders` ⋈ `order_items` ⟕ `kots`, where restaurant = caller's, `orderChannelId` = path, `orderChannel = 'DELIVERY'`, `kots.status != 3`, ordered by `orderId` DESC (random). Grouped by `orderId`; each order takes `kotId`, `kotStatus`, `kotNumber` from its first line; `items` = all its lines (status 3 lines excluded). IDs are returned as hex.

## Status update (DLV-D2)

| Step | SQL |
|---|---|
| Lookup | `SELECT * FROM kots WHERE kotId = <int> LIMIT 1` — binary IDs compared as numbers |
| Update | `UPDATE kots SET status = ?, updated_at = now WHERE kotId = <int>` — fails in strict mode when IDs don't read as numbers |

No other column (`chefId`, stage times, `isPickedUp`) is touched.
