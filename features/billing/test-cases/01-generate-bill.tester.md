# 01 — Generate the bill — tester cases

Use case: [01-generate-bill.md](../01-generate-bill.md). Screen text: [ui-ux.md §1](../ui-ux.md#1-entry-points). Fixtures: [README.md](./README.md#fixtures).

Signed in as WTR-A. These cases confirm that the bill page can't be opened today (BILL-D2).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| BILL-01-T01 | BILL-D2 | Phone: Takeaway → add `Paneer Tikka` → send the KOT. On the KOT sheet tap `Collect Payment`. | Nothing happens: no Bill Breakdown or Bill Payment screen, no message. |
| BILL-01-T02 | BILL-D2 | Tablet: same as T01, then tap `Collect Payment` on the KOT banner. | Nothing happens. |
| BILL-01-T03 | BILL-D2 | Open Order History → `Live`. If T01's order is listed, tap its bill button. | A spinner shows on the row, then stops; the bill page doesn't open. (If the order isn't listed, record that — see HIST.) |
