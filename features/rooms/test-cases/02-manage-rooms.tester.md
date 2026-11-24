# 02 — Manage rooms — tester cases

Use case: [02-manage-rooms.md](../02-manage-rooms.md). Screen text: [ui-ux.md §2](../ui-ux.md#2-room-management). Fixtures: [README.md](./README.md#fixtures).

Signed in as OWN-A with RS-ON; Room Management open (menu → `Room Management`).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| ROOM-02-T01 | AC1 (ROOM-D1 fixed); `103` part *Fails until ROOM-D5 is fixed* | FLOOR-G with ROOMS-IMP; open the screen. | `ROOM MANAGEMENT` / `Rest A` / `Manage hotel rooms`; picker `Ground Floor`; cards `101`, `102` with `Capacity: 2`, and `103` faded with its switch off. (Today: `103` is not shown.) |
| ROOM-02-T02 | AC4 | No floors; open the screen. | No picker, no `+`; `No Floors Found`. |
| ROOM-02-T03 | AC2 (ROOM-D1 fixed) | `+`; `201`, capacity `2`; `Save Room`. Leave and reopen. | Sheet closes; `201` first in the list; still there after reopening. |
| ROOM-02-T04 | AC3; `0` part *Fails until ROOM-D3 is fixed* | `+`; save empty; then capacity `abc` (if the keypad allows); then `0`. | `Room name is required` and `Capacity is required`; `Must be a number`; `0`: `Capacity must be at least 1`, nothing sent. (Today `0` passes the form and the room is created with capacity 0.) |
| ROOM-02-T05 | AC1 | Pick `First Floor` in the picker; then `Ground Floor` again. | `Loading rooms...`; First Floor: `No Rooms Found`; Ground Floor: its rooms again. |
| ROOM-02-T06 | AC5 (ROOM-D1 fixed) | Edit `102`; capacity `4`; `Update Room`. | Sheet closes; the card shows `Capacity: 4`. |
| ROOM-02-T07 | AC6; reopen part *Fails until ROOM-D5 is fixed* | Switch `102` off; `Deactivate`. Leave the screen and come back. Switch `102` on; `Activate`. | Dialog `Deactivate room?`; toast `Room "102" deactivated`; card faded. After coming back `102` is still listed, faded; switching on gives `Room "102" activated`. (Today: after coming back `102` is gone and can't be switched on in the app.) |
| ROOM-02-T08 | AC7 — *Fails until ROOM-D3 is fixed*, *Fails until ROOM-D6 is fixed* | `+`; `101`, capacity `2`; `Save Room`. | Red `Room name already exists.` under the name; the sheet stays open; one `101`. (Today: the sheet closes and a second `101` is added.) |
