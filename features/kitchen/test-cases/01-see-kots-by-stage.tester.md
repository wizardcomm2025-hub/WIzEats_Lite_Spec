# 01 — See KOTs by stage — tester cases

Use case: [01-see-kots-by-stage.md](../01-see-kots-by-stage.md). Screen text: [ui-ux.md §1–2](../ui-ux.md#2-kitchen-orders). Fixtures: [README.md](./README.md#fixtures).

Chef device DEV-1 signed in as CHF-A; waiter device DEV-2 signed in as WTR-A.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| KITCH-01-T01 | Chef shell | Sign in as CHF-A. Tap each footer button. | Opens **Kitchen Orders** (`Manage and track food preparation`) with the `New` tab selected. Footer buttons are greyed out; tapping does nothing. |
| KITCH-01-T02 | AC1 | REST-A has no KOTs. Look at the `New` tab. | `No Orders Here` · `No orders in this section. Pull down to refresh.` |
| KITCH-01-T03 | AC2 (KITCH-D1) | On DEV-2 send a Takeaway KOT (`Paneer Tikka` × 2, note `less spicy`). On DEV-1 wait 20 s on `New`. | Still `No Orders Here`. No error message. (The KOT is in the database — ask a developer to confirm if needed.) |
| KITCH-01-T04 | AC2 | After T03 pull down on `New`. | The refresh spinner shows and stops; still `No Orders Here`; no error message. |
| KITCH-01-T05 | AC2 / tabs | After T03 tap `Preparing`, `Completed`, `Ready`, then `New`. | Each tab change shows `Loading KOTs...` briefly, then `No Orders Here`. No tab ever shows a count badge. |
| KITCH-01-T06 | Drawer | Open the drawer. | Section `Kitchen` with `KOT` and `Dashboard` only (no owner setup menus). `KOT` returns to Kitchen Orders. |
| KITCH-01-T07 | AC2 / polling | After T03 put the app in the background for 1 minute, then reopen it. | Kitchen Orders, `No Orders Here`; no error toast or pop-up. |
