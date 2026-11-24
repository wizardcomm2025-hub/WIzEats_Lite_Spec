# 04 — View a running order

## Summary

The cart shows an existing order's KOTs and lines by calling `POST /api/orders/{orderId}`. Since the app fix (ORD-D2) it sends the hex ID and reads the server's flat reply, so re-opening an order from the table or room grid shows everything already sent.

## Prerequisites

- **Backend:** `POST /api/orders/{orderId}` — [api-contract.md](./api-contract.md#post-apiordersorderid)

## Actors & preconditions

- **Actor:** owner or waiter (today any role through the API; target owner and waiter — X-D1).
- **Trigger:** the cart has an order ID: after a KOT, or from a table / room with a running order.

## Main flow

1. The app requests `/orders/<orderId hex>`.
2. The server finds the order in the caller's restaurant.
3. **Status 0** (`source: "live"`): it returns the order number, status, subtotal, tax % and `kots`. Each KOT has its number and its items grouped by item and price: `totalQuantity`, `total`, `taxAmount`. The item name is the **current** menu name, and lines whose menu item no longer exists are dropped (inner join).
4. **Status 1 or 2 with a bill** (`source: "bill"`): it returns the bill's number, status, grand total and lines.
5. The app rebuilds its `{ order, kots }` shape from the flat reply and shows the order's KOTs and lines in the cart.

## Alternative flows

- **Unknown order or another restaurant's.** 404 `Order not found`; the app shows `Failed to fetch order`.
- **Not a hex ID** (API only). Today 404 with the PHP warning text; *target (X-D5)* 404 `Order not found`.

## Acceptance criteria

1. Given the API is called with a running order's hex ID, then 200 with `source: "live"` and its KOTs and lines.
2. Given a billed order, then `source: "bill"` with the bill's figures.
3. Given REST-B's order, then 404 `Order not found`.
4. Given a table with a running order of two KOTs, when it is tapped, then the cart shows both KOTs and their lines (ORD-D2 fixed).

## Platform notes

### POS API

`OrderController::getBill` → `KotService::getMergedBill`. On any error it answers 404 with the exception message (target: `Order not found` only).

### Data — MySQL

Read only.

### POS app

`getAllOrders` / `getOrders` in `cart.api.ts` (handles the flat reply; one request per order at a time).

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- The saved bill for a reprint (`payments/bill/{id}`) — BILL.

## Open questions

- None open. Related decision: ORD-D2 (fixed).
