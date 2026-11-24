# 01 — View the dashboard

## Summary

The owner opens the Dashboard and picks a period. The stat cards, the revenue breakdown, Top Selling and Order Channels load for that period, from the orders paid in the app (BILL-D2 fixed). Revenue is the money actually collected (since 2026-11-18). **Today the periods use UTC days and `30 Days` is the calendar month (RPT-D2, To fix).**

## Prerequisites

- **Backend:** `POST /api/reports/dashboard-summary`, `/top-selling-items`, `/order-count-by-channel` — [api-contract.md](./api-contract.md)
- **Data:** `orders`, `order_items`, `order_channels`, `payments`, `tables`

## Actors & preconditions

- **Actor:** owner (the app shows the screen to owners only). Today any role through the API; target owner only (RPT-D3 / X-D1).
- **Preconditions:** signed in.

## Main flow

1. The owner opens `Dashboard`. The period is `Today`. The app posts `{ filter_type: "today" }` to the three endpoints (and the trend — 02). It loads again each time the screen comes into focus; a new period shows `Loading analytics data...`, a revisit shows `Updating…`. Pull down refreshes.
2. **Stat cards:** `Total Revenue ₹<money collected>`, `Total Orders <n>`, `Avg Order Value ₹<revenue ÷ orders>`, `Table Turnover Rate <x.x>x` — paid orders of the period.
3. **Revenue Breakdown:** `Cash` and `UPI` amounts and shares; `No Payments Yet` · `Cash and UPI collections will appear here once bills are paid.` when both are 0.
4. **Top Selling:** items of the period's fully paid orders, most units first: the first as `Best Seller` with `<n> units sold` and its amount, the rest under `Other Top Performers`. Empty: `No Sales Data` · `Top selling items will appear here once orders are completed.`
5. **Order Channels:** paid orders per channel with percentages; empty: `No Channel Data`.
6. Changing the period reloads these three; the trend stays.

## Alternative flows

- **Only running orders in the period.** Every figure 0, `No Payments Yet`, `No Sales Data`, `No Channel Data`.
- **Order created 00:00–05:29 IST.** Today counted under the previous day (UTC); *target* under its IST day.
- **`30 Days`.** Today the calendar month; *target* today and the 29 days before.
- **Server error.** A toast with the server's message (e.g. `Failed to fetch top selling items`).
- **`filter_type` missing or unknown** (API). 422.

## Acceptance criteria

1. Given REST-A has today only running orders from the app (`Paneer Tikka` × 2), when the Dashboard opens, then `Total Revenue ₹0`, `Total Orders 0`, `Avg Order Value ₹0.00`, `Table Turnover Rate 0.0x`, `No Payments Yet`, `No Channel Data`, and `No Sales Data`.
2. Given ORDER-PAID (paid today, 525.00, CASH received 600 change 75), then `Total Revenue ₹525.00`, `Total Orders 1`, `Avg Order Value ₹525.00`, the breakdown shows Cash 525 (100 %), UPI 0, and Top Selling shows `Paneer Tikka` `2 units sold`.
3. *(Target, RPT-D2)* Given the period `30 Days` on 24 Nov, then orders from 26 Oct onwards count. (Today: from 1 Nov.)
4. Given REST-B's paid orders, then REST-A's figures don't include them.
5. Given a paid Dine In order on Table 1 of 4 tables, then `table_turnover_rate` is 0.25 (the card shows it with one decimal).

## Platform notes

### POS API

`ReportController` (`ReportRequest`: `filter_type` required, in the five values) → `ReportService::getDashboardSummary`, `getTopThreeSellingItems`, `getOrderCountByChannel`.

### Data — MySQL

Read only. See [data-model.md](./data-model.md).

### POS app

`analytics/analyticsScreen.tsx`, `DateFilter`, `StatCard`, `PaymentBreakdown`, `OrderStats` (shows up to 100 items), `OrderChannels`.

### Super-admin panel

None.

### Integrations

None.

## Configuration

Server timezone UTC.

## Out of scope

- Per-staff figures — WAIT / KITCH.

## Open questions

- None open. Related decisions: RPT-D1, RPT-D2, RPT-D3.
