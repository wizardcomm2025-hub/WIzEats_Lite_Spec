# WhizEats POS Lite — Project Overview

> **Purpose of this file:** starting-point context for writing feature specifications and test cases. Read this before drafting any spec, use case, data model or test case. Update it when fundamental product decisions change.

> **Status.** Re-targeted from the RMS template to WhizEats POS Lite on 2026-11-05. Architecture, API conventions and data facts are taken from the code on that date. The product, roles, languages, scope and glossary were filled in on 2026-11-13 (intake answers, and rules read from the code and the feature specs); no `TODO` sections remain. The glossary is the single source of truth for shared terms.

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

The role rules are **the app's navigation as it is today**. The API enforces none of them: any signed-in user of a restaurant can call any endpoint. That gap is accepted per feature (AUTH, STAFF, TBL, MENU, ORD, KITCH, WAIT, BILL, HIST, RPT …) and listed under [Known gaps](#known-gaps-as-of-2026-11-05).

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
| POS API (`C:\whizeats-lite\laravel-pos-api-dev_guid`) | JSON API under `/api/*` for the POS app and the public QR menu | Laravel 12, PHP 8.2, Sanctum personal access tokens, services in `app/Services/<Module>/` |
| Super-admin web panel (same repo, `routes/web.php`) | Approve restaurants; manage their items, tables, categories, zones, config and users | Laravel Blade views, session guard `admin`, locales en / es / de / bn |
| Database | All restaurant data and the super-admin table, in one schema; rows scoped by `restaurantId` | MySQL / MariaDB (InnoDB, utf8mb4), `binary(16)` UUID keys |
| POS app (`C:\whizeats-lite\POS_APP-master`) | The app restaurant staff use: owner / waiter / chef screens | React Native 0.82, React 19, TypeScript, Redux Toolkit, React Navigation 7, NativeWind, axios |
| Integrations | Error logging to Google Sheets (`GoogleSheetLogService`), e-mail (PIN mails, restaurant status mails), image upload storage, Excel import / export, receipt / KOT printing from the app | — |

---

## Folder structure

```
C:\whizeats-lite\
├── App-Specs\                     ← this folder; feature specs, test cases, ADRs
├── laravel-pos-api-dev_guid\      ← Laravel API + super-admin panel; reads specs at ../App-Specs/
│   ├── db_script\tables\tables.sql ← schema dump (source of truth for columns)
│   ├── database\migrations\        ← migrations (not complete — see Known gaps)
│   └── specs\                      ← plan.md / tasks.md / test-results.md per feature (created with the first plan)
└── POS_APP-master\                ← React Native POS app; reads specs at ../App-Specs/
```

Data-model facts: read `laravel-pos-api-dev_guid/db_script/tables/tables.sql`, the migrations and the Eloquent models in `app/Models/`. If a table isn't there, ask the user for a fresh dump.

---

## API conventions

These are facts from the code (2026-11-05). Developer test cases use them as expected results. If the code changes, update this section first.

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
| Success — PIN endpoints | `PinController` | 200 | `{ "success": true, "message": "<text>", "data": … }` — note `success`, not `status` |
| Validation failure | `FormRequest::failedValidation` | 422 | `{ "status": false, "message": "<first error>", "errors": { "<field>": ["…"] } }` |
| Business failure (duplicate, invalid PIN, not found…) | Service | 4xx as the service sets it (400 / 401 / 403 / 404 / 422) | `{ "status": false, "message": "<text>", "data": null }` |
| Unexpected error | Service `catch` | 500 | `{ "status": false, "message": "Failed to <action>" }` — some controllers add `"error": "<exception text>"` (see Known gaps) |
| No / invalid / revoked token | `bootstrap/app.php` exception renderer | 401 | `{ "message": "Unauthenticated." }` |
| Token user has no restaurant | `RestaurantTenantMiddleware` | 403 | `{ "status": false, "message": "Restaurant context missing" }` |
| Too many requests | Laravel throttle | 429 | `{ "message": "Too Many Attempts." }` |

