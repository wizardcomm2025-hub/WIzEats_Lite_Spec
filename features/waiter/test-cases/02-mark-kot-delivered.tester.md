# 02 — Mark a KOT delivered — tester cases

Use case: [02-mark-kot-delivered.md](../02-mark-kot-delivered.md). Screen text: [ui-ux.md §2](../ui-ux.md#2-ready-orders). Fixtures: [README.md](./README.md#fixtures).

WTR-A on DEV-1 (phone) and DEV-2 (tablet); a KOT made Ready first (01-T03). All cases need a ready card on screen, so they fail at the first step until KITCH-D1 and WAIT-D1 are fixed.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| WAIT-02-T01 | AC1 — *Fails until WAIT-D1 is fixed* | On `Pickup` tap `Delivered to Customer` on the ready card. Open `Delivered`. | Spinner on the button; other buttons disabled; the card leaves `Pickup` (count drops) and shows in `Delivered` with a blue `DELIVERED` pill and no button. (Today: no card to tap.) |
| WAIT-02-T02 | AC4 — *Fails until WAIT-D2 is fixed* | After T01 close the app fully, reopen it, open Ready Orders → `Delivered`. | The delivered KOT is listed. (Today: `No Deliveries Yet`.) |
| WAIT-02-T03 | AC4 — *Fails until WAIT-D2 is fixed* | After T01 open Ready Orders → `Delivered` on DEV-2. | The KOT delivered on DEV-1 is listed. (Today: `No Deliveries Yet`.) |
