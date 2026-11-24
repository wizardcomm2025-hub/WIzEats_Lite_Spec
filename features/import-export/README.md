# Import & export

Restaurants can bulk-load their setup from CSV files: zones with tables, menu categories with items, and floors with rooms. The export endpoints give the matching **empty templates** (header row only). This is **API-only**: the app's Import / Export section exists in code but is commented out of Profile (IMPEX-D1, accepted). Since 2026-11-16 rooms can also be created in the app (ROOM-D1 fixed).

The spec records today's behaviour plus targets (re-synced 2026-11-24 against API commit d54d6c8 and `POS_APP-kot`; no IMPEX code changed). IMPEX-D1 stays accepted; IMPEX-D2 and IMPEX-D3 are *To fix*.

**Feature code:** `IMPEX` (test IDs `IMPEX-01-D01`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Import CSV files](./01-import-csv.md) | Today any signed-in user; target owner (API only — IMPEX-D3) | Tester: n/a — no UI (IMPEX-D1) | [developer](./test-cases/01-import-csv.developer.md) |
| 02 | [Download a template](./02-download-template.md) | Today any signed-in user; target owner (API only) | Tester: n/a — no UI (IMPEX-D1) | [developer](./test-cases/02-download-template.developer.md) |

## Key concepts

- **Zone file** — columns `Zone Name, Table Name, Table Capacity`. Each row adds one table, and creates the zone if it isn't there.
- **Menu file** — `Category Name, Item Name, Description, isVeg, Price, Image`. Each row adds one item, and creates the category if needed. `Image` is ignored.
- **Floor file** — `Floor Name, Room Name, Room Capacity`. Each row adds one room, and creates the floor if needed.
- The first line of every file is treated as a header and skipped, whatever it contains.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api/api/specs/import-export/` (created with the first plan)

## Depends on

- **TBL**, **MENU**, **ROOM** — the rows import creates show up there, on Home and on the ordering screen.
- **RST** — the hidden Profile section.

## Decisions

Statuses per [conventions.md → Decision status](../../conventions.md#decision-status), set 2026-11-24. Cross-cutting items: [project-overview.md → Known gaps and targets](../../project-overview.md#known-gaps-and-targets).

| ID | Status | Short | Tests |
|---|---|---|---|
| IMPEX-D1 | Accepted (corrected 2026-11-24) | Import / Export hidden in the app | — |
| IMPEX-D2 | **To fix** | No report of skipped rows; re-import duplicates | 01-D05–D07, 01-D19, 01-D20 |
| IMPEX-D3 | **To fix** (with X-D1, X-D5) | No data rules, any role, raw exception text | 01-D05, 01-D06, 01-D10, 01-D11, 01-D21; 02-D04, 02-D06 |
| AUTH-D2 (inherited) | **To fix** (X-D2) | Switched-off / unapproved callers keep access | 01-D14, 02-D03 |

- **IMPEX-D1 — Accepted** (re-confirmed 2026-11-24, with a correction). The section stays hidden; no tester cases. Correction: the hidden section's export calls `export/zones`, `export/menu`, `export/floors`, which **do** match the server; only its import is wrong (it sends the field `file`, so the server answers 422 `At least one CSV file is required.`).
- **IMPEX-D2 — To fix (2026-11-24, was accepted 2026-11-11).** *Target:* the 200 answer reports what happened — per file `created` and `skipped` counts and a list of skipped rows with the file, the line number (the header is line 1) and a fixed reason ([api-contract.md](./api-contract.md#post-apidataimport)). A row whose table / item / room name already exists in that zone / category / floor (not deleted, any case, trimmed) — or appeared earlier in the same file — is skipped as already existing, so importing a file again creates nothing new. A zone, category or floor is created only together with at least one valid row.
- **IMPEX-D3 — To fix (2026-11-24, was accepted 2026-11-11).** *Target:* rows are checked with the same rules as the screens (TBL-D2, MENU-D4, ROOM-D3): capacity a whole number ≥ 1; price a number > 0 and ≤ 99 999 999.99; names ≤ 255 characters; `isVeg` one of `1`, `0`, `true`, `false`, `veg`, `non-veg` or empty (any case; empty = Non-Veg). A row that fails is skipped with its reason (IMPEX-D2). Import and export are owner only (X-D1: 403). An unexpected failure rolls everything back and answers 500 `{ "status": false, "message": "Import failed", "data": null }` — no exception text (X-D5); an export failure answers 500 `{ "status": false, "message": "Failed to export <zones|menu|floors>", "data": null }` (no `error` key). Still accepted: export returns empty templates, not the restaurant's data; rows may land in an existing **inactive** zone, category or floor; the `Image` column is ignored.
- **Inherited:** AUTH-D2 (To fix, X-D2).

Original decision texts (history):

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
- **Inherited (2026-11-11):** AUTH-D2.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Editing what was imported — TBL / MENU / ROOM.
- Exporting the restaurant's actual data — not offered.
