# 02 — Move a delivery order on — tester cases

Use case: [02-move-delivery-order.md](../02-move-delivery-order.md). Screen text: [ui-ux.md §2](../ui-ux.md#2-delivery-card-never-shown-today--dlv-d1). Fixtures: [README.md](./README.md#fixtures).

Signed in as WTR-A. Because no delivery card is ever shown today (DLV-D1), this case confirms that no stage can be changed from the app.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| DLV-02-T01 | AC1 (DLV-D1/D2) | Open Delivery Orders; pull down. | No card, so no `Accept`, `Reject`, `Out For Delivery` or `Complete Payment` button. |
