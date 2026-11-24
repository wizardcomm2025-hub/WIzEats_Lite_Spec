# Delivery board — data model

Sources:

- `db_script/pos_stage.sql` (`orders`, `order_items`, `kots`, `order_channels`; re-checked 2026-11-24: unchanged)
- `app/Services/Order/OrderService.php` (`getOrdersList`), `app/Services/Order/KotService.php` (`updateOrderKOTStatus`)

## Columns the board reads

| Table | Columns |
|---|---|
| `orders` | `orderId`, `orderNumber`, `restaurantId`, `orderChannelId`, **`orderChannel`** (not in the GUID schema — DLV-D1; target: dropped, the channel is checked through `order_channels.channelCode`), `customerName`, `customerPhone`, `customerAddress`, `grandTotal`, `created_at` |
| `order_items` | `kotId`, `menuItemName`, `quantity` |
| `kots` | `status` (as `kotStatus`), `kotNumber` |
| `order_channels` | target: `channelCode = 'DELIVERY'` of the path channel |

## List query

Today: `orders` ⋈ `order_items` ⟕ `kots`, where restaurant = caller's, `orderChannelId` = path, `orderChannel = 'DELIVERY'` (fails — the column doesn't exist), `kots.status != 3`, ordered by `orderId` DESC (random). Grouped by `orderId`; each order takes `kotId`, `kotStatus`, `kotNumber` from its first line; `items` = all its lines (status 3 lines excluded). IDs are returned as hex.

Target: the same grouping, with the channel checked to be the restaurant's `DELIVERY` channel and the order newest first by `created_at`.

## Status update (DLV-D2)

| Step | Today | Target |
|---|---|---|
| Lookup | `SELECT * FROM kots WHERE kotId = <int> LIMIT 1` — binary IDs compared as numbers | by `hex2bin` of the path, joined to an `order_items` row of the restaurant whose order is on the `DELIVERY` channel |
| Update | `UPDATE kots SET status = ?, updated_at = now WHERE kotId = <int>` — fails in strict mode when IDs don't read as numbers | `status`, `updated_at` of that KOT, after the allowed-change check |

No other column (`chefId`, stage times, `isPickedUp`) is touched.

## WhatsApp (DLV-D3)

Today to a number fixed in `KotService::sendKOTStatusWhatsAppMessage`; target to `orders.customerPhone` of the KOT's order (skipped when empty).
