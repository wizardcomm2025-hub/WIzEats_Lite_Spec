# Floors & rooms — UI / UX

Screen text exactly as in the code on 2026-11-10; re-checked in `POS_APP-kot` 2026-11-24. Both screens are in the owner's menu only while Room Service is on (`Floor Management`, `Room Management`; on a tablet, under Setup).

## 1. Floor Management

| Part | Content |
|---|---|
| Header | `FLOOR MANAGEMENT` · restaurant name · `Manage hotel floors` · green `＋` |
| Card | Floor name · `Rooms: <n>` · edit button · switch; faded when inactive |
| Empty | `No Floors Found` / `Start by adding your first floor to organize your restaurant layout.` |
| Add sheet | `Add Floor` · `Floor name (e.g. Ground Floor)` · `Add Floor`; empty → `Floor name is required` |
| Edit sheet | `Edit Floor` · same field · `Save Changes`; on success the list reloads with the new name |
| Switch dialog | `Deactivate floor?` / `Are you sure you want to deactivate "<name>"?` · `Deactivate`; or `Activate floor?` · `Activate` |
| Switch result | Toast `Floor "<name>" deactivated` / `Floor "<name>" activated` (since 2026-11-16); on failure the server message or `Failed to toggle floor status`. The card shows `Rooms: 0` until the list reloads |
| Add / rename failure | Today nothing shown (the sheet has closed). *Target (ROOM-D6):* the sheet stays open; red text under the field with the server message (e.g. `Floor name already exists.`) or `Something went wrong. Please try again.` |

## 2. Room Management

| Part | Content |
|---|---|
| Header | `ROOM MANAGEMENT` · restaurant name · `Manage hotel rooms` · dark floor picker · green `+` (picker and `+` only when there are floors) |
| List | The selected floor's rooms (ROOM-D1 fixed): today active rooms only; *target (ROOM-D5)* inactive ones too, faded. A floor without rooms: `No Rooms Found` / `Create rooms to better organize your dining areas and service zones.`; with no floors `No Floors Found` |
| Card | Room name · `Capacity: <n>` · edit button · switch; faded when inactive |
| Loading | `Loading rooms...` |
| Add sheet | `Add Room` · `Room Name` · `Capacity` (number pad) · `Save Room`; the new room appears first in the list |
| Edit sheet | `Edit Room` · same fields · `Update Room`; the card updates |
| Field errors | `Room name is required` · `Capacity is required` · `Must be a number`; `0` passes today — *target (ROOM-D3)* `Capacity must be at least 1` |
| Switch dialog | `Deactivate room?` / `Are you sure you want to deactivate "<name>"?` · `Deactivate`; or `Activate room?` · `Activate` |
| Switch result | Toast `Room "<name>" deactivated` / `Room "<name>" activated`; on failure the server message or `Failed to toggle room status` |
| Add / edit failure | Today nothing shown (the sheet has closed). *Target (ROOM-D6):* as for floors, e.g. `Room name already exists.`, `Invalid floor` |
