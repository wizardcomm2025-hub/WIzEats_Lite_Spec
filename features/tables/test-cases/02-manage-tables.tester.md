# 02 — Manage tables in the app — tester cases

Use case: [02-manage-tables.md](../02-manage-tables.md). Screen text: [ui-ux.md §2](../ui-ux.md#2-table-management-app). Fixtures: [README.md](./README.md#fixtures).

Signed in as OWN-A; Table Management open (phone: menu → `Table Management`; tablet: Setup). A developer helps where a case needs an API call.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| TBL-02-T01 | AC1 (TBL-D1 fixed) | Look at the screen. | Header `TABLE MANAGEMENT` / `Rest A` / `Manage restaurant tables`; picker shows `Main Hall`; cards `Table 1`–`Table 4`, each `Capacity: 4`, switch on. |
| TBL-02-T02 | AC1 | Open the picker; choose `Patio`. | List `SELECT ZONE` with `Main Hall` and `Patio` (inactive zones included); `Patio` shows `P1` faded, switch off. |
| TBL-02-T03 | AC2 | `+`; `Table 5`, capacity `4`; `Save Table`. Go Home. | `Saving...`; sheet closes; toast `Table created`; `Table 5` in the list. Home's Main Hall shows `Table 5`. |
| TBL-02-T04 | AC3 | `+`; `table 1`, capacity `2`; `Save Table`. Then edit the name to `Table 6`. | `Table name already exists.` in red under the name; sheet stays open with the values. Typing clears the message; saving `Table 6` works. |
| TBL-02-T05 | AC4 | `+`; save empty. Then capacity `abc` (if the keypad allows), `0`. | `Table name is required`, `Capacity is required`; `Must be a number`; `Capacity must be at least 1`. Nothing created. |
| TBL-02-T06 | Alt: no zones | Signed in as an owner whose zones were all deleted (developer). | No picker, no `+`; `No Zones Found`. |
| TBL-02-T07 | AC5 | Tap `Table 2`'s switch; read the dialog; `Deactivate`. Open Home. Back on the screen, switch it on again. | Dialog `Deactivate table?` / `Are you sure you want to deactivate "Table 2"?`; toast `Table "Table 2" deactivated`; card faded. Home doesn't show `Table 2`. Switching on: `Table "Table 2" activated`; Home shows it again. |
| TBL-02-T08 | AC6 — *Fails until TBL-D2 is fixed* | Edit `Table 2`; name `Table 1`; `Update Table`. | Red `Table name already exists.` under the name; the sheet stays open; `Table 2` unchanged. (Today: saved, and Home shows two `Table 1`.) |
| TBL-02-T09 | Alt: inactive zone | Choose `Patio`; `+`; `P2`, capacity `2`; save. Open Home. | Toast `Table created`; Home shows nothing of Patio. |
| TBL-02-T10 | AC7 — *Fails until TBL-D4 is fixed* | A developer adds a second running order on `Table 1`. Reopen Table Management. | `Table 1` is listed once. (Today: twice.) |
| TBL-02-T11 | Main flow: edit | Edit `Table 3`; capacity `6`; `Update Table`. Go Home. | `Updating...`; sheet closes; toast `Table updated`; the card shows `Capacity: 6`. |
