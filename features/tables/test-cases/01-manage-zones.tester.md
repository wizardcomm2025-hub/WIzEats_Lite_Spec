# 01 — Manage zones in the app — tester cases

Use case: [01-manage-zones.md](../01-manage-zones.md). Screen text: [ui-ux.md §1](../ui-ux.md#1-zone-management-app). Fixtures: [README.md](./README.md#fixtures).

Signed in as OWN-A; Zone Management open (phone: menu → `Zone Management`; tablet: Setup).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| TBL-01-T01 | AC1 | Look at the list. | Header `ZONE MANAGEMENT` / `Rest A` / `Manage restaurant seating areas`. `Main Hall` with `Tables: 4`; `Patio` faded with `Tables: 1`. |
| TBL-01-T02 | AC2 | `+`; type `Garden`; `Add Zone`. Then open Table Management and its zone picker. | Brief `Loading zones...`; `Garden` first in the list with `Tables: 0`. The picker lists `Garden`. |
| TBL-01-T03 | AC3 | Edit `Main Hall`; change to `Hall A`; `Save Changes`. Open Home. | List shows `Hall A` with `Tables: 4`; Home shows `Hall A` with its four tables. |
| TBL-01-T04 | AC4 | Switch `Main Hall` off; read the dialog; `Deactivate`. Open Home. | Dialog `Deactivate zone?` / `Are you sure you want to deactivate "Main Hall"?`. Toast `Zone "Main Hall" deactivated`; card faded. Home shows no Main Hall and none of its tables. |
| TBL-01-T05 | AC5 | After T04, a developer sets `Table 2` inactive on its own. Switch `Main Hall` on; `Activate`. Open Home. | Toast `Zone "Main Hall" activated`. Home shows all four tables, including `Table 2`. |
| TBL-01-T06 | AC6 | `+`; leave the name empty; `Add Zone`. | `Zone name is required`; the sheet stays open; nothing added. |
| TBL-01-T07 | Alt: silent failure | `+`; `Garden`; airplane mode on; `Add Zone`. | The sheet closes; no message; `Garden` is not in the list. |
| TBL-01-T08 | Alt: count after switch | Switch `Patio` on; then leave the screen and come back. | Right after the switch the card shows `Tables: 0`; after coming back, `Tables: 1`. |
| TBL-01-T09 | Alt: duplicate (accepted TBL-D2) | Add `Main Hall` again. | Added; two `Main Hall` zones in the list and in Table Management's picker. |
