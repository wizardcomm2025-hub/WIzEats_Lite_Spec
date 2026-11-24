# 01 — Open the delivery board — tester cases

Use case: [01-open-delivery-board.md](../01-open-delivery-board.md). Screen text: [ui-ux.md §1](../ui-ux.md#1-delivery-orders). Fixtures: [README.md](./README.md#fixtures).

Signed in as WTR-A on DEV-1 unless stated; Delivery switched on.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| DLV-01-T01 | AC1 | REST-A has no delivery orders. On Home tap `DELIVERY`. | `Delivery Orders` opens; subtitle `Accept, reject and track home delivery`; `No Delivery Orders` · `No delivery orders in this section. Pull down to refresh.`; no toast. |
| DLV-01-T02 | AC1 | Pull down; then wait 30 s on the screen. | Spinner shows and stops; still empty; no toast. |
| DLV-01-T03 | AC2 — *Fails until DLV-D1 is fixed* | Create DLV-ORD (developer, or an online order through PUB). Wait up to 20 s on the board. | A card appears: `Order #<number>`, `Pending` (amber), `Ravi`, `9876543210`, `12 Park St`, the items, `Order total` `₹<total>`, buttons `Reject` and `Accept`. (Today: `No Delivery Orders`.) |
| DLV-01-T04 | AC2 — *Fails until DLV-D1 is fixed* | A second delivery order arrives; pull down. | Two cards, the newer first; subtitle `2 orders in this list`. |
| DLV-01-T05 | Alt | Sign in as OWN-A on a tablet; tap `DELIVERY`. Then switch Delivery off in Profile and go back to Home. | The board opens; the rail keeps `Home` highlighted. After switching Delivery off, Home has no `DELIVERY` tile. |
| DLV-01-T06 | DLV-D4 (accepted) | Look for a way to start a new delivery order on the board. | None — the board only lists orders. |
