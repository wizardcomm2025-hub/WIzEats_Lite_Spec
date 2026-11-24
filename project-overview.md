# WhizEats POS Lite — Project Overview

> **Purpose of this file:** starting-point context for writing feature specifications and test cases. Read this before drafting any spec, use case, data model or test case. Update it when fundamental product decisions change.

> **Status.** Re-targeted from the RMS template to WhizEats POS Lite on 2026-11-05. Architecture, API conventions and data facts are taken from the code on that date. The product, roles, languages, scope and glossary were filled in on 2026-11-13 (intake answers, and rules read from the code and the feature specs); no `TODO` sections remain. The glossary is the single source of truth for shared terms.
>
> **Re-synced 2026-11-24** against the API repo `C:\xampp82\htdocs\laravel-pos-api\api` (branch `dev`, commit `d54d6c8`, 2026-11-19), the app `POS_APP-kot`, and the stage schema dump `db_script/pos_stage.sql` (MySQL 8.0). From this date the spec records **today's code plus the target for every known bug** — see [Decision status](./conventions.md#decision-status) and [Known gaps and targets](#known-gaps-and-targets).

---

## What the product is

WhizEats POS Lite (the app is branded **Wizard POS**) is a multi-restaurant **point-of-sale system** for small restaurants. Each restaurant signs up from the mobile app with a phone number, completes onboarding, and is approved by a platform super admin. Restaurant staff then use the app to set up zones, tables, floors, rooms and the menu; take dine-in, takeaway, delivery and room-service orders; fire KOTs (Kitchen Order Tickets) to the kitchen; generate bills; and take payments. Customers can also scan a QR code and place an order themselves (public QR order). A Blade web panel lets the platform super admin approve restaurants and manage their data.

**Customers and market (confirmed 2026-11-13; drafted from the code and accepted by the user):**
- WhizEats POS Lite is for small restaurants in India. Amounts are in ₹, restaurants record GST and FSSAI numbers, and payment is cash or UPI.
- Owners register, are approved by a super admin, and set up tables, rooms and the menu.
- Waiters take dine-in, takeaway, room-service and delivery orders and send KOTs; chefs move KOTs through the kitchen.
- Staff bill, take cash or UPI, and print on a Wi-Fi thermal printer; owners see reports.

Pricing and plans: none in the code. If they're needed, they will be recorded as a new decision.

---

## The user roles

Every restaurant user belongs to exactly one restaurant (`users.restaurantId`). The role is a plain string in `users.role`.

| Role | Where it lives | Signs in with | Notes |
|---|---|---|---|
| Owner | `users`, `role = 'owner'` | Phone + PIN (or legacy OTP) | Created at sign-up / onboarding. Sets up the restaurant and creates staff. |
| Waiter | `users`, `role = 'waiter'` | Phone + PIN | Created by the owner through Staff. Takes orders, picks up ready KOTs. |
| Chef | `users`, `role = 'chef'` | Phone + PIN | Created by the owner through Staff. Sees pending KOTs and updates their status. App opens on the KOT screen. |
| Super admin | `super_admins` (separate table) | Username + password on the web panel (`/admin/login`, guard `admin`) | Platform operator. Approves / rejects restaurants and edits their data in the Blade panel. Not a restaurant user. |
| Customer (public) | Not stored as a user | No sign-in | Uses the public QR menu and order endpoints. |

### What each role may do (decided 2026-11-13)

The role rules are **the app's navigation as it is today**. **Today** the API enforces none of them: any signed-in user of a restaurant can call any endpoint. **Target** ([X-D1](#known-gaps-and-targets), to fix): the API enforces this table, and a call outside the caller's role is refused with 403. Each feature's `api-contract.md` lists the roles allowed per endpoint.

| Area (app) | Owner | Waiter | Chef |
|---|---|---|---|
| Home: order channels, tables, menu ordering, cart, KOT, billing | ✓ | ✓ | — |
| Setup: restaurant profile and settings, zones and tables, floors and rooms, categories and menu items, import / export, How to Setup | ✓ | — | — |
| User Management | ✓ (all staff) | ✓ (own row) | ✓ (own row; tablet rail only) |
| Owner Dashboard (reports) | ✓ | — | — |
| Order History | ✓ | ✓ | — |
| Waiter Dashboard, Ready Orders | — | ✓ | — |
| KOT screen (kitchen) | — | — | ✓ (opens here) |
| Chef Dashboard | — | — | ✓ |
| Delivery board | ✓ | ✓ | — |
| Printer | ✓ | ✓ | — |
| Profile | ✓ | — | — |

