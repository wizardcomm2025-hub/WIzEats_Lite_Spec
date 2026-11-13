# 02 — Mark a KOT delivered — tester cases

Use case: [02-mark-kot-delivered.md](../02-mark-kot-delivered.md). Screen text: [ui-ux.md §2](../ui-ux.md#cards-never-shown-today--wait-d1). Fixtures: [README.md](./README.md#fixtures).

Signed in as WTR-A. Because no ready card is ever shown today (WAIT-D1), these cases confirm that delivery can't be confirmed from the app.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| WAIT-02-T01 | AC1 (WAIT-D1) | With KOT-R Ready (set by a developer), open Ready Orders; check both tabs. | No card, so no `Delivered to Customer` button anywhere. |
| WAIT-02-T02 | Alt (WAIT-D2) | Close the app fully, reopen it, open Ready Orders → `Delivered`. | `No Deliveries Yet` (the tab isn't loaded from the server). |
