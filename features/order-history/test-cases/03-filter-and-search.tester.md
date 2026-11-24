# 03 — Filter and search — tester cases

Use case: [03-filter-and-search.md](../03-filter-and-search.md). Screen text: [ui-ux.md §3](../ui-ux.md#3-filter-sheet). Fixtures: [README.md](./README.md#fixtures) (ORDER-PAID, ORDER-UPI and one running order prepared today).

Signed in as OWN-A; Order History.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| HIST-03-T01 | AC1 | `History`: search `#<ORDER-PAID number's last digits>`; then `₹525`; then `CODE-B`. | First two: ORDER-PAID listed (others whose number contains those digits may show too); `CODE-B`: `No Past Orders`. |
| HIST-03-T02 | AC2 — *Fails until HIST-D2 is fixed* | `Live`: search `zzz`. | `No Live Orders`. (Today: the running order is still listed.) |
| HIST-03-T03 | AC3 (HIST-D1 fixed) | `Filters` → `Order Channel` → `Takeaway` → `Apply`, on `Live` and on `History`. Then tap `Reset`. | Only Takeaway orders on each tab; no error toast. After `Reset` all orders return. |
| HIST-03-T04 | AC3 (HIST-D1 fixed) | With a Dine In order paid today: `History` → `Filters` → `Dine In` → `Apply`. | Only the Dine In order. |
| HIST-03-T05 | AC4 | `History`: `Filters` → `UPI` → `Apply`. | ORDER-UPI listed; ORDER-PAID (cash) not. |
| HIST-03-T06 | Alt | `Filters` → `Custom` → pick only a start date → `Apply`. | Toast `Pick both a start and an end date.`; the sheet stays open. |
| HIST-03-T07 | Alt | Tap the `7 days` chip. | `Filters` badge shows 1; the list reloads; `Reset` appears. |
