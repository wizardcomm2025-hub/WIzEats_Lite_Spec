# 02 — Manage tables in the app

## Summary

The owner opens Table Management, picks a zone, and sees that zone's tables, where they could add, edit or switch one. Today the list is always empty (TBL-D1). Adding a table works: it gets saved, and Home shows it. Editing and switching can't be reached in the app.

## Prerequisites

- **Backend:**
  - `POST /api/tables/getTablesDetails`
  - `POST /api/tables/tableSave`
  - `POST /api/tables/getTablesZoneDetails`

  See [api-contract.md](./api-contract.md).
- **Data:** `tables`, `zones`, `orders` (running order per table) — [data-model.md](./data-model.md)
- **Depends on:** [01](./01-manage-zones.md); RST 01 (Home reads the tables)

## Actors & preconditions

- **Actor:** owner. Through the API, any signed-in user (TBL-D3).
- **Trigger:** menu → `Table Management` (phone), or Setup (tablet).
- **Preconditions:** at least one zone exists.

## Main flow (as it is today)

1. The header shows `TABLE MANAGEMENT` / restaurant name / `Manage restaurant tables`, a dark zone picker (the first zone in the list is selected) and a green `+`. While loading it shows `Loading tables...`.
2. The app loads the zones and all tables (`getTablesDetails` with `{}`). It keeps only the tables whose `Number(zoneId)` equals the selected zone's ID, which is never true (TBL-D1). The screen shows `No Tables Found` / `Set up tables to start taking orders and managing your dining service.`
3. The zone picker opens a list headed `SELECT ZONE` with all zones, including inactive ones. Any choice shows the same empty state.
4. **Add:** `+` opens `Add Table` with the fields `Table Name` and `Capacity` and the button `Save Table`. The app checks:
   - `Table name is required`;
   - `Capacity is required`;
   - `Must be a number`;
   - `Capacity must be at least 1`.

   It sends `{ tableName, capacity, zoneId: <selected zone> }`, and the button reads `Saving...` while it waits.
5. The server checks for a duplicate name in that zone: same name, ignoring case, not deleted. If there is one, it answers 422 `Table name already exists.`, shown in red under the name field with the sheet left open. Otherwise it creates the table (active, status 0) and answers 200.
6. On success the sheet closes and the toast says `Table created`. The list stays empty (TBL-D1). Home shows the new table at its next refresh, if the zone is active.

## Main flow (edit / switch — through the API only today)

- **Edit:** `{ tableId, tableName?, capacity?, zoneId?, isActive?, status?, isDelete? }`. The server finds the table in the caller's restaurant and sets the fields that were sent. There is no duplicate check on rename, and `zoneId` can be any ID (TBL-D2).
- **Switch:** `{ tableId, isActive }`. The app's dialog would be `Deactivate table?` / `Activate table?` with the toast `Table "<name>" deactivated`, but there is no card to tap.

## Alternative flows

- **No zones.** No picker and no `+`; the screen shows `No Zones Found`.
- **Server error on add.** The message appears under the name field, or `Something went wrong. Please try again.`; the values are kept.
- **Name over 255 characters.** 500 `Failed to save table`.
- **Table added to an inactive zone.** Saved; it isn't shown on Home until the zone is on.
- **Table name differing only in case** (`table 1` vs `Table 1`). Refused as a duplicate on create.

## Acceptance criteria

1. Given OWN-A opens Table Management, then the picker shows `Main Hall` and the screen shows `No Tables Found`, although Main Hall has four tables (TBL-D1).
2. Given `Table 5`, capacity `4` is saved, then the toast `Table created` shows, the list stays empty, and Home shows `Table 5` in Main Hall.
3. Given `Table 1` is added to Main Hall again, then `Table name already exists.` shows under the name field and the sheet stays open.
4. Given an empty name or capacity, `abc`, or `0`, then the matching message shows and nothing is sent.
5. Given the API switches `Table 2` off, then Home no longer shows it.
6. Given the API renames `Table 2` to `Table 1`, then it is saved (TBL-D2), and Home shows two `Table 1`.

## Platform notes

### POS API

- `TableController::tableSave` / `getTablesDetails` → `TableService::saveTable` / `getTables`.
- Create rules: `tableName` required string, `zoneId` required string, `capacity` required integer.
- `status` in 0, 1, 2; `isActive` and `isDelete` boolean.

### Data — MySQL

- `tables` insert or update.
- `getTables` left-joins `orders` with status 0 or 1, so a table with two running orders appears twice (TBL-D4).

### POS app

- `TableConfigScreen` filters with `Number(t.zoneId) === selectedZone.zoneId` (TBL-D1).
- `table.slice` keeps `saving` apart from `loading`, so the sheet stays open while saving.

### Super-admin panel

See [03](./03-zones-tables-in-panel.md).

### Integrations

None.

## Configuration

None.

## Out of scope

- Table status changes from orders — ORD / BILL.

## Open questions

- None open. Related decisions: TBL-D1, TBL-D2, TBL-D4.
