# Order history

**Order History** has two tabs. **Live** lists orders still running (in progress or billed, not paid), each with a `Bill Payment` button. **History** lists paid orders with their payments, page by page, and can reprint a saved bill. Both share a filter sheet (channel, payment mode, date range) and a search box.

The spec records today's behaviour plus targets (re-synced 2026-11-24 against API commit d54d6c8 and `POS_APP-kot`). Since 2026-11-18 the channel filter works by channel code (HIST-D1 fixed), History is sorted by time and shows `Paid` / `Partial` / `Unpaid`, and — with payment working in the app (BILL-D2) — History fills with paid orders. HIST-D2 (Live tab) and HIST-D3 are *To fix*.

**Feature code:** `HIST` (test IDs `HIST-01-T01`, `HIST-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [See live orders](./01-see-live-orders.md) | Owner, waiter (today any role through the API — HIST-D3) | [tester](./test-cases/01-see-live-orders.tester.md) | [developer](./test-cases/01-see-live-orders.developer.md) |
| 02 | [See past orders](./02-see-past-orders.md) | same | [tester](./test-cases/02-see-past-orders.tester.md) | [developer](./test-cases/02-see-past-orders.developer.md) |
| 03 | [Filter and search](./03-filter-and-search.md) | same | [tester](./test-cases/03-filter-and-search.tester.md) | [developer](./test-cases/03-filter-and-search.developer.md) |

## Key concepts

- **Live order** — `orderStatus` 0 (in progress) or 1 (bill generated).
- **Past order** — `orderStatus` 2 (fulfilled / paid).
- **Filters** — date range (default `Today`; presets 7 / 15 / 30 days; custom), one order channel (sent as its `channelCode`, e.g. `TAKEAWAY`), one payment mode (`CASH` / `UPI`). All are sent on every request. Target: on Live only channel and search apply (HIST-D2).
- **Search** — text with `#`, `₹` and `,` stripped; matched on the server.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api/api/specs/order-history/` (created with the first plan)

## Depends on

- **ORD** — the orders and lines. **BILL** — bills and payments; the Live `Bill Payment` button opens Bill Payment (BILL-D2 fixed).
- **ORD 01** — the channel list used by the filter sheet.
- **PRN** — `Print Bill No. <n>` on a past order.

## Decisions

Statuses per [conventions.md → Decision status](../../conventions.md#decision-status), set 2026-11-24. Cross-cutting items: [project-overview.md → Known gaps and targets](../../project-overview.md#known-gaps-and-targets).

| ID | Status | Short | Tests |
|---|---|---|---|
| HIST-D1 | **Fixed in code 2026-11-18** (and in the app) | Channel filter always failed | 01-D05, 03-D04; 03-T03, 03-T04 |
| HIST-D2 | **To fix** | Live tab: date-bound (UTC), ignores search, no time | 01-D02, 01-D03, 01-D10; 01-T05, 03-T02 |
| HIST-D3 | **To fix** (sorting fixed 2026-11-18; with X-D1) | Any role; payments rebuilt from joined text | 01-D07, 02-D06, 02-D07, 02-D11 |
| AUTH-D2 (inherited) | **To fix** (X-D2) | Switched-off / unapproved callers keep access | 01-D08, 02-D08 |

- **HIST-D1 — Fixed in code 2026-11-18 and in the app.** The request field is now `channelCode` (`nullable|string|exists:order_channels,channelCode`); both lists filter on `order_channels.channelCode` (upper-cased); the app sends the channel's code (`DINE_IN`, `TAKEAWAY`, `ROOM_SERVICE`, `DELIVERY`). An unknown code → 422 `The selected channel code is invalid.`
- **Also changed 2026-11-18 (History):** sorted newest first by `created_at` (it was the random `orderId`); the total counts distinct orders; `paymentStatus` is worked out from the completed payments — `Paid` when Σ(received − change) ≥ grand total − 0.01, `Partial` when something was paid, else `Unpaid`; IDs are produced as hex in SQL.
- **HIST-D2 — To fix (2026-11-24, was accepted 2026-11-11).** *Today:* Live ignores `search`, `paymentType`, `page` and `perPage`, filters on UTC days, so a running order from an earlier day (or created 00:00–05:29 IST) doesn't show under `Today`, and its rows have no time. *Target:* Live always lists **every** running order of the restaurant (status 0 / 1), whatever `startDate` / `endDate` say; the channel filter and `search` apply (the same matching as History); `paymentType` and paging don't apply; each row adds `orderDateTime` (IST). The app hides the date chips and the payment-mode filter on the Live tab. Still accepted: a billed order's Live card shows the subtotal labelled `Sub Total`.
- **HIST-D3 — To fix (2026-11-24, was accepted 2026-11-11); sorting fixed 2026-11-18.** *Target:* both lists owner and waiter only (X-D1: 403 `You do not have permission to perform this action`); History loads each order's payments as rows (no `type:received:change` text joining, so no garbling and no 1024-character cut-off). Still accepted: search is `LIKE %text%` on the order number (wildcards not escaped) or an exact grand total; each History row runs its own bill and KOT queries (≤ 100 rows a page).
- **Inherited:** BILL-D2 fixed (the Live `Bill Payment` button opens the bill); ORD-D3 (To fix — customer fields); AUTH-D2 (To fix, X-D2); AUTH-D5 / X-D5.

Original decision texts (history):

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
- **Inherited (2026-11-11):** BILL-D2 (the Live `Bill Payment` button never opens the bill), ORD-D3 (no customer fields), AUTH-D2, AUTH-D5.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Billing and payment — BILL. Printing — PRN. Sales reports — RPT.
