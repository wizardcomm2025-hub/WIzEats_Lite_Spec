# Take an order & send KOTs

Staff pick a channel on Home: Dine In, Takeaway, Room Service or Delivery. For Dine In they choose a table, then add menu items to the cart and send them to the kitchen as a **KOT** (kitchen order ticket). The first KOT creates the **order**; later KOTs, sent while the order is still in progress, add to it. Billing and payment follow (BILL).

The spec records today's behaviour. Decisions ORD-D1 to ORD-D4 keep the current rules as accepted risks. **Orders are never linked to their table or room today, and a running order can't be re-opened (ORD-D2).**

**Feature code:** `ORD` (test IDs `ORD-01-T01`, `ORD-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Choose a channel and a table](./01-choose-channel-and-table.md) | Owner, waiter (any role through the API — ORD-D4) | [tester](./test-cases/01-choose-channel-and-table.tester.md) | [developer](./test-cases/01-choose-channel-and-table.developer.md) |
| 02 | [Send the first KOT (create the order)](./02-send-first-kot.md) | same | [tester](./test-cases/02-send-first-kot.tester.md) | [developer](./test-cases/02-send-first-kot.developer.md) |
| 03 | [Add a KOT to a running order](./03-add-kot.md) | same | [tester](./test-cases/03-add-kot.tester.md) | [developer](./test-cases/03-add-kot.developer.md) |
| 04 | [View a running order](./04-view-running-order.md) | same | [tester](./test-cases/04-view-running-order.tester.md) | [developer](./test-cases/04-view-running-order.developer.md) |

## Key concepts

- **Order channel** — the restaurant's `order_channels` rows: `Dine In`, `Takeaway`, `Room Service`, `Delivery` (all active — RST-D1). IDs are GUIDs.
- **Order** — `orders` row. `orderStatus` 0 = in progress (KOTs can be added), 1 = bill generated, 2 = fulfilled. `orderNumber` = `<restaurantCode>/<n>`.
- **KOT** — `kots` row plus its `order_items`. `kotNumber` is a plain number. The kitchen sees it (KITCH).
- **Order line** — `order_items`: the item ID, and the **name and price as sent by the app** (ORD-D1), quantity, total, and the tax % and tax amount taken from the order.
- **Table status** — 0 available, 1 occupied (set by "occupy" in the app), 2 ordered (meant to be set when an order is created, but never matches today — ORD-D2).

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api-dev_guid/specs/orders/` (created with the first plan)

## Depends on

- **RST** — `showDetails` (zones, tables with their running `orderId`), the tax rate.
- **TBL** — tables, and the occupy / leave status (`tableSave`).
- **MENU** — the ordering menu (`getMenuItems`).
- **ROOM** — rooms for Room Service (the room list is empty today — ROOM-D1).
- **KITCH / WAIT** — what happens to a KOT after it's sent. **DLV** — the Delivery channel's own board.
- **BILL** — generating the bill and payment (closes the order to new KOTs).

## Decisions

- **ORD-D1 (2026-11-11):** The server trusts the app's item lines:
  - `menuItemName` and `price` come from the request and are never looked up;
  - any price ≥ 0 and any quantity ≥ 1 is accepted;
  - the `menuItemId` isn't checked, so an inactive item, another restaurant's item or a made-up ID can be ordered;
  - the order's subtotal is the sum the app sent.

  Accepted risk.
- **ORD-D2 (2026-11-11):** The GUID faults in ordering stay:
  - the app sends `tableId` / `roomId` as `Number(<hex>)`, which becomes `null` in JSON, so dine-in and room-service orders are saved with no table or room;
  - the server's table / room status update compares the raw hex with the binary column, so it never matches anyway;
  - so a table never shows its running order, and opening it again starts a **new** order;
  - opening an existing order fails: the app requests `/orders/NaN` (`Number(orderId)`), and it also expects `data.order`, which the server never returns;
  - KOTs added in the same cart visit still go to the right order (the cart keeps the hex order ID).

  Accepted.
- **ORD-D3 (2026-11-11):** Numbering and customer fields stay as they are:
  - the next order number is taken from a "last order today" chosen by `orderId` order, which is a random GUID, so numbers can repeat or jump within a day, and the sequence restarts at `<restaurantCode>/1` each day;
  - KOT numbers work the same way, across the restaurant;
  - customer name and phone are never saved, because the server compares the channel GUID with `2`.

  Accepted.
- **ORD-D4 (2026-11-11):**
  - The server ignores the app's `Idempotency-Key`, so a KOT retried after a timeout is created twice.
  - A table can hold several running orders.
  - Any role (chef included) can create orders and add KOTs.
  - Errors return raw exception text: `error` on create, `message` on add / view.

  Accepted risk.
- **Inherited:** AUTH-D2, RST-D1 (all four channels always shown), RST-D3 / TBL-D3 (no role checks), ROOM-D1, AUTH-D5.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- The Delivery board and online orders — DLV. The kitchen / waiter flows — KITCH / WAIT.
- Bill generation, discounts and payment — BILL. Order history — HIST.
- Public QR orders — PUB.
