# Floors & rooms — data model

Sources: `db_script/pos_stage.sql` (re-checked 2026-11-24: no ROOM table changed), `app/Services/Room/RoomService.php`, `app/Models/Room.php`, `RoomFloor.php`. ROOM adds no tables or columns.

## `floors`

| Column | Type | Notes |
|---|---|---|
| `floorId` | binary(16) PK | new GUID on create |
| `restaurantId` | binary(16) null | the caller's restaurant |
| `floorName` | varchar(255) NOT NULL | no uniqueness today; target unique per restaurant among non-deleted floors, any case (ROOM-D3) |
| `isActive` | tinyint NOT NULL, 1 | |
| `createdBy` | varchar null | the caller's role on create |
| `isDelete` | tinyint NOT NULL, 0 | |

## `rooms`

| Column | Type | Notes |
|---|---|---|
| `roomId` | binary(16) PK | new GUID on create |
| `restaurantId` | binary(16) null | |
| `floorId` | binary(16) null | converted from hex on create since 2026-11-16 (ROOM-D1). Not checked against `floors` today — a valid hex of no own floor gives an orphan (ROOM-D3). On update stored unconverted today, which fails (ROOM-D4) |
| `roomName` | varchar(255) NOT NULL | no uniqueness today; target unique per floor (ROOM-D3) |
| `capacity` | int NOT NULL | any integer today; target ≥ 1 |
| `isActive` | tinyint NOT NULL, 1 | follows the floor when the floor is switched through the API |
| `status` | tinyint NOT NULL, 0 | 0 free · 1 occupied · 2 ordered — set by the Room Service order screen through `roomSave`, and by ORD / BILL |
| `version`, `createdBy` | | not set by ROOM |
| `isDelete` | tinyint NOT NULL, 0 | |

## Reads

| Reader | Returns |
|---|---|
| `getFloorDetails` | restaurant's floors, not deleted (active and inactive), plus `roomCount` = all room rows of the floor |
| `getRoomsDetails` | restaurant's rooms, not deleted, **active only** (target: inactive too with `includeInactive` — ROOM-D5), left-joined with running orders (`orderId`; one row per running order today — ROOM-D4) |

## Rules

- Updates find the floor / room by hex ID within the caller's restaurant (since 2026-11-16). A miss is a 500 today; target 404 (ROOM-D3).
- Switching a floor sets `rooms.isActive` for all its rooms, including deleted ones.
- Deleting a floor doesn't touch its rooms today; target: its rooms get `isDelete = 1` (ROOM-D3).
- IDs are returned as 32-character hex.
