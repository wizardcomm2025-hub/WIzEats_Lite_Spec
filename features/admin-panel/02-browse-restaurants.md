# 02 — Browse restaurants

## Summary

After sign-in the admin sees `Restaurant Management`. Three counters sit at the top. Below is a directory of onboarded restaurants, 10 per page, newest first, with each restaurant's approval state. A row expands to show the restaurant's IDs and a `View Analytics` button.

## Prerequisites

- **Backend:** `GET /admin` (`?page=<n>`); see [api-contract.md](./api-contract.md#restaurant-list)
- **Data:** `restaurants` (with `isOnboardingDone` = 1)

## Actors & preconditions

- **Actor:** a signed-in super admin.

## Main flow

1. The header shows `Restaurant Management` / `Manage all registered restaurants and their compliance details`. Three counters follow, all limited to onboarded restaurants: `Total Restaurants`, `Active` (`status` 1) and `Inactive` (`status` 0). `Logout` is on the right.
2. `Restaurant Directory` has a search box (`Search restaurants...`) and a table with these columns:
   - `#`;
   - `Restaurant Details`: name and `ID: <hex>`;
   - `Owner`, `Contact`, `Location`;
   - `Status`: `Active` / `Inactive`;
   - `Approved`: `Accept` + `Reject` when pending, `Approved` or `Rejected` otherwise. The buttons are ONB-03.
3. Clicking a name expands the details row (one at a time). It shows:
   - `Restaurant ID`;
   - `GST ID`, or `No gst id Given`;
   - `FSSAI License`, or `No fssai id Given`;
   - `View Analytics`, which opens [03](./03-view-restaurant-overview.md).
4. The footer reads `Showing <first> to <last> of <total> results`, with page links.

## Alternative flows

- **Search:** typing hides the rows of **this page** that don't contain the text. Other pages are not searched (ADM-D3).
- **Page 2 and on:** the `#` column restarts at `01` (ADM-D3).
- **Onboarding not done** (e.g. legacy register, ONB): the restaurant isn't listed or counted.
- **No onboarded restaurants:** `No restaurants found` / `Get started by adding your first restaurant to the system.` and an `Add Restaurant` button that does nothing (ADM-D3).
- **`/dashboard`:** a separate page titled `Restaurant Management` with four hard-coded rows (`Spice Hub`, `Tandoori Flames`, `Urban Bites`, `Food Junction`). It is not linked from anywhere (ADM-D3).
- **Accept / Reject:** see ONB-03. The list shows no success message afterwards.

## Acceptance criteria

1. Given 12 onboarded restaurants (REST-MANY) and REST-LEG, then `Total Restaurants` is 12, page 1 lists 10 newest first, page 2 lists 2 numbered `01`, `02`, and REST-LEG is not listed.
2. Given REST-OFF, then its row shows `Inactive`, and it is counted under `Inactive`.
3. Given REST-P (pending), then its row shows `Accept` and `Reject`; REST-A shows `Approved`; REST-R shows `Rejected`.
4. Given REST-A expanded, then `Restaurant ID` = REST-A's 32-character lower-case hex, and `View Analytics` opens `/admin/analytics/<that hex>`.
5. Given the search text of a restaurant on page 2 while on page 1, then no row is shown.

## Platform notes

### Super-admin panel

`AdminController::index` (paginate 10, `orderByDesc('created_at')`), `resources/views/admin/index.blade.php`. The search runs in the browser.

### Data — MySQL

Read only.

### POS app, POS API

None.

## Configuration

None.

## Out of scope

- Approve / reject (ONB). Editing a restaurant (RST).

## Open questions

- None open.
