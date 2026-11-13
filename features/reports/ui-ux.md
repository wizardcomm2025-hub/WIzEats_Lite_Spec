# Reports — UI / UX

Screen text exactly as in the code on 2026-11-12. Only the parts testers check are listed.

## 1. Dashboard

| Part | Content |
|---|---|
| Entry | Owner: drawer `Analytics` (expands) → `Dashboard`; tablet rail `Dashboard` |
| Header | `Dashboard` |
| Period chips | `Today` · `Yesterday` · `7 Days` · `15 Days` · `30 Days` (the last is the calendar month — RPT-D2) |
| Loading | `Loading analytics data...` (new period); `Updating…` bar (revisit) |
| Stat cards | `Total Revenue ₹<x>` · `Total Orders <n>` · `Avg Order Value ₹<x.xx>` · `Table Turnover Rate <x.x>x` |
| Revenue Breakdown | `Collections` / `Revenue Breakdown`: `Cash`, `UPI` with shares; empty `No Payments Yet` · `Cash and UPI collections will appear here once bills are paid.` |
| Top Selling | `Leaderboard` / `Top Selling`: `Best Seller` card (`<n> units sold`, amount, `#1`), then `Other Top Performers` (rank, name, amount, `<n>` `SOLD`, bar); empty `No Sales Data` · `Top selling items will appear here once orders are completed.` |
| Revenue Trend | `Performance` / `Revenue Trend`; tabs `Daily` · `Weekly` · `Monthly`; empty `No Data Available` · `Transactions will appear here once recorded.` |
| Order Trend | `Orders Analytics` / `Order Trend`; empty `No Order Data` · `Order trends will appear here once orders are placed.` |
| Order Channels | `Order Channels` (pie with percentages); empty `No Channel Data` · `Order channel distribution will appear here once orders are placed.` |
| Export | `Export Report (CSV)` |
| Errors | Toast with the server's message |

## 2. Export toasts

| Case | Toast |
|---|---|
| Saved (or partly saved) | `Files saved to Downloads folder` |
| Nothing to export | `No data available` |
| Permission refused (Android ≤ 9) | `Storage permission denied` |
| Unexpected error | `Export failed!` |

Files: `reports_<timestamp>.csv` (`Item Name,Quantity,Total`), `orders_<timestamp>.csv` (`Order ID,Channel,[Table ID],[Room ID],Grand Total,Payment Status,Created At`).
