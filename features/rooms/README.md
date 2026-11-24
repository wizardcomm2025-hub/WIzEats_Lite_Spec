# Floors & rooms

For restaurants that also run rooms, such as a hotel kitchen, the owner sets up **floors** and the **rooms** on each floor, so that Room Service orders can be taken for a room. The two screens appear in the app's menu only while the restaurant's Room Service setting is on (RST). There is no panel page for floors or rooms.

The spec records today's behaviour plus targets (re-synced 2026-11-24 against API commit d54d6c8 and `POS_APP-kot`). The GUID handling was fixed on 2026-11-16 (ROOM-D1), so floors and rooms can now be added, renamed and switched from the app. ROOM-D2 to ROOM-D6 are *To fix*.

**Feature code:** `ROOM` (test IDs `ROOM-01-T01`, `ROOM-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Manage floors](./01-manage-floors.md) | Owner (today any signed-in user through the API — ROOM-D2) | [tester](./test-cases/01-manage-floors.tester.md) | [developer](./test-cases/01-manage-floors.developer.md) |
| 02 | [Manage rooms](./02-manage-rooms.md) | Owner (today any signed-in user through the API — ROOM-D2) | [tester](./test-cases/02-manage-rooms.tester.md) | [developer](./test-cases/02-manage-rooms.developer.md) |

## Key concepts

- **Floor** — `floors` row: name, active, deleted.
- **Room** — `rooms` row on a floor: name, capacity, active, status (0 free · 1 occupied · 2 ordered, set by Room Service orders), deleted.
- **Room count** — the number on a floor card. It counts **every** room row of the floor, including inactive and deleted ones.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api/api/specs/rooms/` (created with the first plan)

## Depends on

- **RST** — the Room Service switch (panel only) shows or hides the Floor and Room Management menus.
- **IMPEX** — importing floors and rooms (since 2026-11-16 rooms can also be added in the app).
- **ORD** — the Room Service order screen (owner and waiter) lists floors and active rooms and sets a room's `status` through `roomSave` (occupy `1`, leave `0`).

## Decisions

Statuses per [conventions.md → Decision status](../../conventions.md#decision-status), set 2026-11-24. Cross-cutting items: [project-overview.md → Known gaps and targets](../../project-overview.md#known-gaps-and-targets).

| ID | Status | Short | Tests |
|---|---|---|---|
| ROOM-D1 | **Fixed in code 2026-11-16** (and in the app) | Hex IDs not converted; app filter by `Number()` | 01-D03, 01-D04, 02-D02, 02-D04, 02-D06; 01-T05, 01-T06, 02-T01, 02-T03, 02-T06, 02-T07 |
| ROOM-D2 | **To fix** (X-D1) | Any role may change floors and rooms | 01-D08, 02-D10 |
| ROOM-D3 | **To fix** (with X-D4, X-D8) | No integrity checks | 01-D06, 01-D14, 01-D15, 01-D16; 02-D05, 02-D08, 02-D09, 02-D11, 02-D17; 01-T09, 02-T04, 02-T08 |
| ROOM-D4 | **To fix** (new 2026-11-24) | Leftover list / ID faults | 01-D09, 02-D03, 02-D14, 02-D16 |
| ROOM-D5 | **To fix** (new 2026-11-24) | Switched-off rooms vanish from Room Management | 02-D19; 02-T07 |
| ROOM-D6 | **To fix** (new 2026-11-24) | Floor / Room sheet failures are silent | 01-T08, 01-T09, 02-T08 |
| AUTH-D2 (inherited) | **To fix** (X-D2) | Switched-off / unapproved callers keep access | 01-D10, 02-D12 |

- **ROOM-D1 — Fixed in code 2026-11-16** (commit `0642e64`), and in the app. `RoomService` converts `floorId` / `roomId` from hex (dashes removed) on floor update, room create, room update lookup and the room list's filters; the app's Room Management compares IDs as strings and asks the server for the selected floor's rooms. Floor rename / switch (cascading to the floor's rooms), room add, edit and switch now work. The leftovers are ROOM-D4.
- **ROOM-D2 — To fix (2026-11-24, was accepted 2026-11-10), via [X-D1](../../project-overview.md#known-gaps-and-targets).** *Target:* `getFloorDetails` and `getRoomsDetails` — owner and waiter (the Room Service order screen); `floorSave` — owner; `roomSave` — owner with any field, waiter **only** `{ roomId, status }` (any other field present → 403); chef — 403 on all four. 403 body `{ "status": false, "message": "You do not have permission to perform this action", "data": null }`. `includeInactive` (ROOM-D5) is owner-only. The Room Service setting is still **not** checked by the API (accepted).
- **ROOM-D3 — To fix (2026-11-24, was accepted 2026-11-10), aligned with TBL-D2.** *Target:* `floorId` on room create / update must be a non-deleted floor of the caller's restaurant → else 422 `Invalid floor` (X-D8, also for a malformed ID); floor names unique per restaurant among non-deleted floors (any case, trimmed) → 422 `Floor name already exists.`; room names unique per floor (same rules) on create and on any update that changes the name or floor → 422 `Room name already exists.`; `capacity` `integer|min:1|max:2147483647` (the app's form too: `Capacity must be at least 1`); `floorName` / `roomName` `max:255`; deleting a floor (`isDelete: true`) also deletes its rooms; updating an unknown, deleted, other restaurant's or malformed ID → 404 `Floor not found` / `Room not found` (X-D4).
- **ROOM-D4 — To fix (new 2026-11-24).** *Today:* `getRoomsDetails` with `roomId` → 500 (the column is ambiguous with `orders.roomId`); `getFloorDetails` with `floorId` compares hex with binary → `[]`; moving a room (`roomSave` update with `floorId`) stores the hex unconverted → 500; a room with two running orders is listed twice. *Target:* both filters take 32-char hex (dashes allowed) and return the match, other values → 422 `Validation error`; `floorId` converted on update (and checked — ROOM-D3); one row per room, `orderId` = the highest running order.
- **ROOM-D5 — To fix (new 2026-11-24).** *Today:* `getRoomsDetails` returns active rooms only, so a room switched off in Room Management disappears at the next load and can't be switched back on from the app. *Target:* `getRoomsDetails` accepts `includeInactive: true` (owner only; others → 403) and then also returns inactive, non-deleted rooms. Room Management sends it and shows inactive rooms faded with their switch off; the Room Service order screen does not send it, so its grid stays active-only.
- **ROOM-D6 — To fix (new 2026-11-24), same as TBL-D5.** *Today:* the Add / Edit Floor and Add / Edit Room sheets close before the server answers, so a failed save shows nothing. *Target:* the sheet stays open while saving, closes only when the server accepts, and on failure shows the server message (e.g. `Floor name already exists.`, `Room name already exists.`) or `Something went wrong. Please try again.` in red under the name field, values kept.
- **Inherited:** AUTH-D2 (To fix, X-D2).

Original decision texts (history):

- **ROOM-D1 (2026-11-10):** The GUID handling stays broken. `RoomService` never converts hex IDs to binary, and the app filters rooms with `Number(floorId)`. As a result:
  - renaming or switching a floor answers 500 `Failed to save floor`;
  - adding a room with a real floor ID answers 500 `Failed to save room` (the 32-character hex is too long for the `binary(16)` column);
  - editing or switching a room answers 500;
  - the room list is filtered by floor, which never matches, so it is always empty;
  - filtering the room list by `roomId` gives a 500;
  - a room with two running orders appears twice.

  Only adding a floor, listing floors, and listing rooms without a filter (API) work. Accepted.
- **ROOM-D2 (2026-11-10):** No role check. Only the owner sees the screens, but any signed-in user of the restaurant can call the API. Today that means adding floors, and adding rooms with a made-up floor ID. The restaurant's Room Service setting isn't checked either. Accepted risk (same as TBL-D3).
- **ROOM-D3 (2026-11-10):** No integrity checks:
  - a room can be added with any `floorId` of up to 16 characters, which saves an orphan room;
  - capacity can be any integer (the app allows 0);
  - names over 255 characters give a 500;
  - floor and room names may repeat;
  - deleting or switching off a floor would leave or cascade to its rooms as in TBL, but updates don't work today;
  - inactive rooms are never listed, so they can't be switched back on.

  Accepted risk.
- **Inherited (2026-11-10):** AUTH-D2.

## Facts worth knowing

- The save messages are lower case: `floor created successfully`, `floor updated successfully`, `room created successfully`, `room updated successfully`.
- A floor's `createdBy` is set to the caller's role. A room's `createdBy` stays empty.
- After adding or switching a floor, its card shows `Rooms: 0` (the reply has no count) until the list is loaded again.
- Switching a floor through the API sets `isActive` on **all** its rooms (including deleted ones); switching it back on re-activates rooms that were off on their own.
- In the app, a failed floor add or rename, or a failed room add or edit, shows no message (ROOM-D6). A failed switch shows a toast with the server message.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Taking a Room Service order — ORD.
- Importing / exporting floors and rooms — IMPEX.
- The Room Service setting itself — RST.
