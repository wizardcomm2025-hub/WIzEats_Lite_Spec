# Zones & tables — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-10.

## Endpoints

| Endpoint | Middleware | Rate limit | Used by | Use case |
|---|---|---|---|---|
| `POST /api/tables/getTablesZoneDetails` | `auth:sanctum`, `restaurant.tenant` | none | App (zones, table zone picker) | [01](./01-manage-zones.md), [02](./02-manage-tables.md) |
| `POST /api/tables/ZoneSave` | same | none | App | [01](./01-manage-zones.md) |
| `POST /api/tables/getTablesDetails` | same | none | App | [02](./02-manage-tables.md) |
| `POST /api/tables/tableSave` | same | none | App | [02](./02-manage-tables.md) |
| Panel routes | `web`, `auth:admin` | none | Panel | [03](./03-zones-tables-in-panel.md) |

There is no role check on any endpoint (TBL-D3). The common failures of the API endpoints:

| Situation | HTTP | Body |
|---|---|---|
| No token / invalid token | 401 | `{ "message": "Unauthenticated." }` |
| User has no `restaurantId` | 403 | `Restaurant context missing` |
| `GET` | 405 | Method Not Allowed |

A switched-off user, or a Pending / Rejected restaurant, still gets through (AUTH-D2).

---

## `POST /api/tables/getTablesZoneDetails`

| Field | Rule |
|---|---|
| `zoneId` | optional string; compared with the binary column as sent, so it never matches (TBL-D4) |

| Situation | HTTP | Body |
|---|---|---|
| OK | 200 | `{ "status": true, "message": "Tables Zone retrieved successfully", "data": [ { zoneId, restaurantId, zoneName, isActive, isDelete, createdBy, created_at, updated_at, tableCount } ] }` |
| Any `zoneId` | 200 | `data: []` |
| `zoneId` not a string | 422 | `{ "status": false, "message": "Validation error", "errors": { … } }` |

## `POST /api/tables/ZoneSave`

| Field | Create (no `zoneId`) | Update (`zoneId` filled) |
|---|---|---|
| `zoneId` | — | string, 32-char hex (dashes allowed) |
| `zoneName` | required string | sometimes, string |
| `isActive` | optional boolean (default true) | optional boolean — also sets all the zone's tables |
| `isDelete` | optional boolean (default false) | optional boolean |

| Situation | HTTP | Body |
|---|---|---|
| Created | 200 | `{ "status": true, "message": "Zone created successfully", "data": { …zone } }` (no `tableCount`) |
| Updated | 200 | `{ "status": true, "message": "Zone updated successfully", "data": { …zone } }` |
| Validation | 422 | `{ "status": false, "message": "<first error>", "errors": {…} }`, e.g. `The zone name field is required.`, `The is active field must be true or false.` |
| Zone not in the caller's restaurant / unknown / `zoneId` not hex / name > 255 | 500 | `{ "status": false, "message": "Failed to save zone", "data": null }` |

## `POST /api/tables/getTablesDetails`

| Field | Rule |
|---|---|
| `tableId`, `zoneId` | optional strings |

| Situation | HTTP | Body |
|---|---|---|
| OK | 200 | `{ "status": true, "message": "Tables retrieved successfully", "data": [ { tableId, restaurantId, zoneId, tableName, capacity, isActive, status, version, createdBy, isDelete, created_at, updated_at, orderId, status_key, status_color } ] }` — one row per running order (TBL-D4) |
| Any `zoneId` | 200 | `data: []` (TBL-D4) |
| Any `tableId` | 500 | `{ "status": false, "message": "Failed to retrieve tables", "data": null }` — ambiguous column (TBL-D4) |
| Field not a string | 422 | `{ "status": false, "message": "Validation error", "errors": {…} }` |

`status_key` / `status_color`: 0 → `available` / `#22c55e`, 1 → `occupied` / `#f59e0b`, 2 → `ordered` / `#ef4444`.

## `POST /api/tables/tableSave`

| Field | Create (no `tableId`) | Update (`tableId` filled) |
|---|---|---|
| `tableId` | — | string, 32-char hex |
| `tableName` | required string | sometimes, string |
| `zoneId` | required string (32-char hex) — **any** zone (TBL-D2) | sometimes, string |
| `capacity` | required integer (any value) | sometimes, integer |
| `isActive` | optional boolean (default true) | optional boolean |
| `status` | optional, integer in 0, 1, 2 (default 0) | same |
| `isDelete` | optional boolean (default false) | optional boolean |

| Situation | HTTP | Body |
|---|---|---|
| Created | 200 | `{ "status": true, "message": "Table created successfully", "data": { …table } }` |
| Updated | 200 | `{ "status": true, "message": "Table updated successfully", "data": { …table } }` |
| Create: name already in that zone (not deleted, any case) | 422 | `{ "status": false, "message": "Table name already exists.", "data": null }` |
| Validation | 422 | `message` e.g. `The table name field is required.`, `The capacity field must be an integer.`, `The selected status is invalid.` |
| Table not in the caller's restaurant / unknown / ID not hex / name > 255 | 500 | `{ "status": false, "message": "Failed to save table", "data": null }` |

---

## Panel routes (web)

All of them need the panel session, and the POSTs need the CSRF header. No validation.

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
