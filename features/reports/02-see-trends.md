# 02 — See revenue and order trends

## Summary

Under the stat cards, **Revenue Trend** (line chart) and **Order Trend** (bar chart) show paid orders over this week, this month or this year, picked with `Daily` / `Weekly` / `Monthly`. **Today they are empty in practice (RPT-D1).**

## Prerequisites

- **Backend:** `POST /api/reports/revenue-and-order-trend` — [api-contract.md](./api-contract.md#post-apireportsrevenue-and-order-trend)
- **Data:** `orders`

## Actors & preconditions

- **Actor:** owner. Any role through the API (RPT-D3).
- **Preconditions:** Dashboard open.

## Main flow

1. The trend type starts as `Daily`. The app posts `{ trend_type: "daily" }`.
2. The server groups the restaurant's paid orders:
   - `daily` — this week Monday–Sunday, by day name (`label` e.g. `Monday`), sorted by `DAYOFWEEK` (Sunday first — RPT-D2);
   - `weekly` — this month, `Week 1` … `Week 5` (`CEIL(day / 7)`);
   - `monthly` — this year, by month name (`January` …).
   It returns `revenue` (`label`, `total_revenue`) and `orders` (`label`, `total_orders`), only for periods that have orders.
3. The charts plot those points. Switching the tab reloads only the trend.

## Alternative flows

- **No paid orders in the window.** `No Data Available` · `Transactions will appear here once recorded.` and `No Order Data` · `Order trends will appear here once orders are placed.`
- **`trend_type` missing or unknown** (API). 422.

## Acceptance criteria

1. Given no paid orders, then both charts show their empty states.
2. Given paid orders on Monday (2) and Sunday (1) of this week, when `daily` is requested, then `orders` = `[ { label: "Sunday", total_orders: 1 }, { label: "Monday", total_orders: 2 } ]` (RPT-D2).
3. Given a paid order on the 30th of this month, when `weekly`, then it is under `Week 5`.

## Platform notes

### POS API

`ReportController::revenueTrend` (`ReportRevenueTrendRequest`) → `ReportService::getRevenueTrend`.

### Data — MySQL

Read only.

### POS app

`RevenueTrend` (with the tabs), `OrdersTrend`.

### Super-admin panel

None.

### Integrations

None.

## Configuration

Week starts on Monday (Carbon default); UTC.

## Out of scope

- Custom ranges — not offered.

## Open questions

- None open.
