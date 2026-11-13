# 03 — Zones and tables in the panel

## Summary

A super admin manages a restaurant's zones and tables on two panel pages. On both pages they can add rows and edit name, capacity, zone and status through dialogs. Only the zone page's on/off switch saves; the table page's switch does nothing. Nothing is validated (TBL-D2).

## Prerequisites

- **Backend (web):** `GET /admin/zones-management/{restaurantId}`, `POST /admin/zones/store`, `/zones/update`, `/zones/toggle`; `GET /admin/table-management/{restaurantId}`, `POST /admin/tables/store`, `/tables/update`, `/admin/table-toggle-status` — [api-contract.md](./api-contract.md#panel-routes-web)
- **Data:** `zones`, `tables`
- **Depends on:** ADM

## Actors & preconditions

- **Actor:** super admin signed in to the panel.
- **Trigger:** opens a restaurant's zone or table page.
- **Preconditions:** the restaurant exists.

## Main flow — zones page

1. The page shows:
   - the tab title `Analytics`, and the heading `Category Overview` / `Category management` (copy as-is);
   - a `+ Add Zone` button;
   - a table with the columns `SL No` · `Zone Name` · `Visible` (switch) · `Actions` (edit icon). There are 10 rows per page, and deleted zones are included.
2. **Add:** the `Add Zone` dialog has `Zone Name`, `Status` (`Active` / `Inactive`), `Cancel` and `Save`. Save posts, then the page reloads (no alert).
3. **Edit:** the `Edit Zone` dialog has the same fields and an `Update` button, which posts and reloads.
4. **Visible switch:** posts at once; there is no message and no reload. The zone's tables are **not** changed (unlike the app).

## Main flow — tables page

1. The page shows:
   - the tab title `Analytics`, and the heading `Tables Overview` / `Tables management`;
   - a `+ Add Table` button;
   - a table with the columns `SL No` · `Zone Name` · `Table Name` · `Capacity` · `Visible` · `Actions`. There are 10 rows per page, and deleted tables are included.
2. **Add:** the `Add Table` dialog has `Zone` (a list of every zone of the restaurant), `Table Name`, `Capacity`, `Status`, `Cancel` and `Save`. On success the alert `Table Added Successfully` shows and the page reloads.
3. **Edit:** the `Edit Table` dialog has the same fields and an `Update` button. On success the alert `Updated Successfully` shows and the page reloads.
4. **Visible switch:** flips on screen only; nothing is saved.

## Alternative flows

- **Empty name or capacity.** The database refuses it (500). The page shows no alert and doesn't reload; nothing is saved.
- **Duplicate table or zone name.** Saved (TBL-D2).
- **Capacity `0`, `-3` or text.** `0` and `-3` are saved; text gives a 500 and nothing is saved.
- **Session or CSRF expired.** Nothing happens.

## Acceptance criteria

1. Given REST-A's zone page, then `Main Hall` is listed with its switch on.
2. Given `Terrace` is added as Active, then the page reloads with `Terrace`, and the app's Zone Management shows it.
3. Given `Main Hall`'s switch is turned off, then after a reload it is off, the app's Home hides Main Hall, and the four tables are still active in the database.
4. Given REST-A's table page, then `Table 1`–`Table 4` are listed with `Main Hall`.
5. Given `Table 9`, capacity `2`, is added in `Main Hall`, then the alert `Table Added Successfully` shows and Home shows `Table 9`.
6. Given `Table 2` is moved to `Terrace` in the edit dialog, then the alert `Updated Successfully` shows and Home shows `Table 2` under Terrace.
7. Given a table's Visible switch is clicked and the page reloaded, then the stored value is back.

## Platform notes

### POS API

- Web routes behind `web` (CSRF) and `auth:admin`. `AdminController` has no validation.
- `updateTable` writes **every** field it is sent except `id`, `tableId`, `zoneId` and `restaurantId` (`zoneId` is converted to binary). So it can also set `status`, `isDelete` or any other column.

### Data — MySQL

- Plain query-builder writes; the zone switch doesn't cascade to tables.
- A table `status` outside 0–2 breaks every API response that includes that table (see the developer cases).

### POS app

Shows the result at the next refresh (Home, Zone Management).

### Super-admin panel

`resources/views/admin/zones.blade.php`, `tables.blade.php`. Values are HTML-escaped.

### Integrations

None.

## Configuration

Page size 10 (hard-coded).

## Out of scope

- Panel sign-in and navigation — ADM.

## Open questions

- None open. Related decision: TBL-D2.
