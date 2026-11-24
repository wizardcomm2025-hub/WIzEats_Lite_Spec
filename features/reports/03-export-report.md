# 03 — Export the report

## Summary

`Export Report (CSV)` saves two files to the device's Downloads folder: today's items sold, and today's orders. Since 2026-11-18/19 the orders file is written too. **Today the items file counts unpaid orders (RPT-D1), and neither file is escaped — an order total of ₹1,000 or more, or an item name with a comma, shifts the columns (RPT-D3).**

## Prerequisites

- **Backend:** `POST /api/reports/get-all-items`, `POST /api/reports/get-all-orders` — [api-contract.md](./api-contract.md#post-apireportsget-all-items)
- **Data:** `order_items`, `orders`, `order_channels`, `payments`
- **Device:** storage permission on Android below 10

## Actors & preconditions

- **Actor:** owner. Today any role through the API; target owner only (RPT-D3 / X-D1).
- **Preconditions:** Dashboard open.

## Main flow

1. The owner taps `Export Report (CSV)` (at the bottom of the Dashboard). On Android 9 or lower the app asks for storage permission first.
2. The app posts `{}` to both endpoints, so both use today: `get-all-items` today's UTC date (target IST — RPT-D2), `get-all-orders` today's IST date.
3. `get-all-items`: today every order line of the day (any status), grouped by item name — `menuItemName`, `quantity`, `total`, `last_created`, newest first. *Target (RPT-D1):* only lines of fully paid orders.
4. `get-all-orders`: the day's orders with channel name, table / room (hex), `grandTotal` (today `en_IN`-formatted, e.g. `1,050.00`; target plain `1050.00`), `paymentStatus` (`Paid` / `Unpaid`), `created_at` in IST.
5. The app writes `reports_<timestamp>.csv` (`Item Name,Quantity,Total`) and `orders_<timestamp>.csv` (`Order ID,Channel,[Table ID],[Room ID],Grand Total,Payment Status,Created At`), then shows `Files saved to Downloads folder`. *Target (RPT-D3):* every field in double quotes (inner quotes doubled), and a cell starting with `=`, `+`, `-` or `@` prefixed with an apostrophe.

## Alternative flows

- **Nothing today.** Both lists empty: toast `No data available`.
- **Storage permission refused.** Toast `Storage permission denied`.
- **Write fails.** Still `Files saved to Downloads folder` (each save's failure is ignored); an exception before that gives `Export failed!`.
- **Item name with a comma, quote or leading `=`, or a total of ₹1,000 or more.** Today written as is, shifting columns or read as a formula; *target* quoted and neutralised.
- **API with dates.** `startDate` / `endDate` (dates; end ≥ start) choose the range.

## Acceptance criteria

1. *(Target, RPT-D1)* Given today one running order and ORDER-PAID (`Paneer Tikka` × 2 each), when exported, then `reports_<ts>.csv` holds `"Item Name","Quantity","Total"` and `"Paneer Tikka","2","500.00"` — the running order not counted. (Today: `Paneer Tikka,4,1000.00`, unquoted.)
2. Given no orders today, then the toast `No data available` and no file.
3. Given today's ORDER-PAID, when exported, then `orders_<ts>.csv` has one row with its hex order ID, `Takeaway`, `525.00`, `Paid` and today's IST time.
4. *(Target, RPT-D3)* Given a paid order of ₹1,050 today, then its `orders_` row keeps 5 columns with `"1050.00"` in `Grand Total`. (Today: `1,050.00` splits into two columns.)

## Platform notes

### POS API

`ReportController::getAllItems` / `getAllOrders` (`ReportOrderRequest`) → `ReportService`.

### Data — MySQL

Read only.

### POS app

`analyticsScreen.handleReportExport`, `convertToCSV`, `convertOrdersToCSV` (today plain `join(',')`), `react-native-blob-util` (Downloads, MediaStore on Android).

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- PDF or emailed reports — not offered.

## Open questions

- None open. Related decisions: RPT-D1, RPT-D2, RPT-D3.
