# Zones & tables — UI / UX

Screen text exactly as in the code on 2026-11-10; re-checked in `POS_APP-kot` 2026-11-24.

## 1. Zone Management (app)

Owner only: on a phone, menu → `Zone Management`; on a tablet, Setup.

| Part | Content |
|---|---|
| Header | `ZONE MANAGEMENT` · restaurant name · `Manage restaurant seating areas` · green `+` |
| Card | Zone name · `Tables: <n>` · edit button · on/off switch; faded when inactive |
| Empty | `No Zones Found` / `Add zones to divide your space into manageable sections for better service.` |
| Loading (also while saving) | `Loading zones...` |
| Add sheet | `Add Zone` · field `Zone name (e.g. Garden)` · button `Add Zone`; empty → `Zone name is required` |
| Edit sheet | `Edit Zone` · same field · button `Save Changes` |
| Switch dialog | `Deactivate zone?` / `Are you sure you want to deactivate "<name>"?` · `Deactivate` (red); or `Activate zone?` · `Activate` |
| Switch result | Toast `Zone "<name>" deactivated` / `… activated`; on failure the server message or `Failed to update status` |
| Add / rename failure | Today nothing shown (the sheet has closed). *Target (TBL-D5):* the sheet stays open; red text under the field with the server message (e.g. `Zone name already exists.`) or `Something went wrong. Please try again.` |

## 2. Table Management (app)

Owner only: on a phone, menu → `Table Management`; on a tablet, Setup.

| Part | Content |
|---|---|
| Header | `TABLE MANAGEMENT` · restaurant name · `Manage restaurant tables` · dark zone picker (selected zone name ▾) · green `+` (picker and `+` only when there are zones) |
| Picker list | `SELECT ZONE` · every zone, including inactive ones |
| List | The selected zone's tables (active and inactive) — TBL-D1 fixed. A zone without tables: `No Tables Found` / `Set up tables to start taking orders and managing your dining service.` With no zones: `No Zones Found` |
| Card | Table name · `Capacity: <n>` · edit button · switch; faded when inactive |
| Switch dialog | `Deactivate table?` / `Are you sure you want to deactivate "<name>"?` · `Deactivate` (red); or `Activate table?` · `Activate` |
| Switch result | Toast `Table "<name>" deactivated` / `Table "<name>" activated`; on failure the server message or `Failed to toggle table status` |
| Loading | `Loading tables...` |
| Add sheet | `Add Table` · `Table Name` · `Capacity` (number pad) · button `Save Table` (`Saving...` while saving) |
| Edit sheet | `Edit Table` · same fields · `Update Table` (`Updating...`) |
| Field errors | `Table name is required` · `Capacity is required` · `Must be a number` · `Capacity must be at least 1` |
| Server error | Red text under Table Name, e.g. `Table name already exists.` (target also `Invalid zone`, and on rename — TBL-D2), or `Something went wrong. Please try again.`; values kept; editing either field clears it |
| Saved | Sheet closes; toast `Table created` / `Table updated` |

## 3. Panel pages

| Page | Titles | Columns | Buttons and dialogs | Feedback |
|---|---|---|---|---|
| Zones `/admin/zones-management/<id>` | Tab `Analytics`; `Category Overview` / `Category management` | `SL No` · `Zone Name` · `Visible` · `Actions` | `+ Add Zone` → `Add Zone` (`Zone Name`, `Status` Active/Inactive, `Cancel`, `Save`); edit → `Edit Zone` (`Update`); Visible switch saves | Page reload after Save / Update; switch silent; errors silent |
| Tables `/admin/table-management/<id>` | Tab `Analytics`; `Tables Overview` / `Tables management` | `SL No` · `Zone Name` · `Table Name` · `Capacity` · `Visible` · `Actions` | `+ Add Table` → `Add Table` (`Zone`, `Table Name`, `Capacity`, `Status`, `Cancel`, `Save`); edit → `Edit Table` (`Update`); Visible switch does **nothing** | Alerts `Table Added Successfully` / `Updated Successfully` then reload; server errors silent (also the target TBL-D2 refusals) |

10 rows per page. `SL No` restarts at 1 on every page.
