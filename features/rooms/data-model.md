# Floors & rooms — data model

Sources: `db_script/tables/tables.sql`, `app/Services/Room/RoomService.php`, `app/Models/Room.php`, `RoomFloor.php`. ROOM adds no tables or columns.

## `floors`

| Column | Type | Notes |
|---|---|---|
| `floorId` | binary(16) PK | new GUID on create |
| `restaurantId` | binary(16) null | the caller's restaurant |
| `floorName` | varchar(255) NOT NULL | no uniqueness |
| `isActive` | tinyint NOT NULL, 1 | |
| `createdBy` | varchar null | the caller's role on create |
| `isDelete` | tinyint NOT NULL, 0 | |

## `rooms`

| Column | Type | Notes |
|---|---|---|
| `roomId` | binary(16) PK | new GUID on create |
| `restaurantId` | binary(16) null | |
| `floorId` | binary(16) null | stored **as sent**, with no hex conversion (ROOM-D1). A 32-character hex fails; a value of 16 characters or fewer is stored (padded), giving an orphan (ROOM-D3) |
| `roomName` | varchar(255) NOT NULL | |
| `capacity` | int NOT NULL | any integer |
| `isActive` | tinyint NOT NULL, 1 | |
| `status` | tinyint NOT NULL, 0 | 0 free · 1 occupied · 2 ordered — set by ORD / BILL |
| `version`, `createdBy` | | not set by ROOM |
| `isDelete` | tinyint NOT NULL, 0 | |

## Reads

| Reader | Returns |
|---|---|
| `getFloorDetails` | restaurant's floors, not deleted (active and inactive), plus `roomCount` = all room rows of the floor |
| `getRoomsDetails` | restaurant's rooms, not deleted, **active only**, left-joined with running orders (`orderId`, one row per running order); filters compared as sent |

## Rules

- Updates look up `floorId` / `roomId` as sent against binary columns, so they never match a hex ID (ROOM-D1).
- In code, switching a floor sets `rooms.isActive` for all its rooms. This is unreachable today.
- IDs are returned as 32-character hex.
