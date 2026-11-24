# Import & export — API contract

Shared shapes: [project-overview.md → API conventions](../../project-overview.md#api-conventions). Facts from the code on 2026-11-11, re-checked 2026-11-24 (API commit d54d6c8 — no change). Rows marked **Target** describe agreed fixes (IMPEX-D2, IMPEX-D3) and are not in the code yet.

| Endpoint | Middleware | Rate limit | Roles — target (X-D1) | Use case |
|---|---|---|---|---|
| `POST /api/data/import` | `auth:sanctum`, `restaurant.tenant` | none | owner | [01](./01-import-csv.md) |
| `POST /api/export/zones`, `/export/menu`, `/export/floors` | same | none | owner | [02](./02-download-template.md) |

Today there is no role check (IMPEX-D3). **Target:** owner only; others → 403 `{ "status": false, "message": "You do not have permission to perform this action", "data": null }`. The common failures: no or invalid token → 401 `{ "message": "Unauthenticated." }`; no `restaurantId` → 403 `Restaurant context missing`; `GET` → 405. Today a switched-off user and a Pending restaurant still get through (AUTH-D2); **target (X-D2)** 403 `Your account is inactive.` / `Your restaurant is not approved.` / `Your restaurant is inactive.`

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
| Processed (rows created or skipped) | 200 | Today `{ "status": true, "message": "Import completed successfully" }` (IMPEX-D2). **Target:** the same plus `data` (below) |
| No file | 422 | `{ "status": false, "message": "At least one CSV file is required.", "errors": { "file": [...] } }` |
| Wrong type | 422 | `message`: `The <zone file|menu file|floor file> field must be a file of type: csv, txt.` |
| Too big | 422 | `message`: `The <…> field must not be greater than 10240 kilobytes.` |
| Not a file (a text field) | 422 | `message`: `The <…> field must be a file.` |
| Unexpected failure | 400 | `{ "status": false, "message": "<raw exception text>" }` (IMPEX-D3). **Target:** 500 `{ "status": false, "message": "Import failed", "data": null }`, nothing written |

**Target `data` (IMPEX-D2):**

```json
{
  "zone":  { "created": 3, "skipped": 1 },
  "menu":  { "created": 0, "skipped": 0 },
  "floor": { "created": 0, "skipped": 0 },
  "skippedRows": [ { "file": "zone", "line": 4, "reason": "Capacity must be a whole number of at least 1" } ]
}
```

Only the files sent appear as keys. `created` counts tables / items / rooms (not zones, categories, floors).

**Target row checks (IMPEX-D3) and reasons.** Rows are read by position; see [01](./01-import-csv.md) for each file's columns. The first failing check gives the reason:

| File | Check (in this order) | `reason` |
|---|---|---|
| all | parent name empty | `Zone name is required` · `Category name is required` · `Floor name is required` |
| all | child name empty | `Table name is required` · `Menu item name is required` · `Room name is required` |
| all | a name over 255 characters | `Name is longer than 255 characters` |
| zone, floor | capacity not a whole number ≥ 1 | `Capacity must be a whole number of at least 1` |
| menu | `isVeg` not one of `1`, `0`, `true`, `false`, `veg`, `non-veg`, empty (any case; empty = Non-Veg) | `isVeg must be 1, 0, true, false, veg or non-veg` |
| menu | price not a number > 0 and ≤ 99 999 999.99 | `Price must be a number greater than 0 and at most 99999999.99` |
| all | the child name already exists in that zone / category / floor (not deleted, any case, trimmed) — also an earlier row of the same file | `Table name already exists.` · `Menu item name already exists in this category` · `Room name already exists.` |

## `POST /api/export/zones` / `menu` / `floors`

No body. 200, `Content-Type: text/csv`, `Content-Disposition: attachment; filename=zone_file.csv` (or `menu_file.csv` / `floor_file.csv`). The body is one header line:

- zones: `Zone Name,Table Name,Table Capacity`
- menu: `Category Name,Item Name,Description,isVeg,Price,Image`
- floors: `Floor Name,Room Name,Room Capacity`

On an unexpected error: 500 `{ "error": "Failed to export zones" }` (or `menu`, `floors`). **Target (IMPEX-D3 / X-D5):** 500 `{ "status": false, "message": "Failed to export zones", "data": null }` — no `error` key.
