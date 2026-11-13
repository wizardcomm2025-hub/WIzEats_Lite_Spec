# Zones & tables

The owner divides the restaurant into **zones** (seating areas such as "Main Hall" or "Garden") and sets up the **tables** in each, with a name and a seat count. Zones and tables can be switched on or off; a switched-off zone also switches off its tables. Active zones and tables are what Home shows for dine-in orders, through the restaurant details endpoint (RST 01). The same data can be edited in the super-admin panel.

The spec records today's behaviour. Decisions TBL-D1 to TBL-D4 keep the current rules as accepted risks. **The app's Table Management screen shows no tables at all today (TBL-D1).**

**Feature code:** `TBL` (test IDs `TBL-01-T01`, `TBL-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Manage zones in the app](./01-manage-zones.md) | Owner (any signed-in user through the API — TBL-D3) | [tester](./test-cases/01-manage-zones.tester.md) | [developer](./test-cases/01-manage-zones.developer.md) |
| 02 | [Manage tables in the app](./02-manage-tables.md) | Owner (any signed-in user through the API — TBL-D3) | [tester](./test-cases/02-manage-tables.tester.md) | [developer](./test-cases/02-manage-tables.developer.md) |
| 03 | [Zones and tables in the panel](./03-zones-tables-in-panel.md) | Super admin (web panel) | [tester](./test-cases/03-zones-tables-in-panel.tester.md) | [developer](./test-cases/03-zones-tables-in-panel.developer.md) |

## Key concepts

See the [glossary](../../project-overview.md#glossary) (Zone, Table). Specific to this feature:

- **Active / inactive** — `isActive`. Home shows only active zones and their active tables.
- **Deleted** — `isDelete = 1`. Hidden from the app's lists. No screen in the app or the panel deletes anything; only a direct API call does.
- **Table status** — `tables.status`: 0 free (`available`), 1 occupied, 2 ordered. Set by orders and payments (ORD / BILL). The API returns it with `status_key` and `status_color`.
- **Table count** — the number shown on a zone card. It counts **every** table row of the zone, including inactive and deleted ones.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api-dev_guid/specs/tables/` (created with the first plan)

## Depends on

- **ONB** — creates zone `Main Hall` with `Table 1`–`Table 4`.
- **RST** — `showDetails` returns the active zones and tables that Home uses.
- **ORD / BILL** — set the table status and the running order shown on a table.
- **IMPEX** — exporting zones and tables.
- **ADM** — panel sign-in and navigation.

## Decisions

- **TBL-D1 (2026-11-10):** The app's Table Management list stays as it is. It keeps only the tables where `Number(table.zoneId) === selectedZone.zoneId`. Both IDs are hex strings, and `Number(hex)` is `NaN`, so every zone shows `No Tables Found`. That holds even right after `Table created`. Tables can still be added from the app, but they can't be edited or switched from it. They do appear on Home. Accepted.
- **TBL-D2 (2026-11-10):** No integrity checks:
  - a table can be created in, or moved to, any `zoneId`: another restaurant's zone, or one that doesn't exist;
  - deleting a zone leaves its tables active;
  - a duplicate table name is refused only on create, within the same zone;
  - zone names may repeat;
  - capacity may be 0 or negative through the API;
  - names over 255 characters give a 500;
  - the panel checks nothing.

  Accepted risk.
- **TBL-D3 (2026-11-10):** No role check. Only the owner sees Zone and Table Management in the app, but any signed-in user of the restaurant can create, rename, switch off or delete zones and tables through the API. Accepted risk.
- **TBL-D4 (2026-11-10):** The list endpoints keep their faults:
  - `getTablesDetails` with a `tableId` answers 500 (ambiguous column);
  - a `zoneId` filter on either list never matches (hex compared with binary);
  - a table with two running orders appears twice.

  The app uses no filters. Accepted.
- **Inherited:** AUTH-D2. RST finding F2: a 16-byte non-ASCII name comes back as hex.

## Facts worth knowing

- Switching a zone on or off through the API sets **all** its tables to the same state. Turning a zone back on also re-activates tables that were off on their own. The panel's zone switch doesn't touch tables.
- After a zone is switched in the app, its card shows `Tables: 0` until the screen is reopened: the save reply has no count.
- Adding or renaming a zone in the app shows no error when it fails. The sheet closes, and the zone simply doesn't appear or change.
- Updating a zone or table that isn't the restaurant's (or doesn't exist) gives a 500 `Failed to save …`, not a 404.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Choosing a table for an order on Home, and table status changes caused by orders — ORD.
- Floors and rooms — ROOM.
- Import / export — IMPEX.
