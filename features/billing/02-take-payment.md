# 02 — Take payment

## Summary

Staff record how the bill was paid — cash, UPI, or a split of both — and the order is closed, its bill marked paid and its table or room freed. An order not billed yet is billed in the same request. The page opens from the app since the BILL-D2 fix. **Today amounts are never checked (BILL-D1, To fix).**

## Prerequisites

- **Backend:** `POST /api/payments/create2` (the app's), `POST /api/payments/create` (older single payment) — [api-contract.md](./api-contract.md#post-apipaymentscreate2)
- **Data:** `orders`, `bills`, `bill_items`, `payments`, `tables`, `rooms`

## Actors & preconditions

- **Actor:** owner or waiter (today any role through the API; target owner and waiter — BILL-D3).
- **Preconditions:** an order of the caller's restaurant with `orderStatus` 0 or 1.

## Main flow

1. On **Bill Payment** the cashier enters one leg per method (CASH with the cash handed over and the change due; UPI with the QR from the restaurant's UPI ID), until the remaining balance is 0, then confirms. The customer display, if any, shows the amount to pay.
2. The app first saves the bill (01) if it isn't saved, checks the saved total equals the screen's, then posts `{ orderId, payments: [ { paymentType, amount, receivedAmount, changeAmount, paymentNote } ] }` with an `Idempotency-Key`.
3. The server, in one transaction: locks the order (the restaurant's, not status 2); if status 1, reads its bill; if status 0, generates the bill (pay-and-bill — today from the totals in the request, *target* from the discount sent); *target:* checks each leg and that the legs add up to the grand total; creates one `payments` row per leg; frees the order's table / room; sets the bill's `billStatus` 1, and the order's `orderStatus` 2 and `paymentStatus` 1. *Target (DLV-D3):* a Delivery-channel order's KOT is set to Completed.
4. It answers 200 `Payment created successfully` with `{ payments, bill }`.
5. The app shows the payment success dialog (`Done`, or `Next Order` for takeaway); the customer display shows the change, if any.

## Alternative flows

- **Already paid.** Today 500 `Order is already paid`; *target* 409.
- **Another restaurant's order, unknown, or a bad / dashed ID.** Today 500 `Order not found` or the `hex2bin()` message; *target* 404 `Order not found` (dashed IDs accepted).
- **Status 0 and no totals** (today: 500 `Bill not generated for this order`, or `Undefined array key "subTotal"` when only some are sent). *Target:* billed from the discount sent (none → no discount).
- **Legs that don't add up** (e.g. one leg of ₹1 on a ₹525 bill). Today the order is paid. *Target:* 422 `Payment total does not match the bill`; nothing written.
- **`paymentType` other than CASH / UPI.** Today stored. *Target:* 422.
- **`payments` missing and no old-payload fields.** 422 (first message, e.g. `The amount field is required when payments is not present.`).
- **Old-payload fields only, on `create2`.** Today 500 `Payment details are required`; *target* 422.
- **Database failure.** 500 `Failed to create payment`; everything rolled back.
- **Fully discounted bill.** The app sends one zero-value leg with the note `Fully discounted — nothing to collect`.
- **`payments/create`** — one payment from `amount`, `paymentType`, `paymentNote`; `amount` is stored as `receivedAmount`. Same rules otherwise (target: `amount` must equal the grand total).

## Acceptance criteria

1. Given ORDER-BILLED (`grandTotal` 525.00), when `create2` posts one CASH leg with `amount` 525 and `receivedAmount` 600, `changeAmount` 75, then 200; one payment (`receivedAmount` 600.00, `changeAmount` 75.00; target also `amount` 525.00); bill `billStatus` 1; order status 2, `paymentStatus` 1.
2. *(Target, BILL-D1)* Given ORDER-BILLED, when `create2` posts one leg with `amount` 1, then 422 `Payment total does not match the bill` and the order stays unpaid. (Today: paid.)
3. Given an order already paid, then today 500 / *(target)* 409 `Order is already paid`.
4. Given ORDER-RUN (status 0) and `payments` (plus, today, the eight totals), then the bill is generated and paid in one request.
5. Given Table 1 holds an app order, when it is paid in the app, then Table 1 is `Available` again (BILL-D2 fixed).

## Platform notes

### POS API

`PaymentController::store2` / `store` (`PaymentStoreRequest`) → `PaymentService::create2` / `create`. Today errors other than database errors return their message, always with HTTP 500.

### Data — MySQL

See [data-model.md](./data-model.md). Target adds `payments.amount` (migration).

### POS app

`useBillPayment.confirmPayment` → `createPayments` thunk (`payments/create2`, with an `Idempotency-Key`). `useCustomerDisplay` mirrors the total and the change on terminals with a customer display.

### Super-admin panel

None.

### Integrations

None (UPI is recorded, not collected; the QR on the bill comes from RST). Customer display on supported terminals.

## Configuration

None.

## Out of scope

- Refunds — not offered (accepted, BILL-D3).

## Open questions

- None open. Related decisions: BILL-D1, BILL-D2 (fixed), BILL-D3, DLV-D3.
