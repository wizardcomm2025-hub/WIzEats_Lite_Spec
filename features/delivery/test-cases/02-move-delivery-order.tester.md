# 02 — Move a delivery order on — tester cases

Use case: [02-move-delivery-order.md](../02-move-delivery-order.md). Screen text: [ui-ux.md §2](../ui-ux.md#2-delivery-card-never-shown-today--dlv-d1-shown-once-it-is-fixed). Fixtures: [README.md](./README.md#fixtures).

WTR-A on the board with DLV-ORD shown (01-T03). All cases need a card on screen, so they fail at the first step until DLV-D1 is fixed.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| DLV-02-T01 | AC1 — *Fails until DLV-D2 is fixed* | Tap `Accept`. | Spinner; other cards dimmed; toast `Order #… accepted`; the pill turns `Accepted` (blue); the button is now `Out For Delivery`. (Today: no card.) |
| DLV-02-T02 | AC3 — *Fails until DLV-D3 is fixed* | After T01 tap `Out For Delivery`. A developer checks WA-CATCH. | Toast `Order #… is out for delivery`; pill `Out for Delivery`; button `Complete Payment`. WA-CATCH shows the accepted and out-for-delivery texts sent to 9876543210. |
| DLV-02-T03 | AC4 — *Fails until DLV-D3 is fixed* | After T02 tap `Complete Payment`; pay the full amount in cash on Bill Payment; go back to the board. | Bill Payment opens for this order with its items and total; after paying, the success screen shows; the order is no longer on the board. A developer sees a bill and a payment for the order and the completed WhatsApp text. (Today: the KOT is set to Completed with no payment recorded.) |
