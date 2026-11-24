# 02 — Manage tables in the app

## Summary

The owner opens Table Management, picks a zone, and sees that zone's tables. From there they can add a table, edit its name and capacity, or switch it on or off. Since the app fix (TBL-D1, fixed by 2026-11-24) the list shows the tables; before, it was always empty. Today the server does not check which zone a table belongs to, allows duplicate names on rename, and lists a table twice when it has two running orders; these are *To fix* (TBL-D2, TBL-D4).

## Prerequisites

- **Backend:**
  - `POST /api/tables/getTablesDetails`
  - `POST /api/tables/tableSave`
  - `POST /api/tables/getTablesZoneDetails`

  See [api-contract.md](./api-contract.md).
- **Data:** `tables`, `zones`, `orders` (running order per table) — [data-model.md](./data-model.md)
- **Depends on:** [01](./01-manage-zones.md); RST 01 (Home reads the tables)

## Actors & preconditions

- **Actor:** owner. Today any signed-in user through the API; target owner only (TBL-D3 / X-D1).
- **Trigger:** menu → `Table Management` (phone), or Setup (tablet).
- **Preconditions:** at least one zone exists.

## Main flow

1. The header shows `TABLE MANAGEMENT` / restaurant name / `Manage restaurant tables`, a dark zone picker (the first zone in the list is selected) and a green `+`. While loading it shows `Loading tables...`.
2. The app loads the zones and all tables (`getTablesDetails` with `{}`) and shows the tables of the selected zone — active and inactive, not deleted. Each card shows the table name, `Capacity: <n>`, an edit button and an on/off switch; inactive tables are faded.
3. The zone picker opens a list headed `SELECT ZONE` with all zones, including inactive ones. Choosing one shows its tables, or `No Tables Found` / `Set up tables to start taking orders and managing your dining service.` when it has none.
4. **Add:** `+` opens `Add Table` with the fields `Table Name` and `Capacity` and the button `Save Table`. The app checks `Table name is required`, `Capacity is required`, `Must be a number`, `Capacity must be at least 1`. It sends `{ tableName, capacity, zoneId: <selected zone> }`; the button reads `Saving...` while it waits.
5. The server checks for a duplicate name in that zone (same name ignoring case, not deleted): 422 `Table name already exists.` Otherwise it creates the table (active, status 0) and answers 200. *Target (TBL-D2):* it also checks the zone belongs to the restaurant and isn't deleted, and that capacity is at least 1.
6. On success the sheet closes, the toast says `Table created`, and the table appears in the list. Home shows it at its next refresh, if the zone is active.
7. **Edit:** the edit button opens `Edit Table` with the values filled in and the button `Update Table` (`Updating...`). The app sends `{ tableId, tableName, capacity }`. On success: toast `Table updated`, the card updates. Today a rename to a name already used in the zone is saved; *target (TBL-D2):* 422 `Table name already exists.`, shown under the name field.
8. **Switch:** the switch opens `Deactivate table?` / `Are you sure you want to deactivate "<name>"?` (or `Activate table?`). On confirm the app sends `{ tableId, isActive }`; the toast reads `Table "<name>" deactivated` / `Table "<name>" activated`. An inactive table is not shown on Home.

## Alternative flows

- **No zones.** No picker and no `+`; the screen shows `No Zones Found`.
- **Server error on add or edit.** The message appears in red under the name field, or `Something went wrong. Please try again.`; the sheet stays open with the values; editing either field clears it.
- **Switch fails.** Toast with the server message, or `Failed to toggle table status`; the switch stays as it was.
- **Name over 255 characters.** Today 500 `Failed to save table`; *target (TBL-D2)* 422 `The table name field must not be greater than 255 characters.`
- **Table added to an inactive zone.** Saved; it isn't shown on Home until the zone is on.
- **Table name differing only in case** (`table 1` vs `Table 1`). Refused as a duplicate on create (and, in the target, on rename).
- **A table with two running orders.** Today listed twice (TBL-D4); *target* once.
- **Waiter or chef through the API.** Today allowed; *target (X-D1)* 403 `You do not have permission to perform this action`.

## Acceptance criteria

1. Given OWN-A opens Table Management, then the picker shows `Main Hall` and the list shows `Table 1`–`Table 4` with `Capacity: 4` (TBL-D1 fixed).
2. Given `Table 5`, capacity `4` is saved, then the toast `Table created` shows, `Table 5` appears in the list, and Home shows `Table 5` in Main Hall.
3. Given `Table 1` is added to Main Hall again, then `Table name already exists.` shows under the name field and the sheet stays open.
4. Given an empty name or capacity, `abc`, or `0`, then the matching message shows and nothing is sent.
5. Given `Table 2` is switched off and confirmed, then the toast `Table "Table 2" deactivated` shows, the card fades, and Home no longer shows it.
6. *(Target, TBL-D2)* Given `Table 2` is renamed to `Table 1`, then `Table name already exists.` shows and nothing is saved. (Today it is saved, and Home shows two `Table 1`.)
7. *(Target, TBL-D4)* Given `Table 1` has two running orders, then the list shows it once.

## Platform notes

### POS API

- `TableController::tableSave` / `getTablesDetails` → `TableService::saveTable` / `getTables`.
- Create rules: `tableName` required string, `zoneId` required string, `capacity` required integer. `status` in 0, 1, 2; `isActive` and `isDelete` boolean.
- Target rules: see [api-contract.md](./api-contract.md#post-apitablestablesave) (TBL-D2, TBL-D4, X-D1, X-D4).

### Data — MySQL

- `tables` insert or update.
- `getTables` left-joins `orders` with status 0 or 1, so a table with two running orders appears twice today (TBL-D4).

### POS app

- `TableConfigScreen` keeps the tables whose `zoneId` equals the selected zone's (string comparison — TBL-D1 fixed; it used `Number(...)`, which never matched).
- `table.slice` keeps `saving` apart from `loading`, so the sheet stays open while saving.
- The switch uses `useStatusToggle` and `toggleTableStatus`.

### Super-admin panel

See [03](./03-zones-tables-in-panel.md).

### Integrations

None.

## Configuration

None.

## Out of scope

- Table status changes from orders — ORD / BILL.
- Deleting a table from a screen — not offered.

## Open questions

- None open. Related decisions: TBL-D1 (fixed), TBL-D2, TBL-D3, TBL-D4.
