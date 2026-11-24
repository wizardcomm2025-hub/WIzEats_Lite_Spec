# 02 — Place a public order

## Summary

A customer sends their items to the restaurant without signing in. The order and its first KOT are created and appear to staff like any other order — on the table grid for Dine In, in the kitchen, in Live, and on the Delivery board for delivery. **Today every create fails with a 500 that leaks the SQL and the customer's details (PUB-D1, PUB-D3 — To fix).**

## Prerequisites

- **Backend:** `POST /api/public/order` without `orderId` — [api-contract.md](./api-contract.md#post-apipublicorder)
- **Data:** `orders`, `kots`, `order_items`, `tables`, `rooms`, `menu_items`, `order_channels`, `restaurant_configs`
- **Depends on:** 01; ORD (numbering, KOTs); DLV (the board)

## Actors & preconditions

- **Actor:** anyone; no sign-in.
- **Preconditions:** an approved, active restaurant with active items.

## Main flow (target)

1. The caller posts `{ restaurantId, channelCode, tableId | roomId (as the channel needs), kotNote, items: [ { menuItemId, quantity } ], customerName, customerPhone, customerAddress }` with an `Idempotency-Key`.
2. The server checks the rate limit, the restaurant (approved, active), the channel (active, of the restaurant), the table / room (active, of the restaurant, no running order), the items (active, in active categories), and — for Delivery — the customer's name, phone and address.
3. In one transaction it creates the order (`<restaurantCode>/<n>`, status 0, tax % from the restaurant, `createdBy` `customer`, customer fields), its first KOT (next number) and the lines with the menu's names and prices; it sets the table / room to status 2.
4. It answers 201 `Order created successfully` with `{ orderId, orderNumber, kotId, kotNumber, subTotal }`.
5. Staff see it: the kitchen gets the KOT (KITCH), Live lists it (HIST), Dine In marks the table `Ordered` (ORD), and a Delivery order appears on the board (DLV).

## Main flow (today)

The controller always uses channel 3, the insert names the missing `orderChannel` column, and nothing has an ID: 500 `Something went wrong` with the full SQL — customer name, phone and address included — in `error`. Nothing is written.

## Alternative flows

- **Channel not active, or not the restaurant's.** 422 `Invalid channel`.
- **Dine In without a valid table / Room Service without a valid room.** 422 `Invalid table` / `Invalid room`.
- **Table / room already has a running order.** 409 `This table already has a running order` (no order ID given out).
- **Inactive, deleted or unknown item.** 422 `Menu item not available`.
- **Delivery without name, phone or address; bad phone.** 422 with the field's message.
- **Retry with the same `Idempotency-Key`.** The first answer again; one order.
- **More than 10 orders a minute from one IP.** 429.
- **Unexpected failure.** 500 `Something went wrong`, no exception text; the Google Sheet log keeps the detail.

## Acceptance criteria

1. *(Target)* Given REST-A and a Takeaway order of `Paneer Tikka` × 2 with `price` 1 in the body, then 201, `orderNumber` `CODE-A/<n>`, and the line costs 250.00 (the menu price).
2. *(Target)* Given a Delivery order with name, phone and address, then the order appears on REST-A's Delivery board with those details, and the kitchen sees the KOT.
3. *(Target)* Given a Dine In order on Table 2, then Table 2 is `Ordered` on the staff grid; a second public order for Table 2 gets 409.
4. *(Target)* Given any failure, then the reply holds no SQL, path or customer data.
5. Given `{}`, then 422 `The restaurant id field is required.`

## Platform notes

### POS API

`PublicController::order` → `PublicService::createPublicOrder` (today `createOrderWithKot` in the public service).

### Data — MySQL

See [data-model.md](./data-model.md#create-writes-use-case-02).

### POS app

None for the customer; staff screens show the result (ORD, KITCH, HIST, DLV).

### Super-admin panel

None.

### Integrations

Failures go to the Google Sheet log (target: without being echoed to the caller). Delivery WhatsApp messages follow DLV-D3.

## Configuration

Rate limits (target) in the route definitions.

## Out of scope

- Payment by the customer; there is no public payment endpoint.

## Open questions

- None open. Related decisions: PUB-D1, PUB-D2, PUB-D3.
