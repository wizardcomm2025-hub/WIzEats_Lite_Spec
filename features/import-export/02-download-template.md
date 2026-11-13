# 02 — Download a template

## Summary

Three endpoints return an empty CSV template, with a header row only, for the zone, menu and floor files. They don't export the restaurant's data (IMPEX-D3).

## Prerequisites

- **Backend:** `POST /api/export/zones`, `/export/menu`, `/export/floors` — [api-contract.md](./api-contract.md#post-apiexportzones--menu--floors)

## Actors & preconditions

- **Actor:** any signed-in user (API only — IMPEX-D1).

## Main flow

1. The caller posts to an export endpoint.
2. The server streams a CSV with `Content-Type: text/csv` and `Content-Disposition: attachment; filename=<name>`:

   | Endpoint | File name | Content |
   |---|---|---|
   | `/export/zones` | `zone_file.csv` | `Zone Name,Table Name,Table Capacity` |
   | `/export/menu` | `menu_file.csv` | `Category Name,Item Name,Description,isVeg,Price,Image` |
   | `/export/floors` | `floor_file.csv` | `Floor Name,Room Name,Room Capacity` |

## Alternative flows

- **`GET` instead of `POST`.** 405.
- **No token.** 401.

## Acceptance criteria

1. Given OWN-A, when each endpoint is called, then the matching single header line comes back as a CSV attachment.
2. Given REST-A has zones, items and rooms, then the files still hold only the header.
3. Given a template filled in and sent to import, then it imports ([01](./01-import-csv.md)).

## Platform notes

### POS API

`ExportController` → `ExportService::exportHeaderOnly`. Errors give 500 `{ "error": "Failed to export …" }`.

### Data — MySQL

Not read.

### POS app

The hidden section would call a different URL (IMPEX-D1).

### Super-admin panel

None.

### Integrations

None.

## Configuration

None.

## Out of scope

- A real data export.

## Open questions

- None open.
