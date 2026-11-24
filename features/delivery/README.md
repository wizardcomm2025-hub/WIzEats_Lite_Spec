# Delivery board

The `DELIVERY` tile on Home (shown while the Delivery channel is switched on — RST) opens **Delivery Orders**: the restaurant's home-delivery orders, mostly placed online by customers. Staff accept or reject each new order, send it `Out For Delivery`, and finally tap `Complete Payment`. Each step is meant to send the customer a WhatsApp message.

The spec records today's behaviour plus targets (re-synced 2026-11-24 against API commit d54d6c8 and `POS_APP-kot`). The app now asks for the real Delivery channel, but the server's list query still fails, so **the board never lists an order, and the status endpoint never updates the KOT it is given (DLV-D1, DLV-D2)**. DLV-D1 to DLV-D3 are *To fix*; staff still can't create a delivery order in the app (DLV-D4, accepted).

**Feature code:** `DLV` (test IDs `DLV-01-T01`, `DLV-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Open the delivery board](./01-open-delivery-board.md) | Owner, waiter (today any role through the API) | [tester](./test-cases/01-open-delivery-board.tester.md) | [developer](./test-cases/01-open-delivery-board.developer.md) |
| 02 | [Move a delivery order on](./02-move-delivery-order.md) | same (DLV-D2) | [tester](./test-cases/02-move-delivery-order.tester.md) | [developer](./test-cases/02-move-delivery-order.developer.md) |

## Key concepts

- **Delivery channel** — the restaurant's `order_channels` row with `channelCode` `DELIVERY` (created at onboarding). Its ID is a GUID.
- **Delivery order** — an `orders` row on that channel, with customer name, phone and address. The app has no way to create one (the tile opens only the board — DLV-D4); online orders come from the public QR / web flow (PUB).
- **Board stage** — the stage of the order's first KOT (`kots.status`), shown with board labels: 0 `Pending`, 1 `Accepted`, 2 `Rejected`, 3 `Completed`, 4 `Out for Delivery`.
- **Board actions** — `Accept` → 1, `Reject` → 2, `Out For Delivery` → 4, `Complete Payment` → 3.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api/api/specs/delivery/` (created with the first plan)

## Depends on

- **ONB** — creates the Delivery channel. **ORD** — Home channels and the cart's channel ID.
- **PUB** — where delivery orders would come from.
- **KITCH** — the same `kots.status` values.
- **BILL** — real payment; today the board's `Complete Payment` doesn't record one; target it opens Bill Payment (DLV-D3).

## Decisions

Statuses per [conventions.md → Decision status](../../conventions.md#decision-status), set 2026-11-24. Cross-cutting items: [project-overview.md → Known gaps and targets](../../project-overview.md#known-gaps-and-targets).

| ID | Status | Short | Tests |
|---|---|---|---|
| DLV-D1 | **To fix** (app part fixed) | Board never lists an order | 01-D01–D04, 01-D09, 01-D10; 01-T01–T04 |
| DLV-D2 | **To fix** (with X-D1, X-D4, X-D5) | Status endpoint casts the ID to an integer; no rules | 01-D05, 02-D01–D08, 02-D10, 02-D12 |
| DLV-D3 | **To fix** | WhatsApp to a fixed number; `Complete Payment` records nothing | 02-D13, 02-D14; 02-T02, 02-T03 |
| DLV-D4 | Accepted (new 2026-11-24) | No way to create a delivery order in the app | 01-T06 |
| AUTH-D2 (inherited) | **To fix** (X-D2) | Switched-off / unapproved callers keep access | 01-D06, 02-D07 |

- **DLV-D1 — To fix (2026-11-24, was accepted 2026-11-11); app part fixed.** *Fixed in the app:* the board takes the Delivery channel's hex ID from Home's channel list (fallback `3` only when that list has no `DELIVERY` channel) and keeps IDs as strings. *Today:* the server's query still filters on `orders.orderChannel`, which isn't in the schema, so it answers 200 with a wrapped `Failed to retrieve delivery orders` and the board is empty (no toast). *Target:* the path must be the restaurant's `DELIVERY` channel (hex, dashes allowed) → else 404 `{ "status": false, "message": "Channel not found", "data": null }`; the reply is `{ "status": true, "message": "Delivery orders retrieved successfully", "data": [ … ] }` — one row per order (its first KOT's stage), hex IDs, newest first by `created_at`, orders whose KOT is Completed left out as today; failures 500 `Something went wrong` with no exception text.
- **DLV-D2 — To fix (2026-11-24, was accepted 2026-11-11).** *Target:* `update-status/{kotId}` finds the KOT by hex ID (dashes allowed) among the caller's restaurant's KOTs **on a Delivery-channel order** → else 404 `{ "status": false, "message": "KOT not found", "data": null }`; owner and waiter only (X-D1: 403 `You do not have permission to perform this action`); `status` 0–4 → else 422 `The selected status is invalid.`; only Pending → Accepted (1), Pending → Rejected (2), Accepted → Out for Delivery (4), Out for Delivery → Completed (3) → else 409 `Invalid status change`; never SQL or exception text (X-D5).
- **DLV-D3 — To fix (2026-11-24, was accepted 2026-11-11).** *Target:* the WhatsApp text for 1 / 3 / 4 goes to the order's `customerPhone` (no phone → not sent, logged), not to the number fixed in the code. `Complete Payment` opens the normal Bill Payment page for the order (BILL); when the bill is fully paid the server sets the order's KOT to Completed (3), which removes it from the board and sends the completed message.
- **DLV-D4 — Accepted (new 2026-11-24).** The app has no way to start a delivery order; the board only handles orders placed online (PUB). Taking phone orders for delivery is out of scope for now.
- **Watch:** a delivery KOT also appears in the kitchen (KITCH). The kitchen's path (Pending → Preparing → Completed → Ready) and the board's (Pending → Accepted → Out for Delivery → Completed) use the same `kots.status`, so a chef's Completed (3) removes the order from the board. Not decided yet — raise it if it shows up in testing.
- **Inherited:** RST-D1 fixed (the tile follows the Delivery switch); ORD-D3 (To fix — customer fields; delivery orders from the app don't exist anyway, DLV-D4); AUTH-D2 (To fix, X-D2); AUTH-D5 / X-D5.

Original decision texts (history):

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
- **Inherited (2026-11-11):** RST-D1 (the Delivery tile always shows), ORD-D3 (customer fields are never saved by the app's create path), AUTH-D2, AUTH-D5.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Placing online orders — PUB. Bills and payments — BILL.
- The KITCH `chef/update-status` endpoint (same `kots.status`, different route).
