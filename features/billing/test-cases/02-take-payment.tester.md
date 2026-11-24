# 02 — Take payment — tester cases

Use case: [02-take-payment.md](../02-take-payment.md). Screen text: [ui-ux.md §3](../ui-ux.md#3-bill-payment). Fixtures: [README.md](./README.md#fixtures).

WTR-A (phone) or OWN-A (tablet).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| BILL-02-T01 | AC1 | Open Bill Payment for a ₹525 order (01-T01). Choose `CASH`, enter ₹600 received, confirm. | Change due `₹75`; payment success dialog; `Next Order` for takeaway (`Done` otherwise). The order leaves Order History → Live. |
| BILL-02-T02 | AC5 (BILL-D2 fixed) | Dine In → occupy Table 1 → send a KOT → `View Bill` → pay in full in cash. Go back to the table grid. | Table 1 is `Available` again. |
| BILL-02-T03 | Customer display | On a terminal with a customer display: open Bill Payment for a ₹525 order; pay ₹600 cash; close the page. (Skip on devices without one.) | The display shows ₹525.00 while the bill is open, ₹75.00 change after paying, and goes blank when the page closes. |
| BILL-02-T04 | Split; AC2 | Open Bill Payment for a ₹525 order. Add a CASH leg of ₹300, then a UPI leg of ₹225; confirm. | Both legs listed; remaining ₹0; success dialog. A developer sees two payments for the order (target: their amounts add up to 525). |
