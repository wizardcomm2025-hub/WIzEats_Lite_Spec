# 03 — Export the report — tester cases

Use case: [03-export-report.md](../03-export-report.md). Screen text: [ui-ux.md §2](../ui-ux.md#2-export-toasts). Fixtures: [README.md](./README.md#fixtures).

Signed in as OWN-A on an Android device; Dashboard → `Export Report (CSV)` at the bottom. Open the files from the device's Downloads folder.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| RPT-03-T01 | AC1, AC3 — *Fails until RPT-D1 is fixed*, *Fails until RPT-D3 is fixed* | Today: send a Takeaway KOT `Paneer Tikka` × 2 (don't pay); send and pay another (CASH). Tap `Export Report (CSV)`. | Toast `Files saved to Downloads folder`. `reports_<n>.csv`: `"Item Name","Quantity","Total"` and `"Paneer Tikka","2","500.00"`. `orders_<n>.csv`: two rows (paid and unpaid), with hex order IDs and IST times. (Today: unquoted, and `Paneer Tikka,4,1000.00`.) |
| RPT-03-T02 | AC2 | On a restaurant with no orders today, tap `Export Report (CSV)`. | Toast `No data available`; no new file. |
| RPT-03-T03 | Alt — *Fails until RPT-D3 is fixed* | Sell and pay ITEM-COMMA and ITEM-FORMULA today. Export; open both CSVs in a spreadsheet. | `Tikka, Paneer` stays in one cell; `=1+1` shows as text (with a leading apostrophe), not 2. (Today: the comma row splits into four columns; `=1+1` shows 2.) |
| RPT-03-T04 | AC4 — *Fails until RPT-D3 is fixed* | Pay ORDER-1050 in the app today. Export; open `orders_<n>.csv` in a spreadsheet. | ORDER-1050's row has 5 columns, `Grand Total` 1050.00. (Today: `1,050.00` splits into two columns and shifts the rest.) |
