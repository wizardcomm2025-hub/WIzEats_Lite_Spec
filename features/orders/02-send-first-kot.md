# 02 — Send the first KOT (create the order)

## Summary

From the menu, staff add items to the cart, tick the ones to send, add an optional KOT note, and tap to generate the KOT. The first KOT creates the order with its number and the restaurant's current tax rate, and marks the table or room as ordered. The app shows the KOT and keeps the order open in the cart for further KOTs. Today the server trusts the app's prices and items (ORD-D1), ignores the app's retry key (ORD-D4) and never captures customer details (ORD-D3) — all *To fix*.

## Prerequisites

- **Backend:** `POST /api/orders/createOrder` — [api-contract.md](./api-contract.md#post-apiorderscreateorder)
- **Data:** `orders`, `kots`, `order_items`, `restaurant_configs` (tax %, code), `tables` / `rooms` (status) — [data-model.md](./data-model.md)
- **Depends on:** [01](./01-choose-channel-and-table.md), MENU (ordering menu)

## Actors & preconditions

- **Actor:** owner or waiter (today any role through the API; target owner and waiter — X-D1).
- **Preconditions:** a channel is chosen. For Dine In a table, for Room Service a room is chosen.

## Main flow

1. The ordering screen lists the active menu items of active categories. Tapping adds an item to the cart. In the cart, quantities can be changed and each item ticked for this KOT.
2. With nothing ticked, the toast says `Tick at least one item to send on this KOT.`
3. On send, the app posts `{ orderChannelId: <channel hex>, tableId: <table hex or null>, roomId: <room hex or null>, kotNote, customerName, customerPhone, items: [ { menuItemId, menuItemName, quantity, price } ] }` with an `Idempotency-Key` header (the same key when the same items are retried).
4. The server (one transaction):
   1. *Target:* checks the role (X-D1), the `Idempotency-Key` (ORD-D4), that the channel, table / room and items are the restaurant's and active (ORD-D1), that the table / room has no running order (ORD-D4), and the customer fields for Delivery (ORD-D3).
   2. Reads the tax %; null counts as 0.
   3. Picks the next order number `<restaurantCode>/<n>` from today's latest order by `created_at` (since 2026-11-16; target: under a lock — ORD-D3).
   4. Creates the order: status 0, `tableId` / `roomId`, tax %, subtotal = Σ quantity × price as sent (target: menu prices), `createdBy` = the role, `createdByUserId`. Customer fields are never saved today (target: saved when sent).
   5. Sets the table / room `status` to 2 (since 2026-11-16).
   6. Creates KOT `<n>` with the note.
   7. Creates one order line per item, with the name and price as sent (target: from the menu) and the tax amount = round(total × tax % / 100, 2).
5. It answers 201 `Order created successfully` with `{ orderId, orderNumber, kotId, kotNumber, subTotal }`.
6. The app removes the sent items from the cart (held items stay), shows the KOT card / modal, and the toast `KOT generated successfully!`. If items are held, the toast adds `<n> held item(s) still in the cart.`

## Alternative flows

- **Takeaway.** Same, with no table. In the tablet panel, `New Order` starts the next customer and the toast says `Order #<number> saved — collect it from Order History.` *Target (ORD-D3):* the cart shows its customer block (`Enter customer name...` / `Enter phone number...`) for Takeaway (optional) and Delivery (required: `Customer name is required` / `Customer phone is required` under the fields).
- **No channel set.** The app sends `orderChannelId: ""` → 422 `The order channel id field is required.`
- **No network / timeout.** The toast says `No reply from the server. The KOT may already have reached the kitchen — check before sending it again.` Today sending again creates a second order; *target (ORD-D4)* the retry carries the same key and gets the first answer back.
- **Price 0, another restaurant's item, or an inactive item, through the API.** Today accepted; *target (ORD-D1)* 422 `Menu item not available` (prices come from the menu).
- **The same table ordered twice** (direct API, or a stale grid). Today two running orders; *target (ORD-D4)* 409 `This table already has a running order` with its `orderId`.

## Acceptance criteria

1. Given Dine In on Table 1 with `Paneer Tikka` × 2 ticked, when the KOT is sent, then the toast `KOT generated successfully!` shows and the KOT card shows the order number and KOT number. A new order exists with subtotal 500.00 and tax % 5.00.
2. Given (1), then the order's `tableId` is Table 1, Table 1 is `Ordered` in the grid, and tapping it opens this order (ORD-D2 fixed).
3. Given two items with one ticked, then only that one is sent, and the toast mentions 1 held item.
4. Given nothing ticked, then `Tick at least one item to send on this KOT.` shows and nothing is sent.
5. *(Target, ORD-D3)* Given Delivery with customer name `Ravi` and phone `9876543210`, then the order holds them; without them, the cart asks for them and the API answers 422.
6. *(Target, ORD-D1)* Given the API is sent `price: 0` for `Paneer Tikka`, then the line costs 250.00 (the menu price).
7. *(Target, ORD-D4)* Given the same request is sent twice with the same `Idempotency-Key`, then one order exists and both answers are the same.

## Platform notes

### POS API

- `OrderController::createOrder` validates with `CreateOrderRequest`, then calls `KotService::createOrderWithKot`.
- Errors give 500 `{ status: false, message: "Something went wrong", error: <text> }` today (target: no `error` key).

### Data — MySQL

- Inserts into `orders`, `kots` and `order_items`; updates `tables.status` / `rooms.status`.
- `orders.grandTotal`, `taxAmount` and `roundOff` stay 0 until the bill is generated (BILL).

### POS app

- `cart.tsx` `submitKot` → `createNewOrder`. The cart keeps the returned hex `orderId`, so later KOTs go to 03.
- `getCustomerInfoRequirement` decides the customer fields from the channel ID (today `'0'`–`'2'` only; target by channel code — ORD-D3).

### Super-admin panel

None.

### Integrations

None. KOT printing / sharing is device-side (PRN).

## Configuration

`restaurant_configs.taxPercent`, `restaurantCode`.

## Out of scope

- The kitchen receiving the KOT — KITCH.

## Open questions

- None open. Related decisions: ORD-D1, ORD-D2 (fixed), ORD-D3, ORD-D4.
