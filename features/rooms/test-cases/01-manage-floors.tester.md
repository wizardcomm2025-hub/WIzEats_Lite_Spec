# 01 — Manage floors — tester cases

Use case: [01-manage-floors.md](../01-manage-floors.md). Screen text: [ui-ux.md §1](../ui-ux.md#1-floor-management). Fixtures: [README.md](./README.md#fixtures).

Signed in as OWN-A. Unless stated, RS-ON is set and Floor Management is open (menu → `Floor Management`).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| ROOM-01-T01 | AC1 | Panel: switch Room Service off; app refresh; open the menu. Then switch it on; refresh; open the menu. | Off: no `Floor Management` / `Room Management`. On: both shown. |
| ROOM-01-T02 | AC2 | No floors exist; open Floor Management. | `FLOOR MANAGEMENT` / `Rest A` / `Manage hotel floors`; `No Floors Found` / `Start by adding your first floor to organize your restaurant layout.` |
| ROOM-01-T03 | AC3 | `＋`; `Upper Floor`; `Add Floor`. | Sheet closes; `Upper Floor` first, `Rooms: 0`. |
| ROOM-01-T04 | AC4 | `＋`; leave empty; `Add Floor`. | `Floor name is required`; nothing added. |
| ROOM-01-T05 | AC5 (ROOM-D1 fixed) | Edit `Ground Floor`; `Level 0`; `Save Changes`. | Sheet closes; the list shows `Level 0`. |
| ROOM-01-T06 | AC6 (ROOM-D1 fixed) | Switch `Ground Floor` off; `Deactivate`. Open Room Management on Ground Floor. Back here, switch it on; `Activate`. | Dialog `Deactivate floor?` / `Are you sure you want to deactivate "Ground Floor"?`; toast `Floor "Ground Floor" deactivated`; card faded. Room Management shows no active rooms on Ground Floor. Switching on: `Floor "Ground Floor" activated`; `101`, `102` and `103` are all active again. |
| ROOM-01-T07 | AC7 | FLOOR-G with ROOMS-IMP; open the screen. | `Ground Floor` shows `Rooms: 3` (inactive room `103` counted). |
| ROOM-01-T08 | Alt: add fails — *Fails until ROOM-D6 is fixed* | Airplane mode on; `＋`; `Upper Floor`; `Add Floor`. | The sheet stays open with `Upper Floor`; red `Something went wrong. Please try again.` (or the server's message) under the field; not added. (Today: the sheet closes; no message.) |
| ROOM-01-T09 | AC8 — *Fails until ROOM-D3 is fixed*, *Fails until ROOM-D6 is fixed* | Add `ground floor`. | Red `Floor name already exists.` under the field; the sheet stays open; one Ground Floor. (Today: added; two Ground Floor cards.) |
