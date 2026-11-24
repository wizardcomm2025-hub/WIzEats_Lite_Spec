# 02 — Move a KOT through its stages — tester cases

Use case: [02-move-kot-through-stages.md](../02-move-kot-through-stages.md). Screen text: [ui-ux.md §2](../ui-ux.md#kot-card-never-shown-today--kitch-d1-shown-once-it-is-fixed). Fixtures: [README.md](./README.md#fixtures).

CHF-A on DEV-1; a waiter on DEV-2 sends a KOT first (ORD). All cases need a KOT card on screen, so they fail at the first step until KITCH-D1 is fixed.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| KITCH-02-T01 | AC1, AC5 — *Fails until KITCH-D1 is fixed* | On `New`, tap `Accept` on the KOT. Open `Preparing`. | Spinner on `Accept`; the other cards dim; the card leaves `New` and shows in `Preparing` with `Mark Completed`. (Today: no card to tap.) |
| KITCH-02-T02 | AC2, AC5 — *Fails until KITCH-D1 is fixed* | In `Preparing` tap `Mark Completed`; open `Completed`; tap `Mark Ready`; open `Ready`. On DEV-2 (waiter) open Ready Orders (WAIT). | The card moves to `Completed`, then `Ready` with no button; the waiter sees it ready for pick-up. |
| KITCH-02-T03 | AC4 — *Fails until KITCH-D1 is fixed* | On a new KOT in `New` tap `Reject`. Look at all four tabs. | The card leaves `New` and is in no tab (there is no Rejected tab). The waiter never sees it as ready. |
