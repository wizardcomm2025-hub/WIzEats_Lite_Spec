# Zones & tables

The owner divides the restaurant into **zones** (seating areas such as "Main Hall" or "Garden") and sets up the **tables** in each, with a name and a seat count. Zones and tables can be switched on or off; a switched-off zone also switches off its tables. Active zones and tables are what Home shows for dine-in orders, through the restaurant details endpoint (RST 01). The same data can be edited in the super-admin panel.

The spec records today's behaviour plus targets (re-synced 2026-11-24 against API commit d54d6c8 and `POS_APP-kot`). TBL-D1 is fixed in the app — Table Management lists, edits and switches tables again. TBL-D2 to TBL-D5 are *To fix*. The tables API code has not changed since the spec was written.

**Feature code:** `TBL` (test IDs `TBL-01-T01`, `TBL-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Manage zones in the app](./01-manage-zones.md) | Owner (today any signed-in user through the API — TBL-D3) | [tester](./test-cases/01-manage-zones.tester.md) | [developer](./test-cases/01-manage-zones.developer.md) |
| 02 | [Manage tables in the app](./02-manage-tables.md) | Owner (today any signed-in user through the API — TBL-D3) | [tester](./test-cases/02-manage-tables.tester.md) | [developer](./test-cases/02-manage-tables.developer.md) |
| 03 | [Zones and tables in the panel](./03-zones-tables-in-panel.md) | Super admin (web panel) | [tester](./test-cases/03-zones-tables-in-panel.tester.md) | [developer](./test-cases/03-zones-tables-in-panel.developer.md) |

## Key concepts

See the [glossary](../../project-overview.md#glossary) (Zone, Table). Specific to this feature:

- **Active / inactive** — `isActive`. Home shows only active zones and their active tables.
- **Deleted** — `isDelete = 1`. Hidden from the app's lists. No screen in the app or the panel deletes anything; only a direct API call does.
- **Table status** — `tables.status`: 0 free (`available`), 1 occupied, 2 ordered. Set by orders and payments (ORD / BILL). The API returns it with `status_key` and `status_color`.
- **Table count** — the number shown on a zone card. It counts **every** table row of the zone, including inactive and deleted ones.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api/api/specs/tables/` (created with the first plan)

## Depends on

- **ONB** — creates zone `Main Hall` with `Table 1`–`Table 4`.
- **RST** — `showDetails` returns the active zones and tables that Home uses.
- **ORD / BILL** — set the table status and the running order shown on a table.
- **IMPEX** — exporting zones and tables.
- **ADM** — panel sign-in and navigation.

## Decisions

Statuses per [conventions.md → Decision status](../../conventions.md#decision-status), set 2026-11-24. Cross-cutting items: [project-overview.md → Known gaps and targets](../../project-overview.md#known-gaps-and-targets).

| ID | Status | Short | Tests |
|---|---|---|---|
| TBL-D1 | **Fixed in the app** (found 2026-11-24) | Table Management list always empty | 02-T01–T05, 02-T07 |
| TBL-D2 | **To fix** (with X-D4, X-D8) | No integrity checks in the API or the panel | 01-D07, 01-D08, 01-D10, 01-D17; 02-D06–D08, 02-D10, 02-D11; 03-D06–D09; 01-T09, 01-T10, 02-T08, 03-T09 |
| TBL-D3 | **To fix** (X-D1) | Any role may change zones and tables | 01-D09, 02-D09, 02-D17 |
| TBL-D4 | **To fix** | List endpoints: duplicate rows, broken filters | 01-D14, 02-D15, 02-D20; 02-T10 |
| TBL-D5 | **To fix** (new 2026-11-24) | Zone add / rename failures are silent in the app | 01-T07, 01-T09, 01-T10 |
| AUTH-D2 (inherited) | **To fix** (X-D2) | Switched-off / unapproved callers keep access | 01-D12, 02-D17 |

- **TBL-D1 — Fixed in the app** (seen in `POS_APP-kot` on 2026-11-24; the app has no history to date it). `TableConfigScreen` compares `zoneId` as strings, so the selected zone's tables are listed (active and inactive, not deleted) and can be edited and switched.
- **TBL-D2 — To fix (2026-11-24, was accepted 2026-11-10).** *Target, API:* `zoneId` on table create / update must be a non-deleted zone of the caller's restaurant, else 422 `Invalid zone` (X-D8, also for a malformed ID); the duplicate table-name check (same zone, not deleted, any case, trimmed) runs on create **and** on any update that changes the name or the zone → 422 `Table name already exists.`; zone names are unique per restaurant among non-deleted zones (any case, trimmed) on create and rename → 422 `Zone name already exists.`; `capacity` `integer|min:1|max:2147483647`; `zoneName` / `tableName` `max:255`; deleting a zone (`isDelete: true`) also sets `isDelete = 1` on its tables; updating a zone or table that is unknown, deleted, another restaurant's or has a malformed ID → 404 `Zone not found` / `Table not found` (X-D4) instead of 500. *Target, panel:* the same name, uniqueness, capacity and zone rules on all store / update routes (JSON 422 `{ message, errors }`); `tables/update` writes only `zoneId`, `tableName`, `capacity`, `isActive`; `isActive` boolean; unknown ids or restaurants → 404. The pages' scripts stay unchanged (failures show nothing).
- **TBL-D3 — To fix (2026-11-24, was accepted 2026-11-10), via [X-D1](../../project-overview.md#known-gaps-and-targets).** *Target:* all four `/api/tables/*` endpoints are owner-only — only the owner's setup screens use them; a waiter or chef gets 403 `You do not have permission to perform this action`. Home keeps using `restaurants/showDetails` (RST), which every role may call.
- **TBL-D4 — To fix (2026-11-24, was accepted 2026-11-10).** *Target:* `getTablesDetails` returns one row per table, with `orderId` = the same running order `showDetails` shows (the highest `orderId` with status 0 or 1); the `tableId` and `zoneId` filters (both endpoints) take 32-char hex, with or without dashes, and return the matching rows; any other value → 422 `Validation error`. Now visible in the app: a table with two running orders shows twice in Table Management.
- **TBL-D5 — To fix (new 2026-11-24).** *Today:* the Add / Edit Zone sheet closes before the server answers, so a failed add or rename shows nothing. *Target:* like the Table sheet — the sheet stays open while saving (button disabled), closes only when the server accepts, and on failure shows the server message (e.g. `Zone name already exists.`) or `Something went wrong. Please try again.` in red under the name field, values kept; typing clears it.
- **Inherited:** AUTH-D2 (To fix, X-D2). RST finding F2: a 16-byte non-ASCII name comes back as hex.

Original decision texts (history):

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
- **Inherited (2026-11-10):** AUTH-D2. RST finding F2: a 16-byte non-ASCII name comes back as hex.

## Facts worth knowing

- Switching a zone on or off through the API sets **all** its tables to the same state. Turning a zone back on also re-activates tables that were off on their own. The panel's zone switch doesn't touch tables.
- After a zone is switched in the app, its card shows `Tables: 0` until the screen is reopened: the save reply has no count.
- Adding or renaming a zone in the app shows no error when it fails. The sheet closes, and the zone simply doesn't appear or change (TBL-D5).
- Updating a zone or table that isn't the restaurant's (or doesn't exist) gives a 500 `Failed to save …`, not a 404 (target 404 — TBL-D2 / X-D4).

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Choosing a table for an order on Home, and table status changes caused by orders — ORD.
- Floors and rooms — ROOM.
- Import / export — IMPEX.
