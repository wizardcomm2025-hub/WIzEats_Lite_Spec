# 01 — Choose a channel and a table

## Summary

Home shows one tile per order channel. Dine In opens the table grid for the active zones; there, a free table is "occupied" before ordering, and an occupied table with no order can be released with a long press. Takeaway opens the menu directly. Room Service opens the room picker, and Delivery opens its board (DLV).

## Prerequisites

- **Backend:**
  - `POST /api/orders/getOrderChannels`
  - `POST /api/restaurants/showDetails` (RST 01)
  - `POST /api/tables/tableSave` with `status` (TBL)

  See [api-contract.md](./api-contract.md).
- **Data:** `order_channels`, `tables.status`, `orders` (running order per table)

## Actors & preconditions

- **Actor:** owner or waiter (their Home). Any role through the API (ORD-D4).
- **Preconditions:** signed in; the restaurant has zones with tables.

## Main flow

1. Home lists the channels returned by `getOrderChannels` (all active, not deleted): `DINE IN`, `TAKEAWAY`, `ROOM SERVICE`, `DELIVERY` (RST-D1).
2. **Dine In** opens the table grid:
   - a zone picker (active zones);
   - filter chips: `All`, plus one per status present (`Available`, `Occupied`, `Ordered`), each with a count;
   - table cards coloured by status.

   The grid refreshes in the background.
3. **A free table** (status 0): a sheet shows the table name, `Capacity: <n>` and the button `Ready to Occupy`. Tapping the button sets the table to status 1 (`tableSave`), then opens the menu for that table.
4. **An occupied table** (status 1) with no running order opens the menu at once. A new order will start (ORD-D2).
5. **A table with a running order** (`orderId` from `showDetails`) opens the cart with that order. Today no table gets an `orderId` from the app's own orders (ORD-D2).
6. **Long press** on an occupied table with no order opens a confirmation to leave the table. Confirming sets it back to status 0.
7. **Takeaway** opens the menu. **Room Service** opens the room screen, which lists no rooms today (ROOM-D1). **Delivery** → DLV.

## Alternative flows

- **No tables in the zone, or none with the filter.** `No <filter> tables found`, e.g. `No all tables found`.
- **Changing channel with items in the cart.** A guard asks before the cart is emptied (`Switch to <channel>?`).
- **Occupy fails.** The sheet stays; there is no message.

## Acceptance criteria

1. Given OWN-A on Home, then four channel tiles show.
2. Given Dine In, then Main Hall's tables show, with chips `All 4` and `Available 4`.
3. Given `Table 1` is tapped and `Ready to Occupy` pressed, then the menu opens, and the grid later shows Table 1 as `Occupied`.
4. Given an occupied Table 1 with no order is long-pressed and the leave confirmed, then it is `Available` again.
5. Given Takeaway, then the menu opens with no table.
6. Given `getOrderChannels` is called with a `channelId`, then all channels still come back (the filter is never applied).

## Platform notes

### POS API

- `OrderService::getOrderChannels`. The request rule is `channelId`, but the service reads `orderChannelId`, so the filter never applies.
- The table status is changed through TBL's `tableSave`.

### Data — MySQL

Read only, apart from `tables.status` through `tableSave`.

### POS app

`Home` / `useOpenChannel`; `Order.tsx` (table grid); `useLiveRefresh`; `cart.slice.setLocation` keeps the table's hex ID.

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- Room picking (ROOM), the Delivery board (DLV).

## Open questions

- None open. Related decision: ORD-D2.
