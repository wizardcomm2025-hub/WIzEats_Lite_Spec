# Staff management — UI / UX

Screen text exactly as in the code on 2026-11-10; re-checked in `POS_APP-kot` 2026-11-24.

## 1. User Management screen

**Where:**

- **Phone:** menu → `Account` → `User Management`, for every role.
- **Tablet:** Setup (owner); the users button in the rail (waiter, chef).

| Part | Owner | Waiter / chef |
|---|---|---|
| Header | `USER MANAGEMENT` · restaurant name · `Manage your staff members` | same |
| Card title | `Staff Members` + button `Add Staff` | `Your Profile` |
| Limit line | `You can have 1 active chef and 1 active waiter.` — or, with both taken, `You have 1 active chef and 1 active waiter. Disable one to add another.` (button greyed) | — |
| Row | Name · phone · role chip (as stored, first letter capital) · switch · `Active` / `Inactive`; inactive rows faded | Name · phone · role chip; no switch |
| Empty | `No users added yet` | same |
| Loading (nothing shown yet) | `Loading staff...` | same |

### Switch confirmation

| Direction | Title | Message | Button |
|---|---|---|---|
| Off | `Deactivate staff member?` | `Are you sure you want to deactivate "<name>"?` | `Deactivate` (red) |
| On | `Activate staff member?` | `Are you sure you want to activate "<name>"?` | `Activate` |

The confirm shows a spinner on the row, then the toast `Staff member "<name>" deactivated` / `Staff member "<name>" activated`, and the row updates (since 2026-11-17 — STAFF-D1 fixed). If the server refuses, the toast shows its message and the switch stays as it was.

Today the switch can activate a member even when that role is already full (no limit check). *Target (STAFF-D5):* tapping the switch of an inactive member whose role is full shows the toast `You can have 1 active chef and 1 active waiter.` and no dialog.

## 2. Add Staff sheet

| Part | Content |
|---|---|
| Title | `Add Staff` |
| `NAME` | placeholder `Enter staff name`; empty → `Name is required` |
| `PHONE` | placeholder `Enter phone number`; phone keypad; max 10; empty → `Phone number is required`; wrong → `Enter a valid 10-digit number starting with 6-9` |
| `ROLE` | chips `Chef`, `Waiter`; a taken role is greyed with `Already added`; none picked → `Please select a role` |
| Button | `Create Staff`; spinner while saving |

Errors show in red under the field after tapping `Create Staff`.

| Result | What the owner sees |
|---|---|
| Created | Toast `Staff created successfully`; sheet closes; list reloads |
| Server error | Toast with the server message, e.g. `Failed to create staff member`, `The name field is required.` (target STAFF-D3: `This phone number is already registered.` for a used phone) |
| No network | Toast `Failed to create staff` |

## 3. Panel — user management page

`/admin/user-management/<restaurant id>`. Tab title `Analytics`; page title `User Overview`, subtitle `User management`.

| Part | Content |
|---|---|
| Button | `+ Add User` — does nothing |
| Table | `SL No` · `Name` · `Phone` · `Role` · `Active` (switch — not saved) · `Actions` (edit icon); 3 rows per page with page links |
| Edit dialog | `Edit User` · `Name` · `Role` (free text) · `Status` (`Active` / `Inactive`) · `Cancel` · `Update` |
| Saved | Alert `User Updated Successfully`; page reloads |
| Error / session gone | Nothing |
| *Target (STAFF-D6)* refused (422 / 404) | Nothing — the script is unchanged |
