# Staff management

The owner adds waiters and chefs to the restaurant from the app's **User Management** screen. A new staff member signs in with their phone number and the first PIN `123456`, and must change it straight away. The app allows one active chef and one active waiter at a time. Waiters and chefs open the same screen and see only their own row. In the super-admin panel, a super admin can list a restaurant's users and edit their name, role and status. The spec records today's behaviour; decisions STAFF-D1 to STAFF-D4 keep the current rules as accepted risks.

**Feature code:** `STAFF` (test IDs `STAFF-01-T01`, `STAFF-03-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [See the staff list](./01-see-staff-list.md) | Owner (all staff); waiter / chef (own row) | [tester](./test-cases/01-see-staff-list.tester.md) | [developer](./test-cases/01-see-staff-list.developer.md) |
| 02 | [Add a staff member](./02-add-staff.md) | Owner (any signed-in user through the API — STAFF-D3) | [tester](./test-cases/02-add-staff.tester.md) | [developer](./test-cases/02-add-staff.developer.md) |
| 03 | [Switch a staff member on or off](./03-switch-staff-on-off.md) | Owner (any signed-in user through the API — STAFF-D2) | [tester](./test-cases/03-switch-staff-on-off.tester.md) | [developer](./test-cases/03-switch-staff-on-off.developer.md) |
| 04 | [Edit users in the panel](./04-edit-users-in-panel.md) | Super admin (web panel) | [tester](./test-cases/04-edit-users-in-panel.tester.md) | [developer](./test-cases/04-edit-users-in-panel.developer.md) |

## Key concepts

See the [glossary](../../project-overview.md#glossary) (User, Role). Specific to this feature:

- **Staff member** — a `users` row with role waiter or chef in the restaurant. The owner is a user too, but is not listed in the app.
- **Active / Inactive** — `users.status` 1 / 0. Inactive blocks the **next** sign-in only (AUTH-D2).
- **First PIN** — `123456` for every staff member created in the app. `mustChangePin = 1` forces a change at first sign-in (AUTH 04).
- **Staff limit** — the app allows at most one active chef and one active waiter. The server doesn't check this.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api-dev_guid/specs/staff/` (created with the first plan)

## Depends on

- **AUTH** — sign-in with the first PIN, the forced PIN change, AUTH-D2 (switching off only blocks the next sign-in).
- **ONB** — the owner account exists.
- **ADM** — panel sign-in and navigation.
- **WAIT / KITCH** — what waiters and chefs do once signed in (their dashboards and KOT endpoints live under `/api/waiter` and `/api/chef`, specified there).

## Decisions

- **STAFF-D1 (2026-11-10):** The app's Active/Inactive switch stays as it is. The app sends `isActive`, but the server requires `status`, so every attempt answers 422 `The status field is required.`. The app shows that as a toast, and the staff member keeps the status they had. Today a status can only be changed through the panel or a direct API call. Accepted.
- **STAFF-D2 (2026-11-10):** The update endpoint finds the user by ID alone. There is no restaurant check and no role check. Any signed-in user can rename, or switch on or off, any user of any restaurant, including owners and themselves. Accepted risk.
- **STAFF-D3 (2026-11-10):** No role checks on any staff endpoint, and no rules on create beyond the field formats:
  - any signed-in user can create staff;
  - `role` can be any text, including `owner`;
  - the one-chef, one-waiter limit is enforced only in the app;
  - a phone number that is already used gives a 500 `Failed to create staff member`.

  Accepted risk.
- **STAFF-D4 (2026-11-10):** Every staff member created in the app gets the fixed first PIN `123456`. Anyone who knows the number can sign in before the staff member does, and set their own PIN. Accepted risk.
- **Inherited:** AUTH-D2 (a switched-off user's existing tokens keep working; the restaurant's approval state isn't checked by the API).

## Facts worth knowing

- The app saves the role as `Chef` / `Waiter`, with a capital first letter. Sign-in and the app compare roles without regard to case, and the database collation does too.
- The update endpoint answers `data: null` even when it succeeds: it looks the user up again with the hex ID, which never matches.
- The panel's `+ Add User` button and its Active switches do nothing (no script behind them). Only the edit icon works.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Resetting a staff member's PIN for them — not offered (staff use Forgot PIN, AUTH 03).
- Deleting staff, or changing a staff member's phone — not offered.
- Waiter / chef dashboards and KOT handling — WAIT / KITCH.
