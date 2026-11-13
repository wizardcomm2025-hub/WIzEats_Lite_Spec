# Reports — data model

Sources: `app/Services/Report/ReportService.php`. Read only; no table is written. All dates are compared in UTC (`config/app.php` timezone).

## Period filter (`applyDateFilter`, on `created_at`)

| `filter_type` | Condition |
|---|---|
| `today` | `DATE(created_at)` = today |
| `yesterday` | `DATE(created_at)` = yesterday |
| `last_7_days` | between today − 6 days 00:00 and today 23:59:59 |
| `last_15_days` | between today − 14 days 00:00 and today 23:59:59 |
| `last_30_days` | `MONTH(created_at)` and `YEAR(created_at)` = now (calendar month — RPT-D2) |

## Figures

| Endpoint | Source rows | Figures |
|---|---|---|
| `dashboard-summary` | `orders`, restaurant, status 2, period | `total_revenue` = Σ `grandTotal` (0 or decimal string); `total_orders` = count; `avg_order_value` = round(revenue ÷ orders, 2); `table_turnover_rate` = round(orders with a `tableId` ÷ all `tables` rows, 2); `revenue_breakdown.cash` / `.upi` = round(Σ(`receivedAmount` − `changeAmount`) of COMPLETED payments of those orders, by `paymentType`, 2) |
| `top-selling-items` | `order_items`, restaurant, period (any order status — RPT-D1) | per (`menuItemId`, `menuItemName`): `total_quantity`, `total_amount` (Σ `total`); most units first; no limit |
| `order-count-by-channel` | `orders` ⟕ `order_channels`, status 2, period | per channel: `orderChannelId` (upper-case hex), `orderChannel` (`channelCode`), `orderChannelName`, `total_orders`, `orderChannelpercentage` (2 dp) |
| `revenue-and-order-trend` | `orders`, status 2, window by type | `revenue` [`label`, `total_revenue`, …], `orders` [`label`, `total_orders`, …] |
| `get-all-items` | `order_items`, restaurant, `created_at` between `startDate 00:00:00` and `endDate 23:59:59` (default today), any status | per `menuItemName`: `quantity`, `total`, `last_created`; newest first |
| `get-all-orders` | `orders` ⟕ `order_channels`, restaurant, same range, any status | `orderId`, `tableId`, `roomId` (**raw binary** — RPT-D3), `orderChannel`, `grandTotal`, `orderStatus`, `paymentStatus` (`Paid` if status 2 else `Unpaid`), `created_at`; newest first |

## Trend windows

| `trend_type` | Window | Group / label | Order |
|---|---|---|---|
| `daily` | Monday 00:00 – Sunday 23:59:59 this week | `DAYNAME` (+ `day_number` = `DAYOFWEEK`, Sunday = 1) | `day_number` |
| `weekly` | first – last day of this month | `Week <CEIL(DAY/7)>` | week number |
| `monthly` | 1 Jan – 31 Dec this year | `MONTHNAME` (+ `month_number`) | month number |
