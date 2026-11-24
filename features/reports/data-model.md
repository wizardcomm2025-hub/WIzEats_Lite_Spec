# Reports — data model

Sources: `app/Services/Report/ReportService.php` (changed 2026-11-18 / 19). Read only; no table is written. Periods and trends compare dates in UTC today (`config/app.php` timezone); `get-all-orders` takes IST dates (target: everything IST — RPT-D2).

## Period filter (`applyDateFilter`, on `created_at`)

| `filter_type` | Today (UTC) | Target (IST days) |
|---|---|---|
| `today` | `DATE(created_at)` = today | today 00:00 – 23:59:59 IST |
| `yesterday` | `DATE(created_at)` = yesterday | yesterday IST |
| `last_7_days` | today − 6 days 00:00 … today 23:59:59 | same, IST |
| `last_15_days` | today − 14 days 00:00 … today 23:59:59 | same, IST |
| `last_30_days` | `MONTH(created_at)` and `YEAR(created_at)` = now (calendar month) | today − 29 days 00:00 … today 23:59:59 IST |

## Figures

| Endpoint | Source rows | Figures |
|---|---|---|
| `dashboard-summary` | `orders`, restaurant, status 2, period | `total_orders` = count; `total_revenue` = round(Σ(`receivedAmount` − `changeAmount`) of COMPLETED payments of those orders, 2) (since 2026-11-18); `avg_order_value` = round(revenue ÷ orders, 2); `table_turnover_rate` = round(orders with a `tableId` ÷ all `tables` rows, 2); `revenue_breakdown.cash` / `.upi` = the CASH / UPI parts of the revenue |
| `top-selling-items` | `order_items` of the period ⋈ `orders` (status 2) ⋈ completed payments per order, where Σ(received − change) ≥ `grandTotal` (since 2026-11-18) | per (`menuItemId`, `menuItemName`): `total_quantity`, `total_amount` (Σ `total`); most units first; no limit |
| `order-count-by-channel` | `orders` ⟕ `order_channels`, status 2, period | per channel: `orderChannelId` (upper-case hex), `orderChannel` (`channelCode`), `orderChannelName`, `total_orders`, `orderChannelpercentage` (2 dp) |
| `revenue-and-order-trend` | `orders`, status 2, window by type | `revenue` [`label`, `total_revenue` (Σ `grandTotal`), …], `orders` [`label`, `total_orders`, …] |
| `get-all-items` | `order_items`, restaurant, `created_at` between `startDate 00:00:00` and `endDate 23:59:59` (UTC; default today), any status — target: fully paid orders only, IST days | per `menuItemName`: `quantity`, `total`, `last_created`; newest first |
| `get-all-orders` | `orders` ⟕ `order_channels`, restaurant, IST range converted to UTC, any status | `orderId`, `tableId`, `roomId` (lower-case hex), `orderChannel`, `grandTotal` (`FORMAT(…, 2, 'en_IN')` — target plain), `orderStatus`, `paymentStatus` (`Paid` if status 2 else `Unpaid`), `created_at` (IST); newest first |

## Trend windows

| `trend_type` | Window | Group / label | Order |
|---|---|---|---|
| `daily` | Monday 00:00 – Sunday 23:59:59 this week | `DAYNAME` (+ `day_number` = `DAYOFWEEK`, Sunday = 1) | `day_number` today (Sunday first); target Monday first |
| `weekly` | first – last day of this month | `Week <CEIL(DAY/7)>` | week number |
| `monthly` | 1 Jan – 31 Dec this year | `MONTHNAME` (+ `month_number`) | month number |

Windows are UTC today; target IST.
