# Project overview — intake

> Working notes for this topic, before a formal spec exists. Accumulates inbound
> information from clients/stakeholders and Q&A from interview sessions. Specs
> supersede this file when they're written; nothing here is ever deleted.

Status: collecting

<!-- Status values: collecting | spec-written | abandoned -->

---

## What we've been told

Standalone facts, business rules, constraints, and client input — anything we know about this topic that needs to land somewhere. No question needed.

- History: this folder held RMS (a .NET restaurant-management product) material until 2026-11-05. The RMS facts recorded here earlier were removed when the folder was re-targeted to WhizEats POS Lite, because they describe a different product.
- From the code (2026-11-05): WhizEats POS Lite is multi-restaurant. The tenant is the signed-in user's `restaurantId`, bound by `RestaurantTenantMiddleware`; all restaurants share one MySQL schema.
- From the code: restaurant users are `owner`, `waiter` and `chef` (`users.role`). Sign-in is phone + 6-digit PIN; a legacy phone + OTP flow still exists. Super admins are a separate table and use a Blade web panel.
- From the code: restaurants go through onboarding (`isOnboardingDone`) and super-admin approval (`isApproved`: pending / accepted / rejected). The app shows a Pending-approval screen until accepted.
- From the code: customers can order without signing in through the public QR endpoints (`/api/public/menu`, `/api/public/order`).
- From the user (2026-11-05): test cases come in two kinds — developer cases (direct API calls with garbage, unauthenticated or manipulated input, to test results and security) and tester cases (UI-driven, since testers can't manipulate the API or data much).
- From the user (2026-11-05): for now only specs and test cases are written; developers make the code changes later.
- From the code (2026-11-13), recorded in `project-overview.md` → Ordering and billing rules:
  - **Channels:** four codes per restaurant (`DINE_IN`, `TAKEAWAY`, `ROOM_SERVICE`, `DELIVERY`).
  - **Variants:** none in use.
  - **Tax:** one rate per order, copied from the settings.
  - **Discount:** a percent or rupee amount taken off the subtotal, capped at it.
  - **Rounding:** the grand total is rounded to the rupee, and `roundOff` keeps the difference.
  - **Numbers:** order `<restaurantCode>/<n>` per day; KOT a plain number; bill `max + 1` per financial year (April–March), with no prefix. The invoice prefix and service charge aren't used.
  - **Payment types:** `CASH` and `UPI` (split allowed), with no card or refund.

---

## Interview Q&A

Questions and answers from grill-me / spec-intake sessions or any other clarify-before-draft conversation. Append new entries at the bottom; do not edit prior entries.

- **Q:** Re-target App-Specs to WhizEats POS Lite in place, or keep the RMS version and make a separate copy?
  **A:** Re-target in place (2026-11-05).
- **Q:** Which feature first?
  **A:** AUTH, then the 18-feature list in `project-overview.md` → Core features (2026-11-05).
- **Q:** AUTH — remove, secure or keep the legacy OTP / register endpoints?
  **A:** Keep as-is for now; risk accepted (AUTH-D1, 2026-11-05).
- **Q:** AUTH — what happens to existing tokens when a user is switched off or a restaurant rejected / switched off?
  **A:** Keep current behaviour: only the next sign-in is blocked (AUTH-D2, 2026-11-05).
- **Q:** AUTH — token lifetime?
  **A:** Never expire, as today (AUTH-D3, 2026-11-05).
- **Q:** AUTH — sign out other devices on PIN change / reset?
  **A:** No (AUTH-D4, 2026-11-05).
- **Q:** AUTH open questions Q1–Q7?
  **A:** Keep 500 exception text (D5), keep Forgot PIN as is (D6), no limit on Change PIN (D7), accept phone lookup with rate limits (D8), no Change PIN in Profile for now (D9), keep `success` / `status` envelopes (D10). Parallel legacy register must create exactly one restaurant + owner with no leftovers (D11 — the only behaviour change). (2026-11-05)
- **Q:** ONB — restrict the token path of `/api/restaurants/setup`; validate the approval endpoint; recovery for a mistyped restaurant e-mail; what to do with legacy owners whose onboarding isn't done?
  **A:** Keep all four as they are today (ONB-D1 to ONB-D4, accepted risks). (2026-11-06)
- **Q:** RST — the Takeaway switch doesn't change the order channels; an app save clears the service charge and company address; any role can save settings (phone UI and API); the API and panel barely validate settings. Fix or keep?
  **A:** Keep all four as they are today (RST-D1 to RST-D4, accepted risks). (2026-11-06)
- **Q:** STAFF — the app's Active switch always fails (`isActive` vs `status`); the update endpoint has no restaurant or role check; create has no role check, accepts any role text, has no server-side limit and gives a 500 for a used phone; every new staff member gets PIN 123456. Fix or keep?
  **A:** Keep all four as they are today (STAFF-D1 to STAFF-D4, accepted risks). (2026-11-10)
- **Q:** TBL — the app's Table Management list is always empty (`Number(zoneId)` on GUIDs); no integrity checks (foreign or missing zones, zone delete leaving tables, duplicate names on rename); any role can change zones and tables through the API; the list filters are broken and a table with two running orders appears twice. Fix or keep?
  **A:** Keep all four as they are today (TBL-D1 to TBL-D4, accepted risks). (2026-11-10)
- **Q:** ROOM — the floors/rooms code never converts hex IDs (only add-floor and list work; room create, all updates and the app's room list fail); any role can use the API; no integrity checks (orphan rooms, any capacity, inactive rooms unlisted). Fix or keep?
  **A:** Keep all three as they are today (ROOM-D1 to ROOM-D3, accepted risks). (2026-11-10)
- **Q:** MENU — the app's Add Menu Item always fails (`menuItemId: 0` → 422); category and item edits, switches and filters fail on hex IDs (Item Management always empty); the panel Items page add / edit / switch are broken; no role, price, category-ownership or image checks, and the image upload answers 201 on failure. Fix or keep?
  **A:** Keep all four as they are today (MENU-D1 to MENU-D4, accepted risks). (2026-11-10)
- **Q:** IMPEX — the Import / Export UI is hidden (and would send the wrong field); import always reports success, skips bad rows silently and duplicates on re-import; no value or role checks, raw exception text on failure, and export gives empty templates only. Fix or keep?
  **A:** Keep all three as they are today (IMPEX-D1 to IMPEX-D3, accepted risks). (2026-11-11)
- **Q:** ORD — client-supplied prices / names / item IDs; orders not linked to tables or rooms and running orders can't be re-opened (GUID faults); order and KOT numbers can repeat and customer fields are dropped; no idempotency, several orders per table, any role, raw exception text. Fix or keep?
  **A:** Keep all four as they are today (ORD-D1 to ORD-D4, accepted risks). (2026-11-11)
- **Q:** KITCH — the chef's KOT list fails with a 500 whenever it has KOTs and status updates never match a row (GUID faults); the status endpoint has no role, restaurant, range or stage-order checks; the chef dashboard counts only KOTs the chef last changed and shows only for role exactly `Chef`. Fix or keep?
  **A:** Keep all three as they are today (KITCH-D1 to KITCH-D3, accepted risks). (2026-11-11)
- **Q:** WAIT — the Ready Orders list fails with a 500 whenever a KOT is ready and hex pick-ups never match (GUID faults); the pick-up endpoint has no role, restaurant or stage checks, its reply spans every restaurant, and the Delivered tab isn't kept; the waiter dashboard's Total Orders is all-time and top items count only today's paid orders. Fix or keep?
  **A:** Keep all three as they are today (WAIT-D1 to WAIT-D3, accepted risks). (2026-11-11)
- **Q:** DLV — the delivery board never loads (the app sends channel `3`; with a real ID the query names a missing `orders.orderChannel` column); the status endpoint casts the KOT ID to an integer (never the intended KOT, cross-restaurant lookup, SQL text in errors, no rules); WhatsApp goes to a phone number fixed in the code, `Complete Payment` records no payment, and the list shape is lossy. Fix or keep?
  **A:** Keep all three as they are today (DLV-D1 to DLV-D3, accepted risks). (2026-11-11)
- **Q:** BILL — bill totals and payment amounts come from the client and are never reconciled (unnamed lines can't be billed); the app can't open the bill page, payment doesn't free app orders' tables, and the saved-bill endpoint 500s on a bad ID; any role can bill / pay, no void or refund, bill numbering relies on the config-row lock, exception text in errors. Fix or keep?
  **A:** Keep all three as they are today (BILL-D1 to BILL-D3, accepted risks). (2026-11-11)
- **Q:** HIST — the channel filter always errors (app codes 0–3 vs GUID channel IDs); the Live tab ignores search / payment / paging, uses UTC days and drops earlier unpaid orders, while History is empty in practice (no app payments); lists sorted by random GUID, loose search, fragile payment parsing, any role, per-row queries. Fix or keep?
  **A:** Keep all three as they are today (HIST-D1 to HIST-D3, accepted risks). (2026-11-11)
- **Q:** RPT — dashboard figures count paid orders only (all 0 today) while Top Selling and the items export count every line; UTC days, "30 Days" = calendar month, trend ordering / week 5, Top Selling returns all items; the orders export 500s on raw binary IDs while the app reports success, CSV unescaped, any role reads revenue. Fix or keep?
  **A:** Keep all three as they are today (RPT-D1 to RPT-D3, accepted risks). (2026-11-12)
- **Q:** PUB — public QR ordering never succeeds: the menu is empty for hex IDs, every create 500s on the missing `orders.orderChannel` column with no KOT / line IDs, and add-to-order fails on the `int` order ID. Client prices and IDs are trusted, and any restaurant's full menu is readable (inactive / deleted items, unapproved restaurants). No rate limit and raw exception text with customer details in the replies. Fix or keep?
  **A:** Keep all three as they are today (PUB-D1 to PUB-D3, accepted risks). (2026-11-12)
- **Q:** PRN — after a failed Wi-Fi print the app silently prints to and saves the single other device found on the port, and any open port counts as a printer. Thin feedback: no port check, an alert and the print screen at once, outcome unknown, `Share Failed` on bills, iOS share sheet only. The setting is per device and survives sign-out. Fix or keep?
  **A:** Keep all three as they are today (PRN-D1 to PRN-D3, accepted risks). (2026-11-12)
- **Q:** ADM — what does the super-admin panel feature cover?
  **A:** Only the remaining pages: sign-in and out, the restaurant list, the Overview, Order History and Item History pages, the language switch and `/dashboard`. The other panel pages stay in ONB, RST, STAFF, TBL and MENU. Tester and developer cases. (2026-11-12)
- **Q:** ADM — SQL-only admin accounts, sessions outlive deactivation, the "30-day" remember cookie lasts about 400 days, no admin roles, IP-only throttle, Referer redirect on `/set-locale`. Mixed figure rules (paid revenue ÷ all orders), UTC and n + 1-day periods, static subtitles, today-only history pages, random default sort, 500 on a bad sort. Fake `/dashboard`, dead buttons, page-only search, unknown IDs show zeros, untranslated tables. Fix or keep?
  **A:** Keep all three as they are today (ADM-D1 to ADM-D3, accepted risks). (2026-11-12)
- **Q:** Product description: use the draft inferred from the code (small restaurants in India; ₹, GST, FSSAI, UPI; owner / waiter / chef flows; Wi-Fi printing; reports)?
  **A:** Yes, use the draft. It is recorded in `project-overview.md` → What the product is. (2026-11-13)
- **Q:** Role permissions: record the app's menus, or define target rules for the API to enforce?
  **A:** Record the app's menus as the role rules. API enforcement stays an accepted gap, per feature. (2026-11-13)
- **Q:** Developer test runner?
  **A:** PHPUnit 11 feature tests in `laravel-pos-api-dev_guid/tests/Feature/<FeatureFolder>/`: one class per use case, one method per case ID. (2026-11-13)
- **Q:** POS app languages?
  **A:** English only. The app has no translation library. (2026-11-13)

---

## Open

Things still unresolved. When an item is answered, move it into "What we've been told" or "Interview Q&A" above, and append the resolution date in italics to the closing entry. Do not delete the open item — its history matters.

- What the product is, in the client's words; who the customers are. _Resolved 2026-11-13: the draft from the code was accepted by the user (not yet in the client's own words)._
- Which feature gets specified first, and the order of the rest. Suggested: Auth (OTP / PIN), Onboarding, Zones & tables, Menu, Orders & KOT, Billing & payment, Staff, Public QR order, Reports. _Resolved 2026-11-05: AUTH first, then the order in `project-overview.md` → Core features._
- What each role (owner / waiter / chef) may do, and whether the API must enforce it (today only the app hides screens). _Resolved 2026-11-13: the app's menus are the rules; API enforcement is an accepted gap._
- For known bugs (static OTP, client-supplied prices on public orders, no role checks…): do developer tests state the **intended** behaviour (failing until fixed) or record today's behaviour? `conventions.md` currently assumes intended behaviour. _Partly resolved 2026-11-05: decided per feature. For AUTH the user kept today's behaviour (AUTH-D1 to D4), so AUTH tests record it and flag accepted risks._ _Resolved 2026-11-12: every feature (AUTH to ADM) records today's behaviour, with risks accepted as `CODE-Dn` decisions._
- Is the super-admin web panel in scope for specs and tests now, or only the POS app + API? _Resolved 2026-11-12: in scope. Its pages are specified in ONB, RST, STAFF, TBL, MENU and ADM._
- Developer test format: Postman collection, or PHPUnit / Pest feature tests in the Laravel repo? _Resolved 2026-11-13: PHPUnit feature tests._
- POS app supported languages. _Resolved 2026-11-13: English only._

---

## Linked specs

Once formal specs are written from this material, link them here. Until then this section stays empty.

- [features/auth/](../features/auth/) — Sign-in & PIN (2026-11-05)
- [features/onboarding/](../features/onboarding/) — Onboarding & approval (2026-11-06)
- [features/restaurant-settings/](../features/restaurant-settings/) — Restaurant profile & settings (2026-11-06)
- [features/staff/](../features/staff/) — Staff management (2026-11-10)
- [features/tables/](../features/tables/) — Zones & tables (2026-11-10)
- [features/rooms/](../features/rooms/) — Floors & rooms (2026-11-10)
- [features/menu/](../features/menu/) — Menu categories & items (2026-11-10)
- [features/import-export/](../features/import-export/) — Import & export (2026-11-11)
- [features/orders/](../features/orders/) — Take an order & send KOTs (2026-11-11)
- [features/kitchen/](../features/kitchen/) — Kitchen (chef) (2026-11-11)
- [features/waiter/](../features/waiter/) — Waiter: ready orders & dashboard (2026-11-11)
- [features/delivery/](../features/delivery/) — Delivery board (2026-11-11)
- [features/billing/](../features/billing/) — Bill & payment (2026-11-11)
- [features/order-history/](../features/order-history/) — Order history (2026-11-11)
- [features/reports/](../features/reports/) — Reports / analytics (2026-11-12)
- [features/public-order/](../features/public-order/) — Public QR ordering (2026-11-12)
- [features/printer/](../features/printer/) — Printer settings & printing (2026-11-12)
- [features/admin-panel/](../features/admin-panel/) — Super-admin panel: shell & reports (2026-11-12)
