# 01 — Manage floors — tester cases

Use case: [01-manage-floors.md](../01-manage-floors.md). Screen text: [ui-ux.md §1](../ui-ux.md#1-floor-management). Fixtures: [README.md](./README.md#fixtures).

Signed in as OWN-A. Unless stated, RS-ON is set and Floor Management is open (menu → `Floor Management`).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| ROOM-01-T01 | AC1 | Panel: switch Room Service off; app refresh; open the menu. Then switch it on; refresh; open the menu. | Off: no `Floor Management` / `Room Management`. On: both shown. |
| ROOM-01-T02 | AC2 | No floors exist; open Floor Management. | `FLOOR MANAGEMENT` / `Rest A` / `Manage hotel floors`; `No Floors Found` / `Start by adding your first floor to organize your restaurant layout.` |
| ROOM-01-T03 | AC3 | `＋`; `Ground Floor`; `Add Floor`. | Sheet closes; `Ground Floor` first, `Rooms: 0`. |
| ROOM-01-T04 | AC4 | `＋`; leave empty; `Add Floor`. | `Floor name is required`; nothing added. |
| ROOM-01-T05 | AC5 (accepted ROOM-D1) | Edit `Ground Floor`; `Level 0`; `Save Changes`. | Sheet closes; no message; list still shows `Ground Floor`. |
| ROOM-01-T06 | AC6 (accepted ROOM-D1) | Switch `Ground Floor` off; `Deactivate`. | Dialog `Deactivate floor?` / `Are you sure you want to deactivate "Ground Floor"?`; then toast `Failed to save floor`; still active. |
| ROOM-01-T07 | AC7 | FLOOR-G with ROOMS-IMP; open the screen. | `Ground Floor` shows `Rooms: 3` (inactive room `103` counted). |
| ROOM-01-T08 | Alt: silent failure | Airplane mode on; `＋`; `Upper Floor`; `Add Floor`. | Sheet closes; no message; not added. |
| ROOM-01-T09 | Alt: duplicate (accepted ROOM-D3) | Add `Ground Floor` again. | Added; two `Ground Floor` cards. |
