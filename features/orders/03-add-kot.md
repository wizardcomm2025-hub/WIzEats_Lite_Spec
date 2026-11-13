# 03 — Add a KOT to a running order

## Summary

While the order is still in progress, more items can be sent as further KOTs on the same order. In the app this works only within the same cart visit, right after the first KOT. A table never re-opens its order (ORD-D2).

## Prerequisites

- **Backend:** `POST /api/orders/kots/{orderId}` — [api-contract.md](./api-contract.md#post-apiorderskotsorderid)
- **Data:** `orders`, `kots`, `order_items`

## Actors & preconditions

- **Actor:** owner or waiter (any role through the API — ORD-D4).
- **Preconditions:** the cart holds an order ID with `orderStatus` 0.

## Main flow

1. After the first KOT, staff add more items (`Continue Order` → menu), tick them and send again.
2. The app posts `{ items, kotNote, … }` to `/orders/kots/<orderId hex>`. Table, room and channel fields are ignored here.
3. The server locks the order row, checks that it belongs to the restaurant and has status 0, creates the next KOT number (ORD-D3), and adds the lines with the order's own tax %. It then raises the subtotal.
4. It answers 201 `KOT added successfully` with `{ orderId, orderNumber, kotId, kotNumber, subTotal }`.
5. The app shows the toast `Items added to KOT successfully!` and adds the KOT to the card.

## Alternative flows

- **Bill already generated** (status 1 or 2). 400 `Bill already generated, cannot add items to this order`, shown as a toast.
- **Another restaurant's order, or an unknown ID.** 400 `Order not found`.
- **Not a hex ID.** 400 with the PHP message (ORD-D4).
- **`kotNote` not sent.** 400 `Undefined array key "kotNote"` (ORD-D4). The app always sends it.
- **Tax rate changed since the first KOT.** New lines use the order's original tax %.

## Acceptance criteria

1. Given an order created in this cart visit, when `Chicken 65` × 1 is sent, then the toast `Items added to KOT successfully!` shows, the order's subtotal rises by 300.00, and a second KOT exists.
2. Given the order's bill was generated, then adding a KOT through the API answers 400 `Bill already generated, cannot add items to this order`.
3. Given REST-B's order ID, then 400 `Order not found`.

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

- None open.
