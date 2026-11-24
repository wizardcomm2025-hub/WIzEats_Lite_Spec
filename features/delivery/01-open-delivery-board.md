# 01 — Open the delivery board

## Summary

Staff tap `DELIVERY` on Home and see the restaurant's open delivery orders: number, stage, customer, address, items and total. **Today the board is always empty: the app asks for the right channel since its fix, but the server's list query fails (DLV-D1, To fix).**

## Prerequisites

- **Backend:** `POST /api/orders/channel/{orderChannelId}` — [api-contract.md](./api-contract.md#post-apiorderschannelorderchannelid)
- **Data:** `orders`, `order_items`, `kots`, `order_channels`

## Actors & preconditions

- **Actor:** owner or waiter (Home is not offered to a chef). Today any role through the API; target owner and waiter (X-D1).
- **Preconditions:** signed in; the Delivery channel is switched on (otherwise there is no tile — RST).

## Main flow

1. Staff tap the `DELIVERY` tile. The app opens **Delivery Orders** (`Accept, reject and track home delivery`).
2. The app takes the restaurant's `DELIVERY` channel ID from Home's channel list (fallback `3` only if there is none) and posts `/orders/channel/<channel hex>`. The first load shows `Loading delivery orders...`; after that the app polls every 15 s while the screen is open and in the foreground, and pull down asks again.
3. **Target:** the server checks the channel is the restaurant's Delivery channel and lists its open orders, one per order, newest first; the app shows a card for each: `Order #<orderNumber>`, stage pill, customer, address, items, `Order total`. **Today:** the server answers 200 with a wrapped `Failed to retrieve delivery orders`, which the app shows as an empty list (DLV-D1).
4. Empty list: `No Delivery Orders` · `No delivery orders in this section. Pull down to refresh.`

## Alternative flows

- **Load failure.** A red toast with the server's message (today only when the server answers with an HTTP error, e.g. the `3` fallback's `hex2bin()` 404).
- **Path not hex, or another restaurant's channel.** Today 404 with the `hex2bin()` message, or the wrapped failure. *Target:* 404 `Channel not found`.
- **No `New Order` button.** The board can't create orders (DLV-D4, accepted).

## Acceptance criteria

1. Given WTR-A and REST-A with no delivery orders, when `DELIVERY` is tapped, then the screen shows `No Delivery Orders` and no toast.
2. *(Target, DLV-D1)* Given REST-A has a Pending online delivery order (PUB), when the board opens, then a card shows `Order #<number>`, `Pending`, the customer's name, phone and address, the items and the total. (Today: `No Delivery Orders`.)
3. *(Target, DLV-D1)* Given REST-B's Delivery channel ID, when posted with REST-A's token, then 404 `Channel not found`.

## Platform notes

### POS API

`OrderController::getOrdersList` (`hex2bin` of the path, after removing dashes; any failure → 404 with the exception message) → `OrderService::getOrdersList` (catches its own failure and returns it inside `data`).

### Data — MySQL

Read only. The query names `orders.orderChannel`, which isn't in `db_script/pos_stage.sql` (it exists only in the old integer migration).

### POS app

`delivery/DeliveryScreen.tsx`, `useDelivery` (channel from `home.channels` by `channelCode` `DELIVERY`; fallback `DELIVERY_CHANNEL_ID = '3'`; error toast whenever a load fails with an HTTP error), `delivery.mapper.ts` (walks `data` → `data` to the first array; IDs kept as strings).

### Super-admin panel

None.

### Integrations

None.

## Configuration

Polling interval 15 s.

## Out of scope

- Creating delivery orders — PUB (DLV-D4).

## Open questions

- None open. Related decisions: DLV-D1, DLV-D4.
