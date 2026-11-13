# Import & export — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-11.

| Endpoint | Middleware | Rate limit | Use case |
|---|---|---|---|
| `POST /api/data/import` | `auth:sanctum`, `restaurant.tenant` | none | [01](./01-import-csv.md) |
| `POST /api/export/zones`, `/export/menu`, `/export/floors` | same | none | [02](./02-download-template.md) |

There is no role check (IMPEX-D3). The common failures: no or invalid token → 401 `{ "message": "Unauthenticated." }`; no `restaurantId` → 403 `Restaurant context missing`; `GET` → 405. A switched-off user and a Pending restaurant still get through (AUTH-D2).

## `POST /api/data/import`

`multipart/form-data`:

| Field | Rule |
|---|---|
| `zone_file` | optional; file; mimes `csv`, `txt`; ≤ 10 240 KB |
| `menu_file` | same |
| `floor_file` | same |
| — | at least one of the three |

| Situation | HTTP | Body |
|---|---|---|
| Processed (rows created or skipped — IMPEX-D2) | 200 | `{ "status": true, "message": "Import completed successfully" }` |
| No file | 422 | `{ "status": false, "message": "At least one CSV file is required.", "errors": { "file": [...] } }` |
| Wrong type | 422 | `message`: `The <zone file|menu file|floor file> field must be a file of type: csv, txt.` |
| Too big | 422 | `message`: `The <…> field must not be greater than 10240 kilobytes.` |
| Not a file (a text field) | 422 | `message`: `The <…> field must be a file.` |
| Unexpected failure | 400 | `{ "status": false, "message": "<raw exception text>" }` (IMPEX-D3) |

Rows are read by position; see [01](./01-import-csv.md) for each file's columns.

## `POST /api/export/zones` / `menu` / `floors`

No body. 200, `Content-Type: text/csv`, `Content-Disposition: attachment; filename=zone_file.csv` (or `menu_file.csv` / `floor_file.csv`). The body is one header line:

- zones: `Zone Name,Table Name,Table Capacity`
- menu: `Category Name,Item Name,Description,isVeg,Price,Image`
- floors: `Floor Name,Room Name,Room Capacity`

On an unexpected error: 500 `{ "error": "Failed to export zones" }` (or `menu`, `floors`).
