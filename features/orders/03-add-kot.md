# 03 — Add a KOT to a running order

## Summary

While the order is still in progress, more items can be sent as further KOTs on the same order — right after the first KOT, or later by tapping the table / room that holds the order (ORD-D2 fixed).

## Prerequisites

- **Backend:** `POST /api/orders/kots/{orderId}` — [api-contract.md](./api-contract.md#post-apiorderskotsorderid)
- **Data:** `orders`, `kots`, `order_items`

## Actors & preconditions

- **Actor:** owner or waiter (today any role through the API; target owner and waiter — X-D1).
- **Preconditions:** the cart holds an order ID with `orderStatus` 0.

## Main flow

1. After a KOT (or after re-opening the order from the grid), staff add more items (`Continue Order` → menu), tick them and send again.
2. The app posts `{ items, kotNote, … }` to `/orders/kots/<orderId hex>` with an `Idempotency-Key`. Table, room and channel fields are ignored here.
3. The server locks the order row, checks that it belongs to the restaurant and has status 0, creates the next KOT number (today's latest KOT by `created_at` + 1), and adds the lines with the order's own tax %. It then raises the subtotal.
4. It answers 201 `KOT added successfully` with `{ orderId, orderNumber, kotId, kotNumber, subTotal }`.
5. The app shows the toast `Items added to KOT successfully!` and adds the KOT to the card.

## Alternative flows

- **Bill already generated** (status 1 or 2). 400 `Bill already generated, cannot add items to this order`, shown as a toast.
- **Another restaurant's order, or an unknown ID.** Today 400 `Order not found`; *target (X-D4)* 404.
- **Not a hex ID.** Today 400 with the PHP message; *target* 404 `Order not found`.
- **`kotNote` not sent.** Today 400 `Undefined array key "kotNote"`; *target* the KOT is added with no note. The app always sends it.
- **Retry with the same `Idempotency-Key`.** Today a second KOT; *target (ORD-D4)* the first answer again.
- **Tax rate changed since the first KOT.** New lines use the order's original tax %.

## Acceptance criteria

1. Given an order created in this cart visit, when `Chicken 65` × 1 is sent, then the toast `Items added to KOT successfully!` shows, the order's subtotal rises by 300.00, and a second KOT exists.
2. Given the order's bill was generated, then adding a KOT through the API answers 400 `Bill already generated, cannot add items to this order`.
3. Given REST-B's order ID, then today 400 / *(target)* 404 `Order not found`.
4. Given Table 1 holds a running order, when staff leave to Home, come back, tap Table 1 and send another KOT, then it is added to the same order (ORD-D2 fixed).

## Platform notes

### POS API

`OrderController::addKot` (the ID goes through `hex2bin`) → `KotService::addKotToOrder`, which uses `lockForUpdate`.

### Data — MySQL

New `kots` and `order_items` rows; `orders.subTotal` and `updated_at` are updated.

### POS app

`addKotOrder` thunk, in the same `submitKot` as 02.

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- Removing or cancelling items — not offered.

## Open questions

- None open. Related decisions: ORD-D2 (fixed), ORD-D4.
