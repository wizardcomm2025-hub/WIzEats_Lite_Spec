# 01 — Open the delivery board — tester cases

Use case: [01-open-delivery-board.md](../01-open-delivery-board.md). Screen text: [ui-ux.md §1](../ui-ux.md#1-delivery-orders). Fixtures: [README.md](./README.md#fixtures).

Signed in as WTR-A (sign out and in again before T01).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| DLV-01-T01 | AC1 (DLV-D1) | On Home tap `DELIVERY`. | `Delivery Orders` opens; a red toast `hex2bin(): Hexadecimal input must have an even length`. |
| DLV-01-T02 | AC1 | After T01 look at the screen. | Subtitle `Accept, reject and track home delivery`; `No Delivery Orders` · `No delivery orders in this section. Pull down to refresh.` |
| DLV-01-T03 | AC2 | Go back to Home and tap `DELIVERY` again. | Same empty screen; **no** toast. |
| DLV-01-T04 | AC2 | Pull down; then wait 30 s on the screen. | Spinner shows and stops; still empty; no toast. |
| DLV-01-T05 | Alt | Sign in as OWN-A on a tablet; tap `DELIVERY`. | Same as T01–T02; the rail keeps `Home` highlighted. |