Super admins use only the web panel ([ADM](./features/admin-panel/)). They have no roles among themselves (ADM-D1). Public customers use only the public endpoints ([PUB](./features/public-order/)).

---

## Core features

Feature list agreed 2026-11-05, in spec order. Each gets `features/<folder>/` when specified.

| # | Code | Feature | Spec |
|---|---|---|---|
| 1 | AUTH | Sign-in & PIN | [features/auth/](./features/auth/) |
| 2 | ONB | Onboarding & approval | [features/onboarding/](./features/onboarding/) |
| 3 | RST | Restaurant profile & settings | [features/restaurant-settings/](./features/restaurant-settings/) |
| 4 | STAFF | Staff management | [features/staff/](./features/staff/) |
| 5 | TBL | Zones & tables | [features/tables/](./features/tables/) |
| 6 | ROOM | Floors & rooms | [features/rooms/](./features/rooms/) |
| 7 | MENU | Menu categories & items | [features/menu/](./features/menu/) |
| 8 | IMPEX | Bulk import / export | [features/import-export/](./features/import-export/) |
| 9 | ORD | Take an order & send KOTs | [features/orders/](./features/orders/) |
| 10 | KITCH | Kitchen (chef) | [features/kitchen/](./features/kitchen/) |
| 11 | WAIT | Waiter pick-up | [features/waiter/](./features/waiter/) |
| 12 | DLV | Delivery / online-orders board | [features/delivery/](./features/delivery/) |
| 13 | BILL | Bill & payment | [features/billing/](./features/billing/) |
| 14 | HIST | Order history | [features/order-history/](./features/order-history/) |
| 15 | RPT | Reports / analytics | [features/reports/](./features/reports/) |
| 16 | PUB | Public QR ordering (developer cases only — no UI in these repos) | [features/public-order/](./features/public-order/) |
| 17 | PRN | Printer settings (tester cases only — device-local) | [features/printer/](./features/printer/) |
| 18 | ADM | Super-admin panel: sign-in, restaurant list, Overview / Order / Item History, language (other panel pages live in ONB, RST, STAFF, TBL, MENU) | [features/admin-panel/](./features/admin-panel/) |

---

## Platforms

| Surface | What it does | Technology |
|---|---|---|
| POS API (`C:\xampp82\htdocs\laravel-pos-api\api`, branch `dev`) | JSON API under `/api/*` for the POS app and the public QR menu | Laravel 12, PHP 8.2, Sanctum personal access tokens, services in `app/Services/<Module>/` |
| Super-admin web panel (same repo, `routes/web.php`) | Approve restaurants; manage their items, tables, categories, zones, config and users | Laravel Blade views, session guard `admin`, locales en / es / de / bn |
| Database | All restaurant data and the super-admin table, in one schema; rows scoped by `restaurantId` | MySQL 8.0 on stage (`pos_stage`); MariaDB 10.4 in local XAMPP. InnoDB, utf8mb4, `binary(16)` UUID keys |
| POS app (`D:\whizeats_Lite_Spec\frontend\POS_APP-kot`) | The app restaurant staff use: owner / waiter / chef screens, phone and tablet layouts | React Native 0.82, React 19, TypeScript, Redux Toolkit, React Navigation 7, NativeWind, axios. API base URL `https://pos.wizardcomm.org/api` |
| Integrations | Error logging to Google Sheets (`GoogleSheetLogService`), e-mail (PIN mails, restaurant status mails), image upload storage, Excel import / export, receipt / KOT printing from the app | — |

---

## Folder structure

```
D:\whizeats_Lite_Spec\
├── spec\                          ← this folder; feature specs, test cases, ADRs
├── db_script\pos_stage.sql        ← stage schema dump, MySQL 8.0 (source of truth for columns, 2026-11-24)
└── frontend\POS_APP-kot\          ← React Native POS app

C:\xampp82\htdocs\laravel-pos-api\api\   ← Laravel API + super-admin panel (git, branch dev)
├── db_script\tables\tables.sql    ← older MariaDB dump (2026-11-05); same structure as pos_stage.sql
├── database\migrations\           ← migrations (not complete — see X-D7)
└── specs\                         ← plan.md / tasks.md / test-results.md per feature
```

