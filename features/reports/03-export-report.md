# 03 — Export the report

## Summary

`Export Report (CSV)` saves two files to the device's Downloads folder: today's items sold, and today's orders. **Today the orders file is never written when there are orders, but the app still reports success (RPT-D3).**

## Prerequisites

- **Backend:** `POST /api/reports/get-all-items`, `POST /api/reports/get-all-orders` — [api-contract.md](./api-contract.md#post-apireportsget-all-items)
- **Data:** `order_items`, `orders`, `order_channels`
- **Device:** storage permission on Android below 10

## Actors & preconditions

- **Actor:** owner. Any role through the API (RPT-D3).
- **Preconditions:** Dashboard open.

## Main flow

1. The owner taps `Export Report (CSV)` (at the bottom of the Dashboard). On Android 9 or lower the app asks for storage permission first.
2. The app posts `{}` to both endpoints; the server uses today (UTC date) for both ends.
3. `get-all-items`: every order line of today (any status), grouped by item name — `menuItemName`, `quantity`, `total`, `last_created`, newest first.
4. `get-all-orders`: today's orders with channel name, table / room, `grandTotal`, `paymentStatus` (`Paid` / `Unpaid`), `created_at`. **Today:** raw binary IDs make the reply fail with 500 `Server Error` whenever there is an order (RPT-D3).
5. The app writes `reports_<timestamp>.csv` (`Item Name,Quantity,Total`) and, if it got orders, `orders_<timestamp>.csv` (`Order ID,Channel,[Table ID],[Room ID],Grand Total,Payment Status,Created At`), then shows `Files saved to Downloads folder`.

## Alternative flows

- **Nothing today.** Both lists empty: toast `No data available`.
- **Storage permission refused.** Toast `Storage permission denied`.
- **Write fails.** Still `Files saved to Downloads folder` (each save's failure is ignored); an exception before that gives `Export failed!`.
- **Item name with a comma, quote or leading `=`.** Written as is (RPT-D3).
- **API with dates.** `startDate` / `endDate` (dates; end ≥ start) choose the range.

## Acceptance criteria

1. Given today's running order (`Paneer Tikka` × 2), when exported, then `reports_<ts>.csv` holds `Item Name,Quantity,Total` and `Paneer Tikka,2,500.00`; no `orders_` file is written; the toast says `Files saved to Downloads folder` (RPT-D3).
2. Given no orders today, then the toast `No data available` and no file.
3. Given `get-all-orders` with a range holding an order, then 500 `{ "message": "Server Error" }`; with an empty range, 200 `data: []`.

## Platform notes

### POS API

`ReportController::getAllItems` / `getAllOrders` (`ReportOrderRequest`) → `ReportService`. The controllers catch only validation exceptions, so the encoding failure reaches Laravel's handler.

### Data — MySQL

Read only.

### POS app

`analyticsScreen.handleReportExport`, `convertToCSV`, `convertOrdersToCSV`, `react-native-blob-util` (Downloads, MediaStore on Android).

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- PDF or emailed reports — not offered.

## Open questions

- None open.
