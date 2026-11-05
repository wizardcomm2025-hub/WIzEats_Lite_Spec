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

---

## Open

Things still unresolved. When an item is answered, move it into "What we've been told" or "Interview Q&A" above, and append the resolution date in italics to the closing entry. Do not delete the open item — its history matters.

- What the product is, in the client's words; who the customers are.
- Which feature gets specified first, and the order of the rest. Suggested: Auth (OTP / PIN), Onboarding, Zones & tables, Menu, Orders & KOT, Billing & payment, Staff, Public QR order, Reports. _Resolved 2026-11-05: AUTH first, then the order in `project-overview.md` → Core features._
- What each role (owner / waiter / chef) may do, and whether the API must enforce it (today only the app hides screens).
- For known bugs (static OTP, client-supplied prices on public orders, no role checks…): do developer tests state the **intended** behaviour (failing until fixed) or record today's behaviour? `conventions.md` currently assumes intended behaviour. _Partly resolved 2026-11-05: decided per feature. For AUTH the user kept today's behaviour (AUTH-D1 to D4), so AUTH tests record it and flag accepted risks._
- Is the super-admin web panel in scope for specs and tests now, or only the POS app + API?
- Developer test format: Postman collection, or PHPUnit / Pest feature tests in the Laravel repo?
- POS app supported languages.

---

## Linked specs

Once formal specs are written from this material, link them here. Until then this section stays empty.

- [features/auth/](../features/auth/) — Sign-in & PIN (2026-11-05)
