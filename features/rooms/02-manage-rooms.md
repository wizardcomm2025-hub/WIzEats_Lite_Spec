# 02 — Manage rooms

## Summary

The owner opens Room Management, picks a floor, and would see that floor's rooms there and add, edit or switch them. Today the list is always empty and adding a room always fails (ROOM-D1). Rooms that exist (created by Import) are used by Room Service orders but can't be managed in the app.

## Prerequisites

- **Backend:**
  - `POST /api/rooms/getRoomsDetails`
  - `POST /api/rooms/roomSave`
  - `POST /api/rooms/getFloorDetails`

  See [api-contract.md](./api-contract.md).
- **Data:** `rooms`, `floors`, `orders`
- **Depends on:** [01](./01-manage-floors.md); IMPEX (the only working way to create rooms)

## Actors & preconditions

- **Actor:** owner. Through the API, any signed-in user (ROOM-D2).
- **Trigger:** menu → `Room Management` (only while Room Service is on).
- **Preconditions:** at least one floor.

## Main flow (as it is today)

1. The header shows `ROOM MANAGEMENT` / restaurant name / `Manage hotel rooms`, a dark floor picker (the first floor is selected) and a green `+`. While loading it shows `Loading rooms...`.
2. The app posts `{ floorId: <selected floor> }` to `getRoomsDetails`. The server compares the hex ID with the binary column and returns `[]` (ROOM-D1). The app's `Number(floorId)` filter would also drop every room. The screen shows `No Rooms Found` / `Create rooms to better organize your dining areas and service zones.`
3. Picking another floor reloads with the same result.
4. **Add:** `+` opens `Add Room` with the fields `Room Name` and `Capacity` and the button `Save Room`. The app checks `Room name is required`, `Capacity is required` and `Must be a number`; `0` is allowed. The sheet closes at once. The app sends `{ roomName, capacity, floorId }`. The server answers 500 `Failed to save room` (ROOM-D1), and no message is shown.
5. **Edit / switch:** unreachable, since no room card is ever shown. Through the API both answer 500 (ROOM-D1).

## Alternative flows

- **No floors.** No picker and no `+`; `No Floors Found`.
- **API with a short made-up `floorId`** (≤ 16 characters). The room is created as an orphan (ROOM-D3), and appears in `getRoomsDetails` without a filter.
- **Inactive rooms.** They are never returned by `getRoomsDetails` (ROOM-D3).

## Acceptance criteria

1. Given Ground Floor with imported rooms `101`, `102`, then Room Management shows `No Rooms Found` (ROOM-D1).
2. Given `201`, capacity `2`, is saved, then the sheet closes, no message shows, and no room is created (ROOM-D1).
3. Given an empty name or capacity, or `abc`, then the matching message shows and nothing is sent.
4. Given no floors, then `No Floors Found` shows, with no picker and no `+`.
5. Given the API lists rooms with `{}`, then the active rooms `101` and `102` are returned with `status_key`.

## Platform notes

### POS API

- `RoomController::roomSave` / `getroomsDetails` → `RoomService::saveRoom` / `getrooms`.
- The create rules: `roomName` required string; `floorId` required string; `capacity` required integer; `status` in 0, 1, 2.

### Data — MySQL

- `rooms` insert. A 32-character `floorId` is too long for `binary(16)` (strict mode), so the insert fails.
- The list joins running orders, one row per order.

### POS app

`RoomConfigScreen`. Its filter is `Number(r.floorId) === selectedFloor.floorId`, the same fault as TBL-D1.

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- Picking a room for a Room Service order — ORD.

## Open questions

- None open. Related decisions: ROOM-D1, ROOM-D3.
