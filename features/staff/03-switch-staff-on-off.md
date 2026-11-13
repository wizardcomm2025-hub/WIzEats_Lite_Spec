# 03 — Switch a staff member on or off

## Summary

The owner taps a staff member's Active/Inactive switch and confirms. The app sends the change, but the server rejects it because of a field-name mismatch, so the switch never takes effect from the app (STAFF-D1). The update endpoint itself works when called with the right field. It changes the name and status of any user in any restaurant (STAFF-D2). A switched-off user is blocked at their **next** sign-in only (AUTH-D2).

## Prerequisites

- **Backend:** `POST /api/staff/update` — [api-contract.md](./api-contract.md#post-apistaffupdate)
- **Data:** `users.name`, `users.status` — [data-model.md](./data-model.md)
- **Depends on:** [01](./01-see-staff-list.md); AUTH (what Inactive means at sign-in)

## Actors & preconditions

- **Actor:** the owner in the app. Through the API, any signed-in user (STAFF-D2).
- **Trigger:** taps the switch on a staff row.
- **Preconditions:** the staff list is shown (01).

## Main flow (app, today)

1. The owner taps the switch on WTR-A.
2. A dialog asks `Deactivate staff member?` / `Are you sure you want to deactivate "<name>"?`, with buttons `Deactivate` and cancel. For an inactive member the dialog says `Activate staff member?` and `Activate`.
3. On confirm, the row shows a spinner. The app sends `{ userId, name, isActive }`.
4. The server answers 422 `The status field is required.` (STAFF-D1).
5. The spinner goes away, the toast shows `The status field is required.`, and the switch stays where it was. Nothing changes.

## Main flow (API, with the right field)

1. A caller sends `{ userId: <32-char hex>, name, status: true | false }`.
2. The server finds the user by ID only, in any restaurant and of any role (STAFF-D2), and sets `name` and `status`.
3. It answers 200 `Staff member updated successfully` with `data: null`.
4. If `status` is now false, the user's next sign-in is refused (AUTH 02). Their existing tokens keep working (AUTH-D2).

## Alternative flows

- **Unknown user ID** (valid hex). 404 `Staff member not found`.
- **ID not 32-character hex.** 500 `Failed to update staff member`.
- **`name` or `status` missing.** 422 `The name field is required.` / `The status field is required.`
- **The owner switches themselves off through the API.** Saved; their next sign-in is refused.
- **Double tap in the app.** Only one request: a second change for the same person is skipped while the first is in flight, and other rows' switches are disabled.
- **Cancel in the dialog.** Nothing is sent.

## Acceptance criteria

1. Given OWN-A taps WTR-A's switch and confirms, then the toast `The status field is required.` shows and WTR-A stays `Active` (STAFF-D1).
2. Given the API is called with `status: false` for WTR-A, then 200, and WTR-A's next sign-in is refused while an existing token still works.
3. Given the API is called by WTR-A with OWN-B's user ID (another restaurant), then OWN-B is renamed or switched off (STAFF-D2).
4. Given cancel in the dialog, then nothing is sent.

## Platform notes

### POS API

- `StaffController::update` → `StaffService::update`.
- Rules: `userId` required string; `name` required string max 255; `status` required boolean.
- No restaurant or role check (STAFF-D2).

### Data — MySQL

Updates `users.name`, `users.status` and `updated_at` for the given ID.

### POS app

- The `useStatusToggle` dialog; `UpdateStaffProfile` thunk with the payload `{ userId, name, isActive }` (STAFF-D1).
- The thunk's `condition` skips a repeat for the same person.
- On success the slice would apply the change locally, without refetching; today it never succeeds.

### Super-admin panel

The panel's own status switch does nothing. Its edit dialog can change the status ([04](./04-edit-users-in-panel.md)).

### Integrations

None.

## Configuration

None.

## Out of scope

- Signing a user out remotely — not offered (AUTH-D2).

## Open questions

- None open. Related decisions: STAFF-D1, STAFF-D2.
