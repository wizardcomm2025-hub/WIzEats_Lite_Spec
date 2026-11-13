# 01 — Generate the bill

## Summary

Staff save the bill before payment, for example to print it for the table. The bill gets its number, the order's totals are fixed, and the order takes no more KOTs. **Today this can only be done through the API (BILL-D2), and the server stores whatever totals it is sent (BILL-D1).**

## Prerequisites

- **Backend:** `POST /api/orders/generateBill/{orderId}` — [api-contract.md](./api-contract.md#post-apiordersgeneratebillorderid)
- **Data:** `orders`, `order_items`, `bills`, `bill_items`, `restaurant_configs`

## Actors & preconditions

- **Actor:** owner or waiter (any role through the API — BILL-D3).
- **Preconditions:** an order of the caller's restaurant with `orderStatus` 0.

## Main flow

1. **Intended app path:** cart `View Bill` / takeaway `Collect Payment`, or Order History → Live → bill button, opens **Bill Breakdown** (discount) and then **Bill Payment**, which works out the totals from the order lines, the order's tax % and the discount, and offers to print or share the bill. Printing or sharing first saves it. **Today the page never opens (BILL-D2).**
2. The client posts the eight totals to `/orders/generateBill/<orderId hex>`.
3. The server, in one transaction: locks the order (must be the restaurant's and at status 0); locks the restaurant's config row; takes `max(billNumber) + 1` for the restaurant and the current financial year; creates the bill with the posted totals (customer fields copied from the order); snapshots the order lines into `bill_items`, grouped by item ID, name, price and tax %; copies the totals onto the order and sets `orderStatus` 1.
4. It answers 201 `Bill generated successfully` with the bill and its items.

## Alternative flows

- **Order already billed or paid.** 400 `Bill already generated for this order`.
- **Another restaurant's order, or unknown.** 400 `Order not found`.
- **ID not hex.** 400 with the `hex2bin()` message (BILL-D3).
- **A line with no item name, or a figure too big for decimal(10,2), or `isDiscountPercent` not 0/1-like.** 400 `Failed to generate bill`; nothing saved.
- **Totals that don't match the lines.** Accepted and stored (BILL-D1).

## Acceptance criteria

1. Given ORDER-RUN (subtotal 500.00, tax 5 %) and the matching totals (`grandTotal` 525.00), then 201, `billNumber` = previous + 1, `financialYear` `2026-27`, `billStatus` 0, one item `Paneer Tikka` × 2, and the order has status 1 and `grandTotal` 525.00.
2. Given the same order billed again, then 400 `Bill already generated for this order`.
3. Given `grandTotal` 1.00 for ORDER-RUN, then 201 and the bill and order store 1.00 (BILL-D1).
4. Given a new order whose line has `menuItemName` null, then 400 `Failed to generate bill` every time (BILL-D1).

## Platform notes

### POS API

`OrderController::generateBill` (`GenerateBillRequest`; non-database exceptions return their message) → `BillService::generateBill`.

### Data — MySQL

See [data-model.md](./data-model.md). After billing, `POST /api/orders/{id}` returns `source: "bill"` (ORD 04) and adding a KOT is refused (ORD 03).

### POS app

`useBillPayment.generateBillNow` → `generateOrderBill` thunk (sends an `Idempotency-Key`, ignored). Unreachable today.

### Super-admin panel

None.

### Integrations

None.

## Configuration

Financial year from the server clock (`now()`); April starts a new sequence.

## Out of scope

- Voiding or editing a bill — not offered (BILL-D3).

## Open questions

- None open.
