# Zones & tables — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-10, re-checked 2026-11-24 (API commit d54d6c8 — no change). Rows marked **Target** describe agreed fixes and are not in the code yet.

## Endpoints

| Endpoint | Middleware | Rate limit | Used by | Use case |
|---|---|---|---|---|
| `POST /api/tables/getTablesZoneDetails` | `auth:sanctum`, `restaurant.tenant` | none | App (zones, table zone picker) | [01](./01-manage-zones.md), [02](./02-manage-tables.md) |
| `POST /api/tables/ZoneSave` | same | none | App | [01](./01-manage-zones.md) |
| `POST /api/tables/getTablesDetails` | same | none | App | [02](./02-manage-tables.md) |
| `POST /api/tables/tableSave` | same | none | App | [02](./02-manage-tables.md) |
| Panel routes | `web`, `auth:admin` | none | Panel | [03](./03-zones-tables-in-panel.md) |

Today there is no role check on any endpoint (TBL-D3). **Target (X-D1):** all four API endpoints owner only → 403 `{ "status": false, "message": "You do not have permission to perform this action", "data": null }`. The common failures of the API endpoints:

| Situation | HTTP | Body |
|---|---|---|
| No token / invalid token | 401 | `{ "message": "Unauthenticated." }` |
| User has no `restaurantId` | 403 | `Restaurant context missing` |
| `GET` | 405 | Method Not Allowed |

Today a switched-off user, or a Pending / Rejected restaurant, still gets through (AUTH-D2). **Target (X-D2):** 403 `Your account is inactive.` / `Your restaurant is not approved.` / `Your restaurant is inactive.`

---

## `POST /api/tables/getTablesZoneDetails`

| Field | Rule |
|---|---|
| `zoneId` | optional string; compared with the binary column as sent, so it never matches (TBL-D4). **Target:** 32-char hex (dashes allowed) → that zone only; other values → 422 |

| Situation | HTTP | Body |
|---|---|---|
| OK | 200 | `{ "status": true, "message": "Tables Zone retrieved successfully", "data": [ { zoneId, restaurantId, zoneName, isActive, isDelete, createdBy, created_at, updated_at, tableCount } ] }` |
| Any `zoneId` | 200 | `data: []` (target: the matching zone) |
| `zoneId` not a string | 422 | `{ "status": false, "message": "Validation error", "errors": { … } }` |

## `POST /api/tables/ZoneSave`

| Field | Create (no `zoneId`) | Update (`zoneId` filled) |
|---|---|---|
| `zoneId` | — | string, 32-char hex (dashes allowed) |
| `zoneName` | required string (**target** `max:255`, unique per restaurant) | sometimes, string (same target) |
| `isActive` | optional boolean (default true) | optional boolean — also sets all the zone's tables |
| `isDelete` | optional boolean (default false) | optional boolean — **target:** `true` also deletes the zone's tables |

| Situation | HTTP | Body |
|---|---|---|
| Created | 200 | `{ "status": true, "message": "Zone created successfully", "data": { …zone } }` (no `tableCount`) |
| Updated | 200 | `{ "status": true, "message": "Zone updated successfully", "data": { …zone } }` |
| Validation | 422 | `{ "status": false, "message": "<first error>", "errors": {…} }`, e.g. `The zone name field is required.`, `The is active field must be true or false.` |
| Zone not in the caller's restaurant / unknown / `zoneId` not hex / name > 255 | 500 | `{ "status": false, "message": "Failed to save zone", "data": null }` |
| **Target (TBL-D2)** zone not in the caller's restaurant / unknown / deleted / `zoneId` not hex | 404 | `{ "status": false, "message": "Zone not found", "data": null }` |
| **Target (TBL-D2)** name > 255 | 422 | `The zone name field must not be greater than 255 characters.` |
| **Target (TBL-D2)** name already used by another non-deleted zone of the restaurant (any case) | 422 | `{ "status": false, "message": "Zone name already exists.", "data": null }` (today: saved) |

## `POST /api/tables/getTablesDetails`

| Field | Rule |
|---|---|
| `tableId`, `zoneId` | optional strings |

