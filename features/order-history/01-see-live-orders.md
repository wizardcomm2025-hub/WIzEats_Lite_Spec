# 01 — See live orders

## Summary

The `Live` tab lists the restaurant's running orders (in progress or billed, not paid), with their items and running total, and a `Bill Payment` button on each that opens the bill (BILL-D2 fixed). **Today the list is limited to the chosen dates in UTC days, so an older running order can be missed; target: every running order, always (HIST-D2).**

## Prerequisites

- **Backend:** `POST /api/orders/orderHistoryLive` — [api-contract.md](./api-contract.md#post-apiordersorderhistorylive)
- **Data:** `orders`, `order_items`, `order_channels`

## Actors & preconditions

- **Actor:** owner or waiter. Today any role through the API; target owner and waiter (HIST-D3 / X-D1).
- **Preconditions:** signed in.

## Main flow

1. Staff open Order History (drawer `Order History`, tablet rail `Orders`). The `Live` tab is selected.
2. The app posts the filters: `page` 1, `perPage` 10, `search` (or null), `paymentType` (or null), today's `startDate` / `endDate` (device date), and `channelCode` if one is picked.
3. The server lists the restaurant's orders with status 0 or 1 — today within the date range (UTC days on `created_at`), *target* all of them — newest first. Each order comes with its lines grouped by name and price.
4. Each card shows `Order No. <number>`, the status tag `Pending` (and `Bill generated` for status 1), `<channel> · <n> items` (target: plus the time), and the total labelled `Sub Total`. Tapping a card expands its lines and the note `Tax and round-off are added when the bill is generated.`
5. The tab header counts the orders. Pull down refreshes. `Bill Payment` opens Bill Payment for that order (`Loading Bill…` while the order loads).

## Alternative flows

- **No running orders.** `No Live Orders` · `Orders still awaiting payment will show up here. Pull down to refresh.`
- **Search typed.** Today the Live list doesn't change; *target* it narrows the list like History.
- **Payment mode picked.** No effect on Live (target: the option is hidden on Live).
- **Order created before today, or 00:00–05:29 IST today.** Today not listed under `Today` (UTC days); *target* always listed.
- **Channel filter on.** Only that channel's running orders (HIST-D1 fixed).
- **Server error.** Toast with the server's message, or `Failed to fetch orders`.

## Acceptance criteria

1. Given WTR-A sent a Takeaway KOT (`Paneer Tikka` × 2) at 14:00 IST today, when Live opens, then the order is listed with `Pending`, `Takeaway · 2 items` and `₹500.00` `Sub Total`.
2. Given that order's bill was generated, then the card also shows `Bill generated`, still with `₹500.00` `Sub Total` (accepted).
3. Given the order was paid, then it leaves Live.
4. Given REST-B's running orders, then REST-A's Live list doesn't include them.
5. *(Target, HIST-D2)* Given a running order from yesterday, when Live opens on its default, then it is listed with its date and time. (Today: hidden under `Today`.)
6. Given `Bill Payment` is tapped on a card, then Bill Payment opens for that order (BILL-D2 fixed).

## Platform notes

### POS API

`OrderController::getOrderHistoryLive` (`orderHistoryRequest`) → `OrderService::getOrderHistoryLive`.

### Data — MySQL

Read only.

### POS app

`orderHistory/OrderHistory.tsx`, `useOrderHistory`, `fetchOrderHistory` (`orders/orderHistoryLive`), `OrderHistoryCard`; `openBillForOrder` opens Bill Payment.

### Super-admin panel

None.

### Integrations

None.

## Configuration

Server timezone UTC (`config/app.php`).

## Out of scope

- Taking the payment — BILL.

## Open questions

- None open. Related decisions: HIST-D1 (fixed), HIST-D2, HIST-D3.
