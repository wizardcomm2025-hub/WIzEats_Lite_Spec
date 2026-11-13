# 02 — Manage rooms — tester cases

Use case: [02-manage-rooms.md](../02-manage-rooms.md). Screen text: [ui-ux.md §2](../ui-ux.md#2-room-management). Fixtures: [README.md](./README.md#fixtures).

Signed in as OWN-A with RS-ON; Room Management open (menu → `Room Management`).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| ROOM-02-T01 | AC1 (accepted ROOM-D1) | FLOOR-G with ROOMS-IMP; open the screen. | `ROOM MANAGEMENT` / `Rest A` / `Manage hotel rooms`; picker `Ground Floor`; `No Rooms Found` / `Create rooms to better organize your dining areas and service zones.` — although rooms `101`, `102` exist. |
| ROOM-02-T02 | AC4 | No floors; open the screen. | No picker, no `+`; `No Floors Found`. |
| ROOM-02-T03 | AC2 (accepted ROOM-D1) | `+`; `201`, capacity `2`; `Save Room`. Leave and reopen. | Sheet closes; no message; still `No Rooms Found`. A developer confirms no row was created. |
| ROOM-02-T04 | AC3 | `+`; save empty; then capacity `abc` (if the keypad allows); then `0`. | `Room name is required` and `Capacity is required`; `Must be a number`; `0` passes the form (and then fails on the server like T03). |
| ROOM-02-T05 | AC1 | Add a second floor; pick it in the picker; pick `Ground Floor` again. | `Loading rooms...` then `No Rooms Found` each time. |
