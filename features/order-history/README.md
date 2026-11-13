# Order history

**Order History** has two tabs. **Live** lists orders still running (in progress or billed, not paid), each with a `Bill Payment` button. **History** lists paid orders with their payments, page by page, and can reprint a saved bill. Both share a filter sheet (channel, payment mode, date range) and a search box.

The spec records today's behaviour. Decisions HIST-D1 to HIST-D3 keep the current rules as accepted risks. **Today the Live tab works for orders created today (UTC day). The History tab stays empty in practice, because the app can't take payment (BILL-D2), and any channel filter answers with an error (HIST-D1).**

**Feature code:** `HIST` (test IDs `HIST-01-T01`, `HIST-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [See live orders](./01-see-live-orders.md) | Owner, waiter (any role through the API — HIST-D3) | [tester](./test-cases/01-see-live-orders.tester.md) | [developer](./test-cases/01-see-live-orders.developer.md) |
| 02 | [See past orders](./02-see-past-orders.md) | same | [tester](./test-cases/02-see-past-orders.tester.md) | [developer](./test-cases/02-see-past-orders.developer.md) |
| 03 | [Filter and search](./03-filter-and-search.md) | same | [tester](./test-cases/03-filter-and-search.tester.md) | [developer](./test-cases/03-filter-and-search.developer.md) |

## Key concepts

- **Live order** — `orderStatus` 0 (in progress) or 1 (bill generated).
- **Past order** — `orderStatus` 2 (fulfilled / paid).
- **Filters** — date range (default `Today`; presets 7 / 15 / 30 days; custom), one order channel, one payment mode (`CASH` / `UPI`). All are sent on every request.
- **Search** — text with `#`, `₹` and `,` stripped; matched on the server.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api-dev_guid/specs/order-history/` (created with the first plan)

## Depends on

- **ORD** — the orders and lines. **BILL** — bills and payments; the Live `Bill Payment` button (never opens — BILL-D2).
- **ORD 01** — the channel list used by the filter sheet.
- **PRN** — `Print Bill No. <n>` on a past order.

## Decisions

- **HIST-D1 (2026-11-11):** The channel filter always fails.
  - The app sends a channel code: `Dine In` 0, `Takeaway` 1, `Room Service` 2, `Delivery` 3, or the channel's own ID for any other name.
  - The server's rule for `orderChannelId` is `integer` + `exists` against the binary channel IDs:
    - codes 1, 2 and 3 → 422 `The selected order channel id is invalid.`;
    - code 0 passes, because MySQL compares the binary IDs as numbers and most read as 0. The service then runs `hex2bin("0")`, which fails: 500 `Failed to retrieve order history` (History) or `Failed to retrieve live order history` (Live);
    - a real hex channel ID → 422 `The order channel id field must be an integer.`
  - The app shows the error as a toast and keeps an empty list until the filter is reset.

  Accepted.
- **HIST-D2 (2026-11-11):** The two tabs behave differently.
  - **Live:**
    - ignores `search`, `paymentType`, `page` and `perPage`, and returns every matching running order in one reply;
    - its date filter uses UTC days on the creation time, so `Today` covers orders created 05:30 IST today to 05:29 IST tomorrow;
    - an unpaid order from an earlier day doesn't appear under `Today`;
    - rows carry no time and no bill, so the card shows `Sub Total` even for a billed order (with the `Bill generated` tag).
  - **History:**
    - lists only paid orders, which the app can't create today (BILL-D2), so it is empty in practice;
    - its date filter treats the dates as IST.

  Accepted.
- **HIST-D3 (2026-11-11):**
  - Both lists are sorted by `orderId`, a random GUID, not by time, so the order of rows and pages is arbitrary.
  - Search matches any part of the order number (`LIKE %text%`), or an exact grand total when the text is numeric.
  - Payment details are rebuilt from `type:received:change` text joined with `|`, so a `paymentType` containing `:` or `|` (possible — BILL-D1) garbles them. MySQL's default `GROUP_CONCAT` length of 1024 can cut off long lists.
  - Any role, chef included, can read both lists through the API.
  - Each History row runs its own bill and KOT queries (up to 100 rows a page).

  Accepted risk.
- **Inherited:** BILL-D2 (the Live `Bill Payment` button never opens the bill), ORD-D3 (no customer fields), AUTH-D2, AUTH-D5.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Billing and payment — BILL. Printing — PRN. Sales reports — RPT.
