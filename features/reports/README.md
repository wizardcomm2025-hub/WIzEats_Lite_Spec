# Reports / analytics

The owner's **Dashboard** (drawer `Analytics` → `Dashboard`, tablet rail `Dashboard`) shows sales figures for a period (`Today`, `Yesterday`, `7 Days`, `15 Days`, `30 Days`): total revenue, orders, average order value, table turnover, the cash / UPI split, top-selling items and the channel mix. It also shows revenue and order trends (daily / weekly / monthly) and can export CSV files.

The spec records today's behaviour. Decisions RPT-D1 to RPT-D3 keep the current rules as accepted risks. **Today almost every figure is 0 because the app can't take payment (BILL-D2). Only Top Selling and the items export show data, and they count unpaid orders too (RPT-D1).**

**Feature code:** `RPT` (test IDs `RPT-01-T01`, `RPT-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [View the dashboard](./01-view-dashboard.md) | Owner (any role through the API — RPT-D3) | [tester](./test-cases/01-view-dashboard.tester.md) | [developer](./test-cases/01-view-dashboard.developer.md) |
| 02 | [See revenue and order trends](./02-see-trends.md) | same | [tester](./test-cases/02-see-trends.tester.md) | [developer](./test-cases/02-see-trends.developer.md) |
| 03 | [Export the report](./03-export-report.md) | same | [tester](./test-cases/03-export-report.tester.md) | [developer](./test-cases/03-export-report.developer.md) |

## Key concepts

- **Paid order** — `orders.orderStatus` 2. Only these count for revenue, orders, turnover, channel mix and trends (RPT-D1).
- **Period** — `filter_type`: `today`, `yesterday`, `last_7_days` (today + 6 days before), `last_15_days` (today + 14), `last_30_days` (**the current calendar month** — RPT-D2). Days are UTC days.
- **Trend type** — `daily` (this Monday–Sunday), `weekly` (this month, weeks 1–5 by `CEIL(day / 7)`), `monthly` (this January–December).
- **Revenue breakdown** — per payment type: Σ(`receivedAmount` − `changeAmount`) of completed CASH / UPI payments of the period's paid orders.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api-dev_guid/specs/reports/` (created with the first plan)

## Depends on

- **ORD** — order lines (Top Selling, items export). **BILL** — paid orders, totals and payments (everything else).
- **TBL** — table count for the turnover rate.

## Decisions

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
- **Inherited:** BILL-D1, BILL-D2, ORD-D2, AUTH-D2, AUTH-D5.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- The waiter and chef dashboards — WAIT, KITCH. Order lists — HIST. The super-admin panel's statistics — ADM.
