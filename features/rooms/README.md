# Floors & rooms

For restaurants that also run rooms, such as a hotel kitchen, the owner sets up **floors** and the **rooms** on each floor, so that Room Service orders can be taken for a room. The two screens appear in the app's menu only while the restaurant's Room Service setting is on (RST). There is no panel page for floors or rooms.

The spec records today's behaviour. Decisions ROOM-D1 to ROOM-D3 keep the current rules as accepted risks. **Today only "add a floor" and "list floors" work from the app (ROOM-D1).**

**Feature code:** `ROOM` (test IDs `ROOM-01-T01`, `ROOM-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Manage floors](./01-manage-floors.md) | Owner (any signed-in user through the API — ROOM-D2) | [tester](./test-cases/01-manage-floors.tester.md) | [developer](./test-cases/01-manage-floors.developer.md) |
| 02 | [Manage rooms](./02-manage-rooms.md) | Owner (any signed-in user through the API — ROOM-D2) | [tester](./test-cases/02-manage-rooms.tester.md) | [developer](./test-cases/02-manage-rooms.developer.md) |

## Key concepts

- **Floor** — `floors` row: name, active, deleted.
- **Room** — `rooms` row on a floor: name, capacity, active, status (0 free · 1 occupied · 2 ordered, set by Room Service orders), deleted.
- **Room count** — the number on a floor card. It counts **every** room row of the floor, including inactive and deleted ones.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api-dev_guid/specs/rooms/` (created with the first plan)

## Depends on

- **RST** — the Room Service switch (panel only) shows or hides the Floor and Room Management menus.
- **IMPEX** — importing is today the only working way to create rooms.
- **ORD** — Room Service orders pick a room and set its status.

## Decisions

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
- **Inherited:** AUTH-D2.

## Facts worth knowing

- The save messages are lower case: `floor created successfully`, `room created successfully`.
- A floor's `createdBy` is set to the caller's role. A room's `createdBy` stays empty.
- After adding a floor, its card shows `Rooms: 0` (the reply has no count) until the list is loaded again.
- In the app, a failed rename, a failed room add or a failed room edit shows no message. A failed switch shows a toast with the server message.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Taking a Room Service order — ORD.
- Importing / exporting floors and rooms — IMPEX.
- The Room Service setting itself — RST.
