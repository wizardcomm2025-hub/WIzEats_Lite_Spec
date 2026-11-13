# 01 — View the dashboard

## Summary

The owner opens the Dashboard and picks a period. The stat cards, the revenue breakdown, Top Selling and Order Channels load for that period. **Today, with no paid orders, everything but Top Selling is 0 or empty (RPT-D1).**

## Prerequisites

- **Backend:** `POST /api/reports/dashboard-summary`, `/top-selling-items`, `/order-count-by-channel` — [api-contract.md](./api-contract.md)
- **Data:** `orders`, `order_items`, `order_channels`, `payments`, `tables`

## Actors & preconditions

- **Actor:** owner (the app shows the screen to owners only). Any role through the API (RPT-D3).
- **Preconditions:** signed in.

## Main flow

1. The owner opens `Dashboard`. The period is `Today`. The app posts `{ filter_type: "today" }` to the three endpoints (and the trend — 02). It loads again each time the screen comes into focus; a new period shows `Loading analytics data...`, a revisit shows `Updating…`. Pull down refreshes.
2. **Stat cards:** `Total Revenue ₹<sum of grandTotal>`, `Total Orders <n>`, `Avg Order Value ₹<revenue ÷ orders>`, `Table Turnover Rate <x.x>x` — paid orders of the period (RPT-D1).
3. **Revenue Breakdown:** `Cash` and `UPI` amounts and shares; `No Payments Yet` · `Cash and UPI collections will appear here once bills are paid.` when both are 0.
4. **Top Selling:** every item sold in the period (any order status), most units first: the first as `Best Seller` with `<n> units sold` and its amount, the rest under `Other Top Performers`. Empty: `No Sales Data` · `Top selling items will appear here once orders are completed.`
5. **Order Channels:** paid orders per channel with percentages; empty: `No Channel Data`.
6. Changing the period reloads these three; the trend stays.

## Alternative flows

- **Server error.** A toast with the server's message (e.g. `Failed to fetch top selling items`).
- **`filter_type` missing or unknown** (API). 422.

## Acceptance criteria

1. Given REST-A has today only running orders from the app (`Paneer Tikka` × 2), when the Dashboard opens, then `Total Revenue ₹0`, `Total Orders 0`, `Avg Order Value ₹0.00`, `Table Turnover Rate 0.0x`, `No Payments Yet`, `No Channel Data`, and Top Selling shows `Paneer Tikka` `2 units sold` `₹500` (RPT-D1).
2. Given ORDER-PAID (paid today, 525.00, CASH received 600 change 75), then `Total Revenue ₹525.00`, `Total Orders 1`, `Avg Order Value ₹525.00`, and the breakdown shows Cash 525 (100 %), UPI 0.
3. Given the period `30 Days` on 12 Nov, then only orders from 1 Nov onwards count (RPT-D2).
4. Given REST-B's paid orders, then REST-A's figures don't include them.

## Platform notes

### POS API

`ReportController` (`ReportRequest`: `filter_type` required, in the five values) → `ReportService::getDashboardSummary`, `getTopThreeSellingItems`, `getOrderCountByChannel`.

### Data — MySQL

Read only. See [data-model.md](./data-model.md).

### POS app

`analytics/analyticsScreen.tsx`, `DateFilter`, `StatCard`, `PaymentBreakdown`, `OrderStats`, `OrderChannels`.

### Super-admin panel

None.

### Integrations

None.

## Configuration

Server timezone UTC.

## Out of scope

- Per-staff figures — WAIT / KITCH.

## Open questions

- None open.
