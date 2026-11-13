# 02 — Take payment — tester cases

Use case: [02-take-payment.md](../02-take-payment.md). Fixtures: [README.md](./README.md#fixtures).

Signed in as WTR-A. Payment can't be taken from the app today (BILL-D2 — see BILL-01-T01 to T03); this case checks what the app shows after a payment made through the API.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| BILL-02-T01 | AC5 (BILL-D2) | Dine In → occupy `Table 1` → send a KOT. Ask a developer to bill and pay that order through the API (TOT-525 + PAY-525). Go back to the table grid. | `Table 1` still shows Occupied (the app's order has no table, so payment frees none). |
