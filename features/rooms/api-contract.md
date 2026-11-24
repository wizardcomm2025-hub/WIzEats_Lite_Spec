# Floors & rooms — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-10, re-checked 2026-11-24 (API commit d54d6c8; `RoomService` changed 2026-11-16 — ROOM-D1 fixed). Rows marked **Target** describe agreed fixes and are not in the code yet.

## Endpoints

| Endpoint | Middleware | Rate limit | Used by | Roles — target (X-D1 / ROOM-D2) | Use case |
|---|---|---|---|---|---|
| `POST /api/rooms/getFloorDetails` | `auth:sanctum`, `restaurant.tenant` | none | Floor / Room Management, Room Service order screen | owner, waiter | [01](./01-manage-floors.md), [02](./02-manage-rooms.md) |
| `POST /api/rooms/floorSave` | same | none | Floor Management | owner | [01](./01-manage-floors.md) |
| `POST /api/rooms/getRoomsDetails` | same | none | Room Management, Room Service order screen | owner, waiter (`includeInactive` owner only) | [02](./02-manage-rooms.md) |
| `POST /api/rooms/roomSave` | same | none | Room Management; Room Service order screen (`{ roomId, status }`) | owner any field; waiter only `{ roomId, status }` | [02](./02-manage-rooms.md) |

Today there is no role check and no check of the Room Service setting (ROOM-D2). **Target:** roles as above; otherwise 403 `{ "status": false, "message": "You do not have permission to perform this action", "data": null }`. The Room Service setting stays unchecked. Common failures:

| Situation | HTTP | Body |
|---|---|---|
| No token or an invalid token | 401 | `{ "message": "Unauthenticated." }` |
| User has no `restaurantId` | 403 | `Restaurant context missing` |
| `GET` | 405 | Method Not Allowed |
| Validation | 422 | `{ "status": false, "message": "<first error>", "errors": {…} }` |

Today a switched-off user and a Pending or Rejected restaurant still get through (AUTH-D2). **Target (X-D2):** 403 `Your account is inactive.` / `Your restaurant is not approved.` / `Your restaurant is inactive.`

---

## `POST /api/rooms/getFloorDetails`

| Field | Rule |
|---|---|
| `floorId` | optional string; today compared as sent, so a hex ID returns `[]` (ROOM-D4). **Target:** 32-char hex (dashes allowed) → that floor; other values → 422 `Validation error` |

200 `{ "status": true, "message": "Room floors retrieved successfully", "data": [ { floorId, restaurantId, floorName, isActive, createdBy, isDelete, created_at, updated_at, roomCount } ] }` — not deleted, active and inactive. `roomCount` counts every room row of the floor.

## `POST /api/rooms/floorSave`

| Field | Create (no `floorId`) | Update (`floorId` filled) |
|---|---|---|
| `floorId` | — | string — 32-char hex, dashes and upper case allowed |
| `floorName` | required string (**target** `max:255`, unique per restaurant) | sometimes, string (same target) |
| `isActive` | optional boolean (default true) | optional boolean — also sets `isActive` on **all** the floor's rooms |
| `isDelete` | optional boolean (default false) | optional boolean — **target:** `true` also deletes the floor's rooms |

| Situation | HTTP | Body |
|---|---|---|
| Created | 200 | `{ "status": true, "message": "floor created successfully", "data": { …floor, createdBy: "<caller role>" } }` |
| Updated (since 2026-11-16) | 200 | `{ "status": true, "message": "floor updated successfully", "data": { …floor } }` |
| Unknown / other restaurant's / malformed `floorId`; a name over 255 characters | 500 | `{ "status": false, "message": "Failed to save floor", "data": null }` |
| **Target (ROOM-D3)** unknown / deleted / other restaurant's / malformed `floorId` | 404 | `{ "status": false, "message": "Floor not found", "data": null }` |
| **Target (ROOM-D3)** name over 255 | 422 | `The floor name field must not be greater than 255 characters.` |
| **Target (ROOM-D3)** name used by another non-deleted floor (any case) | 422 | `{ "status": false, "message": "Floor name already exists.", "data": null }` |
| Validation | 422 | `The floor name field is required.` · `The floor name field must be a string.` · `The is active field must be true or false.` |

## `POST /api/rooms/getRoomsDetails`

| Field | Rule |
|---|---|
| `floorId` | optional string — 32-char hex, dashes allowed; returns that floor's rooms (works since 2026-11-16). Malformed → 500 today; **target** 422 |
| `roomId` | optional string — today any value gives a 500 (ambiguous column — ROOM-D4). **Target:** 32-char hex → that room; other values → 422 |
| `includeInactive` | **Target (ROOM-D5):** optional boolean, owner only; `true` also returns inactive, non-deleted rooms |

| Situation | HTTP | Body |
|---|---|---|
| OK | 200 | `{ "status": true, "message": "Rooms retrieved successfully", "data": [ { roomId, restaurantId, floorId, roomName, capacity, isActive, status, version, createdBy, isDelete, created_at, updated_at, orderId, status_key, status_color } ] }` — active, non-deleted rooms; today one row per running order (ROOM-D4); **target** one row per room, `orderId` = highest running order |
| Any `roomId` | 500 | `{ "status": false, "message": "Failed to retrieve rooms", "data": null }` (target: 200) |
| **Target** `includeInactive` from a waiter | 403 | permission body |

## `POST /api/rooms/roomSave`

| Field | Create (no `roomId`) | Update (`roomId` filled) |
|---|---|---|
| `roomId` | — | string — 32-char hex, dashes allowed |
| `roomName` | required string (**target** `max:255`, unique per floor) | sometimes, string (same target) |
| `floorId` | required string — converted from hex since 2026-11-16; **any** floor today (ROOM-D3); **target** an own non-deleted floor | sometimes, string — today stored **unconverted**, so a hex ID gives 500 (ROOM-D4); **target** converted and checked |
| `capacity` | required integer (**target** `min:1`) | sometimes, integer (same) |
| `isActive`, `isDelete` | optional boolean | optional boolean |
| `status` | optional, integer in 0, 1, 2 (strings `"0"`–`"2"` accepted) | same — the Room Service order screen sends `"1"` / `"0"` |

| Situation | HTTP | Body |
|---|---|---|
| Created | 200 | `{ "status": true, "message": "room created successfully", "data": { …room } }` |
| Updated (since 2026-11-16) | 200 | `{ "status": true, "message": "room updated successfully", "data": { …room } }` |
| Create with a `floorId` that is valid hex of no own floor (e.g. 32 hex of nothing, or `abcd`) | 200 | created as an orphan room (ROOM-D3). **Target:** 422 `{ "status": false, "message": "Invalid floor", "data": null }` |
| Create with a malformed `floorId` (odd length, not hex); update with `floorId`; unknown / other restaurant's `roomId`; a name over 255 | 500 | `{ "status": false, "message": "Failed to save room", "data": null }`. **Target:** 422 `Invalid floor` / 200 moved / 404 `Room not found` / 422 max length |
| **Target (ROOM-D3)** room name already on that floor (any case) | 422 | `{ "status": false, "message": "Room name already exists.", "data": null }` |
| **Target (ROOM-D3)** `capacity` < 1 | 422 | `The capacity field must be at least 1.` |
| **Target (ROOM-D2)** waiter sends any field besides `roomId`, `status` | 403 | permission body |
| Validation | 422 | `The room name field is required.` · `The floor id field is required.` · `The capacity field is required.` · `The capacity field must be an integer.` · `The selected status is invalid.` |
