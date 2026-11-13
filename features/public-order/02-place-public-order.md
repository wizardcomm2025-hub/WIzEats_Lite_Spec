# 02 — Place a public order

## Summary

A customer sends their basket to `POST /api/public/order` with no `orderId`. The server is meant to create an order, its first KOT and the order lines. It would also mark the table or room as ordered and return the order and KOT numbers with the totals. **Today every create fails with a 500 and writes nothing (PUB-D1). The reply carries the SQL insert, with the customer's details (PUB-D3).**

## Prerequisites

- **Backend:** `POST /api/public/order` (no `orderId`); see [api-contract.md](./api-contract.md#post-apipublicorder)
- **Data:** `restaurant_configs`, `order_channels`, `orders`, `kots`, `order_items`, `tables`, `rooms`

## Actors & preconditions

- **Actor:** anyone; no token is needed.
- **Preconditions:** none checked.

## Main flow

1. The caller posts:
   - `restaurantId`, `tableId`, `roomId`: both of the last two must be present as keys, and either may be `null`;
   - `orderChannelId`: a required integer, which is then ignored;
   - `kotNote` (optional);
   - `items[]` with `menuItemId`, `menuItemName`, `quantity`, `price`;
   - the optional `customerName`, `customerPhone`, `customerAddress`.
2. The server validates the body ([api-contract.md](./api-contract.md#post-apipublicorder)).
3. The controller computes `subTotal` = Σ quantity × price from the body (PUB-D2). It sets `orderChannelId` 3 and `paymentStatus` 1.
4. **Intended** (inside one transaction):
   - read `taxPercent` (0 when not found); work out the tax, round the grand total to the rupee, and keep the difference as `roundOff`;
   - build the order number `DDMMYY/<restaurantId>/OD-<n>`, where `n` follows today's last order;
   - insert the order (`orderStatus` 0, `createdBy` `customer`), then KOT `…/KO-1`, then the lines;
   - set the table's or room's `status` to 2;
   - answer 201 `{ status: true, message: "Order created successfully", data: { orderId, orderNumber, kotId, kotNumber, subTotal, taxAmount, grandTotal, roundOff } }`.
5. **Today:** the order insert names the missing `orderChannel` column. The transaction rolls back, and the reply is 500 (see the alternative flows).

## Alternative flows

- **Valid body, hex IDs.** 500 `{ status: false, message: "Something went wrong", error: "Failed to create order with KOT: SQLSTATE[42S22]: Column not found: 1054 Unknown column 'orderChannel' in 'field list' (Connection: mysql, SQL: insert into `orders` (…) values (…))" }`:
  - the SQL values include the order number, the client-computed totals and the customer's name, phone and address;
  - nothing is written;
  - the tax is 0, because the hex restaurant ID matches no config row.
- **Valid body, raw `restaurantId` (form-encoded).** The same failure, but the error text now contains raw bytes and can't be encoded as JSON. The reply is 500 `{ "message": "Server Error" }`.
- **`tableId` or `roomId` key missing.** 500 `{ status: false, message: "Something went wrong", error: "Undefined array key \"tableId\"" }` (or `"roomId"`).
- **Validation failure.** 422 `{ status: false, message: <first error>, errors }`.
- **`orderId` present but empty (`""` or `null`).** It counts as absent: this flow.

## Acceptance criteria

1. Given PUB-BODY, when it is posted, then 500 with `message` `Something went wrong` and `error` starting `Failed to create order with KOT: SQLSTATE[42S22]: Column not found: 1054 Unknown column 'orderChannel'`, and the row counts match ROW-SNAP.
2. Given PUB-BODY without the `tableId` key, then 500 `error` `Undefined array key "tableId"`, and nothing is written.
3. Given PUB-BODY with `items: []`, then 422 `The items field is required.`
4. Given PUB-BODY, then the `error` text contains `9990001111` and `5 B Road` (PUB-D3).

## Platform notes

### POS API

`PublicController::order` → `PublicService::createPublicOrderWithKot`. The service wraps any error as `Failed to create order with KOT: <message>`. The controller answers every `Throwable` with 500 and the message. Both log to the Google Sheet (once per exception object).

### Data — MySQL

Strict mode. The `orders` insert names `orderChannel` (missing). The `kots` / `order_items` inserts give no primary key, and there's no default or trigger.

### POS app

None.

### Super-admin panel

None.

### Integrations

Google Sheet error log, when configured (PUB-D3).

## Configuration

As [01](./01-load-public-menu.md#configuration).

## Out of scope

- Customer payment. Notifying the restaurant (the order would appear on DLV / KITCH).

## Open questions

- None open.
