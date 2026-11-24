# Public QR ordering

Customers who scan a restaurant's QR code are meant to see its menu and order without signing in. The server has two open endpoints for this: `POST /api/public/menu` and `POST /api/public/order`. **Neither repository has a customer page**: the POS app's QR codes are UPI payment codes, not menu links. So this feature has developer cases only; there are no tester cases.

The spec records today's behaviour plus targets (re-synced 2026-11-24 against API commit d54d6c8; no public code changed). **Today the menu answers with empty data for a normal (hex) restaurant ID, and no public order can be created or added to: every attempt ends in a 500 (PUB-D1).** All three decisions are now *To fix*: the target is a working, checked public API — it is the only source of Delivery orders (DLV-D4). The customer page itself stays out of scope.

**Feature code:** `PUB` (test IDs `PUB-01-D01`, `PUB-03-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Load the public menu](./01-load-public-menu.md) | Anyone (no sign-in) | — (no UI) | [developer](./test-cases/01-load-public-menu.developer.md) |
| 02 | [Place a public order](./02-place-public-order.md) | Anyone | — | [developer](./test-cases/02-place-public-order.developer.md) |
| 03 | [Add to a public order](./03-add-to-public-order.md) | Anyone | — | [developer](./test-cases/03-add-to-public-order.developer.md) |

## Key concepts

- **Public endpoint** — a route outside `auth:sanctum` and `restaurant.tenant`. The restaurant is whatever `restaurantId` the caller sends. No token is read, and a token that is sent is ignored.
- **Hex ID vs raw ID** — every ID column is `binary(16)`. Today the public code passes the request's `restaurantId` / `orderId` straight into queries without `hex2bin`, so a 32-character hex ID matches nothing and only the 16 raw bytes match (form-encoded only). Target: hex IDs (dashes allowed), like every other endpoint.
- **Create vs add** — `public/order` without `orderId` creates a new order and its first KOT (use case 02). With `orderId` it adds a KOT to that order (use case 03). Today `orderId` is not a validated field (the controller only checks `isset`); target: 32-char hex.
- **Public order** — target: an order with `createdBy` `customer`, on the channel the caller picks by `channelCode`, numbered like staff orders (`<restaurantCode>/<n>`), visible to staff like any other order (Home grid, kitchen, Live, and for Delivery the board).

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api/api/specs/public-order/` (created with the first plan)

## Depends on

- **ONB**: creates the four order channels (`DINE_IN`, `TAKEAWAY`, `ROOM_SERVICE`, `DELIVERY`) the menu lists.
- **MENU**: the items and categories the menu returns.
- **ORD**: the same `orders` / `kots` / `order_items` tables and the number formats.
- **DLV**: the board that would show public delivery orders.
- **TBL / ROOM**: the table and room statuses a created order would set.

## Decisions

Statuses per [conventions.md → Decision status](../../conventions.md#decision-status), set 2026-11-24. Cross-cutting items: [project-overview.md → Known gaps and targets](../../project-overview.md#known-gaps-and-targets).

| ID | Status | Short | Tests |
|---|---|---|---|
| PUB-D1 | **To fix** | Public menu and order don't work | 01-D01–D03, 02-D01–D03, 02-D12–D14, 03-D01–D05, 03-D07 |
| PUB-D2 | **To fix** (X-D10) | Caller trusted; any restaurant; inactive items served | 01-D03–D05, 02-D05–D07, 02-D11, 02-D15 |
| PUB-D3 | **To fix** (with X-D5) | No rate limit or idempotency; exception text with customer data | 01-D09, 02-D04, 02-D08, 02-D10, 03-D10, 03-D11 |

- **PUB-D1 — To fix (2026-11-24, was accepted 2026-11-12).** *Target:* both endpoints take hex IDs (dashes allowed). **Menu:** the restaurant's name, tax %, active channel codes and items (see PUB-D2). **Create** (no `orderId`): the caller picks the channel by `channelCode` — an active channel of the restaurant, else 422 `Invalid channel`; `DINE_IN` needs `tableId` (`Invalid table`), `ROOM_SERVICE` needs `roomId` (`Invalid room`), `DELIVERY` needs `customerName`, `customerPhone` and `customerAddress` (`Customer name is required for delivery orders`, …); the server generates every ID, numbers the order `<restaurantCode>/<n>` and the KOT with the next number, as ORD does (under a lock — ORD-D3 target), sets `orderStatus` 0, `createdBy` `customer`, the tax % from the restaurant, and the table / room `status` 2; a table / room that already has a running order → 409 as ORD-D4. Answer 201 `{ "status": true, "message": "Order created successfully", "data": { orderId, orderNumber, kotId, kotNumber, subTotal } }`. **Add** (`orderId`): an in-progress order of that restaurant created publicly (`createdBy` `customer`) → else 404 `Order not found`; billed → 409 `Bill already generated, cannot add items to this order`; adds a KOT like ORD 03 and answers 201 `KOT added successfully`. The `orderId` returned at create is the customer's handle for adding (32 hex characters, not guessable).
- **PUB-D2 — To fix (2026-11-24, was accepted 2026-11-12); replaces X-D10 (Accepted).** *Target:* only an approved (`isApproved` 1), active (`status` 1) restaurant is served → else 404 `{ "status": false, "message": "Restaurant not found", "data": null }`; the menu lists only active, not-deleted items in active, not-deleted categories; an order line's item must be such an item → else 422 `Menu item not available`, and its name and price come from the menu (sent values ignored); `tableId` / `roomId` must be an active table / room of the restaurant; `tableId` / `roomId` on the menu request stay ignored.
- **PUB-D3 — To fix (2026-11-24, was accepted 2026-11-12).** *Target:* per-IP rate limits — menu 60 / min, order 10 / min → 429 `Too Many Attempts.`; `Idempotency-Key` honoured on `public/order` like ORD-D4; no exception text in any reply (500 `{ "status": false, "message": "Something went wrong", "data": null }`) — the Google Sheet log keeps the detail; `customerPhone` `^[6-9][0-9]{9}$` when sent (`The customer phone field format is invalid.`), names and address ≤ 255. Captcha / bot checks stay out of scope (accepted).

Original decision texts (history):

- **PUB-D1 (2026-11-12):** The public flow stays as it is, and orders never succeed:
  - **Menu:** a hex `restaurantId` matches nothing, so the menu answers 200 with empty data. Only the raw 16-byte ID returns a menu.
  - **Create:** the controller always uses `orderChannelId` 3 ("Delivery"), whatever the caller sends. The insert names an `orders.orderChannel` column that the GUID schema doesn't have, so every create fails. The reply is 500 `Something went wrong` with `error` `Failed to create order with KOT: SQLSTATE[42S22] … Unknown column 'orderChannel' …`. Even without that error, the `kots` and `order_items` inserts supply no ID and would fail in strict mode (1364). Nothing is written.
  - **`tableId` / `roomId` keys:** the controller reads both keys from the validated data. If either key is left out of the body (rather than sent as `null`), the reply is 500 `Undefined array key "tableId"` (or `"roomId"`).
  - **Add:** the service takes `int $orderId`:
    - a hex ID starting with a letter fails PHP's type check: 500 with the `TypeError` text and the server file path;
    - a hex ID starting with a digit raises the PHP warning `A non-numeric value encountered`: 500;
    - a number such as `0` is compared with the binary `orderId`, and most IDs read as 0. With a raw `restaurantId` it therefore finds some order of that restaurant. The KOT insert then fails: 1364 `Field 'kotId' doesn't have a default value`, or `Undefined array key "kotNote"` when `kotNote` is left out. With a hex `restaurantId` the reply is `Order not found`.

  Accepted.
- **PUB-D2 (2026-11-12):** The caller is trusted:
  - prices, item names and item IDs come from the body, with no check against the menu, and the totals are computed from them;
  - `tableId`, `roomId` and the items aren't checked against the restaurant;
  - the menu is served for any restaurant ID, including restaurants that are pending, rejected or switched off. It lists every item of the restaurant, including switched-off (`isActive` false) and deleted (`isDelete` 1) items, and items whose category is switched off;
  - the controller passes `paymentStatus` 1, but the insert never writes it;
  - the customer fields are free text with no format check;
  - `tableId` / `roomId` on the menu request are validated and then ignored.

  Accepted risk. Not reachable for orders today (PUB-D1), but visible in the error text.
- **PUB-D3 (2026-11-12):** The abuse and error behaviour stays:
  - there is no rate limit, captcha, idempotency key or bot check;
  - every 500 from `public/order` carries the raw exception text in `error`. That includes the full SQL insert with the customer's name, phone and address, and for a `TypeError` the server file path;
  - when `GOOGLE_SHEET_LOG_URL` is set, each failure is also sent to the Google Sheet error log with the same message. A failed create sends two rows, one from the service and one from the controller. The SQL text contains `now()` timestamps, so the 5-minute throttle almost never merges calls;
  - the menu's own 500 is generic (`Failed to retrieve public menu items`).

  Accepted risk.
- **Inherited (2026-11-12):** ORD-D1 (client-supplied lines, the same pattern), MENU (items can't be created through the app, so the fixtures use the API or SQL), AUTH-D5 (exception text in replies). *2026-11-24:* MENU items can now be created in the app (MENU-D1 / D2 fixed).

## Cross-cutting open questions

- None open. (Whether a customer page will be built is a product question. It is not needed for these specs.)

## Feature-wide out of scope

- Payment by the customer; there is no public payment endpoint.
- Showing public orders to staff (DLV, KITCH, HIST).
- The UPI QR code on the bill (BILL).
