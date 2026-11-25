# Super-admin panel (shell & reports)

The super-admin panel is a Blade web app in the Laravel project (`routes/web.php`, session guard `auth:admin`, table `super_admins`). Super admins sign in with a username and password, see every onboarded restaurant, approve or reject new ones, and open any restaurant's pages.

**Scope (decided 2026-11-12):** ADM covers only the panel parts that no other feature specifies:
- sign-in and logout;
- the restaurant list;
- each restaurant's Overview, Order History and Item History pages;
- the language switch;
- the leftover `/dashboard` page.

The other panel pages are specified where their data lives:

| Panel page / action | Feature |
|---|---|
| Accept / Reject on the restaurant list | [ONB-03](../onboarding/03-approve-or-reject.md) |
| Restaurant Settings | [RST-03](../restaurant-settings/03-edit-in-panel.md) |
| User Management | [STAFF-04](../staff/04-edit-users-in-panel.md) |
| Zone / Table Management | [TBL-03](../tables/03-zones-tables-in-panel.md) |
| Category / Item Management | [MENU-03](../menu/03-menu-in-panel.md) |

The spec records today's behaviour plus targets (re-synced 2026-11-24; no panel code changed). ADM-D2 is *To fix* — the figures and history pages follow the RPT rules (paid only, IST days); ADM-D1 and ADM-D3 stay accepted.

