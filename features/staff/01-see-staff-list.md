# 01 — See the staff list

## Summary

The owner opens User Management and sees every waiter and chef of the restaurant, with each one's phone, role and Active / Inactive state. A waiter or chef opening the same screen sees only their own row, under `Your Profile`.

## Prerequisites

- **Backend:** `POST /api/staff/getStaffDetails` — [api-contract.md](./api-contract.md#post-apistaffgetstaffdetails)
- **Data:** `users` — [data-model.md](./data-model.md)
- **Depends on:** AUTH (token)

## Actors & preconditions

- **Actor:** any signed-in user of the restaurant.
- **Trigger:** opens User Management:
  - **Phone:** menu → `Account` → `User Management`, for every role.
  - **Tablet:** Setup for the owner; the rail's users button for waiter and chef.
- **Preconditions:** signed in, PIN changed.

## Main flow

1. The screen shows the header `USER MANAGEMENT` / restaurant name / `Manage your staff members` and, while loading, `Loading staff...`.
2. **Owner:** the app posts `{}`. The server returns all users of the caller's restaurant whose role is waiter or chef, in either case (the owner isn't included).
3. **Waiter / chef:** the app posts `{ userId: <own id> }`. The server returns that row, but only if it is a waiter or chef of the same restaurant.
4. **Owner view:** card `Staff Members`, the `Add Staff` button, and a limit line:
   - `You can have 1 active chef and 1 active waiter.`
   - once both are taken: `You have 1 active chef and 1 active waiter. Disable one to add another.`

   Each row shows name, phone, a role chip, an Active/Inactive switch and the label `Active` / `Inactive`. Inactive rows are faded.
5. **Waiter / chef view:** card `Your Profile`, their own row, and no switch, button or limit line.

## Alternative flows

- **No staff yet.** `No users added yet`.
- **Server error / no network.** The error toast shows the server message, or `Failed to fetch staff`; the list stays as it was.
- **A user whose role was changed to something else in the panel** (for example `manager`, or empty). That user is not listed.

## Acceptance criteria

1. Given OWN-A with WTR-A and CHF-A, when User Management opens, then both rows show with name, phone, role chip and `Active`, and the owner isn't listed.
2. Given WTR-A, when the screen opens, then only WTR-A's own row shows under `Your Profile`, with no switch and no `Add Staff`.
3. Given an active chef and an active waiter, then `Add Staff` is greyed and the limit line says `Disable one to add another.`
4. Given a restaurant with no staff, then `No users added yet` shows.
5. Given any request, then only the caller's restaurant's staff are returned.

## Platform notes

### POS API

- `StaffController::getStaffDetails` → `StaffService::fetch`.
- `userId` is optional and must be 32-character hex; anything else gives a 500.
- No role check today (STAFF-D3): a waiter calling with `{}` gets the whole list. *Target (X-D1):* a waiter or chef always gets only their own row.

### Data — MySQL

Read only: `users` where `restaurantId` is the caller's and `role IN ('waiter','chef')`, compared without regard to case.

### POS app

`UserManagement` screen. `GetStaffProfiles(userId)` passes no `userId` for the owner and the user's own ID for anyone else. `StaffListCard` shows the switch only to the owner.

### Super-admin panel

See [04](./04-edit-users-in-panel.md).

### Integrations

None.

## Configuration

None.

## Out of scope

- The owner's own account details — RST / AUTH.

## Open questions

- None open.
