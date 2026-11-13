# 01 — Manage floors

## Summary

The owner opens Floor Management and sees the floors with their room counts. They can add a floor. Renaming or switching a floor fails today (ROOM-D1).

## Prerequisites

- **Backend:** `POST /api/rooms/getFloorDetails`, `POST /api/rooms/floorSave` — [api-contract.md](./api-contract.md)
- **Data:** `floors`, `rooms` — [data-model.md](./data-model.md)
- **Depends on:** RST (Room Service on, so the menu shows)

## Actors & preconditions

- **Actor:** owner. Through the API, any signed-in user (ROOM-D2).
- **Trigger:** menu → `Floor Management` (shown only while Room Service is on).
- **Preconditions:** signed in as the owner; Room Service on (set in the panel).

## Main flow

1. The screen shows the header `FLOOR MANAGEMENT` / restaurant name / `Manage hotel floors` and a green `＋`. While loading, it shows the loading state.
2. The app posts to `getFloorDetails`. Each card shows the floor name, `Rooms: <count>`, an edit button and a switch. Inactive floors are faded.
3. **Add:** `＋` opens `Add Floor` with the field `Floor name (e.g. Ground Floor)` and the button `Add Floor`. An empty name shows `Floor name is required`. On submit the sheet closes. The server creates the floor (active, `createdBy` = the caller's role) and answers `floor created successfully`. The floor appears first, with `Rooms: 0`.
4. **Rename:** the edit button opens `Edit Floor` with the name filled in and a `Save Changes` button. The app sends `{ floorId, floorName }`. The server answers 500 `Failed to save floor` (ROOM-D1). No message is shown; the list reloads with the old name.
5. **Switch:** the dialog `Deactivate floor?` / `Are you sure you want to deactivate "<name>"?`. On confirm the server answers 500 (ROOM-D1). The toast shows `Failed to save floor`, and the floor stays as it was.

## Alternative flows

- **No floors.** `No Floors Found` / `Start by adding your first floor to organize your restaurant layout.`
- **Add fails** (no network, name over 255 characters). The sheet closes and there is no message; the floor isn't added.
- **Duplicate name.** Allowed (ROOM-D3).
- **Room Service off.** The menu doesn't show Floor Management. The API still accepts floors (ROOM-D2).

## Acceptance criteria

1. Given Room Service is off, then the owner's menu has no Floor Management or Room Management.
2. Given Room Service is on and no floors exist, then `No Floors Found` shows.
3. Given `Ground Floor` is added, then it appears first with `Rooms: 0`.
4. Given an empty name, then `Floor name is required` shows and nothing is sent.
5. Given `Ground Floor` is renamed to `Level 0`, then the list still shows `Ground Floor` and no message appears (ROOM-D1).
6. Given `Ground Floor` is switched off and confirmed, then the toast `Failed to save floor` shows and the floor stays active (ROOM-D1).
7. Given a floor with imported rooms, then its card shows the number of all its rooms, including inactive ones.

## Platform notes

### POS API

- `RoomController::floorSave` / `getRoomFloorDetails` → `RoomService::saveFloor` / `getRoomFloorDetails`.
- An update searches for `floorId` as sent (hex) against the binary column (ROOM-D1).

### Data — MySQL

`floors` insert. The update and the cascade to `rooms.isActive` are in the code but never reached.

### POS app

- `FloorManagementScreen`, `floor.thunks`, `floor.slice`.
- `loading` covers every action and shows the full-screen loading state.
- `state.error` is never shown.

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- Deleting floors — no screen.

## Open questions

- None open. Related decisions: ROOM-D1, ROOM-D2, ROOM-D3.
