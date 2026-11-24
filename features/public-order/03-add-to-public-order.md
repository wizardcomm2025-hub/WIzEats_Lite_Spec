# 03 — Add to a public order

## Summary

A customer whose order is still being prepared sends more items to it, using the `orderId` they got when placing it. **Today every attempt fails with a 500 and exception text (PUB-D1, PUB-D3 — To fix).**

## Prerequisites

- **Backend:** `POST /api/public/order` with `orderId` — [api-contract.md](./api-contract.md#post-apipublicorder)
- **Data:** `orders`, `kots`, `order_items`

## Actors & preconditions

- **Actor:** anyone holding a public order's ID.
- **Preconditions:** the order was created publicly (02) and is in progress.

## Main flow (target)

1. The caller posts `{ orderId, restaurantId, kotNote, items: [ { menuItemId, quantity } ] }` with an `Idempotency-Key`.
2. The server finds the order: the restaurant's, created publicly (`createdBy` `customer`), locked for update. Not found → 404; billed → 409.
3. It checks the items as in 02, adds a KOT with the next number and the lines with menu prices, and raises the subtotal.
4. It answers 201 `KOT added successfully` with `{ orderId, orderNumber, kotId, kotNumber, subTotal }`. The kitchen gets the new KOT.

## Main flow (today)

The service takes `int $orderId`: a hex ID gives a `TypeError` (with the server path) or `A non-numeric value encountered`; `0` can match some order of the restaurant (raw ID) and then fails on the KOT insert (1364) or a missing `kotNote` key. Every reply is 500 with the text in `error`; nothing is written.

## Alternative flows

- **A staff-created order's ID, another restaurant's order, or an unknown / malformed ID.** 404 `Order not found`.
- **Order billed or paid.** 409 `Bill already generated, cannot add items to this order`.
- **`kotNote` left out.** The KOT has no note.
- **Unexpected failure.** 500 `Something went wrong`, no exception text.

## Acceptance criteria

1. *(Target)* Given a public order from 02, when `Chicken 65` × 1 is added with its `orderId`, then 201, a second KOT, and the subtotal rises by 300.00.
2. *(Target)* Given ORD's ORDER-RUN (created by staff), then 404 `Order not found`.
3. *(Target)* Given the public order after its bill was generated, then 409.

## Platform notes

### POS API

`PublicController::order` (`isset($request->orderId)`) → `PublicService::addKotToPublicOrder`.

### Data — MySQL

See [data-model.md](./data-model.md#add-writes-use-case-03).

### POS app

None.

### Super-admin panel

None.

### Integrations

Google Sheet log on failure.

## Configuration

None.

## Out of scope

- Cancelling or changing items.

## Open questions

- None open. Related decisions: PUB-D1, PUB-D2, PUB-D3.
