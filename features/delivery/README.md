# Delivery board

The `DELIVERY` tile on Home opens **Delivery Orders**: the restaurant's home-delivery orders, mostly placed online by customers. Staff accept or reject each new order, send it `Out For Delivery`, and finally tap `Complete Payment`. Each step is meant to send the customer a WhatsApp message.

The spec records today's behaviour. Decisions DLV-D1 to DLV-D3 keep the current rules as accepted risks. **Today the board never lists an order, and the status endpoint never updates the KOT it is given (DLV-D1, DLV-D2).**

**Feature code:** `DLV` (test IDs `DLV-01-T01`, `DLV-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Open the delivery board](./01-open-delivery-board.md) | Owner, waiter (any role through the API) | [tester](./test-cases/01-open-delivery-board.tester.md) | [developer](./test-cases/01-open-delivery-board.developer.md) |
| 02 | [Move a delivery order on](./02-move-delivery-order.md) | same (DLV-D2) | [tester](./test-cases/02-move-delivery-order.tester.md) | [developer](./test-cases/02-move-delivery-order.developer.md) |

## Key concepts

- **Delivery channel** — the restaurant's `order_channels` row with `channelCode` `DELIVERY` (created at onboarding). Its ID is a GUID.
- **Delivery order** — an `orders` row on that channel, with customer name, phone and address. The app has no way to create one (the tile opens only the board); online orders come from the public QR / web flow (PUB).
- **Board stage** — the stage of the order's first KOT (`kots.status`), shown with board labels: 0 `Pending`, 1 `Accepted`, 2 `Rejected`, 3 `Completed`, 4 `Out for Delivery`.
- **Board actions** — `Accept` → 1, `Reject` → 2, `Out For Delivery` → 4, `Complete Payment` → 3.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api-dev_guid/specs/delivery/` (created with the first plan)

## Depends on

- **ONB** — creates the Delivery channel. **ORD** — Home channels and the cart's channel ID.
- **PUB** — where delivery orders would come from.
- **KITCH** — the same `kots.status` values.
- **BILL** — real payment; the board's `Complete Payment` doesn't record one (DLV-D3).

## Decisions

- **DLV-D1 (2026-11-11):** The board never loads:
  - the app turns the channel's hex ID into `Number(<hex>)`, which is `NaN`, and falls back to the old fixed ID `3`. It requests `/orders/channel/3`;
  - the server's `hex2bin("3")` fails, so the answer is 404 `{ status: false, message: "hex2bin(): Hexadecimal input must have an even length" }`. The app shows that text as an error toast on the first visit of each sign-in; later polls and pulls fail silently;
  - with a real channel ID, the query filters on `orders.orderChannel`, a column the GUID schema doesn't have. The server answers 200 `{ status: true, data: { status: false, message: "Failed to retrieve delivery orders", code: 500 } }`, which the app shows as an empty list;
  - the app can't create a delivery order itself.

  Accepted.
- **DLV-D2 (2026-11-11):** The status endpoint `POST /api/orders/update-status/{kotId}` stays as it is:
  - the controller casts the ID to an integer: a hex ID starting with digits becomes those digits, and one starting with a letter or `0` becomes `0`;
  - for a non-zero number, normally no KOT matches: 400 `KOT not found`;
  - for `0`, MySQL compares every binary `kotId` as a number. Most read as 0, so the lookup finds **some KOT of any restaurant**. The update that follows (`WHERE kotId = 0`) then fails in strict mode (error 1292). The reply is 400 with the SQL error text, or 500 `Server Error` when that text can't be JSON-encoded. Nothing is changed;
  - there is no role, restaurant or stage check, and any integer status passes validation. If the update ever ran (strict mode off), it would change every KOT whose ID reads as that number, in every restaurant.

  Accepted risk.
- **DLV-D3 (2026-11-11):** The flow design stays:
  - after a successful update to status 1, 3 or 4, the server sends a WhatsApp text (`Your order has been accepted.` / `Your order has been completed.` / `Your order is out for delivery.`) to a **phone number fixed in the code**, not the customer's. Status 2 and 0 send nothing; a send failure is only logged;
  - `Complete Payment` only sets the KOT to status 3; no payment or bill is recorded;
  - the list takes one row per order from the first KOT line found, drops order lines whose KOT is status 3 (so a completed order disappears), and is sorted by `orderId`, a random GUID.

  None of this is reached today. Accepted.
- **Inherited:** RST-D1 (the Delivery tile always shows), ORD-D3 (customer fields are never saved by the app's create path), AUTH-D2, AUTH-D5.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Placing online orders — PUB. Bills and payments — BILL.
- The KITCH `chef/update-status` endpoint (same `kots.status`, different route).
