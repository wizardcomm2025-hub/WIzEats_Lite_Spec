# Super-admin panel — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api/api/specs/admin-panel/test-results.md`, never here.

**Expected results describe today's behaviour**, as fixed by ADM-D1 to ADM-D3 and the inherited decisions ([README](../README.md#decisions)). No case is blocked. Tester cases use a desktop browser; developer cases send HTTP requests with a session cookie (and the CSRF token for POSTs).

## Fixtures

The AUTH, ONB, ORD and RPT fixtures apply ([AUTH](../../auth/test-cases/README.md#fixtures), [ONB](../../onboarding/test-cases/README.md#fixtures), [ORD](../../orders/test-cases/README.md#fixtures), [RPT](../../reports/test-cases/README.md#fixtures)). Additional:

| Fixture | State |
|---|---|
| ADM-1 | A super admin (`status` 1), username `qa.admin`; the password is in the test environment's seed notes |
| ADM-OFF | A super admin with `status` 0 and a known password |
| REST-MANY | Exactly 12 onboarded restaurants (REST-A, REST-B, REST-P, REST-R, REST-OFF and 7 more), with distinct `created_at`; REST-LEG (onboarding not done) also exists |
| ORDER-LIVE / ORDER-PAID | As in RPT: REST-A Takeaway orders today, `Paneer Tikka` × 2 each. ORDER-PAID billed with TOT-525 and paid (`orderStatus` 2, `grandTotal` 525.00) |
| ORDER-OLD | As in HIST: a REST-A order with `created_at` moved to yesterday by SQL |
| PAID-AT(x) | As in RPT: a paid REST-A order whose `created_at` is moved to `x` by SQL |
| SESS | A signed-in ADM-1 browser session (cookie `laravel_session` + `XSRF-TOKEN`), for the developer cases |

`APP_DEBUG=false` unless a case says otherwise. "Today" means the UTC date (ADM-D2).

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 sign in | T01 | D01 |
| 01 | AC2 / AC3 wrong password, switched off | T02, T03 | D02 |
| 01 | AC4 throttle | T04 | D03 |
| 01 | AC5 logout; guest redirect | T05, T06 | D04, D05 |
| 01 | Alt: empty fields, expired page, dead links | T07, T08, T09 | D06 |
| 01 | Alt: remember cookie, sessions outlive status (ADM-D1) | — | D07, D08 |
| 02 | AC1 counts, paging, onboarding filter | T01, T02 | D01 |
| 02 | AC2 / AC3 status and approval columns | T03 | — |
| 02 | AC4 details row | T04 | — |
| 02 | AC5 page-only search (ADM-D3) | T05 | — |
| 02 | Alt: `/dashboard`, page query, escaping | T06 | D02, D03, D04 |
| 03 | AC1 / AC2 figures (ADM-D2) | T01, T02 | D01 |
| 03 | AC3 / AC4 periods, UTC | T03, T04 | D02 |
| 03 | Alt: unknown ID, unknown filter, dead buttons, sidebar, rank format | T05, T06, T07 | D03, D04, D05 |
| 04 | AC1 / AC4 Order History | T01, T03 | D01 |
| 04 | AC2 / AC3 Item History and sorting | T02, T04 | D02 |
| 04 | Alt: header period, bad sort, untranslated headings | T05, T06 | D03, D04 |
| 05 | AC1–AC3 language | T01, T02, T03 | D01 |
| 05 | Alt: unsupported locale, Referer redirect, no sign-in (ADM-D1) | — | D02, D03 |

## Developer endpoint checklist

| Route | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `POST /admin/login` (01) | D01, D06 | D02, D08 | n/a¹ | n/a² | D05 | D06 | D02 | D06 | n/a³ | D03 | D06 | D02 |
| `POST /admin/logout` (01) | D04 | — | n/a¹ | n/a² | — | — | — | — | n/a³ | n/a | — | D04 |
| `GET /admin`, `/dashboard` (02) | D02 | D08 | n/a¹ | n/a² | D05 (01) | D03 | — | — | n/a³ | D02 | D04 | D03 |
| `GET /admin/analytics/{id}` (03) | D03 | D08 (01) | D03 | n/a² | D05 (01) | D03, D04 | D04 | — | n/a³ | D05 | D03 | D03 |
| `GET /admin/orders`, `/items` (04) | D03 | — | D01 | n/a² | D05 (01) | D03 | — | — | n/a³ | D04 | D03 | D03 |
| `GET /set-locale/{locale}` (05) | D03 | — | n/a¹ | n/a² | — | D02 | — | — | n/a³ | D03 | D02 | D03 |

1. Super admins see every restaurant by design (ADM-D1).
2. No roles among super admins (ADM-D1).
3. Read-only pages, or a sign-in with no shared state.

## How to run

- **Tester cases:** a desktop browser (Chrome), the panel at `APP_URL`.
- **Developer cases:** PHPUnit feature tests (decided 2026-11-13) in `laravel-pos-api/api/tests/Feature/AdminPanel/`, one class per use case and one test method per case ID. In PHPUnit, sign in with `actingAs($admin, 'admin')`, except for the sign-in cases themselves. By hand, to get a session: `GET /` (it sets the cookies), read the `csrf-token` meta tag, then `POST /admin/login` with `X-CSRF-TOKEN`. Check the database with a SQL client.
