# 04 — View a running order

## Summary

The cart shows an existing order's KOTs and lines by calling `POST /api/orders/{orderId}`. The server answers correctly, but the app can't use it today (ORD-D2).

## Prerequisites

- **Backend:** `POST /api/orders/{orderId}` — [api-contract.md](./api-contract.md#post-apiordersorderid)

## Actors & preconditions

- **Actor:** owner or waiter (any role through the API — ORD-D4).
- **Trigger:** the cart has an order ID: after a KOT, or from a table with a running order.

## Main flow (server)

1. The server finds the order in the caller's restaurant.
2. **Status 0** (`source: "live"`): it returns the order number, status, subtotal, tax % and `kots`. Each KOT has its number and its items grouped by item and price: `totalQuantity`, `total`, `taxAmount`. The item name is the **current** menu name, and lines whose menu item no longer exists are dropped (inner join).
3. **Status 1 or 2 with a bill** (`source: "bill"`): it returns the bill's number, status, grand total and lines.

## Main flow (app, today)

1. The app requests `/orders/<Number(orderId)>`, which is `/orders/NaN`.
2. The server answers 404 with the PHP `hex2bin` message (ORD-D2, ORD-D4).
3. The cart can't show the existing KOTs. Even with a valid ID, the app looks for `data.order`, which isn't in the reply, and reports `Failed to fetch order`.

## Alternative flows

- **Unknown order or another restaurant's.** 404 `Order not found`.

## Acceptance criteria

1. Given the API is called with a running order's hex ID, then 200 with `source: "live"` and its KOTs and lines.
2. Given a billed order, then `source: "bill"` with the bill's figures.
3. Given REST-B's order, then 404 `Order not found`.
4. Given the app, after sending a KOT, then the sent KOT shows on the KOT card, but the earlier KOTs of the order aren't loaded (ORD-D2).

## Platform notes

### POS API

`OrderController::getBill` → `KotService::getMergedBill`. On any error it answers 404 with the exception message.

### Data — MySQL

Read only.

### POS app

`getAllOrders` / `getOrders` in `cart.api.ts`.

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- The saved bill for a reprint (`payments/bill/{id}`) — BILL.

## Open questions

- None open.
