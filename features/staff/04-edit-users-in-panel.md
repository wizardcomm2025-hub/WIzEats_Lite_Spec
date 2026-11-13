# 04 — Edit users in the panel

## Summary

A super admin opens a restaurant's user management page in the web panel. It lists **all** of the restaurant's users (owner, waiters, chefs), three per page. The edit icon opens a dialog that changes name, role (free text) and status. The `+ Add User` button and the Active switches in the table are shown but do nothing.

## Prerequisites

- **Backend (web):** `GET /admin/user-management/{restaurantId}`, `POST /admin/user/update` — [api-contract.md](./api-contract.md#panel-routes-web)
- **Data:** `users` — [data-model.md](./data-model.md)
- **Depends on:** ADM (panel sign-in, navigation)

## Actors & preconditions

- **Actor:** super admin signed in to the panel.
- **Trigger:** opens the page; clicks the edit icon on a row.
- **Preconditions:** the restaurant exists.

## Main flow

1. The page shows:
   - the browser tab title `Analytics`, page title `User Overview`, subtitle `User management` (copy as-is);
   - a `+ Add User` button;
   - a table with `SL No` · `Name` · `Phone` · `Role` · `Active` (switch) · `Actions` (edit icon).

   Users appear in database order, 3 per page, with page links. `SL No` restarts at 1 on every page.
2. The admin clicks the edit icon. The dialog `Edit User` opens with `Name`, `Role` (free text) and `Status` (`Active` / `Inactive`), and the buttons `Cancel` and `Update`.
3. On `Update`, the page posts `{ id, name, role, status }` as JSON with the CSRF token.
4. The server writes the three values to that user, with no checks.
5. The page shows the alert `User Updated Successfully` and reloads.

## Alternative flows

- **`+ Add User`.** Nothing happens (the script it calls doesn't exist).
- **Active switch in the table.** It flips on screen only; nothing is saved, and a reload shows the stored value.
- **Role changed.** Any text is saved:
  - `owner` gives a waiter the owner UI at the next sign-in;
  - a role in another case (`WAITER`) works as before;
  - anything else (`manager`, empty) leaves the user with no menus, and drops them from the app's staff list.
- **Name emptied.** Saved as empty.
- **Status Inactive.** The user's next sign-in is refused; their existing tokens still work (AUTH-D2). This is the only UI way to free a staff slot today (STAFF-D1).
- **Server error, session expired, or CSRF expired.** No alert and no reload; nothing is saved.
- **Unknown user ID** (crafted). Answers success; nothing changes.

## Acceptance criteria

1. Given REST-A's page, then OWN-A, WTR-A and CHF-A are listed (3 per page) with name, phone and role.
2. Given WTR-A's name changed to `Waiter Two` and `Update` clicked, then the alert `User Updated Successfully` shows, the page reloads with the new name, and the owner's app list shows `Waiter Two` after a refresh.
3. Given WTR-A's status set to `Inactive`, then the app list shows WTR-A as `Inactive`, WTR-A's next sign-in is refused, and the owner can add a new waiter.
4. Given `+ Add User` is clicked, then nothing happens.
5. Given an Active switch in the table is clicked and the page reloaded, then the stored status shows again (nothing saved).
6. Given WTR-A's role set to `owner`, then at the next sign-in WTR-A gets the owner's menus.
7. Given no panel session, then nothing changes.

## Platform notes

### POS API

- Web routes behind `web` (session, CSRF) and `auth:admin`.
- `AdminController::userManagement` paginates by 3.
- `AdminController::updateUser` writes `name`, `role` and `status` exactly as sent, and always answers `{ "success": true }` unless an exception occurs.

### Data — MySQL

- Updates `users.name`, `role` and `status` by ID.
- `status` is NOT NULL, so an empty value gives a 500.
- `updated_at` is **not** set (query-builder update without it).

### POS app

Reflects the change on the next list load (01) and the next sign-in.

### Super-admin panel

`resources/views/admin/user-management.blade.php`. The values in the list and the dialog are HTML-escaped.

### Integrations

None.

## Configuration

Page size 3 (hard-coded).

## Out of scope

- Panel sign-in and navigation — ADM.
- Phone and PIN changes — not offered in the panel.

## Open questions

- None open. Related decisions: STAFF-D1, STAFF-D3.
