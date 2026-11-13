# 02 — Move a KOT through its stages — tester cases

Use case: [02-move-kot-through-stages.md](../02-move-kot-through-stages.md). Screen text: [ui-ux.md §2](../ui-ux.md#kot-card-never-shown-today--kitch-d1). Fixtures: [README.md](./README.md#fixtures).

Chef device DEV-1 signed in as CHF-A. Because no KOT card is ever shown today (KITCH-D1), these cases confirm that no stage change can be made from the app.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| KITCH-02-T01 | AC1 (KITCH-D1) | A waiter sends a KOT. On DEV-1 check all four tabs. | No KOT card in any tab, so no `Accept`, `Reject`, `Mark Completed` or `Mark Ready` button anywhere. |
| KITCH-02-T02 | AC2 (KITCH-D1) | Ask a developer to move KOT-P to Preparing (KITCH-02-D02). On DEV-1 open `Preparing` and pull down. | `No Orders Here`. |
