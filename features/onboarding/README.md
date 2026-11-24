# Onboarding & approval

A new restaurant owner enters an unregistered mobile number on the Login screen and fills in a short restaurant profile. The server creates the restaurant, its owner account and starter data (settings, a "Main Hall" zone with four tables, four order channels) and e-mails the owner a PIN. The restaurant then waits for a super admin to approve or reject it in the web panel; the owner is told by e-mail and the app shows the matching screen. The spec records today's behaviour plus targets (re-synced 2026-11-24 against API commit d54d6c8): ONB-D1 and ONB-D2 are *To fix*, ONB-D3 and ONB-D4 stay accepted, and the cross-cutting X-D2 target (AUTH-D2) makes the API, not only the app, hold back unapproved restaurants.

**Feature code:** `ONB` (test IDs `ONB-01-T01`, `ONB-03-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Register a new restaurant](./01-register-restaurant.md) | New owner (not signed in) | [tester](./test-cases/01-register-restaurant.tester.md) | [developer](./test-cases/01-register-restaurant.developer.md) |
| 02 | [Wait for approval](./02-wait-for-approval.md) | Owner / staff of a Pending or Rejected restaurant | [tester](./test-cases/02-wait-for-approval.tester.md) | [developer](./test-cases/02-wait-for-approval.developer.md) |
| 03 | [Approve or reject a restaurant](./03-approve-or-reject.md) | Super admin (web panel) | [tester](./test-cases/03-approve-or-reject.tester.md) | [developer](./test-cases/03-approve-or-reject.developer.md) |
| 04 | [Legacy setup with a token](./04-legacy-setup-with-token.md) | Old app versions / any signed-in caller | Tester: n/a — no UI | [developer](./test-cases/04-legacy-setup-with-token.developer.md) |

## Key concepts

See the [glossary](../../project-overview.md#glossary) (Restaurant, Onboarding, Approval, Restaurant config, Zone, Table, Order channel). Specific to this feature:

- **Starter data** — what a first onboarding creates: a `restaurant_configs` row with a unique 6-character `restaurantCode`, zone **Main Hall**, tables **Table 1–4** (capacity 4) in it, and order channels `DINE_IN` / `TAKEAWAY` / `ROOM_SERVICE` / `DELIVERY`. Since 2026-11-16 each channel's `isActive` follows the request: `DINE_IN` always on, `TAKEAWAY` = `enableTakeAway`, `ROOM_SERVICE` = `enableRoomService`, `DELIVERY` = `enableDelivery`. With what the app sends, a new restaurant has Dine In, plus Takeaway if the owner switched it on.
- **Onboarding PIN** — the random 6-digit PIN e-mailed to the restaurant e-mail at registration. The owner must replace it at first sign-in (AUTH 04).
- **Approval** — `restaurants.isApproved`: 0 Pending, 1 Accepted, 2 Rejected. Decided only in the super-admin panel.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api/api/specs/onboarding/` (created with the first plan)

## Depends on

- **AUTH** — the Login phone step routes a new number here ([AUTH 01](../auth/01-check-phone.md)); approved owners sign in with the e-mailed PIN and are forced to change it ([AUTH 02](../auth/02-sign-in-with-pin.md), [AUTH 04](../auth/04-change-pin.md)). Today approval is **not** enforced by the API; the target (AUTH-D2 / [X-D2](../../project-overview.md#known-gaps-and-targets)) refuses every protected call of an unapproved restaurant with 403, except `restaurants/showDetails` (the Pending screen's check) and `auth/logout`.
- **RST** — the restaurant details endpoint (`/api/restaurants/showDetails`) used to re-read approval; later profile edits ([features/restaurant-settings/](../restaurant-settings/)). The Takeaway switch chosen here now really turns the Takeaway channel on or off (RST-D1 fixed 2026-11-16).
- **ADM** — super-admin sign-in and the rest of the panel.

## Decisions

Statuses per [conventions.md → Decision status](../../conventions.md#decision-status), set 2026-11-24. Cross-cutting items: [project-overview.md → Known gaps and targets](../../project-overview.md#known-gaps-and-targets).

| ID | Status | Short | Tests |
|---|---|---|---|
| ONB-D1 | **To fix** (with X-D1, X-D2) | Token path of `setup` open to any role and any restaurant | 04-D02, 04-D07, 04-D10–D13 |
| ONB-D2 | **To fix** (with X-D4) | Approve / reject stores any value; unknown id looks like success | 03-D06, 03-D07, 03-D09, 03-D18 |
| ONB-D3 | Accepted | No recovery for a mistyped e-mail or lost PIN mail | 01-D13 |
| ONB-D4 | Accepted | Legacy owners stuck on the Pending screen | 02-D04, 02-T12 |
| AUTH-D2 (inherited) | **To fix** (X-D2) | Unapproved restaurants use the API freely | 01-D01, 02-D02, 02-D03, 02-D06; 02-T13 |

- **ONB-D1 — To fix (2026-11-24, was accepted 2026-11-06).** *Today:* see the original text below; since 2026-11-16 a token call also overwrites each channel's `isActive` from the flags. *Target:* with a valid token, after validation, the server checks in this order and writes nothing on failure — (1) caller `users.status` = 0 → 403 `Your account is inactive.`; (2) caller has no restaurant → 404 `Restaurant not found` (as today); (3) restaurant `status` = 0 → 403 `Your restaurant is inactive.`; (4) caller's role is not `owner` → 403 `You do not have permission to perform this action` (X-D1); (5) restaurant `isOnboardingDone` = 1 → 409 `Restaurant is already onboarded`. Approval is **not** checked (a legacy restaurant is Pending). Bodies keep the setup shape: `{ "status": false, "message": …, "data": null, "token": null, "mustChangePin": null }`. Two parallel calls for the same restaurant: one 200, the other 409. A legacy owner can still finish setup once; nobody can re-run it.
- **ONB-D2 — To fix (2026-11-24, was accepted 2026-11-06).** *Today:* see below. *Target:* `isApproved` is validated `required|in:0,1,2`; a bad or missing value changes nothing and redirects back with Laravel's validation errors (with `Accept: application/json`: 422 `The is approved field is required.` / `The selected is approved is invalid.`) — never a 500. An unknown or malformed `{restaurantId}` → 404, nothing changes, no e-mail (X-D4). Changing an existing decision (including back to Pending) stays possible by a direct request; the panel still offers buttons only while Pending.
- **ONB-D3, ONB-D4 — Accepted** (re-confirmed 2026-11-24).
- **AUTH-D2 (inherited) — To fix.** *Today:* Pending and Rejected restaurants are blocked only by the app's screens; their tokens work on every API endpoint. *Target:* X-D2 — 403 `Your restaurant is not approved.` on every protected call except `restaurants/showDetails` and `auth/logout`, so the Pending screen keeps working. A switched-off user or restaurant is refused even on `showDetails`, and the app then signs out ([AUTH 06](../auth/06-resume-or-end-session.md)).

Original decision texts (history):

- **ONB-D1 (2026-11-06):** `/api/restaurants/setup` keeps accepting a token (old app versions). With any valid token — any role, any approval state, onboarding done or not — it overwrites the caller's restaurant profile and settings, renames the restaurant's first zone to "Main Hall", sets the contact phone to the caller's phone, renames the caller to the owner name given, and issues a new token. Accepted risk. See [04](./04-legacy-setup-with-token.md).
- **ONB-D2 (2026-11-06):** The super-admin approve / reject endpoint keeps accepting any `isApproved` value with no check of the current state. The panel only offers Accept / Reject while Pending, but a crafted request can store any value or reverse a decision. Accepted risk. See [03](./03-approve-or-reject.md).
- **ONB-D3 (2026-11-06):** No recovery path for a mistyped restaurant e-mail or a lost onboarding PIN e-mail: Forgot PIN goes to the same address and the panel can't edit the e-mail. Support must fix the database. Accepted risk.
- **ONB-D4 (2026-11-06):** Owners of restaurants created by the legacy register endpoint (onboarding not done) stay on the Pending screen after sign-in; their restaurant never appears in the panel. Accepted.
- **Inherited from AUTH-D2 (2026-11-06):** Pending and Rejected restaurants are blocked only by the app's screens. Their tokens — including the one `/api/restaurants/setup` returns, which the app discards — work on every API endpoint.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Super-admin sign-in, restaurant list paging, analytics and data editing — ADM.
- Editing the restaurant profile and settings after onboarding — RST.
- Switching a restaurant on / off (`restaurants.status`) — ADM / RST.
