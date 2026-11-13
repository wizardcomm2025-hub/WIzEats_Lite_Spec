# 02 — Manage tables in the app — tester cases

Use case: [02-manage-tables.md](../02-manage-tables.md). Screen text: [ui-ux.md §2](../ui-ux.md#2-table-management-app). Fixtures: [README.md](./README.md#fixtures).

Signed in as OWN-A; Table Management open (phone: menu → `Table Management`; tablet: Setup). A developer helps where a case needs an API call.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| TBL-02-T01 | AC1 (accepted TBL-D1) | Look at the screen. | Header `TABLE MANAGEMENT` / `Rest A` / `Manage restaurant tables`; picker shows `Main Hall`; body `No Tables Found` / `Set up tables to start taking orders and managing your dining service.` — although Main Hall has four tables. |
| TBL-02-T02 | AC1 | Open the picker; choose `Patio`. | List `SELECT ZONE` with `Main Hall` and `Patio` (inactive zones included); `Patio` shows the same empty state. |
| TBL-02-T03 | AC2 | `+`; `Table 5`, capacity `4`; `Save Table`. Go Home. | `Saving...`; sheet closes; toast `Table created`; list still empty. Home's Main Hall shows `Table 5`. |
| TBL-02-T04 | AC3 | `+`; `table 1`, capacity `2`; `Save Table`. Then edit the name to `Table 6`. | `Table name already exists.` in red under the name; sheet stays open with the values. Typing clears the message; saving `Table 6` works. |
| TBL-02-T05 | AC4 | `+`; save empty. Then capacity `abc` (if the keypad allows), `0`. | `Table name is required`, `Capacity is required`; `Must be a number`; `Capacity must be at least 1`. Nothing created. |
| TBL-02-T06 | Alt: no zones | Signed in as an owner whose zones were all deleted (developer). | No picker, no `+`; `No Zones Found`. |
| TBL-02-T07 | AC5 | Developer switches `Table 2` off through the API. Open Home. | `Table 2` not shown. |
| TBL-02-T08 | AC6 (accepted TBL-D2) | Developer renames `Table 2` to `Table 1` through the API. Open Home. | Main Hall shows two `Table 1`. |
| TBL-02-T09 | Alt: inactive zone | Choose `Patio`; `+`; `P2`, capacity `2`; save. Open Home. | Toast `Table created`; Home shows nothing of Patio. |