**Feature code:** `ADM` (test IDs `ADM-01-T01`, `ADM-03-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Sign in and out](./01-sign-in-and-out.md) | Super admin | [tester](./test-cases/01-sign-in-and-out.tester.md) | [developer](./test-cases/01-sign-in-and-out.developer.md) |
| 02 | [Browse restaurants](./02-browse-restaurants.md) | Super admin | [tester](./test-cases/02-browse-restaurants.tester.md) | [developer](./test-cases/02-browse-restaurants.developer.md) |
| 03 | [View a restaurant's overview](./03-view-restaurant-overview.md) | Super admin | [tester](./test-cases/03-view-restaurant-overview.tester.md) | [developer](./test-cases/03-view-restaurant-overview.developer.md) |
| 04 | [See orders and items](./04-see-orders-and-items.md) | Super admin | [tester](./test-cases/04-see-orders-and-items.tester.md) | [developer](./test-cases/04-see-orders-and-items.developer.md) |
| 05 | [Switch language](./05-switch-language.md) | Super admin (anyone, for the route) | [tester](./test-cases/05-switch-language.tester.md) | [developer](./test-cases/05-switch-language.developer.md) |

## Key concepts

- **Super admin:** a `super_admins` row: `username`, hashed `password`, `status` (1 on, 0 off), `last_login_at`. There are no roles: every super admin can see and change every restaurant (ADM-D1). Accounts are created and reset only by SQL.
- **Restaurant pages:** `/admin/<page>/<restaurantId>`. The ID is 32-character hex or a dashed GUID, converted with `hex2bin`. Any other value is used as is and matches nothing, so the page shows zeros or empty tables rather than a 404 (ADM-D3).
- **Panel day:** today `whereDate(created_at, today())` in UTC (`config/app.php`), so a panel "day" runs from 05:30 to 05:29 IST. Target: IST days (ADM-D2).
- **Paid** on the panel means `orders.orderStatus = 2`. Payment sets that (BILL).

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api/api/specs/admin-panel/` (created with the first plan)

## Depends on

- **ONB**: which restaurants appear (onboarding done), and approval.
- **ORD / BILL**: the orders and payments behind the figures.
- **RPT**: the owner's own dashboard. Today its figures are computed differently; target: both count paid orders only on IST days (RPT-D1 / D2, ADM-D2).

## Decisions

Statuses per [conventions.md → Decision status](../../conventions.md#decision-status), set 2026-11-24. No panel code changed since 2026-11-12 (API commit d54d6c8).

| ID | Status | Short | Tests |
|---|---|---|---|
| ADM-D1 | Accepted (confirmed 2026-11-24) | SQL-only accounts; sessions outlive deactivation; long remember cookie; Referer redirect | 01-D07, 01-D08, 05-D02, 05-D03 |
| ADM-D2 | **To fix** (aligned with RPT) | Mixed paid / all figures; UTC and N + 1-day periods; static subtitles; today-only, unpaged history pages | 03-T01–T05, 03-T08, 03-D01, 03-D02, 03-D04, 03-D06, 04-T01–T05, 04-T07, 04-D01–D03, 04-D05 |
| ADM-D3 | Accepted (confirmed 2026-11-24) | Placeholder `/dashboard`, dead buttons, page-only search, zeros for unknown IDs, English-only tables | 02-T05, 02-T06, 03-T06, 03-D03, 04-T06 |

- **ADM-D1 — Accepted (confirmed 2026-11-24).** Unchanged from 2026-11-12 (below). Note: the app-side checks targeted by X-D2 / AUTH-D2 don't cover panel sessions.
- **ADM-D2 — To fix (2026-11-24, was accepted 2026-11-12); same rules as RPT-D1 / RPT-D2 targets.** *Target:*
  - **Paid only:** `Total Revenue` (Σ `grandTotal`), `Total Orders`, `Total Items Sold`, `Avg Order Value` (revenue ÷ paid orders), the revenue chart, `Order Channel` and `Popular Items` count only paid orders (`orderStatus` 2) and their lines. `Recent Orders` and Order History still list every order with `Paid` / `Unpaid`.
  - **Periods in IST:** day boundaries at 00:00 Asia/Kolkata (`created_at` stays UTC in the database). `Today`, `Yesterday`; `Last 7 / 15 / 30 Days` = today and the 6 / 14 / 29 days before it. A missing, empty or unknown `filter` means `Today`, as the dropdown shows.
  - **Texts follow the period:** subtitles `Daily revenue — <period>`, `Top performing menu items — <period>`, header `Welcome back! Here's what happened: <period>.` (`<period>` = the dropdown label; translated in all four languages). The revenue chart has one point per IST date of the period (0 when nothing was paid).
  - **Formats:** `Total Revenue` with `₹` (`₹525.00`); `Popular Items` shows the top 10, ranked `01` … `10`.
  - **Order History / Item History:** follow the header's period (the dropdown stays on the page, `?filter=`); 25 rows a page with `Showing <a> to <b> of <n> results`; times in IST; Order History adds `Order No.` (`orders.orderNumber`) before `Order ID`; Item History sums paid lines only and sorts by quantity, highest first, by default; a `sort_by` or `sort_order` outside the allowed values → 422 page (`The selected sort order is invalid.`), not a 500.
  - *Still accepted:* items with the same name are one row in Item History; order IDs upper-case hex.
- **ADM-D3 — Accepted (confirmed 2026-11-24).** Unchanged from 2026-11-12 (below). With ADM-D2 fixed, the history pages keep the `Dashboard Overview` title and English headings, but new texts (`Order No.`, the 422 page) follow the same rule.
- **Inherited (2026-11-24):** ONB-D2; X-D2 / AUTH-D2 (target: app calls check the restaurant's and account's state, so a panel approval change or switch-off takes effect at once in the app); RPT-D1 / RPT-D2 targets — the panel and the app dashboard now count the same orders, though the app's revenue is money collected (received − change) and the panel's is Σ `grandTotal`.

Original decision texts (history):

- **ADM-D1 (2026-11-12):** Accounts and sessions stay as they are:
  - super admins are created, switched off and reset only by SQL; `Forgot password?` is a dead link;
  - switching an admin off (`status` 0) or changing their password blocks only the next sign-in. Open sessions and "remember me" cookies keep working;
  - `Remember me for 30 days` sets Laravel's default remember cookie, which lasts about 400 days;
  - there are no roles or per-restaurant limits; any admin can see and change every restaurant;
  - sign-in is limited to 5 attempts a minute per IP, with no account lockout;
  - `GET /set-locale/{locale}` needs no sign-in and redirects to the request's `Referer`, whatever site that is.

  Accepted risk.
- **ADM-D2 (2026-11-12):** The figures stay as they are:
  - **Overview:**
    - `Total Revenue` and the revenue chart sum `grandTotal` of paid orders (`orderStatus` 2) only;
    - `Total Orders` counts every order, and `Avg Order Value` = paid revenue ÷ all orders;
    - `Total Items Sold`, `Popular Items` and `Order Channel` count every order or line, paid or not;
    - `Recent Orders` shows `Paid` / `Unpaid` from `orderStatus`.
  - **Periods:** UTC days. `Last 7 Days` / `15` / `30` include today and the 7 / 15 / 30 days before it (8 / 16 / 31 calendar days). An unknown `filter` value means all time, while the dropdown shows `Today`.
  - **Static subtitles:** `Daily revenue for the current month`, `Top performing menu items this week` and `Welcome back! Here's what's happening today.` don't follow the filter.
  - **Order History and Item History:** they show today (UTC) only, with no paging. The header's period dropdown takes you back to the Overview. Times are shown in UTC; order IDs are upper-case hex, with no order number.
  - **Item History:** rows are grouped by item name. The default order is by the largest line ID, which is effectively random. An invalid `sort_order` gives a 500 error page.
  - **Number formats:** Popular Items numbers its ranks `01` … `09`, `010`. `Total Revenue` has no `₹`, while `Avg Order Value` has one.

  Accepted.
- **ADM-D3 (2026-11-12):** Placeholder and dead UI stays:
  - `/dashboard` shows four hard-coded restaurants (`Spice Hub`, `Tandoori Flames`, `Urban Bites`, `Food Junction`);
  - these do nothing: `Export Report`, the bell, `Add Restaurant` (empty list), `Privacy Policy`, `Terms of Service`;
  - the restaurant search filters only the rows of the current page;
  - row numbers restart at `01` on every page;
  - an unknown restaurant ID shows a page of zeros;
  - the Order History and Item History pages are titled `Dashboard Overview`, and their table headings are always in English.

  Accepted.
- **Inherited (2026-11-12):** ONB-D2 (approval endpoint rules), AUTH-D2 (app tokens are unaffected by panel changes), RPT-D1 (the app's dashboard counts differently).

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- The pages listed in the scope table above (owned by other features).
- Managing super-admin accounts (there is no UI).
