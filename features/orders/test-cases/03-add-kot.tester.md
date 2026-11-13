# 03 — Add a KOT to a running order — tester cases

Use case: [03-add-kot.md](../03-add-kot.md). Screen text: [ui-ux.md §3](../ui-ux.md#3-cart-and-kot). Fixtures: [README.md](./README.md#fixtures).

Signed in as WTR-A.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| ORD-03-T01 | AC1 | Takeaway: send a first KOT (`Paneer Tikka`). On the KOT card tap `Continue Order`; add `Chicken 65`; send. | Toast `Items added to KOT successfully!`; the KOT card lists both KOTs under the same order number. Kitchen sees two KOTs. |
| ORD-03-T02 | AC1 | Dine In Table 1: first KOT, then add a second KOT without leaving the cart flow. | Same order number on both. |
| ORD-03-T03 | AC2 | After T01, generate the bill (BILL) for that order; then try to send another KOT from the same cart. | Toast `Bill already generated, cannot add items to this order` (or the cart no longer offers sending — record which). |
| ORD-03-T04 | Re-open (ORD-D2) | After T02 leave to Home, come back to Table 1, add an item, send. | A **new** order number — not added to the first order. |
