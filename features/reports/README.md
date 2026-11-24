# Reports / analytics

The owner's **Dashboard** (drawer `Analytics` → `Dashboard`, tablet rail `Dashboard`) shows sales figures for a period (`Today`, `Yesterday`, `7 Days`, `15 Days`, `30 Days`): total revenue, orders, average order value, table turnover, the cash / UPI split, top-selling items and the channel mix. It also shows revenue and order trends (daily / weekly / monthly) and can export CSV files.

The spec records today's behaviour plus targets (re-synced 2026-11-24 against API commit d54d6c8 and `POS_APP-kot`). Payments are taken in the app now (BILL-D2 fixed), so the figures fill up. Since 2026-11-18 revenue is the money actually collected and Top Selling counts fully paid orders only; since 2026-11-19 the orders export works (hex IDs, IST days). RPT-D1 to RPT-D3 are *To fix* for what remains.

**Feature code:** `RPT` (test IDs `RPT-01-T01`, `RPT-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [View the dashboard](./01-view-dashboard.md) | Owner (today any role through the API — RPT-D3) | [tester](./test-cases/01-view-dashboard.tester.md) | [developer](./test-cases/01-view-dashboard.developer.md) |
| 02 | [See revenue and order trends](./02-see-trends.md) | same | [tester](./test-cases/02-see-trends.tester.md) | [developer](./test-cases/02-see-trends.developer.md) |
| 03 | [Export the report](./03-export-report.md) | same | [tester](./test-cases/03-export-report.tester.md) | [developer](./test-cases/03-export-report.developer.md) |

## Key concepts

- **Paid order** — `orders.orderStatus` 2. Only these count for revenue, orders, turnover, channel mix and trends; Top Selling also requires the payments to cover the grand total (since 2026-11-18); target the items export too (RPT-D1).
- **Period** — `filter_type`: `today`, `yesterday`, `last_7_days` (today + 6 days before), `last_15_days` (today + 14), `last_30_days` (today **the current calendar month**; target today + 29 — RPT-D2). Days are UTC days today; target IST days.
- **Trend type** — `daily` (this Monday–Sunday), `weekly` (this month, weeks 1–5 by `CEIL(day / 7)`), `monthly` (this January–December).
- **Revenue (collected)** — since 2026-11-18: Σ(`receivedAmount` − `changeAmount`) of the completed payments of the period's paid orders, all payment types; the **breakdown** shows the CASH and UPI parts.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api/api/specs/reports/` (created with the first plan)

## Depends on

- **ORD** — order lines (Top Selling, items export). **BILL** — paid orders, totals and payments (everything else).
- **TBL** — table count for the turnover rate.

## Decisions

Statuses per [conventions.md → Decision status](../../conventions.md#decision-status), set 2026-11-24. Cross-cutting items: [project-overview.md → Known gaps and targets](../../project-overview.md#known-gaps-and-targets).

| ID | Status | Short | Tests |
|---|---|---|---|
| RPT-D1 | **To fix** (revenue and Top Selling fixed in code 2026-11-18) | Items export counts unpaid orders | 03-D01, 03-D08; 03-T01 |
| RPT-D2 | **To fix** | UTC days; `30 Days` = calendar month; daily trend Sunday-first | 01-D05, 01-D06, 02-D01, 03-D09 |
| RPT-D3 | **To fix** (orders export fixed in code 2026-11-18/19; with X-D1, X-D5) | Unsafe CSV; formatted total breaks the orders CSV; any role; exception text | 01-D09, 01-D13, 02-D07, 03-D02, 03-D05, 03-D07; 01-T04, 03-T03, 03-T04 |
| AUTH-D2 (inherited) | **To fix** (X-D2) | Switched-off / unapproved callers keep access | 01-D10, 02-D07, 03-D05 |

- **Fixed in code 2026-11-18 / 19** (part of RPT-D1 and RPT-D3): `total_revenue` is the money collected — Σ(received − change) of completed payments of the period's paid orders — and `avg_order_value` divides that by the paid orders; `top-selling-items` counts only lines of paid orders whose completed payments cover the grand total, and now has the message `Top selling items fetched successfully`; `get-all-orders` returns hex IDs (no more 500), takes `startDate` / `endDate` as IST days, returns `created_at` in IST and `grandTotal` formatted `en_IN` (e.g. `1,050.00`). With the ORD-D2 fix, app orders carry their table, so table turnover is no longer 0.
- **RPT-D1 — To fix (2026-11-24, was accepted 2026-11-12).** *Target:* `get-all-items` counts only lines of paid orders whose payments cover the grand total, like Top Selling. Still accepted: an order paid for less than its total (possible until BILL-D1 is fixed) counts in `Total Orders` but not in Top Selling or the items export; the trends sum `grandTotal` of paid orders.
- **RPT-D2 — To fix (2026-11-24, was accepted 2026-11-12).** *Target:* every period, both trends and both exports use IST calendar days (converted to UTC for the query); `last_30_days` = today and the 29 days before; the daily trend lists Monday first. Still accepted: the weekly trend's `Week 1`–`Week 5` by `CEIL(day / 7)`; `top-selling-items` returns every item (the app shows up to 100).
- **RPT-D3 — To fix (2026-11-24, was accepted 2026-11-12).** *Target:* `get-all-orders` returns `grandTotal` as a plain decimal string (`"1050.00"`); the app quotes every CSV field (RFC 4180: in double quotes, inner quotes doubled) and prefixes a cell starting with `=`, `+`, `-` or `@` with an apostrophe; all six endpoints owner only (X-D1: 403 `You do not have permission to perform this action`); 500s carry no `error` key or exception text (X-D5).
- **Inherited:** BILL-D1 (To fix — once fixed only CASH / UPI exist and payments add up); BILL-D2 and ORD-D2 fixed; AUTH-D2 (To fix, X-D2); AUTH-D5 / X-D5.

Original decision texts (history):

- **RPT-D1 (2026-11-12):** What is counted stays as it is:
  - revenue, order count, average, turnover, revenue breakdown, channel mix and both trends count **paid orders only**. With no app payments (BILL-D2) they are all 0 / empty in practice;
  - Top Selling and the items export count **every order line** in the period, of running, billed and paid orders alike, so they disagree with the other figures;
  - revenue is the sum of the orders' `grandTotal`, which comes from the client (BILL-D1);
  - table turnover = paid orders with a table ÷ all the restaurant's tables. App orders have no table (ORD-D2), so it is 0;
  - the cash / UPI split ignores any other `paymentType` text.

  Accepted.
- **RPT-D2 (2026-11-12):** Periods stay as they are:
  - every period uses UTC days, so `Today` covers 05:30 IST today to 05:29 IST tomorrow;
  - `30 Days` (`last_30_days`) is the current calendar month, not the last 30 days;
  - the daily trend covers this Monday–Sunday but is sorted by `DAYOFWEEK`, so Sunday comes first;
  - the weekly trend numbers weeks by `CEIL(day / 7)`, so days 29–31 form `Week 5`;
  - `top-selling-items` returns every item of the period (the app shows up to 100), not a top three.

  Accepted.
- **RPT-D3 (2026-11-12):**
  - `get-all-orders` returns raw binary `orderId` / `tableId` / `roomId`, so whenever the range holds an order the reply can't be encoded: 500 `Server Error`. The export then writes only the items CSV but still shows `Files saved to Downloads folder`.
  - The CSV files aren't escaped: a comma in an item name shifts the columns, and a name starting with `=`, `+`, `-` or `@` is read as a formula by spreadsheet programs.
  - No role check: a waiter or chef token can read all the reports, including revenue.

  Accepted risk.
- **Inherited (2026-11-12):** BILL-D1, BILL-D2, ORD-D2, AUTH-D2, AUTH-D5.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- The waiter and chef dashboards — WAIT, KITCH. Order lists — HIST. The super-admin panel's statistics — ADM.
