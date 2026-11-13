# 04 — View a running order — tester cases

Use case: [04-view-running-order.md](../04-view-running-order.md). Fixtures: [README.md](./README.md#fixtures).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| ORD-04-T01 | AC4 (accepted ORD-D2) | Send two KOTs on one Takeaway order (03-T01); look at the cart's order section. | The KOT card shows the KOTs sent in this visit; the cart does not load the order's earlier KOTs from the server (error toast `Failed to fetch order` may appear — record it). |
| ORD-04-T02 | AC4 | Ask a developer to create a running order linked to Table 4 through the API (ORD-02-D02); refresh the grid; tap Table 4. | Table 4 shows as having an order; tapping opens the cart, which cannot load it (`Failed to fetch order` or empty). |
| ORD-04-T03 | AC4 | From T02's cart, try `View Bill` / bill actions. | Record what opens — the bill flow depends on BILL; note any error text. |
