# Floors & rooms — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-10.

## Endpoints

| Endpoint | Middleware | Rate limit | Used by | Use case |
|---|---|---|---|---|
| `POST /api/rooms/getFloorDetails` | `auth:sanctum`, `restaurant.tenant` | none | App | [01](./01-manage-floors.md), [02](./02-manage-rooms.md) |
| `POST /api/rooms/floorSave` | same | none | App | [01](./01-manage-floors.md) |
| `POST /api/rooms/getRoomsDetails` | same | none | App | [02](./02-manage-rooms.md) |
| `POST /api/rooms/roomSave` | same | none | App | [02](./02-manage-rooms.md) |

There is no role check and no check of the Room Service setting (ROOM-D2). Common failures:

| Situation | HTTP | Body |
|---|---|---|
| No token or an invalid token | 401 | `{ "message": "Unauthenticated." }` |
| User has no `restaurantId` | 403 | `Restaurant context missing` |
| `GET` | 405 | Method Not Allowed |
| Validation | 422 | `{ "status": false, "message": "<first error>", "errors": {…} }` |

A switched-off user and a Pending or Rejected restaurant still get through (AUTH-D2).

---

## `POST /api/rooms/getFloorDetails`

| Field | Rule |
|---|---|
| `floorId` | optional string; compared as sent, so a hex ID returns `[]` |

200 `{ "status": true, "message": "Room floors retrieved successfully", "data": [ { floorId, restaurantId, floorName, isActive, createdBy, isDelete, created_at, updated_at, roomCount } ] }`.

## `POST /api/rooms/floorSave`

| Field | Create (no `floorId`) | Update (`floorId` filled) |
|---|---|---|
| `floorId` | — | string |
| `floorName` | required string | sometimes, string |
| `isActive` | optional boolean (default true) | optional boolean |
| `isDelete` | optional boolean (default false) | optional boolean |

| Situation | HTTP | Body |
|---|---|---|
| Created | 200 | `{ "status": true, "message": "floor created successfully", "data": { …floor, createdBy: "<caller role>" } }` |
| Any update by hex ID (ROOM-D1); a name over 255 characters | 500 | `{ "status": false, "message": "Failed to save floor", "data": null }` |
| Validation | 422 | `The floor name field is required.` · `The floor name field must be a string.` · `The is active field must be true or false.` |

## `POST /api/rooms/getRoomsDetails`

| Field | Rule |
|---|---|
| `floorId` | optional string; compared as sent, so a hex ID returns `[]` (ROOM-D1) |
| `roomId` | optional string; the column name is ambiguous, so any value gives a 500 (ROOM-D1) |

| Situation | HTTP | Body |
|---|---|---|
| `{}` | 200 | `{ "status": true, "message": "Rooms retrieved successfully", "data": [ { roomId, restaurantId, floorId, roomName, capacity, isActive: true, status, version, createdBy, isDelete, created_at, updated_at, orderId, status_key, status_color } ] }` — active rooms only; one row per running order |
| Any `floorId` | 200 | `data: []` |
| Any `roomId` | 500 | `{ "status": false, "message": "Failed to retrieve rooms", "data": null }` |

## `POST /api/rooms/roomSave`

| Field | Create (no `roomId`) | Update (`roomId` filled) |
|---|---|---|
| `roomId` | — | string |
| `roomName` | required string | sometimes, string |
| `floorId` | required string — stored as sent | sometimes, string |
| `capacity` | required integer | sometimes, integer |
| `isActive`, `isDelete` | optional boolean | optional boolean |
| `status` | optional, integer in 0, 1, 2 | same |

| Situation | HTTP | Body |
|---|---|---|
| Created (`floorId` ≤ 16 characters — ROOM-D3) | 200 | `{ "status": true, "message": "room created successfully", "data": { …room } }` |
| Create with a 32-character hex `floorId`; any update by hex ID; a name over 255 characters | 500 | `{ "status": false, "message": "Failed to save room", "data": null }` |
| Validation | 422 | `The room name field is required.` · `The floor id field is required.` · `The capacity field is required.` · `The capacity field must be an integer.` · `The selected status is invalid.` |
