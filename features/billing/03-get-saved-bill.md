# 03 — Get a saved bill

## Summary

Reads an order's saved bill (for a reprint). The app has a call for it but no screen uses it; the same snapshot also comes back from `POST /api/orders/{id}` once an order is billed (ORD 04).

## Prerequisites

- **Backend:** `POST /api/payments/bill/{orderId}` — [api-contract.md](./api-contract.md#post-apipaymentsbillorderid)
- **Data:** `bills`, `bill_items`

## Actors & preconditions

- **Actor:** any signed-in restaurant user.
- **Preconditions:** none.

## Main flow

1. The client posts `/payments/bill/<orderId hex>`.
2. The server finds the bill by order ID and the caller's restaurant and answers 200 `Bill retrieved successfully` with the bill and its items.

## Alternative flows

- **No bill, or another restaurant's order.** 404 `{ status: false, message: "Bill not found" }`.
- **ID not hex / odd length.** 500 `{ "message": "Server Error" }` — the conversion happens outside the error handling (BILL-D2).

## Acceptance criteria

1. Given ORDER-BILLED, then 200 with its `billNumber`, `billStatus` 0 and items.
2. Given ORDER-RUN (not billed) or ORDER-B, then 404 `Bill not found`.
3. Given `xyz`, then 500 `Server Error`.

## Platform notes

### POS API

`PaymentController::getBill` → `PaymentService::getBill` → `BillService::getBillByOrderId`.

### Data — MySQL

Read only.

### POS app

`cart.api.getBill` (treats 404 as "no bill"); not called by any screen.

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- Printing — PRN.

## Open questions

- None open.
