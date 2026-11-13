# 01 — Manage zones in the app

## Summary

The owner opens Zone Management. They see every zone that isn't deleted, each with its table count, and can add a zone, rename one, or switch one on or off.

## Prerequisites

- **Backend:**
  - `POST /api/tables/getTablesZoneDetails`
  - `POST /api/tables/ZoneSave`

  See [api-contract.md](./api-contract.md).
- **Data:** `zones`, `tables` — [data-model.md](./data-model.md)
- **Depends on:** AUTH, ONB (the first zone)

## Actors & preconditions

- **Actor:** owner. Through the API, any signed-in user (TBL-D3).
- **Trigger:** menu → `Zone Management` (phone), or Setup (tablet).
- **Preconditions:** signed in as the owner.

## Main flow

1. The screen shows the header `ZONE MANAGEMENT` / restaurant name / `Manage restaurant seating areas` and a green `+` button. While loading it shows `Loading zones...`.
2. The app posts `{}` to `getTablesZoneDetails`. Each zone card shows the zone name, `Tables: <count>`, and an edit button and an on/off switch. Inactive zones are faded.
3. **Add:** `+` opens the sheet `Add Zone` with the field `Zone name (e.g. Garden)` and the button `Add Zone`. An empty name shows `Zone name is required`. On submit the sheet closes. The app sends `{ zoneName }`, and the new zone appears first in the list with `Tables: 0`.
4. **Rename:** the edit button opens `Edit Zone` with the name filled in, and a `Save Changes` button. On submit the sheet closes. The app sends `{ zoneId, zoneName }` and then reloads the list.
5. **Switch:** the switch opens a dialog: `Deactivate zone?` / `Are you sure you want to deactivate "<name>"?` (or `Activate zone?`). On confirm the app sends `{ zoneId, isActive }`. The server sets the zone **and all its tables** to that state, and the toast reads `Zone "<name>" deactivated` / `… activated`.
6. While a save is in flight, the whole screen shows `Loading zones...`.

## Alternative flows

- **No zones.** Empty state `No Zones Found` / `Add zones to divide your space into manageable sections for better service.`
- **Add or rename fails** (server error, no network). There is no message: the sheet has already closed, and the zone isn't added or renamed.
- **Switch fails.** Toast with the server message, or `Failed to update status`.
- **After a switch.** The card shows `Tables: 0` until the screen is reopened.
- **Duplicate zone name.** Allowed (TBL-D2).
- **Zone switched off.** Home no longer shows it or its tables. Switching it back on re-activates **all** its tables, including ones that were off on their own.
- **Delete** (API only, `isDelete: true`). The zone disappears from the lists and from Home. Its tables stay active in the database and in `getTablesDetails` (TBL-D2).

## Acceptance criteria

1. Given OWN-A, when Zone Management opens, then `Main Hall` shows with `Tables: 4`.
2. Given `Garden` is added, then it appears first with `Tables: 0`, and the Table Management zone list includes it.
3. Given `Main Hall` is renamed to `Hall A`, then the list shows `Hall A`, and Home shows `Hall A`.
4. Given `Main Hall` is switched off and confirmed, then the toast `Zone "Main Hall" deactivated` shows, the card is faded, and Home shows neither the zone nor its tables.
5. Given (4) and then switched back on, then the zone and all its tables are active.
6. Given an empty name, then `Zone name is required` shows and nothing is sent.
7. Given WTR-A calls `ZoneSave` through the API, then the change is saved (TBL-D3).

## Platform notes

### POS API

- `TableController::ZoneSave` / `getTablesZoneDetails` → `TableService::saveZone` / `getTablesZone`.
- A save with a `zoneId` is an update; without one it is a create.
- An update looks for the zone in the caller's restaurant only. If it isn't there, the answer is 500 `Failed to save zone`.

### Data — MySQL

- `zones` insert or update.
- When `isActive` is sent, `tables.isActive` is updated for every table of the zone, including deleted ones.

### POS app

- `ZoneManagementScreen`, `zone.thunks`, `zone.slice`.
- `loading` covers fetch, add, edit and toggle, and shows the full-screen loader for all of them.
- `state.error` is set, but the screen never shows it.

### Super-admin panel

See [03](./03-zones-tables-in-panel.md).

### Integrations

None.

## Configuration

None.

## Out of scope

- Deleting a zone from a screen — not offered.

## Open questions

- None open. Related decisions: TBL-D2, TBL-D3.