### Known gaps (as of 2026-11-05)

Developer tests describe the **intended** behaviour that the spec defines; a test that hits one of these gaps fails until the code is fixed. Each gap is decided in the spec of the feature that owns it.

- **Static OTP.** `OtpService` accepts `123456` for every phone and returns it in the `sendOtp` response.
- **Default PIN.** Legacy registration (`/api/auth/register`) creates owners with PIN `123456` (`mustChangePin = 1`).
- **No role checks on the API.** Any signed-in user of a restaurant (waiter, chef) can call owner endpoints such as `staff/create`, `item/menuItemSave`, `restaurants/updateDetails`. `staff/create` accepts any `role` string.
- **Approval and status not enforced after login.** The tenant middleware does not check `restaurants.isApproved`, `restaurants.status` or `users.status`; a token issued before rejection or deactivation keeps working.
- **Tokens never expire** (`sanctum.expiration = null`); logout revokes only the current token.
- **Public order trusts the client.** `/api/public/order` takes `restaurantId`, `items.*.price` and totals from the body, with no check against `menu_items`.
- **Cross-restaurant references.** Some updates accept a foreign key (for example `zoneId` on `tables/tableSave`) without checking it belongs to the caller's restaurant.
- **Error text leaks.** Several controllers return `"error": $e->getMessage()` on 500. Target: no SQL, stack trace or internal names in any response.
- **Not-found is a 500.** `firstOrFail()` inside a service `try` becomes `500 "Failed to …"` instead of 404.
- **Inconsistent envelope.** PIN endpoints use `success`; everything else uses `status`.
- **Unused handler.** `app/Exceptions/Handler.php` is not wired in Laravel 12; `bootstrap/app.php` decides the 401 body.
- **Schema drift.** `bills`, `bill_items`, `customers`, `customer_addresses` exist in `tables.sql` but have no migration; `menu_item_variants` has a migration but is missing from `tables.sql`.

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
| Channel codes | Onboarding creates four channels per restaurant: `DINE_IN` (Dine In), `TAKEAWAY` (Takeaway), `ROOM_SERVICE` (Room Service), `DELIVERY` (Delivery). Each has a GUID `orderChannelId`. The Takeaway, Delivery and Room Service switches in the settings don't change the channels, so all four stay active (RST-D1). The app still sends old fixed numbers in places, such as `3` for Delivery (DLV-D1) and `0`–`3` as the history filter (HIST-D1). |
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
| Backend stack | Laravel 12 / PHP 8.2 (`laravel-pos-api-dev_guid`): route → controller → FormRequest → service → Eloquent model |
| Database | One MySQL / MariaDB schema; every restaurant row filtered by `restaurantId`; `binary(16)` UUID keys |
| Data access | Eloquent and the query builder inside `app/Services/<Module>/` |
| Authentication | Sanctum personal access tokens for the app (phone + PIN, legacy OTP); session guard `admin` for the super-admin panel. Restaurant and user come from the token, never from the body |
| API shape | `POST /api/<module>/<action>`, body `{ status, message, data }`, real HTTP status codes (422 validation, 401, 403, 404, 500) |
| Client surfaces | React Native POS app (`POS_APP-master`); Blade super-admin panel |
| Supported languages | Super-admin panel: en, es, de, bn. POS app: English only. All screen texts are hard-coded; there is no translation library (confirmed 2026-11-13). |
| API contract | Each feature's `api-contract.md`. `api/openapi.yaml` will be generated from the Laravel code later and is never hand-edited; when the two disagree, `api-contract.md` wins (decided 2026-11-13) |
| Developer test runner | PHPUnit 11 feature tests in `laravel-pos-api-dev_guid/tests/Feature/<FeatureFolder>/`: one test class per use case (e.g. `Kitchen/SeeKotsByStageTest.php`), one test method per case ID (`test_KITCH_01_D03_…`). Tester cases stay manual (decided 2026-11-13) |
