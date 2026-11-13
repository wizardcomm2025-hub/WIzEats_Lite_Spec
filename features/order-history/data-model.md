# Order history — data model

Sources:

- `app/Services/Order/OrderService.php` (`getOrderHistory`, `getOrderHistoryLive`)
- `app/Services/Bill/BillService.php` (`getBillByOrderId`), `app/Services/Order/KotService.php` (`getKotsByOrderId`)

Read only. No table is written.

## Live (status 0, 1)

`orders` ⋈ `order_items` ⟕ `order_channels` (same restaurant), restaurant = caller's, optional channel and date (UTC). Sorted by `orderId` DESC (random). Grouped per order; orders without lines are not listed.

| Field | Source |
|---|---|
| `orderId`, `orderChannelId` | hex |
| `orderNumber`, `subTotal`, `taxAmount`, `grandTotal`, `roundOff`, `isDiscountPercent`, `discountPercent`, `discountAmount` | `orders` |
| `orderChannel` | `order_channels.channelName` |
| `orderStatus` | int |
| `paymentStatus` | `Unpaid` (always, for status 0 / 1) |
| `menuItems` | lines grouped by `menuItemName` + `price`: `quantity` and `taxAmount` summed, `taxPercent` of the first |

## History (status 2)

1. Count the matching paid orders.
2. Take page `page` of `perPage` order IDs, `orderId` DESC.
3. Load those orders with lines, channel and a payment summary: per order, `GROUP_CONCAT(paymentType:receivedAmount:changeAmount SEPARATOR '|')` over COMPLETED payments (HIST-D3).

Each row has the Live fields plus: `orderDateTime` (IST text), `paymentStatus` `Paid`, `payments` `[ { paymentType, receivedAmount, changeAmount } ]` (strings), `bill` (bill object or null — [BILL](../billing/api-contract.md)), `kots` `[ { kotId, kotNumber, kotStatus, kotNote, items } ]`.

## Filters

| Filter | Live | History |
|---|---|---|
| `startDate` + `endDate` (both needed) | `created_at` in [start 00:00 UTC, end+1 00:00 UTC) | `created_at` in [start 00:00 IST, end+1 00:00 IST), as UTC |
| `orderChannelId` | `hex2bin` of the value (HIST-D1) | same |
| `paymentType` | ignored | `EXISTS` a COMPLETED payment of that type |
| `search` | ignored | `orderNumber LIKE %s%` OR (numeric) `grandTotal = s` |
| `page`, `perPage` | ignored | default 1 / 10 |
