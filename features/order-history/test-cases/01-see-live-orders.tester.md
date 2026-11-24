# 01 — See live orders — tester cases

Use case: [01-see-live-orders.md](../01-see-live-orders.md). Screen text: [ui-ux.md](../ui-ux.md). Fixtures: [README.md](./README.md#fixtures).

Signed in as WTR-A; Order History from the drawer (phone) or the `Orders` rail button (tablet). Filters at their defaults (`Today`).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| HIST-01-T01 | AC1 | Send a Takeaway KOT (`Paneer Tikka` × 2). Open Order History. | `Live` tab selected; the order listed first: `Order No. <number>`, `Pending`, `Takeaway · 2 items`, `₹500.00` `SUB TOTAL`, button `Bill Payment`. The `Live` tab count includes it. |
| HIST-01-T02 | AC1 | Tap the card. | Expands: `Paneer Tikka`, `₹250.00 × 2`, `₹500.00`; `Sub Total ₹500.00`; note `Tax and round-off are added when the bill is generated.` |
| HIST-01-T03 | AC2 | Generate the bill for that order (share it from Bill Payment, BILL 01). Back in Order History, pull down. | Card shows `Bill generated` next to `Pending`, and still `₹500.00` `SUB TOTAL` (accepted). |
| HIST-01-T04 | AC6 (BILL-D2 fixed) | Tap `Bill Payment`. | Button shows `Loading Bill…`, then Bill Payment opens for that order. |
| HIST-01-T05 | AC5 — *Fails until HIST-D2 is fixed* | With ORDER-OLD prepared, open `Live`. | ORDER-OLD is listed with yesterday's date and time; no date chips on the Live tab. (Today: hidden under `Today`; shown under `7 days`.) |
| HIST-01-T06 | AC3 | Pay T01's order in the app (BILL 02). Pull down on `Live`. | The order is gone from `Live` and is at the top of `History`. |
