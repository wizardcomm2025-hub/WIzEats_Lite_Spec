# 01 — Generate the bill — tester cases

Use case: [01-generate-bill.md](../01-generate-bill.md). Screen text: [ui-ux.md](../ui-ux.md). Fixtures: [README.md](./README.md#fixtures).

WTR-A (phone) or OWN-A (tablet); a printer is not needed (sharing also saves the bill).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| BILL-01-T01 | AC5 (BILL-D2 fixed) | Phone: Takeaway → add `Paneer Tikka` × 2 → send the KOT. On the KOT sheet tap `Collect Payment`. | Bill Payment opens for this order: `Paneer Tikka` × 2, subtotal `₹500.00`, tax `₹25.00`, total `₹525`, a discount entry, `CASH` / `UPI`. |
| BILL-01-T02 | AC1, AC5 | Dine In → Table 1 → send a KOT (`Paneer Tikka` × 2). In the cart tap `View Bill`; continue to Bill Payment; share (or print) the bill. Go back to the cart and try to send another KOT. | Bill Breakdown, then Bill Payment; after sharing, the receipt shows a bill number. Sending another KOT is refused: `Bill already generated, cannot add items to this order` (or the cart no longer offers it — record which). |
| BILL-01-T03 | AC5 | Order History → `Live`; tap the bill button on T01's order (phone) / select it and tap `Bill Payment` (tablet). | A spinner on the row, then Bill Payment for that order. |
| BILL-01-T04 | AC4 | In T02's flow before sharing, apply a 10 % discount on Bill Breakdown; continue; share. | `Discount Applied!` · `−₹50.00` celebration; Bill Payment shows discount `₹50.00`, tax `₹22.50`, round-off `₹0.50`, total `₹473`; the saved bill matches (no mismatch message). |
