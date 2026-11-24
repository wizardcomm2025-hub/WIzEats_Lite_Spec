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
| TBL-01-T07 | Alt: add fails — *Fails until TBL-D5 is fixed* | `+`; `Garden`; airplane mode on; `Add Zone`. | The sheet stays open with `Garden`; red text `Something went wrong. Please try again.` under the field (or the server's message); `Garden` is not in the list. (Today: the sheet closes; no message.) |
| TBL-01-T08 | Alt: count after switch | Switch `Patio` on; then leave the screen and come back. | Right after the switch the card shows `Tables: 0`; after coming back, `Tables: 1`. |
| TBL-01-T09 | Alt: duplicate — *Fails until TBL-D2 is fixed*, *Fails until TBL-D5 is fixed* | Add `main hall`. | Red `Zone name already exists.` under the field; the sheet stays open; one `Main Hall` in the list. (Today: added; two `Main Hall` zones.) |
| TBL-01-T10 | Alt: rename to a used name — *Fails until TBL-D2 is fixed*, *Fails until TBL-D5 is fixed* | Edit `Patio`; change to `Main Hall`; `Save Changes`. | Red `Zone name already exists.` under the field; the sheet stays open; `Patio` keeps its name. (Today: the sheet closes and `Patio` is renamed.) |
