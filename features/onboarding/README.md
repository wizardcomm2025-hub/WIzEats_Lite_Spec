# Onboarding & approval

A new restaurant owner enters an unregistered mobile number on the Login screen and fills in a short restaurant profile. The server creates the restaurant, its owner account and starter data (settings, a "Main Hall" zone with four tables, four order channels) and e-mails the owner a PIN. The restaurant then waits for a super admin to approve or reject it in the web panel; the owner is told by e-mail and the app shows the matching screen. The spec records today's behaviour; decisions ONB-D1 to ONB-D4 keep the current rules as accepted risks.

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

- **Starter data** — what a first onboarding creates: a `restaurant_configs` row with a unique 6-character `restaurantCode`, zone **Main Hall**, tables **Table 1–4** (capacity 4) in it, and order channels `DINE_IN` / `TAKEAWAY` / `ROOM_SERVICE` / `DELIVERY`.
- **Onboarding PIN** — the random 6-digit PIN e-mailed to the restaurant e-mail at registration. The owner must replace it at first sign-in (AUTH 04).
- **Approval** — `restaurants.isApproved`: 0 Pending, 1 Accepted, 2 Rejected. Decided only in the super-admin panel.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api-dev_guid/specs/onboarding/` (created with the first plan)

## Depends on

- **AUTH** — the Login phone step routes a new number here ([AUTH 01](../auth/01-check-phone.md)); approved owners sign in with the e-mailed PIN and are forced to change it ([AUTH 02](../auth/02-sign-in-with-pin.md), [AUTH 04](../auth/04-change-pin.md)). AUTH-D2 decides that approval is **not** enforced by the API.
- **RST** — the restaurant details endpoint (`/api/restaurants/showDetails`) used to re-read approval; later profile edits ([features/restaurant-settings/](../restaurant-settings/)). RST-D1: the Takeaway switch chosen here is only stored.
- **ADM** — super-admin sign-in and the rest of the panel.

## Decisions

- **ONB-D1 (2026-11-06):** `/api/restaurants/setup` keeps accepting a token (old app versions). With any valid token — any role, any approval state, onboarding done or not — it overwrites the caller's restaurant profile and settings, renames the restaurant's first zone to "Main Hall", sets the contact phone to the caller's phone, renames the caller to the owner name given, and issues a new token. Accepted risk. See [04](./04-legacy-setup-with-token.md).
- **ONB-D2 (2026-11-06):** The super-admin approve / reject endpoint keeps accepting any `isApproved` value with no check of the current state. The panel only offers Accept / Reject while Pending, but a crafted request can store any value or reverse a decision. Accepted risk. See [03](./03-approve-or-reject.md).
- **ONB-D3 (2026-11-06):** No recovery path for a mistyped restaurant e-mail or a lost onboarding PIN e-mail: Forgot PIN goes to the same address and the panel can't edit the e-mail. Support must fix the database. Accepted risk.
- **ONB-D4 (2026-11-06):** Owners of restaurants created by the legacy register endpoint (onboarding not done) stay on the Pending screen after sign-in; their restaurant never appears in the panel. Accepted.
- **Inherited from AUTH-D2:** Pending and Rejected restaurants are blocked only by the app's screens. Their tokens — including the one `/api/restaurants/setup` returns, which the app discards — work on every API endpoint.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Super-admin sign-in, restaurant list paging, analytics and data editing — ADM.
- Editing the restaurant profile and settings after onboarding — RST.
- Switching a restaurant on / off (`restaurants.status`) — ADM / RST.
