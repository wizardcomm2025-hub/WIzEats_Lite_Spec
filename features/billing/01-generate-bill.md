# 01 — Generate the bill

## Summary

Staff save the bill before payment, for example to print it for the table. The bill gets its number, the order's totals are fixed, and the order takes no more KOTs. The bill pages open from the app since the BILL-D2 fix. **Today the server stores whatever totals it is sent (BILL-D1, To fix).**

## Prerequisites

- **Backend:** `POST /api/orders/generateBill/{orderId}` — [api-contract.md](./api-contract.md#post-apiordersgeneratebillorderid)
- **Data:** `orders`, `order_items`, `bills`, `bill_items`, `restaurant_configs`

## Actors & preconditions

- **Actor:** owner or waiter (today any role through the API; target owner and waiter — BILL-D3).
- **Preconditions:** an order of the caller's restaurant with `orderStatus` 0.

## Main flow

1. Staff open the bill: cart `View Bill` (Dine In / Room Service; opens **Bill Breakdown** first), takeaway `Collect Payment`, or Order History → Live → bill button (opens **Bill Payment**). Bill Breakdown shows the lines, subtotal, tax and — if the restaurant's discount switch is on — the discount entry; applying a discount shows `Discount Applied!` with `−₹<amount>`.
2. **Bill Payment** works out the totals from the order lines, the order's tax % and the discount (discount first, tax on the rest, rounded to the nearest rupee) and offers to print or share the bill. Printing or sharing first saves it; on terminals with a customer display the amount to pay is shown there.
3. The app posts the eight totals to `/orders/generateBill/<orderId hex>` (with an `Idempotency-Key`). *Target (BILL-D1):* the server uses only the discount from the request.
4. The server, in one transaction: locks the order (must be the restaurant's and at status 0); locks the restaurant's config row; takes `max(billNumber) + 1` for the restaurant and the current financial year; creates the bill — today with the posted totals, *target* with its own calculation; copies the customer fields from the order; snapshots the order lines into `bill_items`, grouped by item ID, name, price and tax %; copies the totals onto the order and sets `orderStatus` 1.
5. It answers 201 `Bill generated successfully` with the bill and its items. The app checks the saved `grandTotal` against its own; if they differ it shows `The bill was saved at ₹<x>, not ₹<y>. Check the amount and confirm again.`

## Alternative flows

- **Order already billed or paid.** Today 400 `Bill already generated for this order`; *target* 409.
- **Another restaurant's order, unknown, or a bad ID.** Today 400 `Order not found` / the `hex2bin()` message; *target* 404 `Order not found`.
- **A line with no item name, a figure too big for decimal(10,2), or `isDiscountPercent` not 0/1-like.** Today 400 `Failed to generate bill`; nothing saved. *Target:* names come from the menu (ORD-D1) and the server computes the figures, so these can't happen.
- **Totals that don't match the lines.** Today accepted and stored. *Target:* ignored — the server's figures are stored.
- **Discount above the subtotal, or a discount while the switch is off** (API). *Target:* 422 `The discount can't be more than the subtotal` / `Discounts are turned off for this restaurant`.
- **Chef through the API.** Today allowed; *target* 403.

## Acceptance criteria

1. Given ORDER-RUN (subtotal 500.00, tax 5 %) and the matching totals (`grandTotal` 525.00), then 201, `billNumber` = previous + 1, `financialYear` `2026-27`, `billStatus` 0, one item `Paneer Tikka` × 2, and the order has status 1 and `grandTotal` 525.00.
2. Given the same order billed again, then today 400 / *(target)* 409 `Bill already generated for this order`.
3. *(Target, BILL-D1)* Given `grandTotal` 1.00 for ORDER-RUN, then 201 and the bill stores 525.00 (the server's figure). (Today: 1.00.)
4. *(Target, BILL-D1)* Given a 10 % discount on ORDER-RUN, then `discountAmount` 50.00, `taxAmount` 22.50, `roundOff` 0.50, `grandTotal` 473.00.
5. Given the app, when `View Bill` is tapped on a Dine In order with KOTs, then Bill Breakdown opens with the order's lines (BILL-D2 fixed).

## Platform notes

### POS API

`OrderController::generateBill` (`GenerateBillRequest`; non-database exceptions return their message today) → `BillService::generateBill`.

### Data — MySQL

See [data-model.md](./data-model.md). After billing, `POST /api/orders/{id}` returns `source: "bill"` (ORD 04) and adding a KOT is refused (ORD 03).

### POS app

`BillBreakdownScreen` (discount, `DiscountCelebration`), `BillPaymentScreen`, `useBillPayment.generateBillNow` → `generateOrderBill` thunk (sends an `Idempotency-Key`), `billing.utils.calculateBill`.

### Super-admin panel

None.

### Integrations

The terminal's customer display, when present (`useCustomerDisplay`).

## Configuration

Financial year from the server clock (`now()`); April starts a new sequence.

## Out of scope

- Voiding or editing a bill — not offered (accepted, BILL-D3).

## Open questions

- None open. Related decisions: BILL-D1, BILL-D2 (fixed), BILL-D3.
