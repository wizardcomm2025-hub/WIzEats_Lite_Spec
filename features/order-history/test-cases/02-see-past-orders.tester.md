# 02 — See past orders — tester cases

Use case: [02-see-past-orders.md](../02-see-past-orders.md). Screen text: [ui-ux.md](../ui-ux.md). Fixtures: [README.md](./README.md#fixtures).

Signed in as OWN-A; Order History → `History`.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| HIST-02-T01 | AC3 | No paid orders today (only running ones). | `No Past Orders` · `Settled and closed orders will appear here. Pull down to refresh.` |
| HIST-02-T02 | AC1 | With ORDER-PAID prepared, pull down. | Card: `Order No. <number>`, `Paid`, `<channel> · 2 items · <today's date> <time>` (IST), `₹525.00` `GRAND TOTAL`; no `Bill Payment` button. |
| HIST-02-T03 | AC1 | Tap the card. | Chip `CASH`; `Paneer Tikka ₹250.00 × 2`; `Sub Total ₹500.00`, `Tax ₹25.00`, `Change Returned ₹75.00`, `Grand Total ₹525.00`; button `Print Bill No. <n>`. |
| HIST-02-T04 | AC2 | With PAID-25 prepared, scroll to the end of the list (repeat until nothing more loads). | 25 paid orders in total (plus ORDER-PAID if present), each once; the tab count matches. |
| HIST-02-T05 | AC4 (BILL-D2 fixed) | Pay an order in the app (BILL 02); open `History`. | The order is first in the list with `Paid`, its time, and the payment chip(s). |
