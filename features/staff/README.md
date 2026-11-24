# Staff management

The owner adds waiters and chefs to the restaurant from the app's **User Management** screen. A new staff member signs in with their phone number and the first PIN `123456`, and must change it straight away. The app allows one active chef and one active waiter at a time (today only when adding; target also when switching on, and on the server — STAFF-D5). Waiters and chefs open the same screen and see only their own row. In the super-admin panel, a super admin can list a restaurant's users and edit their name, role and status. The spec records today's behaviour plus targets (re-synced 2026-11-24 against API commit d54d6c8 and `POS_APP-kot`): STAFF-D1 is fixed in code; STAFF-D2, D3, D5 and D6 are *To fix*; STAFF-D4 stays accepted.

**Feature code:** `STAFF` (test IDs `STAFF-01-T01`, `STAFF-03-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [See the staff list](./01-see-staff-list.md) | Owner (all staff); waiter / chef (own row) | [tester](./test-cases/01-see-staff-list.tester.md) | [developer](./test-cases/01-see-staff-list.developer.md) |
| 02 | [Add a staff member](./02-add-staff.md) | Owner (today any signed-in user through the API — STAFF-D3) | [tester](./test-cases/02-add-staff.tester.md) | [developer](./test-cases/02-add-staff.developer.md) |
| 03 | [Switch a staff member on or off](./03-switch-staff-on-off.md) | Owner (today any signed-in user through the API — STAFF-D2) | [tester](./test-cases/03-switch-staff-on-off.tester.md) | [developer](./test-cases/03-switch-staff-on-off.developer.md) |
| 04 | [Edit users in the panel](./04-edit-users-in-panel.md) | Super admin (web panel) | [tester](./test-cases/04-edit-users-in-panel.tester.md) | [developer](./test-cases/04-edit-users-in-panel.developer.md) |

## Key concepts

See the [glossary](../../project-overview.md#glossary) (User, Role). Specific to this feature:

- **Staff member** — a `users` row with role waiter or chef in the restaurant. The owner is a user too, but is not listed in the app.
- **Active / Inactive** — `users.status` 1 / 0. Today Inactive blocks the **next** sign-in only; target (AUTH-D2 / X-D2) every call is refused too.
- **First PIN** — `123456` for every staff member created in the app. `mustChangePin = 1` forces a change at first sign-in (AUTH 04).
- **Staff limit** — at most one active chef and one active waiter. Today only the app's Add Staff sheet checks it; target the server checks it on create and on switching on, and the app's switch too (STAFF-D5). The super-admin panel is not limited.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api/api/specs/staff/` (created with the first plan)

## Depends on

- **AUTH** — sign-in with the first PIN, the forced PIN change, AUTH-D2 (today switching off only blocks the next sign-in; target X-D2 blocks every call).
- **ONB** — the owner account exists.
- **ADM** — panel sign-in and navigation.
- **WAIT / KITCH** — what waiters and chefs do once signed in (their dashboards and KOT endpoints live under `/api/waiter` and `/api/chef`, specified there).

## Decisions

Statuses per [conventions.md → Decision status](../../conventions.md#decision-status), set 2026-11-24. Cross-cutting items: [project-overview.md → Known gaps and targets](../../project-overview.md#known-gaps-and-targets).

| ID | Status | Short | Tests |
|---|---|---|---|
| STAFF-D1 | **Fixed in code 2026-11-17** | App switch sent `isActive`, server wanted `status` | 03-D01, 03-D02, 03-T01, 03-T02 |
| STAFF-D2 | **To fix** (with X-D1, X-D4) | Update finds any user of any restaurant; any role may call it | 03-D03–D05, 03-D07 |
| STAFF-D3 | **To fix** (with X-D1) | Any role may create staff; any `role` text; used phone → 500 | 02-D05–D07, 02-D12; 02-T06 |
| STAFF-D4 | Accepted (X-D6) | Fixed first PIN `123456` | 02-T02, 02-T11 |
| STAFF-D5 | **To fix** (new 2026-11-24) | One-chef / one-waiter limit not enforced on the server or by the app's switch | 02-D08, 02-D15, 03-D17; 03-T06 |
| STAFF-D6 | **To fix** (new 2026-11-24, with X-D4) | Panel user edit stores anything; unknown id looks like success | 04-D02, 04-D04–D06, 04-D10; 04-T07 |
| X-D1 (list) | **To fix** | A waiter / chef gets the whole staff list from the API | 01-D03 |
| AUTH-D2 (inherited) | **To fix** (X-D2) | Switched-off / unapproved callers keep access | 01-D08, 02-D11, 03-D01, 03-D11; 03-T05 |

- **STAFF-D1 — Fixed in code 2026-11-17.** `UpdateStaffRequest` now requires `isActive` (boolean), which the service writes to `users.status`, and the response carries the updated user. The app's switch works.
- **STAFF-D2 — To fix (2026-11-24, was accepted 2026-11-10).** *Target:* `staff/update` is owner-only (X-D1: 403 `You do not have permission to perform this action`). The user must be a waiter or chef (any case) of the caller's restaurant; otherwise — another restaurant's user, an owner (including the caller), unknown or malformed ID — 404 `Staff member not found`, nothing written.
- **STAFF-D3 — To fix (2026-11-24, was accepted 2026-11-10).** *Target:* `staff/create` is owner-only (X-D1). `role` must be `chef` or `waiter` in any case and is stored as sent → otherwise 422 `The selected role is invalid.` A phone already in `users` → 422 `This phone number is already registered.` (checked before insert; the unique key still guards a race, which also answers 422, never 500).
- **STAFF-D4 — Accepted** (re-confirmed 2026-11-24, covered by X-D6).
- **STAFF-D5 — To fix (new 2026-11-24).** *Today:* the limit is checked only by the Add Staff sheet. The switch deliberately allows activating (`userManagement.tsx`: "Enabling and disabling are always allowed"), and the server checks nothing, so a restaurant can end up with two active chefs or waiters. *Target:* the server refuses, with 422 `You can have 1 active chef and 1 active waiter.`, a `create`, or an `update` with `isActive: true`, when another active user of the restaurant has the same role (any case). The app's switch shows that text as a toast instead of the Activate dialog when the role is already full. The super-admin panel is exempt.
- **STAFF-D6 — To fix (new 2026-11-24).** *Today:* see [04](./04-edit-users-in-panel.md) — the panel writes `name`, `role`, `status` exactly as sent; empty or non-numeric status → 500; `status` 5 stored; any role text; unknown id answers success. *Target:* `name` required; `role` in `owner`, `waiter`, `chef` (any case); `status` in `0`, `1` → 422 JSON (Laravel messages); unknown or missing `id` → 404 / 422 (X-D4). The page's script is unchanged, so failures still show nothing.
- **X-D1 on the list.** *Target:* `getStaffDetails` for a waiter or chef returns only the caller's own row, whatever `userId` is sent (the app already asks only for that).
- **Inherited:** AUTH-D2 (To fix, X-D2).

Original decision texts (history):

- **STAFF-D1 (2026-11-10):** The app's Active/Inactive switch stays as it is. The app sends `isActive`, but the server requires `status`, so every attempt answers 422 `The status field is required.`. The app shows that as a toast, and the staff member keeps the status they had. Today a status can only be changed through the panel or a direct API call. Accepted.
- **STAFF-D2 (2026-11-10):** The update endpoint finds the user by ID alone. There is no restaurant check and no role check. Any signed-in user can rename, or switch on or off, any user of any restaurant, including owners and themselves. Accepted risk.
- **STAFF-D3 (2026-11-10):** No role checks on any staff endpoint, and no rules on create beyond the field formats:
  - any signed-in user can create staff;
  - `role` can be any text, including `owner`;
  - the one-chef, one-waiter limit is enforced only in the app;
  - a phone number that is already used gives a 500 `Failed to create staff member`.

  Accepted risk.
- **STAFF-D4 (2026-11-10):** Every staff member created in the app gets the fixed first PIN `123456`. Anyone who knows the number can sign in before the staff member does, and set their own PIN. Accepted risk.
- **Inherited (2026-11-10):** AUTH-D2 (a switched-off user's existing tokens keep working; the restaurant's approval state isn't checked by the API).

## Facts worth knowing

- The app saves the role as `Chef` / `Waiter`, with a capital first letter. Sign-in and the app compare roles without regard to case, and the database collation does too.
- Since 2026-11-17 the update endpoint answers with the updated user in `data` (it used to be `null`).
- The panel's `+ Add User` button and its Active switches do nothing (no script behind them). Only the edit icon works.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Resetting a staff member's PIN for them — not offered (staff use Forgot PIN, AUTH 03).
- Deleting staff, or changing a staff member's phone — not offered.
- Waiter / chef dashboards and KOT handling — WAIT / KITCH.
