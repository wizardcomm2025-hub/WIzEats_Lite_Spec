# 02 — See past orders

## Summary

The `History` tab lists paid orders page by page (10 at a time), each with its payments, totals and lines, and a `Print Bill No. <n>` button. **It stays empty in practice, because the app can't take payment today (BILL-D2); orders paid through the API do appear.**

## Prerequisites

- **Backend:** `POST /api/orders/orderHistory` — [api-contract.md](./api-contract.md#post-apiordersorderhistory)
- **Data:** `orders`, `order_items`, `order_channels`, `payments`, `bills`, `bill_items`, `kots`

## Actors & preconditions

- **Actor:** owner or waiter. Any role through the API (HIST-D3).
- **Preconditions:** signed in.

## Main flow

1. Staff tap `History`. The app posts the same filters as Live (01).
2. The server counts the restaurant's paid orders (status 2) matching the filters, takes one page in `orderId` order (HIST-D3), and returns for each: number, IST time, channel name, totals, discount, `paymentStatus` `Paid`, its completed payments (`paymentType`, `receivedAmount`, `changeAmount`), its bill and KOTs, and its lines grouped by name and price. It also returns `pagination`.
3. Each card shows `Order No. <number>`, `Paid`, `<channel> · <n> items · <dd/Mon/yyyy hh:mm AM>`, and `₹<grand total>` `Grand Total`. Expanded: payment chips, lines, `Sub Total`, `Discount (<n>%)`, `Tax`, `Round Off`, `Change Returned`, `Grand Total`, and `Print Bill No. <n>` when the order has a saved bill.
4. Scrolling to the end loads the next page (`Load More`), until a page adds nothing new.

## Alternative flows

- **No paid orders.** `No Past Orders` · `Settled and closed orders will appear here. Pull down to refresh.`
- **`page` beyond the last.** Empty `data` with `pagination` (`lastPage` = pages, no `from` / `to`).
- **A `paymentType` with `:` or `|`.** Its chip text is cut or split (HIST-D3).
- **Server error.** 500 `Failed to retrieve order history`, shown as a toast.

## Acceptance criteria

1. Given ORDER-PAID (paid through the API today, CASH received 600 change 75), when History opens, then it is listed with `Paid`, the time in IST, `₹525.00` `Grand Total`, chip `CASH`, and `Change Returned ₹75.00` when expanded.
2. Given 25 paid orders today and `perPage` 10, then pages 1–3 hold 10, 10 and 5 orders, `pagination.total` 25, `lastPage` 3.
3. Given only unpaid orders, then History is empty.

## Platform notes

### POS API

`OrderController::getOrderHistory` → `OrderService::getOrderHistory` (bill via `BillService::getBillByOrderId`, KOTs via `KotService::getKotsByOrderId`).

### Data — MySQL

Read only. `orderDateTime` = `CONVERT_TZ(created_at, '+00:00', '+05:30')`.

### POS app

Same screen as 01; `PrintPastBillButton` (PRN).

### Super-admin panel

None.

### Integrations

None.

## Configuration

`perPage` 10 in the app (`ORDER_HISTORY_PAGE_SIZE`); the server accepts 1–100.

## Out of scope

- Refunds — none (BILL-D3).

## Open questions

- None open.
