# 02 — Send the first KOT (create the order)

## Summary

From the menu, staff add items to the cart, tick the ones to send, add an optional KOT note, and tap to generate the KOT. The first KOT creates the order with its number and the restaurant's current tax rate. The app shows the KOT and keeps the order open in the cart for further KOTs.

## Prerequisites

- **Backend:** `POST /api/orders/createOrder` — [api-contract.md](./api-contract.md#post-apiorderscreateorder)
- **Data:** `orders`, `kots`, `order_items`, `restaurant_configs` (tax %, code) — [data-model.md](./data-model.md)
- **Depends on:** [01](./01-choose-channel-and-table.md), MENU (ordering menu)

## Actors & preconditions

- **Actor:** owner or waiter (any role through the API — ORD-D4).
- **Preconditions:** a channel is chosen. For Dine In, a table is chosen.

## Main flow

1. The ordering screen lists the active menu items. Tapping adds an item to the cart. In the cart, quantities can be changed and each item ticked for this KOT.
2. With nothing ticked, the toast says `Tick at least one item to send on this KOT.`
3. On send, the app posts:

   ```
   {
     orderChannelId: <channel hex>,
     tableId: Number(<hex>) → null,
     roomId: … → null,
     kotNote, customerName, customerPhone,
     items: [ { menuItemId, menuItemName, quantity, price } ]
   }
   ```

   It also sends an `Idempotency-Key` header, which the server ignores (ORD-D4).
4. The server (one transaction):
   1. Reads the tax %; null counts as 0.
   2. Picks the next order number `<restaurantCode>/<n>` (ORD-D3).
   3. Creates the order: status 0, tax %, subtotal = Σ quantity × price as sent (ORD-D1), `createdBy` = the role, `createdByUserId`. Customer fields are never saved (ORD-D3).
   4. Tries to set the table / room status to 2. It matches nothing (ORD-D2).
   5. Creates KOT `<n>` with the note.
   6. Creates one order line per item, with the name and price as sent and the tax amount = round(total × tax % / 100, 2).
5. It answers 201 `Order created successfully` with `{ orderId, orderNumber, kotId, kotNumber, subTotal }`.
6. The app removes the sent items from the cart (held items stay), shows the KOT card / modal, and the toast `KOT generated successfully!`. If items are held, the toast adds `<n> held item(s) still in the cart.`

## Alternative flows

- **Takeaway.** Same, with no table. In the tablet panel, `New Order` starts the next customer and the toast says `Order #<number> saved — collect it from Order History.`
- **No channel set.** The app sends `orderChannelId: "1"`, and the server answers 500 `Something went wrong` (hex conversion). The app shows that message.
- **No network / timeout.** The toast says `No reply from the server. The KOT may already have reached the kitchen — check before sending it again.` Sending again creates a second order (ORD-D4).
- **Price 0, or another restaurant's item, through the API.** Accepted (ORD-D1).
- **The same table ordered twice.** Two running orders (ORD-D2, ORD-D4).

## Acceptance criteria

1. Given Dine In on Table 1 with `Paneer Tikka` × 2 ticked, when the KOT is sent, then the toast `KOT generated successfully!` shows and the KOT card shows the order number and KOT number. A new order exists with subtotal 500.00 and tax % 5.00.
2. Given (1), then the order's `tableId` is NULL, and Table 1 in the grid has no running order (ORD-D2).
3. Given two items with one ticked, then only that one is sent, and the toast mentions 1 held item.
4. Given nothing ticked, then `Tick at least one item to send on this KOT.` shows and nothing is sent.
5. Given Takeaway with customer name and phone filled, then the order has no customer name or phone (ORD-D3).
6. Given the API is sent `price: 0`, then the order line costs 0.00 (ORD-D1).

## Platform notes

### POS API

- `OrderController::createOrder` validates with `CreateOrderRequest`, then calls `KotService::createOrderWithKot`.
- Errors give 500 `{ status: false, message: "Something went wrong", error: <text> }`.

### Data — MySQL

- Inserts into `orders`, `kots` and `order_items`.
- `orders.grandTotal`, `taxAmount` and `roundOff` stay 0 until the bill is generated (BILL).

### POS app

- `cart.tsx` `submitKot` → `createNewOrder`. The cart keeps the returned hex `orderId`, so later KOTs in the same visit go to 03.

### Super-admin panel

None.

### Integrations

None. KOT printing / sharing is device-side (PRN).

## Configuration

`restaurant_configs.taxPercent`, `restaurantCode`.

## Out of scope

- The kitchen receiving the KOT — KITCH.

## Open questions

- None open. Related decisions: ORD-D1 to ORD-D4.
