# 02 — Take payment

## Summary

Staff record how the bill was paid — cash, UPI, or a split of both — and the order is closed, its bill marked paid and its table or room freed. An order not billed yet is billed in the same request. **Today this can only be done through the API (BILL-D2), and amounts are never checked (BILL-D1).**

## Prerequisites

- **Backend:** `POST /api/payments/create2` (the app's), `POST /api/payments/create` (older single payment) — [api-contract.md](./api-contract.md#post-apipaymentscreate2)
- **Data:** `orders`, `bills`, `bill_items`, `payments`, `tables`, `rooms`

## Actors & preconditions

- **Actor:** owner or waiter (any role through the API — BILL-D3).
- **Preconditions:** an order of the caller's restaurant with `orderStatus` 0 or 1.

## Main flow

1. **Intended app path:** on **Bill Payment** the cashier enters one leg per method (CASH with the cash handed over and change due; UPI), until the remaining balance is 0, then confirms. The app first saves the bill (01) if it isn't saved, checks the saved total equals the screen's, then posts `{ orderId, payments: [ { paymentType, amount, receivedAmount, changeAmount, paymentNote } ] }`. **Today the page never opens (BILL-D2).**
2. The server, in one transaction: locks the order (the restaurant's, not status 2); if status 1, reads its bill; if status 0, generates the bill from the totals in the request (pay-and-bill); creates one `payments` row per entry; then frees the order's table / room (if it has one), sets the bill's `billStatus` 1, and the order's `orderStatus` 2 and `paymentStatus` 1.
3. It answers 200 `Payment created successfully` with `{ payments, bill }`.

## Alternative flows

- **Already paid.** 500 `Order is already paid`.
- **Another restaurant's order, or unknown.** 500 `Order not found`.
- **Status 0 and no `grandTotal`.** 500 `Bill not generated for this order`.
- **Status 0 with `grandTotal` but other totals missing.** 500 `Undefined array key "subTotal"` (or the first missing key) — BILL-D3.
- **`payments` missing and no old-payload fields.** 422 (first message, e.g. `The amount field is required when payments is not present.`).
- **Old-payload fields only, on `create2`.** 500 `Payment details are required`.
- **Dashed UUID on `create2`.** 500 `hex2bin(): Input string must be hexadecimal string`.
- **Database failure.** 500 `Failed to create payment`; everything rolled back.
- **`payments/create`** — one payment from `amount`, `paymentType`, `paymentNote`; `amount` is stored as `receivedAmount`. Same rules otherwise.

## Acceptance criteria

1. Given ORDER-BILLED (`grandTotal` 525.00), when `create2` posts one CASH entry with `amount` 525 and `receivedAmount` 600, `changeAmount` 75, then 200; one payment (`receivedAmount` 600.00, `changeAmount` 75.00); bill `billStatus` 1; order status 2, `paymentStatus` 1.
2. Given ORDER-BILLED, when `create2` posts one entry with `amount` 1, then the order is paid all the same (BILL-D1).
3. Given an order already paid, then 500 `Order is already paid`.
4. Given ORDER-RUN (status 0) and the eight totals plus `payments`, then the bill is generated and paid in one request.
5. Given an order linked to Table 1 (created through the API), when paid, then Table 1 `status` is 0; an order with no table changes no table (BILL-D2).

## Platform notes

### POS API

`PaymentController::store2` / `store` (`PaymentStoreRequest`) → `PaymentService::create2` / `create`. Errors other than database errors return their message, always with HTTP 500.

### Data — MySQL

See [data-model.md](./data-model.md).

### POS app

`useBillPayment.confirmPayment` → `createPayments` thunk (`payments/create2`, with an `Idempotency-Key`, ignored). A fully discounted bill is sent as one zero-value leg with the note `Fully discounted — nothing to collect`. Unreachable today.

### Super-admin panel

None.

### Integrations

None (UPI is recorded, not collected; the QR on the bill comes from RST).

## Configuration

None.

## Out of scope

- Refunds — not offered (BILL-D3).

## Open questions

- None open.
