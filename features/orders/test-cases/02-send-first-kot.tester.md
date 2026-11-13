# 02 — Send the first KOT — tester cases

Use case: [02-send-first-kot.md](../02-send-first-kot.md). Screen text: [ui-ux.md §3](../ui-ux.md#3-cart-and-kot). Fixtures: [README.md](./README.md#fixtures).

Signed in as WTR-A (phone) unless stated. A developer checks the database where the expected result says so.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| ORD-02-T01 | AC1 | Dine In → occupy Table 1 → add `Paneer Tikka`, set qty 2 → open the cart; all ticked → send the KOT. | Toast `KOT generated successfully!`; KOT card shows order number `CODE-A/<n>`, KOT number, `Paneer Tikka × 2`. Developer: order subtotal 500.00, tax % 5.00, 1 KOT, 1 line at 250.00. The kitchen (CHF-A) sees the KOT. |
| ORD-02-T02 | AC2 (accepted ORD-D2) | After T01, go back to the grid. | Table 1 still `Occupied` (amber), no `Ordered`. Developer: the order's `tableId` is NULL. |
| ORD-02-T03 | AC3 | Add `Paneer Tikka` and `Chicken 65`; untick `Chicken 65`; send. | Toast `KOT generated successfully! 1 held item still in the cart.`; `Chicken 65` still in the cart. |
| ORD-02-T04 | AC4 | Untick all items; send. | Toast `Tick at least one item to send on this KOT.`; nothing sent. |
| ORD-02-T05 | AC5 (accepted ORD-D3) | Takeaway → add an item → fill customer name `Ravi` and phone `9876543210` (if the form shows them) → send. | KOT sent. Developer: `customerName` / `customerPhone` NULL. |
| ORD-02-T06 | Numbering (accepted ORD-D3) | Send five Takeaway orders in a row today. | Each KOT card shows an order number `CODE-A/<n>`; record the numbers — repeats or gaps are possible and accepted. |
| ORD-02-T07 | Double order (ORD-D2 / D4) | Dine In: occupy Table 3, send a KOT, go back, tap Table 3, send another KOT. | Two separate orders (two order numbers); the second does not include the first's items. |
| ORD-02-T08 | Alt: no reply | Slow / cut network right after tapping send. | Toast `No reply from the server. The KOT may already have reached the kitchen — check before sending it again.` If sent again after the first went through: two KOTs in the kitchen (accepted ORD-D4). |
| ORD-02-T09 | Alt: inactive item | Check the ordering screen. | `Old Dish` (inactive) not offered. |
| ORD-02-T10 | Room Service | Tap `ROOM SERVICE`. | No rooms to pick (ROOM-D1); no order can be started there. |
