# 01 — See KOTs by stage — tester cases

Use case: [01-see-kots-by-stage.md](../01-see-kots-by-stage.md). Screen text: [ui-ux.md §1–2](../ui-ux.md#2-kitchen-orders). Fixtures: [README.md](./README.md#fixtures).

CHF-A signed in on DEV-1; a waiter on DEV-2 sends orders (ORD).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| KITCH-01-T01 | Chef shell | Sign in as CHF-A. Tap each footer button. | Opens **Kitchen Orders** (`Manage and track food preparation`) with the `New` tab selected. Footer buttons are greyed out; tapping does nothing. |
| KITCH-01-T02 | AC1 | REST-A has no KOTs. Look at the `New` tab. | `No Orders Here` · `No orders in this section. Pull down to refresh.` |
| KITCH-01-T03 | AC2 — *Fails until KITCH-D1 is fixed* | On DEV-2 send a Dine In KOT on Table 1 (`Paneer Tikka` × 2, note `less spicy`). On DEV-1 wait up to 20 s on `New`. | A card appears by itself: `KOT #<kot number>`, `Pending` pill, `Just now`, `Zone : Main Hall  Table 1`, `2  Paneer Tikka`, `📝 less spicy`, buttons `Reject` and `Accept`. (Today: still `No Orders Here`.) |
| KITCH-01-T04 | AC2 — *Fails until KITCH-D1 is fixed* | After T03 send a Takeaway KOT on DEV-2; on DEV-1 pull down on `New`. | Two cards, the newer first; the Takeaway card's place is `N/A`. (Today: `No Orders Here`.) |
| KITCH-01-T05 | AC2 / tabs | After T03 tap `Preparing`, `Completed`, `Ready`, then `New`. | Each tab change shows `Loading KOTs...` briefly; `Preparing`, `Completed`, `Ready` show `No Orders Here`; `New` shows T03's card again (target — today `No Orders Here` everywhere). No tab shows a count badge. |
| KITCH-01-T06 | Drawer | Open the drawer. | Section `Kitchen` with `KOT` and `Dashboard` only (no owner setup menus). `KOT` returns to Kitchen Orders. |
| KITCH-01-T07 | AC2 / polling — *Fails until KITCH-D1 is fixed* | Put the app in the background for 1 minute; meanwhile DEV-2 sends a KOT; reopen the app. | Kitchen Orders shows the new KOT within 15 s; no error toast or pop-up. (Today: `No Orders Here`.) |
