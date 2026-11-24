# 03 — Zones and tables in the panel — tester cases

Use case: [03-zones-tables-in-panel.md](../03-zones-tables-in-panel.md). Screen text: [ui-ux.md §3](../ui-ux.md#3-panel-pages). Fixtures: [README.md](./README.md#fixtures).

Desktop browser, signed in to the panel as ADMIN-1, on REST-A's zones or tables page. A second tester holds the app as OWN-A where needed.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| TBL-03-T01 | AC1 | Open the zones page. | Tab `Analytics`; `Category Overview` / `Category management`; rows `Main Hall` (switch on) and `Patio` (off). |
| TBL-03-T02 | AC2 | `+ Add Zone`; `Terrace`, `Active`; `Save`. App: reopen Zone Management. | Page reloads with `Terrace`; the app lists `Terrace` with `Tables: 0`. |
| TBL-03-T03 | AC3 | Turn `Main Hall`'s switch off; reload. App: Home; Zone Management. | Switch stays off after reload. Home hides Main Hall. (A developer confirms the four tables are still active in the database.) |
| TBL-03-T04 | AC4 | Open the tables page. | `Tables Overview` / `Tables management`; `Table 1`–`4` with `Main Hall`, and `P1` with `Patio`. |
| TBL-03-T05 | AC5 | `+ Add Table`; Zone `Main Hall`, `Table 9`, capacity `2`, `Active`; `Save`. App: Home. | Alert `Table Added Successfully`; reload shows `Table 9`; Home shows it. |
| TBL-03-T06 | AC6 | Edit `Table 2`; set Zone `Terrace` (added in T02); `Update`. App: Home. | Alert `Updated Successfully`; row shows `Terrace`; Home shows `Table 2` under Terrace. |
| TBL-03-T07 | AC7 | Click `Table 3`'s Visible switch off; reload. | Switch back on; Home still shows `Table 3`. |
| TBL-03-T08 | Alt: empty | `+ Add Table` with an empty name; `Save`. `+ Add Zone` with an empty name; `Save`. | No alert, no reload; nothing added. |
| TBL-03-T09 | Alt: duplicate — *Fails until TBL-D2 is fixed* | `+ Add Table`; Zone `Main Hall`, `Table 1`; `Save`. | No alert, no reload; one `Table 1`. (Today: `Table Added Successfully`; two `Table 1` rows; Home shows both.) |
