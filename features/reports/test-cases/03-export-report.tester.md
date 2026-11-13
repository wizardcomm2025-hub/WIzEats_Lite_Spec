# 03 — Export the report — tester cases

Use case: [03-export-report.md](../03-export-report.md). Screen text: [ui-ux.md §2](../ui-ux.md#2-export-toasts). Fixtures: [README.md](./README.md#fixtures).

Signed in as OWN-A on an Android device; Dashboard → `Export Report (CSV)` at the bottom. Open the files from the device's Downloads folder.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| RPT-03-T01 | AC1 (RPT-D3) | Send a Takeaway KOT `Paneer Tikka` × 2 today. Tap `Export Report (CSV)`. | Toast `Files saved to Downloads folder`. Downloads has `reports_<number>.csv` with `Item Name,Quantity,Total` and `Paneer Tikka,2,500.00`; **no** `orders_<number>.csv`. |
| RPT-03-T02 | AC2 | On a restaurant with no orders today, tap `Export Report (CSV)`. | Toast `No data available`; no new file. |
| RPT-03-T03 | Alt (RPT-D3) | Ask a developer to create ITEM-COMMA and send one KOT with it today. Export; open the reports CSV in a spreadsheet. | The `Tikka, Paneer` row is split: `Tikka` / ` Paneer` / quantity / total over four columns. |