Data-model facts: read `D:\whizeats_Lite_Spec\db_script\pos_stage.sql`, the migrations and the Eloquent models in `app/Models/`. If a table isn't there, ask the user for a fresh dump. Short names used in the feature files: `laravel-pos-api/api` = the API repo, `POS_APP-kot` = the app, `db_script/pos_stage.sql` = the schema dump (all renamed from `laravel-pos-api-dev_guid` / `POS_APP-master` / `tables.sql` on 2026-11-24).

---

## API conventions

These are facts from the code (2026-11-05; re-verified unchanged 2026-11-24). Developer test cases use them as expected results, except where a [target](#known-gaps-and-targets) replaces them. If the code changes, update this section first.

### Request routing

- All app endpoints are under `/api/` and are `POST` (including reads). Some take an ID in the path, for example `POST /api/orders/kots/{orderId}`, `POST /api/chef/update-status/{kotId}`.
- **Public (no token):** `/api/auth/sendOtp`, `/api/auth/verifyOtp`, `/api/auth/register`, `/api/auth/checkPhone`, `/api/auth/loginPin`, `/api/auth/forgotPin`, `/api/restaurants/setup` (token optional), `/api/public/menu`, `/api/public/order`.
- **Everything else** runs behind `auth:sanctum` + `restaurant.tenant` (`app/Http/Middleware/RestaurantTenantMiddleware.php`).
- **Tenant:** the middleware binds `tenant.restaurantId` from the **signed-in user's** `restaurantId`. Services read it with `app('tenant.restaurantId')` and filter every query on it. `restaurantId` and `userId` in a request body must never be trusted.
- **Super-admin panel:** `routes/web.php`, session guard `auth:admin`, CSRF-protected forms; login throttled to 5 per minute.
- **IDs** are `binary(16)` UUIDs. The API returns them as 32-character hex (no dashes) and accepts them with or without dashes.

### Rate limits

| Limiter | Endpoint | Limit |
|---|---|---|
| `checkPhone` | `/api/auth/checkPhone` | 20 / min per IP |
| `loginPin` | `/api/auth/loginPin` | 5 / min per phone, 30 / min per IP |
| `forgotPin` | `/api/auth/forgotPin` | 3 / 10 min per phone, 10 / min per IP |
| `onboarding` | `/api/restaurants/setup` | 10 / min per IP |
| `throttle:5,1` | `/admin/login` (web) | 5 / min |

Over the limit → HTTP 429, body `{ "message": "Too Many Attempts." }`.

### Response shapes

| Situation | Produced by | HTTP | Body |
|---|---|---|---|
| Success | Controller | 200 (201 for `/api/public/order`) | `{ "status": true, "message": "<text>", "data": … }` |
| Success — PIN endpoints | `PinController` | 200 | Today `{ "success": true, "message": "<text>", "data": … }` — note `success`, not `status`. Target (X-D9): `status`, like every other endpoint |
| Validation failure | `FormRequest::failedValidation` | 422 | `{ "status": false, "message": "<first error>", "errors": { "<field>": ["…"] } }` |
| Business failure (duplicate, invalid PIN, not found…) | Service | 4xx as the service sets it (400 / 401 / 403 / 404 / 422) | `{ "status": false, "message": "<text>", "data": null }` |
| Unexpected error | Service `catch` | 500 | `{ "status": false, "message": "Failed to <action>" }` — some controllers add `"error": "<exception text>"` (see Known gaps) |
| No / invalid / revoked token | `bootstrap/app.php` exception renderer | 401 | `{ "message": "Unauthenticated." }` |
| Token user has no restaurant | `RestaurantTenantMiddleware` | 403 | `{ "status": false, "message": "Restaurant context missing" }` |
| Token user switched off, or restaurant not approved / inactive | **Target only** (X-D2): `RestaurantTenantMiddleware` | 403 | `{ "status": false, "message": "Your account is inactive." }` / `"Your restaurant is not approved."` / `"Your restaurant is inactive."` |
| Role not allowed | **Target only** (X-D1): role check | 403 | `{ "status": false, "message": "You do not have permission to perform this action", "data": null }` |
| ID not found / another restaurant's ID / deleted | **Target** (X-D4); today mostly 500 `Failed to …` | 404 | `{ "status": false, "message": "<Thing> not found", "data": null }` — the exact text per endpoint is in each `api-contract.md` |
| Too many requests | Laravel throttle | 429 | `{ "message": "Too Many Attempts." }` |

### Known gaps and targets

Cross-cutting decisions, `X-Dn`, decided 2026-11-24 (first listed 2026-11-05 as "Known gaps"). Status rules are in [Decision status](./conventions.md#decision-status): tests of a **To fix** item assert the target and carry `Fails until X-Dn is fixed`. Feature decisions cite these instead of restating them.

| ID | Status | Today | Target |
|---|---|---|---|
| **X-D1** | To fix | **No role checks on the API.** Any signed-in user of a restaurant (waiter, chef) can call owner endpoints such as `staff/create`, `item/menuItemSave`, `restaurants/updateDetails`. `staff/create` accepts any `role` string. | The API enforces the [role table](#what-each-role-may-do-decided-2026-11-13). A call outside the caller's role → 403 `{ "status": false, "message": "You do not have permission to perform this action", "data": null }`, nothing written. Each `api-contract.md` lists the allowed roles per endpoint. |
| **X-D2** | To fix | **Approval and status not enforced after login.** The tenant middleware does not check `users.status`, `restaurants.isApproved` or `restaurants.status`. A token issued before switch-off, rejection or deactivation keeps working. | `RestaurantTenantMiddleware` checks on every request, in this order: `users.status = 0` → 403 `{ "status": false, "message": "Your account is inactive." }`; `restaurants.isApproved ≠ 1` → 403 `"Your restaurant is not approved."`; `restaurants.status = 0` → 403 `"Your restaurant is inactive."`. Nothing is written. `POST /api/auth/loginPin` also refuses a restaurant with `status = 0` (403 `"Your restaurant is inactive."`, no token); pending and rejected restaurants still sign in so the app can show their screen. **Exemptions:** `POST /api/auth/logout` skips all three checks (signing out always works); `POST /api/restaurants/showDetails` skips the approval check only (the Pending-approval screen polls it). The app (today it reacts only to 401): on `Your account is inactive.` or `Your restaurant is inactive.` it signs out to Login and shows the message in red; on `Your restaurant is not approved.` it opens the Pending-approval screen (ONB) without signing out. See [AUTH 06](./features/auth/06-resume-or-end-session.md). |
| **X-D3** | Accepted | **Tokens never expire** (`sanctum.expiration = null`); logout revokes only the current token. | — POS devices stay signed in through a shift; X-D2 blocks switched-off users. |
| **X-D4** | To fix | **Not-found is a 500.** `firstOrFail()` inside a service `try` becomes `500 "Failed to …"`. | A missing, deleted or other-restaurant ID → 404 `{ "status": false, "message": "<Thing> not found", "data": null }`. The exact text per endpoint is in each `api-contract.md`. |
| **X-D5** | To fix | **Error text leaks.** Some controllers and services return `"error": $e->getMessage()` on 500 (e.g. `reports/dashboard-summary`). | No response contains an `error` key, SQL, a stack trace or internal names. A 500 is `{ "status": false, "message": "Failed to <action>" }` only. |
| **X-D6** | Accepted | **Legacy OTP and default PIN.** `OtpService` accepts `123456` for every phone and returns it in the `sendOtp` response. `/api/auth/register` creates owners with PIN `123456` (`mustChangePin = 1`). The current app does not call `sendOtp`, `verifyOtp` or `register`. | — Kept as is ([AUTH 07](./features/auth/07-legacy-otp-and-register.md)). |
| **X-D7** | To fix | **Schema drift.** `bills`, `bill_items`, `customers`, `customer_addresses` exist in the schema dump but have no migration; `menu_item_variants` has a migration but is missing from the dump. | Migrations and `db_script` describe the same tables: add the four missing migrations, and add `menu_item_variants` to the dump. No column changes. |
| **X-D8** | To fix | **Cross-restaurant references.** Some saves accept a foreign key (for example `zoneId` on `tables/tableSave`) without checking it belongs to the caller's restaurant. | Every foreign key in a body is checked against the caller's restaurant (and must not be deleted). Failing → 422 with the field's message from the feature's `api-contract.md`; nothing written. |
| **X-D9** | To fix | **Inconsistent envelope.** PIN endpoints (`checkPhone`, `loginPin`, `forgotPin`, `changePin`) return `success`; everything else returns `status`. The app never reads either flag (it uses the HTTP status and `data`). | PIN endpoints return `status` (same values) instead of `success`. Not breaking for the app (checked 2026-11-24); breaking only for external callers. |
| **X-D10** | To fix (changed 2026-11-24, was Accepted) | **Public order trusts the client.** `/api/public/order` takes `restaurantId`, `items.*.price` and totals from the body, with no check against `menu_items`. No UI calls it. | The server takes names and prices from the menu and checks items, channel, table / room and the restaurant's state — see [PUB-D2](./features/public-order/README.md#decisions). |
| **X-D11** | Accepted | **Unused handler.** `app/Exceptions/Handler.php` is not wired in Laravel 12; `bootstrap/app.php` decides the 401 body. | — Informational. |

---

## Glossary

| Term | Meaning |
|---|---|
| Restaurant | One customer business, `restaurants` row, identified by `restaurantId`. The data boundary for every query. |
| Tenant | The restaurant the signed-in user belongs to; bound by `RestaurantTenantMiddleware` as `tenant.restaurantId`. |
| Restaurant config | Per-restaurant switches and values, `restaurant_configs`: which channels are enabled (dine-in, takeaway, delivery, room service), KOT on/off and grouping, discount on/off, tax %, service charge %, invoice prefix, `restaurantCode`, QR. |
| Onboarding | First-time restaurant setup through `/api/restaurants/setup`; sets `restaurants.isOnboardingDone = 1`. |
| Approval | Super-admin decision on a restaurant, `restaurants.isApproved`: 0 = Pending, 1 = Accepted, 2 = Rejected. The app shows a Pending-approval screen until accepted. |
| Restaurant status | `restaurants.status`: 0 = inactive, 1 = active. |
| User | A person who signs in to the POS app, `users` row, one restaurant, one role. Identified by phone number. `users.status`: 0 = inactive, 1 = active. |
| Owner / Waiter / Chef | The three restaurant roles (`users.role`). See [The user roles](#the-user-roles). |
| Super admin | Platform operator in `super_admins`, signs in to the web panel. Not a restaurant user. |
| PIN | 6-digit secret used with the phone number to sign in, stored hashed in `users.pin`. `mustChangePin = 1` forces a change at next sign-in (after a reset or a default PIN). |
| OTP | One-time code for the legacy phone sign-in (`sendOtp` / `verifyOtp`). |
| Zone | A dining area that groups tables (Garden, AC Hall), `zones`. Switching a zone off switches its tables off. |
| Table | A dine-in table in a zone, `tables`, with capacity. `tables.status`: 0 = free, 1 = occupied, 2 = ordered. |
| Floor | A level that groups rooms, `floors`. |
| Room | A room on a floor for room-service orders, `rooms`. Same status values as tables. |
| Item category | A menu group (Starters, Beverages), `item_categories`. |
| Menu item | Something sold, `menu_items`, in one category, with price, veg flag, image. May have variants (`menu_item_variants`). |
| Order channel | How an order arrives (dine-in, takeaway, delivery, room service), `order_channels`, per restaurant. |
| Order | A customer's order, `orders` + `order_items`. `orderStatus`: 0 = in progress, 1 = bill generated, 2 = fulfilled. `paymentStatus`: 0 = unpaid, 1 = paid. |
| KOT | Kitchen Order Ticket, `kots`: one batch of items sent to the kitchen for an order. `status`: 0 = pending, 1 = preparing, 2 = rejected, 3 = completed, 4 = ready; `isPickedUp` once the waiter collects it. |
| Bill | The invoice for an order, `bills` + `bill_items`, numbered per financial year. Generating it moves the order from 0 to 1. `billStatus`: 0 = unpaid, 1 = paid. |
| Payment | Money received against an order, `payments` (type, status, received and change amounts). |
| Public QR order | An order a customer places by scanning the restaurant's QR, through `/api/public/menu` and `/api/public/order`, without signing in. |
| Active / Inactive / Deleted | Row state on master data: `isActive = 1, isDelete = 0` active; `isActive = 0, isDelete = 0` inactive; `isDelete = 1` deleted (soft). |

### Ordering and billing rules (from the code and the feature specs, 2026-11-13)

| Term | Rule today |
|---|---|
| Channel codes | Onboarding creates four channels per restaurant: `DINE_IN` (Dine In), `TAKEAWAY` (Takeaway), `ROOM_SERVICE` (Room Service), `DELIVERY` (Delivery). Each has a GUID `orderChannelId`. Dine In is always active. Takeaway, Room Service and Delivery start active only if switched on during onboarding, and the settings switches turn them on and off by `channelCode` (RST-D1, fixed in code 2026-11-16). Home lists only active channels. Order history filters by `channelCode` (HIST-D1, fixed in code 2026-11-18). The Delivery board falls back to the old fixed number `3` when it has no channel id (DLV-D1). |
| Variants | The `menu_item_variants` table exists, but no endpoint or screen uses it. An item has one price. |
| Tax | The restaurant's `taxPercent` (Settings → Tax Rate) is copied onto each **new** order (ORD). The bill uses the order's rate, or the restaurant's when the order has none, and 0 when neither is set. One rate covers the whole bill; there is no per-item tax or CGST / SGST split. |
| Discount | Only when the restaurant's `enableDiscount` is on. It is entered on Bill Breakdown as a percent or a rupee amount, is always taken off the **subtotal** (before tax), and is capped at the subtotal. |
| Bill arithmetic (app) | The sum of the line totals, minus the discount, gives the taxable amount. Tax = taxable × rate ÷ 100, rounded to paise. Grand total = taxable + tax, rounded to the nearest rupee (.50 rounds up), and `roundOff` is the signed difference. The server stores the figures as sent and does not recompute them (BILL-D1). The public order path computes its own totals (PUB, never reached). |
| Service charge, invoice prefix | `serviceChargePercent` and `invoicePrefix` are stored in the settings but not used on bills or prints. An app save clears the service charge (RST-D2). |
| Order number | `<restaurantCode>/<n>`, with `n` restarting each (UTC) day. It may repeat or jump (ORD-D3). |
| KOT number | A plain number per restaurant, with the same caveat (ORD-D3). The ticket's label is the restaurant's KOT display name (`kotDisplayName`, column default `kot`). |
| Bill number | `max + 1` per restaurant per financial year (`financialYear` such as `2026-27`, April–March). It is a plain number with no prefix. There is no unique key; numbering relies on the lock on the config row (BILL-D3). It is assigned when the bill is generated, or when an unbilled order is paid. |
| Payment types | The app sends `CASH` and `UPI`, or a split of both. The server accepts any text (BILL-D1). The UPI QR on Bill Payment is built from the restaurant's UPI ID (`restaurant_configs.qr`). Each payment gets `transactionId` `TXN-<12 random>` and `paymentStatus` `COMPLETED`. There is no card, wallet, void or refund. |
| Paid | The order gets `paymentStatus` 1 and `orderStatus` 2, and the bill `billStatus` 1. Reports differ in which flag they read (RPT-D1, ADM-D2). |

---

## What is not in scope (as of this document)

The super-admin web panel **is** in scope (decided 2026-11-12); its pages are split across ONB, RST, STAFF, TBL, MENU and ADM. Out of scope, as recorded in the feature specs:

- **Changing code:** specs record today's behaviour, with risks accepted as `CODE-Dn` decisions. Developers change the code later, under new decisions.
- **Missing pieces:**
  - a customer-facing QR menu page (PUB);
  - customer payment;
  - bill void and refund (BILL);
  - managing super-admin accounts (ADM).
- **Printers:** Bluetooth or USB printers, and more than one printer per device (PRN).
- **Languages:** translating the POS app (English only).

---

## Resolved platform decisions

These are settled. Do not reopen without a formal ADR. The same table appears in `CLAUDE.md`; keep them in sync.

| Decision | Resolution |
|---|---|
| Backend stack | Laravel 12 / PHP 8.2 (`laravel-pos-api/api`): route → controller → FormRequest → service → Eloquent model |
| Database | One MySQL / MariaDB schema; every restaurant row filtered by `restaurantId`; `binary(16)` UUID keys |
| Data access | Eloquent and the query builder inside `app/Services/<Module>/` |
| Authentication | Sanctum personal access tokens for the app (phone + PIN, legacy OTP); session guard `admin` for the super-admin panel. Restaurant and user come from the token, never from the body |
| API shape | `POST /api/<module>/<action>`, body `{ status, message, data }`, real HTTP status codes (422 validation, 401, 403, 404, 500) |
| Client surfaces | React Native POS app (`POS_APP-kot`); Blade super-admin panel |
| Supported languages | Super-admin panel: en, es, de, bn. POS app: English only. All screen texts are hard-coded; there is no translation library (confirmed 2026-11-13). |
| API contract | Each feature's `api-contract.md`. `api/openapi.yaml` will be generated from the Laravel code later and is never hand-edited; when the two disagree, `api-contract.md` wins (decided 2026-11-13) |
| Developer test runner | PHPUnit 11 feature tests in `laravel-pos-api/api/tests/Feature/<FeatureFolder>/`: one test class per use case (e.g. `Kitchen/SeeKotsByStageTest.php`), one test method per case ID (`test_KITCH_01_D03_…`). Tester cases stay manual (decided 2026-11-13) |
