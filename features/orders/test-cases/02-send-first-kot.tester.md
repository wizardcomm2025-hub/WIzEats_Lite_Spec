# 02 — Send the first KOT — tester cases

Use case: [02-send-first-kot.md](../02-send-first-kot.md). Screen text: [ui-ux.md §3](../ui-ux.md#3-cart-and-kot). Fixtures: [README.md](./README.md#fixtures).

Signed in as WTR-A (phone) unless stated. A developer checks the database where the expected result says so.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| ORD-02-T01 | AC1 | Dine In → occupy Table 1 → add `Paneer Tikka`, set qty 2 → open the cart; all ticked → send the KOT. | Toast `KOT generated successfully!`; KOT card shows order number `CODE-A/<n>`, KOT number, `Paneer Tikka × 2`. Developer: order subtotal 500.00, tax % 5.00, 1 KOT, 1 line at 250.00. The kitchen (CHF-A) sees the KOT. |
| ORD-02-T02 | AC2 (ORD-D2 fixed) | After T01, go back to the grid; tap Table 1. | Table 1 `Ordered` (red); tapping opens the cart with this order. Developer: the order's `tableId` is Table 1. |
| ORD-02-T03 | AC3 | Add `Paneer Tikka` and `Chicken 65`; untick `Chicken 65`; send. | Toast `KOT generated successfully! 1 held item still in the cart.`; `Chicken 65` still in the cart. |
| ORD-02-T04 | AC4 | Untick all items; send. | Toast `Tick at least one item to send on this KOT.`; nothing sent. |
| ORD-02-T05 | AC5 — *Fails until ORD-D3 is fixed* | Takeaway → add an item → open the cart. Fill the customer block (`Ravi`, `9876543210`) → send. | The cart shows the customer block (today: not shown). Developer: `customerName` `Ravi`, `customerPhone` `9876543210` (today: NULL). |
| ORD-02-T06 | Numbering (changed 2026-11-16) | Send five Takeaway orders in a row today (the first of the day). | Order numbers `CODE-A/1` … `CODE-A/5` in order. |
| ORD-02-T07 | One order per table — *Fails until ORD-D4 is fixed* | DEV-1 and DEV-2 both on the Dine In grid. DEV-1: occupy Table 3, send a KOT. Before DEV-2's grid refreshes, DEV-2: tap Table 3 (still amber) and send a KOT. | DEV-2: toast `This table already has a running order`; nothing new; after refreshing, Table 3 opens DEV-1's order. (Today: a second, separate order on Table 3.) |
| ORD-02-T08 | AC7 — *Fails until ORD-D4 is fixed* | Slow / cut network right after tapping send so the reply is lost; restore it; send the same items again. | First: toast `No reply from the server. The KOT may already have reached the kitchen — check before sending it again.` Second: the KOT card shows the **same** order and KOT numbers; the kitchen has one KOT. (Today: two KOTs / orders.) |
| ORD-02-T09 | Alt: inactive item | Check the ordering screen. | `Old Dish` (inactive) not offered. |
| ORD-02-T10 | AC2 — Room Service (ORD-D2 fixed) | Room Service → `101` → `Ready to Occupy` → add `Chicken 65` → send. Back to the room grid. | Toast `KOT generated successfully!`; `101` is `Ordered`. Developer: the order's `roomId` is `101`. |
