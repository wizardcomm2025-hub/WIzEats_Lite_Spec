# 03 — Add to a public order

## Summary

A customer who already ordered sends more items with the order's `orderId`. The server is meant to add a new KOT and its lines to that order and recompute the totals. **Today it never succeeds (PUB-D1):**
- a hex `orderId` fails PHP's `int` parameter;
- a numeric `orderId` matches the wrong order or none;
- the KOT insert has no ID.

## Prerequisites

- **Backend:** `POST /api/public/order` with `orderId`; see [api-contract.md](./api-contract.md#post-apipublicorder)
- **Data:** `orders`, `kots`, `order_items`

## Actors & preconditions

- **Actor:** anyone; no token is needed.
- **Preconditions:** intended: an in-progress order (`orderStatus` 0) of the restaurant sent.

## Main flow

1. The caller posts the same body as [02](./02-place-public-order.md#main-flow) plus `orderId`. The full validation still applies, including the ignored `orderChannelId`. `orderId` itself is not validated.
2. The controller calls `addKotToPublicOrder(int $orderId, …)`.
3. **Intended** (inside one transaction):
   - lock the order with that ID and the given `restaurantId`. If there's none: `Order not found`;
   - if `orderStatus` isn't 0: `Bill already generated, cannot add items to this order`;
   - insert a KOT with `kotNote` and `createdBy` `customer`. It gets **no** `kotNumber`;
   - insert the lines;
   - add Σ quantity × price to `subTotal`, recompute tax from the order's `taxPercent`, and round the grand total;
   - answer 201 `{ status: true, message: "Order created successfully", data: { orderId, kotId, subTotal, taxAmount, grandTotal } }`.
4. **Today:** see the alternative flows; nothing is written.

## Alternative flows

- **Hex `orderId` starting with a letter.** 500 `{ status: false, message: "Something went wrong", error: "App\\Services\\Public\\PublicService::addKotToPublicOrder(): Argument #1 ($orderId) must be of type int, string given, called in <server path>\\app\\Http\\Controllers\\Public\\PublicController.php on line 42" }`.
- **Hex `orderId` starting with a digit.** The leading digits are used, with a PHP warning, which Laravel turns into an exception: 500 `error` `A non-numeric value encountered`.
- **Numeric `orderId` with a hex `restaurantId`.** No order matches: 500 `error` `Order not found`.
- **`orderId` `0` with a raw `restaurantId` (form-encoded).** MySQL compares each binary `orderId` as a number, and most read as 0. So the lookup takes some order of that restaurant. Then:
  - that order is billed (`orderStatus` ≠ 0): 500 `error` `Bill already generated, cannot add items to this order`;
  - it is in progress and `kotNote` is left out: 500 `error` `Undefined array key "kotNote"`;
  - it is in progress and `kotNote` is sent: 500 `error` `SQLSTATE[HY000]: General error: 1364 Field 'kotId' doesn't have a default value (Connection: mysql, SQL: insert into `kots` …)`.

  Any restaurant's raw ID works; no credentials are needed.
- **`orderId` `""` or `null`.** Treated as absent: use case 02.
- **`orderId` an array.** `TypeError` (`array given`): 500.
- **Validation failure.** 422, before any lookup.

## Acceptance criteria

1. Given ORDER-L's hex ID as `orderId` and PUB-BODY, then 500 with `error` starting `App\Services\Public\PublicService::addKotToPublicOrder(): Argument #1 ($orderId) must be of type int, string given`. ORDER-L is unchanged.
2. Given `orderId` `0` and PUB-BODY (hex `restaurantId`), then 500 `error` `Order not found`.
3. Given `orderId` `0`, RAWB(REST-A) and `kotNote` `more`, with ORDER-L REST-A's only order, then 500 `error` starting `SQLSTATE[HY000]: General error: 1364 Field 'kotId' doesn't have a default value`. ORDER-L's totals and lines are unchanged.

## Platform notes

### POS API

`PublicController::order` (`isset($request->orderId)`) → `PublicService::addKotToPublicOrder(int $orderId, array $data)`. Errors are logged to the Google Sheet and rethrown, and the controller answers 500 with the message.

### Data — MySQL

The `SELECT … FOR UPDATE` compares binary with an integer, which in a SELECT only gives warnings. Even if the KOT insert worked, the later `UPDATE orders … WHERE orderId = <int>` would fail with 1292 in strict mode (see DLV-D2).

### POS app

None.

### Super-admin panel

None.

### Integrations

Google Sheet error log, when configured.

## Configuration

As [01](./01-load-public-menu.md#configuration).

## Out of scope

- Re-opening a billed order.

## Open questions

- None open.
