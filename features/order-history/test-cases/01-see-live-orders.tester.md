# 01 — See live orders — tester cases

Use case: [01-see-live-orders.md](../01-see-live-orders.md). Screen text: [ui-ux.md](../ui-ux.md). Fixtures: [README.md](./README.md#fixtures).

Signed in as WTR-A; Order History from the drawer (phone) or the `Orders` rail button (tablet). Filters at their defaults (`Today`).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| HIST-01-T01 | AC1 | Send a Takeaway KOT (`Paneer Tikka` × 2). Open Order History. | `Live` tab selected; the order listed: `Order No. <number>`, `Pending`, `Takeaway · 2 items`, `₹500.00` `SUB TOTAL`, button `Bill Payment`. The `Live` tab count includes it. |
| HIST-01-T02 | AC1 | Tap the card. | Expands: `Paneer Tikka`, `₹250.00 × 2`, `₹500.00`; `Sub Total ₹500.00`; note `Tax and round-off are added when the bill is generated.` |
| HIST-01-T03 | AC2 (HIST-D2) | Ask a developer to generate the bill for that order (TOT-525). Pull down. | Card shows `Bill generated` next to `Pending`, and still `₹500.00` `SUB TOTAL`. |
| HIST-01-T04 | Alt (BILL-D2) | Tap `Bill Payment`. | Button shows `Loading Bill…`, then returns to `Bill Payment`; no bill page opens. |
| HIST-01-T05 | Alt (HIST-D2) | With ORDER-OLD prepared, look at `Today`; then tap the `7 days` chip. | Not listed under `Today`; listed under `7 days`. |
| HIST-01-T06 | AC3 | Ask a developer to pay T01's order (PAY-525). Pull down on `Live`. | The order is gone from `Live`. |
