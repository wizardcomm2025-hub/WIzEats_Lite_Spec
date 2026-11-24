# 03 — Switch a staff member on or off

## Summary

The owner taps a staff member's Active/Inactive switch and confirms. The app sends the change, and the server saves the new status (working since 2026-11-17 — STAFF-D1 fixed). A switched-off user is refused at their next sign-in; today their existing tokens keep working, and in the target they are refused on every call (AUTH-D2 / X-D2). Today the endpoint also changes any user of any restaurant (STAFF-D2) and does not check the one-chef / one-waiter limit (STAFF-D5); both are *To fix*.

## Prerequisites

- **Backend:** `POST /api/staff/update` — [api-contract.md](./api-contract.md#post-apistaffupdate)
- **Data:** `users.name`, `users.status` — [data-model.md](./data-model.md)
- **Depends on:** [01](./01-see-staff-list.md); AUTH (what Inactive means at sign-in and, in the target, on every call)

## Actors & preconditions

- **Actor:** the owner in the app. Today any signed-in user through the API (STAFF-D2); target owner only (X-D1).
- **Trigger:** taps the switch on a staff row.
- **Preconditions:** the staff list is shown (01).

## Main flow

1. The owner taps the switch on WTR-A.
2. A dialog asks `Deactivate staff member?` / `Are you sure you want to deactivate "<name>"?`, with buttons `Deactivate` and cancel. For an inactive member the dialog says `Activate staff member?` and `Activate`.
3. On confirm, the row shows a spinner. The app sends `{ userId, name, isActive }`.
4. The server finds the user and sets `name` and `status` (= `isActive`). *Target:* it first checks the caller is the owner (STAFF-D2 / X-D1), that the user is a waiter or chef of the caller's restaurant (STAFF-D2), and, when switching on, that no other active member has the same role (STAFF-D5).
5. It answers 200 `Staff member updated successfully` with the updated user in `data` (no `pin`).
6. The app updates the row locally (switch and `Active` / `Inactive`, faded when inactive) and shows the toast `Staff member "<name>" deactivated` / `Staff member "<name>" activated`.
7. If the member is now inactive, their next sign-in is refused (AUTH 02). Today their existing tokens keep working; target: their next call is refused with 403 `Your account is inactive.` and the app signs them out ([AUTH 06](../auth/06-resume-or-end-session.md)).

## Alternative flows

- **Activating when the role is full.** Today the app sends it and the server saves it, giving two active chefs (or waiters). *Target (STAFF-D5):* the app shows the toast `You can have 1 active chef and 1 active waiter.` without a dialog and sends nothing; a direct API call gets 422 with the same message.
- **Unknown user ID** (valid hex). 404 `Staff member not found`.
- **Another restaurant's user, an owner, or the caller themselves.** Today saved (STAFF-D2). *Target:* 404 `Staff member not found`, nothing changes.
- **Caller is a waiter or chef (API).** Today saved. *Target (X-D1):* 403 `You do not have permission to perform this action`.
- **ID not 32-character hex.** 500 `Failed to update staff member`. *Target (X-D4):* 404 `Staff member not found`.
- **`name` or `isActive` missing.** 422 `The name field is required.` / `The is active field is required.`
- **Server refuses.** The spinner goes away, the switch stays where it was, and the toast shows the server's message.
- **Double tap in the app.** Only one request: a second change for the same person is skipped while the first is in flight, and other rows' switches are disabled.
- **Cancel in the dialog.** Nothing is sent.

## Acceptance criteria

1. Given OWN-A taps WTR-A's switch and confirms, then the toast `Staff member "<WTR-A name>" deactivated` shows, the row shows `Inactive` (faded), and after reopening the screen WTR-A is still `Inactive`.
2. Given WTR-A is switched off, then WTR-A's next sign-in is refused. Today an existing token still works; *(target, AUTH-D2)* it gets 403 `Your account is inactive.`
3. *(Target, STAFF-D2)* Given the API is called with OWN-B's user ID (another restaurant), OWN-A's own ID, or by WTR-A, then nothing changes (404 / 404 / 403).
4. Given cancel in the dialog, then nothing is sent.
5. *(Target, STAFF-D5)* Given CHF-A is active and CHF-A2 is inactive, when the owner taps CHF-A2's switch, then the toast `You can have 1 active chef and 1 active waiter.` shows and CHF-A2 stays `Inactive`.
6. Given WTR-A inactive and no other active waiter, when the owner activates WTR-A, then the toast `Staff member "<name>" activated` shows and WTR-A can sign in again.

## Platform notes

### POS API

- `StaffController::update` → `StaffService::update`.
- Rules: `userId` required string; `name` required string max 255; `isActive` required boolean (renamed from `status` 2026-11-17).
- Today no restaurant, role or limit check (STAFF-D2, STAFF-D5).

### Data — MySQL

Updates `users.name`, `users.status` and `updated_at` for the given ID.

### POS app

- The `useStatusToggle` dialog; `UpdateStaffProfile` thunk with the payload `{ userId, name, isActive }`.
- The thunk's `condition` skips a repeat for the same person.
- On success the slice applies the change locally, without refetching.
- Today the switch is not limited ("Enabling and disabling are always allowed" in `userManagement.tsx`); target STAFF-D5 blocks activation when the role is full.

### Super-admin panel

The panel's own status switch does nothing. Its edit dialog can change the status ([04](./04-edit-users-in-panel.md)); the panel is not held to the staff limit.

### Integrations

None.

## Configuration

None.

## Out of scope

- Signing a user out remotely — not offered; the X-D2 target refuses their next call instead.

## Open questions

- None open. Related decisions: STAFF-D1 (fixed), STAFF-D2, STAFF-D5, AUTH-D2.
