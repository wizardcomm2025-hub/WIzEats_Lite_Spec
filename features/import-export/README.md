# Import & export

Restaurants can bulk-load their setup from CSV files: zones with tables, menu categories with items, and floors with rooms. The export endpoints give the matching **empty templates** (header row only). Today this is **API-only**: the app's Import / Export section exists in code but is commented out of Profile. For rooms, import is today the only way that works (ROOM-D1).

The spec records today's behaviour. Decisions IMPEX-D1 to IMPEX-D3 keep the current rules as accepted risks.

**Feature code:** `IMPEX` (test IDs `IMPEX-01-D01`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Import CSV files](./01-import-csv.md) | Any signed-in user (API only — IMPEX-D3) | Tester: n/a — no UI (IMPEX-D1) | [developer](./test-cases/01-import-csv.developer.md) |
| 02 | [Download a template](./02-download-template.md) | Any signed-in user (API only) | Tester: n/a — no UI (IMPEX-D1) | [developer](./test-cases/02-download-template.developer.md) |

## Key concepts

- **Zone file** — columns `Zone Name, Table Name, Table Capacity`. Each row adds one table, and creates the zone if it isn't there.
- **Menu file** — `Category Name, Item Name, Description, isVeg, Price, Image`. Each row adds one item, and creates the category if needed. `Image` is ignored.
- **Floor file** — `Floor Name, Room Name, Room Capacity`. Each row adds one room, and creates the floor if needed.
- The first line of every file is treated as a header and skipped, whatever it contains.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api-dev_guid/specs/import-export/` (created with the first plan)

## Depends on

- **TBL**, **MENU**, **ROOM** — the rows import creates show up there, on Home and on the ordering screen.
- **RST** — the hidden Profile section.

## Decisions

- **IMPEX-D1 (2026-11-11):** The Import / Export section stays hidden in Profile, so there are no tester cases. If it is switched back on, it won't work: the import sends the field `file` (the server expects `zone_file` / `menu_file` / `floor_file`, so the answer is 422 `At least one CSV file is required.`), and the export posts to a different URL. Accepted.
- **IMPEX-D2 (2026-11-11):** Import always answers 200 `Import completed successfully` once the files pass validation.
  - Rows missing a name, and rows the database refuses (a name over 255 characters, a price out of range), are skipped and written only to the log. The response has no counts and no row numbers.
  - Importing the same file again adds every table, item and room again. Zones, categories and floors are matched by name, ignoring case, and reused.

  Accepted.
- **IMPEX-D3 (2026-11-11):** No data checks and no role check:
  - capacity and price are cast to numbers, so text becomes 0, and negative values are kept;
  - `isVeg` is the number the text starts with: `1` (any non-zero number) means Veg, anything else means Non-Veg;
  - `Image` is ignored;
  - rows land in an inactive zone, category or floor if the name matches one;
  - any signed-in user can import;
  - an unexpected failure answers 400 with the raw exception text;
  - export returns empty templates, not the restaurant's data.

  Accepted risk.
- **Inherited:** AUTH-D2.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Editing what was imported — TBL / MENU / ROOM.
- Exporting the restaurant's actual data — not offered.
