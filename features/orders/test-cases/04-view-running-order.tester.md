# 04 — View a running order — tester cases

Use case: [04-view-running-order.md](../04-view-running-order.md). Fixtures: [README.md](./README.md#fixtures).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| ORD-04-T01 | AC4 (ORD-D2 fixed) | Send two KOTs on Table 1 (03-T02); go to Home; come back and tap Table 1. | The cart shows the order number and both KOTs with their lines. |
| ORD-04-T02 | AC4 (ORD-D2 fixed) | Ask a developer to create a running order on Table 4 through the API (ORD-02-D02); refresh the grid; tap Table 4. | Table 4 is `Ordered`; tapping opens the cart with that order's KOT. |
| ORD-04-T03 | AC4 | From T02's cart, try `View Bill` / bill actions. | Record what opens — the bill flow is BILL; note any error text. |
