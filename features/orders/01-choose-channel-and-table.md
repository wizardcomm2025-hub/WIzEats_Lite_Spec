# 01 — Choose a channel and a table

## Summary

Home shows one tile per active order channel. Dine In opens the table grid for the active zones; there, a free table is "occupied" before ordering, an occupied table with no order can be released with a long press, and a table with a running order opens that order. Takeaway opens the menu directly. Room Service opens the room grid, which works the same way as the table grid. Delivery opens its board (DLV).

## Prerequisites

- **Backend:**
  - `POST /api/orders/getOrderChannels`
  - `POST /api/restaurants/showDetails` (RST 01)
  - `POST /api/tables/tableSave` with `status` (TBL); `POST /api/rooms/getFloorDetails`, `getRoomsDetails`, `roomSave` with `status` (ROOM)

  See [api-contract.md](./api-contract.md).
- **Data:** `order_channels`, `tables.status`, `rooms.status`, `orders` (running order per table / room)

## Actors & preconditions

- **Actor:** owner or waiter (their Home). Today any role through the API; target owner and waiter only (X-D1).
- **Preconditions:** signed in; the restaurant has zones with tables (or floors with rooms for Room Service).

## Main flow

1. Home lists the channels returned by `getOrderChannels` — only the active ones (RST-D1 fixed 2026-11-16): `DINE IN` always, `TAKEAWAY`, `ROOM SERVICE`, `DELIVERY` when switched on.
2. **Dine In** opens the table grid:
   - a zone picker (active zones);
   - filter chips: `All`, plus one per status present (`Available`, `Occupied`, `Ordered`), each with a count;
   - table cards coloured by status.

   The grid refreshes in the background.
3. **A free table** (status 0): a sheet shows the table name, `Capacity: <n>` and the button `Ready to Occupy`. Tapping the button sets the table to status 1 (`tableSave`), then opens the menu for that table.
4. **An occupied table** (status 1) with no running order opens the menu at once; the next KOT starts a new order.
5. **A table with a running order** (status 2, `orderId` from `showDetails`) opens the cart with that order and its KOTs (ORD-D2 fixed).
6. **Long press** on an occupied table with no order opens a confirmation to leave the table. Confirming sets it back to status 0.
7. **Takeaway** opens the menu. **Room Service** opens the room grid: floor picker, rooms coloured by status, `Ready to Occupy` for a free room (`roomSave` `{ roomId, status: "1" }`), a room with a running order opens its cart, long press leaves a room with no order. **Delivery** → DLV.

## Alternative flows

- **No tables in the zone, or none with the filter.** `No <filter> tables found`, e.g. `No all tables found`.
- **Changing channel with items in the cart.** A guard asks before the cart is emptied (`Switch to <channel>?`).
- **Occupy fails.** The sheet stays; there is no message.
- **Channel switched off in the settings.** Its tile is gone (RST).
- **Waiter or chef.** Chefs have no Home; *target (X-D1)* the API refuses them (403).

## Acceptance criteria

1. Given OWN-A on Home with Takeaway on and Delivery / Room Service off, then the tiles `DINE IN` and `TAKEAWAY` show.
2. Given Dine In, then Main Hall's tables show, with chips `All 4` and `Available 4`.
3. Given `Table 1` is tapped and `Ready to Occupy` pressed, then the menu opens, and the grid later shows Table 1 as `Occupied`.
4. Given an occupied Table 1 with no order is long-pressed and the leave confirmed, then it is `Available` again.
5. Given Takeaway, then the menu opens with no table.
6. Given `getOrderChannels` is called with a `channelId`, then all active channels still come back (the filter is never applied).
7. Given a KOT has been sent on Table 2, then the grid shows Table 2 as `Ordered`, and tapping it opens the cart with that order's KOTs (ORD-D2 fixed).

## Platform notes

### POS API

- `OrderService::getOrderChannels` (active, not deleted). The request rule is `channelId`, but the service reads `orderChannelId`, so the filter never applies.
- The table status is changed through TBL's `tableSave`; the room status through ROOM's `roomSave`.

### Data — MySQL

Read only, apart from `tables.status` / `rooms.status` through `tableSave` / `roomSave`.

### POS app

`Home` / `useOpenChannel`; `Order.tsx` (table grid); `roomOrder.tsx` (room grid); `useLiveRefresh`; `cart.slice.setLocation` keeps the table's / room's hex ID; a table / room with an `orderId` sets the cart's order.

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- The Delivery board (DLV).

## Open questions

- None open. Related decision: ORD-D2 (fixed).
