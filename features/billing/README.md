# Bill & payment

When the guests are done, staff open the order's bill, optionally add a discount, and either print the bill first (it then gets its bill number and the order takes no more KOTs) or take payment straight away in cash, UPI or a split of both. Payment closes the order and frees its table or room.

The spec records today's behaviour. Decisions BILL-D1 to BILL-D3 keep the current rules as accepted risks. **Today the app can't open the bill page at all (BILL-D2); through the API, every money figure is taken from the client (BILL-D1).**

**Feature code:** `BILL` (test IDs `BILL-01-T01`, `BILL-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Generate the bill](./01-generate-bill.md) | Owner, waiter (any role through the API — BILL-D3) | [tester](./test-cases/01-generate-bill.tester.md) | [developer](./test-cases/01-generate-bill.developer.md) |
| 02 | [Take payment](./02-take-payment.md) | same | [tester](./test-cases/02-take-payment.tester.md) | [developer](./test-cases/02-take-payment.developer.md) |
| 03 | [Get a saved bill](./03-get-saved-bill.md) | same | — (no UI path) | [developer](./test-cases/03-get-saved-bill.developer.md) |

## Key concepts

- **Bill** — `bills` row, one per order: the totals as sent, `billNumber`, `financialYear` (`2026-27` style, April–March), `billStatus` 0 unpaid / 1 paid, plus `bill_items` — the order lines grouped by item ID, name, price and tax %.
- **Order status** — 0 in progress → 1 bill generated (no more KOTs, ORD) → 2 fulfilled (paid). `paymentStatus` 1 once paid.
- **Payment** — `payments` row: `transactionId` `TXN-<12 random>`, `paymentType` (any text; the app sends `CASH` / `UPI`), `receivedAmount`, `changeAmount`, `paymentStatus` `COMPLETED`.
- **Pay-and-bill** — paying an order still at status 0 generates its bill in the same request, from totals sent with the payment.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api-dev_guid/specs/billing/` (created with the first plan)

## Depends on

- **ORD** — the order and its lines; `POST /api/orders/{id}` returns the bill snapshot once billed (ORD 04).
- **TBL / ROOM** — table and room status, freed on payment.
- **RST** — tax rate, discount switch, UPI QR on the bill.
- **HIST** — the Live list's bill button. **PRN** — printing the bill.

## Decisions

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
- **Inherited:** ORD-D1 (line prices from the client), ORD-D2, ORD-D3 (customer fields never saved, so bills have none), AUTH-D2, AUTH-D5.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Order history lists and filters — HIST. Reports — RPT. Printing and sharing — PRN.
- The delivery board's `Complete Payment` (sets a KOT status only) — DLV.
