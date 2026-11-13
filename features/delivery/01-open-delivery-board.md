# 01 — Open the delivery board

## Summary

Staff tap `DELIVERY` on Home and see the restaurant's open delivery orders: number, stage, customer, address, items and total. **Today the board is always empty, with an error toast on the first visit (DLV-D1).**

## Prerequisites

- **Backend:** `POST /api/orders/channel/{orderChannelId}` — [api-contract.md](./api-contract.md#post-apiorderschannelorderchannelid)
- **Data:** `orders`, `order_items`, `kots`

## Actors & preconditions

- **Actor:** owner or waiter (Home is not offered to a chef). Any role through the API.
- **Preconditions:** signed in.

## Main flow

1. Staff tap the `DELIVERY` tile. The app stores the channel's hex ID on the cart and opens **Delivery Orders** (`Accept, reject and track home delivery`).
2. The app computes `Number(<channel hex>) || 3` and posts `/orders/channel/<that>`. The first load shows `Loading delivery orders...`; after that the app polls every 15 s while the screen is open and in the foreground, and pull down asks again.
3. **Intended:** the server lists the restaurant's delivery orders on that channel, one per order, and the app shows a card for each. **Today:**
   - the app sends `3`, the server answers 404 with the `hex2bin()` message, and the app shows it as a red toast on the first load of the sign-in; the list is empty;
   - with a real channel ID (API only), the server answers 200 with a wrapped `Failed to retrieve delivery orders`, and the app would show an empty list.
4. Empty list: `No Delivery Orders` · `No delivery orders in this section. Pull down to refresh.`

## Alternative flows

- **Later loads in the same sign-in.** Failures are silent; the list keeps what it had.
- **Path not hex.** 404 with `hex2bin(): Input string must be hexadecimal string`.
- **Another restaurant's channel ID.** Same wrapped failure (the query never runs).

## Acceptance criteria

1. Given a fresh sign-in as WTR-A, when `DELIVERY` is tapped, then the toast `hex2bin(): Hexadecimal input must have an even length` appears, and the screen shows `No Delivery Orders`.
2. Given the board was opened once in this sign-in, when it is opened again, then no toast appears.
3. Given CH-DLV's hex ID, when posted directly, then 200 `{ status: true, data: { status: false, message: "Failed to retrieve delivery orders", code: 500 } }`.

## Platform notes

### POS API

`OrderController::getOrdersList` (`hex2bin` of the path, after removing dashes; any failure → 404 with the exception message) → `OrderService::getOrdersList` (catches its own failure and returns it inside `data`).

### Data — MySQL

Read only. The query names `orders.orderChannel`, which isn't in `db_script/tables/tables.sql` (it exists only in the old integer migration).

### POS app

`delivery/DeliveryScreen.tsx`, `useDelivery` (channel fallback `DELIVERY_CHANNEL_ID = 3`; the error toast only when the first load fails), `delivery.mapper.ts` (walks `data` → `data` to the first array; `orderId` and `kotId` go through `Number()`).

### Super-admin panel

None.

### Integrations

None.

## Configuration

Polling interval 15 s.

## Out of scope

- Creating delivery orders — PUB.

## Open questions

- None open.
