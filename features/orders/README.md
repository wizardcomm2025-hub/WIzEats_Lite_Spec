# Take an order & send KOTs

Staff pick a channel on Home: Dine In, Takeaway, Room Service or Delivery. For Dine In they choose a table, then add menu items to the cart and send them to the kitchen as a **KOT** (kitchen order ticket). The first KOT creates the **order**; later KOTs, sent while the order is still in progress, add to it. Billing and payment follow (BILL).

The spec records today's behaviour plus targets (re-synced 2026-11-24 against API commit d54d6c8 and `POS_APP-kot`). Orders are now linked to their table or room, the table turns `Ordered`, and tapping it re-opens the running order (ORD-D2 fixed). ORD-D1, ORD-D3 and ORD-D4 are *To fix*.

**Feature code:** `ORD` (test IDs `ORD-01-T01`, `ORD-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Choose a channel and a table](./01-choose-channel-and-table.md) | Owner, waiter (today any role through the API — X-D1) | [tester](./test-cases/01-choose-channel-and-table.tester.md) | [developer](./test-cases/01-choose-channel-and-table.developer.md) |
| 02 | [Send the first KOT (create the order)](./02-send-first-kot.md) | same | [tester](./test-cases/02-send-first-kot.tester.md) | [developer](./test-cases/02-send-first-kot.developer.md) |
| 03 | [Add a KOT to a running order](./03-add-kot.md) | same | [tester](./test-cases/03-add-kot.tester.md) | [developer](./test-cases/03-add-kot.developer.md) |
| 04 | [View a running order](./04-view-running-order.md) | same | [tester](./test-cases/04-view-running-order.tester.md) | [developer](./test-cases/04-view-running-order.developer.md) |

## Key concepts

- **Order channel** — the restaurant's `order_channels` rows: `Dine In`, `Takeaway`, `Room Service`, `Delivery`. Only active channels are listed, and the settings switch Takeaway / Delivery / Room Service on and off (RST-D1 fixed). IDs are GUIDs; the code is `DINE_IN`, `TAKEAWAY`, `ROOM_SERVICE`, `DELIVERY`.
- **Order** — `orders` row. `orderStatus` 0 = in progress (KOTs can be added), 1 = bill generated, 2 = fulfilled. `orderNumber` = `<restaurantCode>/<n>`.
- **KOT** — `kots` row plus its `order_items`. `kotNumber` is a plain number. The kitchen sees it (KITCH).
- **Order line** — `order_items`: the item ID, and the **name and price as sent by the app** today (target: taken from the menu — ORD-D1), quantity, total, and the tax % and tax amount taken from the order.
- **Table status** — 0 available, 1 occupied (set by "occupy" in the app), 2 ordered (set when an order is created on the table — since 2026-11-16). Rooms work the same way.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api/api/specs/orders/` (created with the first plan)

## Depends on

- **RST** — `showDetails` (zones, tables with their running `orderId`), the tax rate.
- **TBL** — tables, and the occupy / leave status (`tableSave`).
- **MENU** — the ordering menu (`getMenuItems`).
- **ROOM** — rooms for Room Service (the room grid and its occupy / leave — ROOM-D1 fixed 2026-11-16).
- **KITCH / WAIT** — what happens to a KOT after it's sent. **DLV** — the Delivery channel's own board.
- **BILL** — generating the bill and payment (closes the order to new KOTs).

## Decisions

Statuses per [conventions.md → Decision status](../../conventions.md#decision-status), set 2026-11-24. Cross-cutting items: [project-overview.md → Known gaps and targets](../../project-overview.md#known-gaps-and-targets).

| ID | Status | Short | Tests |
|---|---|---|---|
| ORD-D1 | **To fix** (with X-D8) | Server trusts the app's item lines and IDs | 02-D05, 02-D06, 02-D10, 02-D11, 02-D21 |
| ORD-D2 | **Fixed in code 2026-11-16 and in the app** | Orders not linked to tables / rooms; running order can't be re-opened | 02-D02, 02-D03; 01-T06, 01-T08, 02-T02, 02-T10, 03-T04, 04-T01, 04-T02 |
| ORD-D3 | **To fix** (numbering changed 2026-11-16) | Duplicate numbers possible; customer name / phone never captured | 02-D04, 02-D15, 03-D09 (context: 02-D22, 02-T06); 02-T05 |
| ORD-D4 | **To fix** (with X-D1, X-D4, X-D5) | Idempotency-Key ignored; several running orders per table; any role; raw errors | 01-D08, 02-D07, 02-D09, 02-D12, 02-D13, 02-D18, 03-D03–D05, 03-D08, 03-D11, 04-D04, 04-D05; 02-T07, 02-T08 |
| AUTH-D2 (inherited) | **To fix** (X-D2) | Switched-off / unapproved callers keep access | 01-D06, 02-D17, 03-D08 |

- **ORD-D2 — Fixed in code 2026-11-16 and in the app.** The app sends the table's / room's hex ID (it used `Number(hex)` → null); the server sets `tables.status` / `rooms.status` to 2 with the ID converted (it compared raw hex). So `showDetails` gives the table its running `orderId`, the grid shows it `Ordered`, and tapping it opens the cart with that order; the app requests `/orders/<hex>` and reads the server's flat reply. Room Service orders work the same way through the room grid.
- **ORD-D1 — To fix (2026-11-24, was accepted 2026-11-11).** *Target:* each line's `menuItemId` must be an active, not-deleted item of the restaurant in an active, not-deleted category → else 422 `{ "status": false, "message": "Menu item not available", "data": null }`; the line's name and price are taken from the menu (sent `menuItemName` / `price` are ignored), so the subtotal is the menu's. `orderChannelId` must be an active channel of the restaurant → else 422 `Invalid channel`; `tableId` / `roomId` an active, not-deleted table / room of the restaurant → else 422 `Invalid table` / `Invalid room` (X-D8). Malformed IDs give the same 422s (no 500).
- **ORD-D3 — To fix (2026-11-24, was accepted 2026-11-11).** *Changed 2026-11-16:* the last order / KOT of the day is now picked by `created_at` (it was by the random GUID), so numbers count up in normal use. *Target:* (a) numbers are allocated under a lock, so order numbers are unique per restaurant per day and KOT numbers unique per restaurant per day, even for parallel requests; (b) customer capture works by channel code: the app shows `customerName` / `customerPhone` for Takeaway and Delivery (today it checks for channel IDs `0`–`2`, which never match); Delivery requires both → 422 `Customer name is required for delivery orders` / `Customer phone is required for delivery orders`; the server saves them for any channel when sent (today it compares the GUID with `2`).
- **ORD-D4 — To fix (2026-11-24, was accepted 2026-11-11).** *Target:* (a) `Idempotency-Key` honoured on `createOrder` and `kots/{orderId}` for 24 hours per user: the same key with the same body returns the first answer again and writes nothing; the same key with a different body → 422 `Idempotency-Key reused with a different request`; (b) `createOrder` on a table or room that already has a running order (status 0 or 1) → 409 `{ "status": false, "message": "This table already has a running order", "data": { "orderId": "<hex>" } }` (`This room …` for rooms); (c) X-D1: all four endpoints owner and waiter only (chef 403); (d) X-D4 / X-D5: an unknown, other restaurant's or malformed order ID → 404 `Order not found`; unexpected errors → 500 `{ "status": false, "message": "Something went wrong", "data": null }` with no `error` key or exception text; a missing `kotNote` counts as no note.
- **Inherited:** AUTH-D2 (To fix, X-D2); RST-D1 and ROOM-D1 fixed; TBL-D3 / ROOM-D2 (roles, To fix).

Original decision texts (history):

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
- **Inherited (2026-11-11):** AUTH-D2, RST-D1 (all four channels always shown), RST-D3 / TBL-D3 (no role checks), ROOM-D1, AUTH-D5.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- The Delivery board and online orders — DLV. The kitchen / waiter flows — KITCH / WAIT.
- Bill generation, discounts and payment — BILL. Order history — HIST.
- Public QR orders — PUB.
