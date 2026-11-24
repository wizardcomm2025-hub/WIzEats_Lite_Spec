# 02 — Manage rooms

## Summary

The owner opens Room Management, picks a floor, and sees that floor's rooms. They can add a room, edit its name and capacity, or switch it on or off. All of this works since 2026-11-16 (ROOM-D1 fixed in the API and the app). Today a switched-off room disappears at the next load (ROOM-D5), and the server doesn't check the floor, duplicate names or capacity (ROOM-D3).

## Prerequisites

- **Backend:**
  - `POST /api/rooms/getRoomsDetails`
  - `POST /api/rooms/roomSave`
  - `POST /api/rooms/getFloorDetails`

  See [api-contract.md](./api-contract.md).
- **Data:** `rooms`, `floors`, `orders`
- **Depends on:** [01](./01-manage-floors.md); IMPEX (importing rooms); ORD (the Room Service order screen uses the same rooms)

## Actors & preconditions

- **Actor:** owner. Today any signed-in user through the API; target per [api-contract.md](./api-contract.md#endpoints) (ROOM-D2).
- **Trigger:** menu → `Room Management` (only while Room Service is on).
- **Preconditions:** at least one floor.

## Main flow

1. The header shows `ROOM MANAGEMENT` / restaurant name / `Manage hotel rooms`, a dark floor picker (the first floor is selected) and a green `+`. While loading it shows `Loading rooms...`.
2. The app posts `{ floorId: <selected floor> }` to `getRoomsDetails` (target: also `includeInactive: true` — ROOM-D5) and shows that floor's rooms. Each card shows the room name, `Capacity: <n>`, an edit button and a switch.
3. Picking another floor reloads with that floor's rooms, or `No Rooms Found` / `Create rooms to better organize your dining areas and service zones.`
4. **Add:** `+` opens `Add Room` with the fields `Room Name` and `Capacity` and the button `Save Room`. The app checks `Room name is required`, `Capacity is required` and `Must be a number` (target also `Capacity must be at least 1` — ROOM-D3). The sheet closes (today at once; target only once saved — ROOM-D6). The app sends `{ roomName, capacity, floorId }`; the server creates the room (active, status 0) and the room appears first in the list.
5. **Edit:** the edit button opens `Edit Room` with the values filled in and `Update Room`. The app sends `{ roomId, roomName, capacity }`; the card updates.
6. **Switch:** the dialog `Deactivate room?` / `Are you sure you want to deactivate "<name>"?` (or `Activate room?`). On confirm the app sends `{ roomId, isActive }`; the toast reads `Room "<name>" deactivated` / `Room "<name>" activated` and the card fades. Today the room is gone after the next load and can't be switched back on in the app; *target (ROOM-D5)* it stays listed, faded, with its switch off.

## Alternative flows

- **No floors.** No picker and no `+`; `No Floors Found`.
- **Add or edit fails.** Today no message (the sheet has closed). *Target (ROOM-D6):* the sheet stays open with the server message (e.g. `Room name already exists.`) or `Something went wrong. Please try again.` in red.
- **Capacity `0`.** Today the form accepts it and the server saves it. *Target:* refused by the form and the server.
- **Duplicate room name on a floor.** Today saved. *Target:* 422 `Room name already exists.`
- **API with a floor ID that isn't the restaurant's.** Today an orphan room is created if the ID is valid hex, or 500 if it isn't. *Target:* 422 `Invalid floor`.
- **Moving a room to another floor (API).** Today 500. *Target:* saved (ROOM-D4), if the floor is valid.
- **A room with two running orders.** Today listed twice; *target* once (ROOM-D4).
- **Room Service order screen.** Owners and waiters occupy / free rooms through `roomSave` `{ roomId, status }` — specified in ORD. In the target a waiter may send only those two fields.

## Acceptance criteria

1. Given Ground Floor with rooms `101`, `102` (active) and `103` (inactive), then Room Management shows `101` and `102` (ROOM-D1 fixed); *(target, ROOM-D5)* `103` too, faded.
2. Given `201`, capacity `2`, is saved, then the sheet closes and `201` appears first in the list.
3. Given an empty name or capacity, or `abc`, then the matching message shows and nothing is sent. *(Target, ROOM-D3)* `0` too.
4. Given no floors, then `No Floors Found` shows, with no picker and no `+`.
5. Given `102` is edited to capacity `4`, then the card shows `Capacity: 4`.
6. Given `102` is switched off and confirmed, then the toast `Room "102" deactivated` shows and the card fades; *(target, ROOM-D5)* after reopening it is still listed and can be switched back on.
7. *(Target, ROOM-D3 / ROOM-D6)* Given `101` exists on Ground Floor, when `101` is added there again, then `Room name already exists.` shows and nothing is added.

## Platform notes

### POS API

- `RoomController::roomSave` / `getroomsDetails` → `RoomService::saveRoom` / `getrooms`.
- The create rules: `roomName` required string; `floorId` required string (converted from hex); `capacity` required integer; `status` in 0, 1, 2.

### Data — MySQL

- `rooms` insert or update.
- The list joins running orders, one row per order today (ROOM-D4).

### POS app

- `RoomConfigScreen` (`roomScreen.tsx`) asks for the selected floor's rooms and keeps those whose `floorId` equals the floor's (string comparison — ROOM-D1 fixed; it used `Number(...)`).
- The room slice's `loading` covers saves, so the screen shows `Loading rooms...` while saving.

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- Picking a room for a Room Service order — ORD.

## Open questions

- None open. Related decisions: ROOM-D1 (fixed), ROOM-D2 to ROOM-D6.
