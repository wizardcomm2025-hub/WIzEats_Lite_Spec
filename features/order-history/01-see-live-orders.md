# 01 — See live orders

## Summary

The `Live` tab lists the restaurant's running orders (in progress or billed, not paid), with their items and running total, and a `Bill Payment` button on each. **The list works for orders created today (UTC day). The button never opens the bill (BILL-D2).**

## Prerequisites

- **Backend:** `POST /api/orders/orderHistoryLive` — [api-contract.md](./api-contract.md#post-apiordersorderhistorylive)
- **Data:** `orders`, `order_items`, `order_channels`

## Actors & preconditions

- **Actor:** owner or waiter. Any role through the API (HIST-D3).
- **Preconditions:** signed in.

## Main flow

1. Staff open Order History (drawer `Order History`, tablet rail `Orders`). The `Live` tab is selected.
2. The app posts the filters: `page` 1, `perPage` 10, `search` (or null), `paymentType` (or null), and today's `startDate` / `endDate` (device date), plus `orderChannelId` if one is picked.
3. The server lists the restaurant's orders with status 0 or 1, within the date range (UTC days on `created_at` — HIST-D2), in `orderId` order (HIST-D3). Each order comes with its lines grouped by name and price.
4. Each card shows `Order No. <number>`, the status tag `Pending` (and `Bill generated` for status 1), `<channel> · <n> items`, and the total labelled `Sub Total`. Tapping a card expands its lines and the note `Tax and round-off are added when the bill is generated.`
5. The tab header counts the orders. Pull down refreshes.

## Alternative flows

- **No running orders.** `No Live Orders` · `Orders still awaiting payment will show up here. Pull down to refresh.`
- **Search typed or payment mode picked.** The Live list doesn't change (HIST-D2).
- **Order created before today.** Not listed under `Today`; pick `7 days` to see it.
- **Order created between 00:00 and 05:29 IST today.** Listed under yesterday's date, not `Today` (HIST-D2).
- **`Bill Payment` tapped.** `Loading Bill…` on the button, then nothing (BILL-D2).
- **Channel filter on.** Error toast, empty list (HIST-D1).
- **Server error.** Toast with the server's message, or `Failed to fetch orders`.

## Acceptance criteria

1. Given WTR-A sent a Takeaway KOT (`Paneer Tikka` × 2) at 14:00 IST today, when Live opens, then the order is listed with `Pending`, `Takeaway · 2 items` and `₹500.00` `Sub Total`.
2. Given that order's bill was generated through the API, then the card also shows `Bill generated`, but still `₹500.00` `Sub Total`, not the grand total `₹525.00` (HIST-D2).
3. Given the order was paid, then it leaves Live.
4. Given REST-B's running orders, then REST-A's Live list doesn't include them.

## Platform notes

### POS API

`OrderController::getOrderHistoryLive` (`orderHistoryRequest`) → `OrderService::getOrderHistoryLive`.

### Data — MySQL

Read only.

### POS app

`orderHistory/OrderHistory.tsx`, `useOrderHistory`, `fetchOrderHistory` (`orders/orderHistoryLive`), `OrderHistoryCard`.

### Super-admin panel

None.

### Integrations

None.

## Configuration

Server timezone UTC (`config/app.php`).

## Out of scope

- Taking the payment — BILL.

## Open questions

- None open.
