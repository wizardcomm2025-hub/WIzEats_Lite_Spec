# 01 — Manage floors

## Summary

The owner opens Floor Management and sees the floors with their room counts. They can add a floor, rename it, or switch it on or off; switching a floor switches all its rooms. Rename and switch work since 2026-11-16 (ROOM-D1 fixed).

## Prerequisites

- **Backend:** `POST /api/rooms/getFloorDetails`, `POST /api/rooms/floorSave` — [api-contract.md](./api-contract.md)
- **Data:** `floors`, `rooms` — [data-model.md](./data-model.md)
- **Depends on:** RST (Room Service on, so the menu shows)

## Actors & preconditions

- **Actor:** owner. Today any signed-in user through the API; target owner only for `floorSave`, owner and waiter for the list (ROOM-D2).
- **Trigger:** menu → `Floor Management` (shown only while Room Service is on).
- **Preconditions:** signed in as the owner; Room Service on (set in the panel).

## Main flow

1. The screen shows the header `FLOOR MANAGEMENT` / restaurant name / `Manage hotel floors` and a green `＋`. While loading, it shows the loading state.
2. The app posts to `getFloorDetails`. Each card shows the floor name, `Rooms: <count>`, an edit button and a switch. Inactive floors are faded.
3. **Add:** `＋` opens `Add Floor` with the field `Floor name (e.g. Ground Floor)` and the button `Add Floor`. An empty name shows `Floor name is required`. On submit the sheet closes (today at once; target only once saved — ROOM-D6). The server creates the floor (active, `createdBy` = the caller's role) and answers `floor created successfully`. The floor appears first, with `Rooms: 0`.
4. **Rename:** the edit button opens `Edit Floor` with the name filled in and a `Save Changes` button. The app sends `{ floorId, floorName }`; the server answers `floor updated successfully` and the list reloads with the new name.
5. **Switch:** the dialog `Deactivate floor?` / `Are you sure you want to deactivate "<name>"?` (or `Activate floor?`). On confirm the app sends `{ floorId, isActive }`. The server sets the floor **and all its rooms** to that state; the toast reads `Floor "<name>" deactivated` / `Floor "<name>" activated`. The card shows `Rooms: 0` until the list reloads.

## Alternative flows

- **No floors.** `No Floors Found` / `Start by adding your first floor to organize your restaurant layout.`
- **Add or rename fails** (no network, name over 255 characters, duplicate in the target). Today the sheet has closed and there is no message. *Target (ROOM-D6):* the sheet stays open with the server message or `Something went wrong. Please try again.` in red.
- **Duplicate name.** Today allowed. *Target (ROOM-D3):* 422 `Floor name already exists.` (any case), on add and rename.
- **Switch fails.** Toast with the server message or `Failed to toggle floor status`.
- **Switched back on.** All the floor's rooms become active again, including ones switched off on their own.
- **Delete** (API only, `isDelete: true`). The floor disappears from the lists. Today its rooms stay; *target (ROOM-D3)* they are deleted too.
- **Room Service off.** The menu doesn't show Floor Management. The API still accepts floors (accepted — ROOM-D2).
- **Waiter or chef through the API.** Today allowed. *Target:* `floorSave` 403 for both; `getFloorDetails` 403 for a chef.

## Acceptance criteria

1. Given Room Service is off, then the owner's menu has no Floor Management or Room Management.
2. Given Room Service is on and no floors exist, then `No Floors Found` shows.
3. Given `Ground Floor` is added, then it appears first with `Rooms: 0`.
4. Given an empty name, then `Floor name is required` shows and nothing is sent.
5. Given `Ground Floor` is renamed to `Level 0`, then the list shows `Level 0` (ROOM-D1 fixed).
6. Given `Ground Floor` is switched off and confirmed, then the toast `Floor "Ground Floor" deactivated` shows, the card fades, and all its rooms are inactive (ROOM-D1 fixed).
7. Given a floor with rooms, then its card shows the number of all its rooms, including inactive ones.
8. *(Target, ROOM-D3 / ROOM-D6)* Given `Ground Floor` exists, when `ground floor` is added, then `Floor name already exists.` shows under the field and nothing is added.

## Platform notes

### POS API

- `RoomController::floorSave` / `getRoomFloorDetails` → `RoomService::saveFloor` / `getRoomFloorDetails`.
- An update finds the floor by hex ID (dashes removed) within the caller's restaurant (since 2026-11-16). A miss is a 500 today; target 404 `Floor not found`.

### Data — MySQL

`floors` insert or update; when `isActive` is sent, `rooms.isActive` for every room of the floor.

### POS app

- `FloorManagementScreen` (`floorScreen.tsx`), `floor.thunks`, `floor.slice`.
- `loading` covers every action and shows the full-screen loading state.
- `state.error` is never shown (ROOM-D6).

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- Deleting floors — no screen.

## Open questions

- None open. Related decisions: ROOM-D1 (fixed), ROOM-D2, ROOM-D3, ROOM-D4, ROOM-D6.
