# Floors & rooms — UI / UX

Screen text exactly as in the code on 2026-11-10. Both screens are in the owner's menu only while Room Service is on (`Floor Management`, `Room Management`; on a tablet, under Setup).

## 1. Floor Management

| Part | Content |
|---|---|
| Header | `FLOOR MANAGEMENT` · restaurant name · `Manage hotel floors` · green `＋` |
| Card | Floor name · `Rooms: <n>` · edit button · switch; faded when inactive |
| Empty | `No Floors Found` / `Start by adding your first floor to organize your restaurant layout.` |
| Add sheet | `Add Floor` · `Floor name (e.g. Ground Floor)` · `Add Floor`; empty → `Floor name is required` |
| Edit sheet | `Edit Floor` · same field · `Save Changes` |
| Switch dialog | `Deactivate floor?` / `Are you sure you want to deactivate "<name>"?` · `Deactivate`; or `Activate floor?` · `Activate` |
| Switch result today | Toast `Failed to save floor` (ROOM-D1) |
| Add / rename failure | Nothing shown |

## 2. Room Management

| Part | Content |
|---|---|
| Header | `ROOM MANAGEMENT` · restaurant name · `Manage hotel rooms` · dark floor picker · green `+` (picker and `+` only when there are floors) |
| List | Today always `No Rooms Found` / `Create rooms to better organize your dining areas and service zones.` (ROOM-D1); with no floors `No Floors Found` |
| Loading | `Loading rooms...` |
| Add sheet | `Add Room` · `Room Name` · `Capacity` (number pad) · `Save Room` |
| Edit sheet (unreachable) | `Edit Room` · same fields · `Update Room` |
| Field errors | `Room name is required` · `Capacity is required` · `Must be a number` |
| Save result today | Sheet closes; nothing shown; no room (ROOM-D1) |
