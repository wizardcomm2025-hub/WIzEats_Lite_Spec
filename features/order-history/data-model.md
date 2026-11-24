# Order history — data model

Sources:

- `app/Services/Order/OrderService.php` (`getOrderHistory`, `getOrderHistoryLive` — changed 2026-11-18)
- `app/Services/Bill/BillService.php` (`getBillByOrderId`), `app/Services/Order/KotService.php` (`getKotsByOrderId`)

Read only. No table is written.

## Live (status 0, 1)

`orders` ⋈ `order_items` ⟕ `order_channels` (same restaurant), restaurant = caller's, optional channel (by `channelCode`) and date (UTC days today; target: no date filter, plus `search` — HIST-D2). Sorted by `created_at` DESC. Grouped per order; orders without lines are not listed.

| Field | Source |
|---|---|
| `orderId`, `orderChannelId` | hex |
| `orderNumber`, `subTotal`, `taxAmount`, `grandTotal`, `roundOff`, `isDiscountPercent`, `discountPercent`, `discountAmount` | `orders` |
| `orderChannel` | `order_channels.channelName` |
| `orderStatus` | int |
| `paymentStatus` | `Unpaid` (always, for status 0 / 1) |
| `orderDateTime` | **target only** — IST text, as in History |
| `menuItems` | lines grouped by `menuItemName` + `price`: `quantity` and `taxAmount` summed, `taxPercent` of the first |

## History (status 2)

1. Count the distinct matching paid orders.
2. Take page `page` of `perPage` order IDs, `created_at` DESC.
3. Load those orders with lines, channel and a payment summary. Today: per order `GROUP_CONCAT(paymentType:receivedAmount:changeAmount SEPARATOR '|')` over COMPLETED payments (HIST-D3); target: the payment rows themselves.

Each row has the Live fields plus: `orderDateTime` (`CONVERT_TZ(created_at, '+00:00', '+05:30')`), `paymentStatus` (`Paid` / `Partial` / `Unpaid` from Σ(received − change) against the grand total), `payments` `[ { paymentType, receivedAmount, changeAmount } ]` (strings), `bill` (bill object or null — [BILL](../billing/api-contract.md)), `kots` `[ { kotId, kotNumber, kotStatus, kotNote, items } ]`.

## Filters

| Filter | Live | History |
|---|---|---|
| `startDate` + `endDate` (both needed) | today `created_at` in [start 00:00 UTC, end+1 00:00 UTC); target ignored | `created_at` in [start 00:00 IST, end+1 00:00 IST), as UTC |
| `channelCode` | `order_channels.channelCode` = the code, upper-cased | same |
| `paymentType` | ignored | `EXISTS` a COMPLETED payment of that type |
| `search` | today ignored; target as History | `orderNumber LIKE %s%` OR (numeric) `grandTotal = s` |
| `page`, `perPage` | ignored | default 1 / 10 |