| Situation | HTTP | Body |
|---|---|---|
| OK | 200 | `{ "status": true, "message": "Tables retrieved successfully", "data": [ { tableId, restaurantId, zoneId, tableName, capacity, isActive, status, version, createdBy, isDelete, created_at, updated_at, orderId, status_key, status_color } ] }` — one row per running order today (TBL-D4); **target** one row per table, `orderId` = highest running order |
| Any `zoneId` | 200 | `data: []` (TBL-D4). **Target:** that zone's tables |
| Any `tableId` | 500 | `{ "status": false, "message": "Failed to retrieve tables", "data": null }` — ambiguous column (TBL-D4). **Target:** 200 with that table (or `[]`) |
| **Target** `tableId` / `zoneId` not 32-char hex | 422 | `{ "status": false, "message": "Validation error", "errors": {…} }` |
| Field not a string | 422 | `{ "status": false, "message": "Validation error", "errors": {…} }` |

`status_key` / `status_color`: 0 → `available` / `#22c55e`, 1 → `occupied` / `#f59e0b`, 2 → `ordered` / `#ef4444`.

## `POST /api/tables/tableSave`

| Field | Create (no `tableId`) | Update (`tableId` filled) |
|---|---|---|
| `tableId` | — | string, 32-char hex |
| `tableName` | required string (**target** `max:255`) | sometimes, string (same) |
| `zoneId` | required string (32-char hex) — **any** zone today (TBL-D2); **target** a non-deleted zone of the caller's restaurant | sometimes, string (same target) |
| `capacity` | required integer (any value; **target** `min:1`) | sometimes, integer (same) |
| `isActive` | optional boolean (default true) | optional boolean |
| `status` | optional, integer in 0, 1, 2 (default 0) | same |
| `isDelete` | optional boolean (default false) | optional boolean |

| Situation | HTTP | Body |
|---|---|---|
| Created | 200 | `{ "status": true, "message": "Table created successfully", "data": { …table } }` |
| Updated | 200 | `{ "status": true, "message": "Table updated successfully", "data": { …table } }` |
| Create: name already in that zone (not deleted, any case) | 422 | `{ "status": false, "message": "Table name already exists.", "data": null }` — **target** also on an update that changes the name or zone |
| **Target (TBL-D2)** `zoneId` not a non-deleted zone of the restaurant | 422 | `{ "status": false, "message": "Invalid zone", "data": null }` (today: saved) |
| **Target (TBL-D2)** `capacity` < 1 | 422 | `The capacity field must be at least 1.` (today: saved) |
| Validation | 422 | `message` e.g. `The table name field is required.`, `The capacity field must be an integer.`, `The selected status is invalid.` |
| Table not in the caller's restaurant / unknown / ID not hex / name > 255 | 500 | `{ "status": false, "message": "Failed to save table", "data": null }`. **Target:** 404 `Table not found` (name > 255: 422) |

---

## Panel routes (web)

All of them need the panel session, and the POSTs need the CSRF header. Today no validation. **Target (TBL-D2):** names required, max 255, unique (zone per restaurant, table per zone, any case, non-deleted); `capacity` integer ≥ 1; `zoneId` a zone of the posted / stored restaurant; `isActive` boolean; failures 422 JSON `{ message, errors }`; unknown `id` / `restaurantId` → 404; `tables/update` writes only `zoneId`, `tableName`, `capacity`, `isActive`.

| Route | Body | Result |
|---|---|---|
| `GET /admin/zones-management/{restaurantId}` | — | Page, 10 per page, all zones (deleted too) |
| `POST /admin/zones/store` | `restaurantId, name, isActive` | Insert; `{ "success": true }`; empty name → 500 |
| `POST /admin/zones/update` | `id, name, isActive` | Update name and isActive; tables untouched; `{ "success": true }` (also for an unknown id) |
| `POST /admin/zones/toggle` | `id, isActive` | Update isActive; tables untouched |
| `GET /admin/table-management/{restaurantId}` | — | Page, 10 per page, all tables, with the zone name |
| `POST /admin/tables/store` | `restaurantId, zoneId, tableName, capacity, isActive` | Insert; no duplicate check; empty name or capacity → 500 |
| `POST /admin/tables/update` | `id, zoneId?, …any columns` | Writes every posted key except `id`, `tableId`, `zoneId`, `restaurantId` as a column (`zoneId` converted). An unknown column → 500 |
| `POST /admin/table-toggle-status` | `id, isActive` | Update isActive. The page never calls it |

The common failures are the same as for every panel route: no session → 302 to `/` (JSON 401); bad CSRF → 419; a POS bearer token counts as no session.
