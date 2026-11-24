# Bill & payment

When the guests are done, staff open the order's bill, optionally add a discount, and either print the bill first (it then gets its bill number and the order takes no more KOTs) or take payment straight away in cash, UPI or a split of both. Payment closes the order and frees its table or room.

The spec records today's behaviour plus targets (re-synced 2026-11-24 against API commit d54d6c8 and `POS_APP-kot`; no billing server code changed). Since the app and ORD fixes the bill pages open from the cart and from Order History, and paying frees the order's table or room (BILL-D2 fixed in the app). **Every money figure is still taken from the client (BILL-D1)** — BILL-D1 and BILL-D3 are *To fix*.

**Feature code:** `BILL` (test IDs `BILL-01-T01`, `BILL-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Generate the bill](./01-generate-bill.md) | Owner, waiter (today any role through the API — BILL-D3) | [tester](./test-cases/01-generate-bill.tester.md) | [developer](./test-cases/01-generate-bill.developer.md) |
| 02 | [Take payment](./02-take-payment.md) | same | [tester](./test-cases/02-take-payment.tester.md) | [developer](./test-cases/02-take-payment.developer.md) |
| 03 | [Get a saved bill](./03-get-saved-bill.md) | same | — (no UI path) | [developer](./test-cases/03-get-saved-bill.developer.md) |

## Key concepts

- **Bill** — `bills` row, one per order: the totals as sent, `billNumber`, `financialYear` (`2026-27` style, April–March), `billStatus` 0 unpaid / 1 paid, plus `bill_items` — the order lines grouped by item ID, name, price and tax %.
- **Order status** — 0 in progress → 1 bill generated (no more KOTs, ORD) → 2 fulfilled (paid). `paymentStatus` 1 once paid.
- **Payment** — `payments` row: `transactionId` `TXN-<12 random>`, `paymentType` (any text today; the app sends `CASH` / `UPI`; target only those — BILL-D1), `receivedAmount`, `changeAmount`, `paymentStatus` `COMPLETED`; target also the leg's `amount`.
- **Pay-and-bill** — paying an order still at status 0 generates its bill in the same request, from totals sent with the payment (target: from the discount sent; the server works out the totals).
- **Customer display** — on terminals with a small customer-facing display, Bill Payment shows the amount to pay while the bill is open and the change once it is paid; blank when the page closes (new in the app). Devices without one are unaffected.
- **Bill rules (app, and server target)** — the discount comes off the subtotal first (capped at the subtotal), tax is charged on what is left at the order's tax %, and the total is rounded to the nearest rupee (.50 rounds up); the difference is the round-off.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api/api/specs/billing/` (created with the first plan)

## Depends on

- **ORD** — the order and its lines; `POST /api/orders/{id}` returns the bill snapshot once billed (ORD 04).
- **TBL / ROOM** — table and room status, freed on payment.
- **RST** — tax rate, discount switch, UPI QR on the bill.
- **HIST** — the Live list's bill button. **PRN** — printing the bill. **DLV** — a delivery order's `Complete Payment` opens Bill Payment (target DLV-D3).

## Decisions

Statuses per [conventions.md → Decision status](../../conventions.md#decision-status), set 2026-11-24. Cross-cutting items: [project-overview.md → Known gaps and targets](../../project-overview.md#known-gaps-and-targets).

| ID | Status | Short | Tests |
|---|---|---|---|
| BILL-D1 | **To fix** | Money figures and payments taken from the client | 01-D04–D06, 01-D14, 01-D15, 02-D03, 02-D05, 02-D15, 02-D18, 02-D19 |
| BILL-D2 | **Fixed in the app** (with ORD-D2; server leftovers moved to BILL-D3) | App couldn't open the bill pages; tables never freed | 01-T01–T03, 02-T01, 02-T02, 02-D10 |
| BILL-D3 | **To fix** (with X-D1, X-D4, X-D5) | Any role; Idempotency-Key ignored; numbering; error codes and text | 01-D02, 01-D03, 01-D07, 01-D08, 01-D12, 02-D06, 02-D07, 02-D09, 02-D11, 02-D12, 03-D03, 03-D04 |
| AUTH-D2 (inherited) | **To fix** (X-D2) | Switched-off / unapproved callers keep access | 01-D11, 02-D14 |

- **BILL-D2 — Fixed in the app** (seen 2026-11-24), together with the ORD-D2 fix. The cart's `View Bill` and the takeaway `Collect Payment` open Bill Breakdown / Bill Payment with the hex order ID; Order History → Live's bill button loads the order and opens Bill Payment; app orders carry their table / room, so payment frees it. The server leftovers (`payments/bill/{id}` bare 500, `create2` refusing the dashed ID) are part of BILL-D3.
- **BILL-D1 — To fix (2026-11-24, was accepted 2026-11-11).** *Target, bill:* the client sends only the discount — `isDiscountPercent` with `discountPercent` (0–100) or `discountAmount` (≥ 0); any other total sent is ignored. The server works the bill out with the app's rules: `subTotal` = Σ of the order lines; `discountAmount` = the percent of the subtotal or the amount, capped at the subtotal → a larger amount → 422 `The discount can't be more than the subtotal`; tax on (subtotal − discount) at the order's tax %; `grandTotal` rounded to the nearest rupee (.50 up) with the difference as `roundOff`. A non-zero discount while the restaurant's discount switch is off → 422 `Discounts are turned off for this restaurant`. The same applies to pay-and-bill. *Target, payments:* each leg `paymentType` `CASH` or `UPI` → else 422 `The selected payments.0.paymentType is invalid.`; `amount` > 0 (0 only for a ₹0 bill), stored; a CASH leg's `receivedAmount` ≥ its `amount` and `changeAmount` = the difference; the legs' amounts must add up to the bill's `grandTotal` → else 422 `Payment total does not match the bill`; the same on `payments/create` with its single `amount`. Lines without a name can no longer occur once ORD-D1 is fixed (names from the menu).
- **BILL-D3 — To fix (2026-11-24, was accepted 2026-11-11).** *Target:* owner and waiter only (X-D1: 403 `You do not have permission to perform this action`); `Idempotency-Key` honoured on `generateBill`, `create2` and `create` like ORD-D4 (same key + same body → the first answer again); bill numbers unique per restaurant per financial year (unique key, and a lock that doesn't depend on the config row); proper codes: unknown, other restaurant's or malformed (including dashed-form handling — accepted on every route) order ID → 404 `Order not found`; no bill → 404 `Bill not found`; already billed → 409 `Bill already generated for this order`; already paid → 409 `Order is already paid`; missing payment details → 422 `Payment details are required`; unexpected → 500 `Something went wrong` / `Failed to create payment` with no exception text (X-D5). Voiding, editing and refunding a bill stay out of scope (accepted).
- **Also (DLV-D3 target):** when a Delivery-channel order is paid in full, its KOT is set to Completed (3) and the completed WhatsApp text is sent.
- **Inherited:** ORD-D1 (To fix — names and prices from the menu); ORD-D2 fixed; ORD-D3 (To fix — customer fields, which the bill copies); AUTH-D2 (To fix, X-D2); AUTH-D5 / X-D5.

Original decision texts (history):

- **BILL-D1 (2026-11-11):** Money figures come from the client:
  - `generateBill` stores the `subTotal`, `taxPercent`, `taxAmount`, `isDiscountPercent`, `discountPercent`, `discountAmount`, `roundOff` and `grandTotal` it is sent, and copies them onto the order (the order's own `subTotal` is overwritten). Nothing is checked against the order lines or the restaurant's tax rate;
  - payments aren't checked against the bill: any one payment of ≥ 0.01 (old payload) or any `payments` list (new payload) marks the order paid. `paymentType` is any text. In the new payload `payments.*.amount` is required but never stored; only `receivedAmount` / `changeAmount` are;
  - an order line saved with no `menuItemName` (ORD-D1 allows it) can never be billed: `bill_items.menuItemName` is NOT NULL, so every attempt answers `Failed to generate bill`.

  Accepted risk.
- **BILL-D2 (2026-11-11):** The app can't bill or take payment:
  - the cart's `Collect Payment` (takeaway KOT sheet) uses `Number(<hex orderId>)`, which is `NaN`, and does nothing;
  - Order History's bill button first loads the order with `getAllOrders`, which never succeeds (ORD-D2), so the bill page never opens;
  - on the server, payment frees the table / room only when the order has one; app orders have none (ORD-D2), so their table stays Occupied;
  - `POST /api/payments/bill/{id}` converts the ID outside its error handling, so a bad ID gives a bare 500 `Server Error`; `payments/create2` doesn't accept the dashed UUID form.

  Accepted.
- **BILL-D3 (2026-11-11):**
  - Any role, chef included, can generate bills and take payments.
  - `Idempotency-Key` is ignored. A repeated payment is refused (`Order is already paid`) and a repeated bill (`Bill already generated for this order`), because the order row is locked and its status checked.
  - A bill can't be voided, edited or refunded; there is no endpoint.
  - Bill numbers are `max + 1` per restaurant per financial year, with no unique key; the lock that serialises them is on the restaurant's `restaurant_configs` row, so a restaurant without that row can get duplicate numbers.
  - Error replies carry exception text other than database errors (e.g. `Undefined array key "subTotal"`, `hex2bin()` messages).

  Accepted risk.
- **Inherited (2026-11-11):** ORD-D1 (line prices from the client), ORD-D2, ORD-D3 (customer fields never saved, so bills have none), AUTH-D2, AUTH-D5.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Order history lists and filters — HIST. Reports — RPT. Printing and sharing — PRN.
- The delivery board itself — DLV (its `Complete Payment` opens Bill Payment in the target).
