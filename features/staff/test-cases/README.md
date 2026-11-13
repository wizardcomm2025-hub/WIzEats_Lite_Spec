# Staff management — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api-dev_guid/specs/staff/test-results.md`, never here.

**Expected results describe today's behaviour**, as fixed by STAFF-D1 to STAFF-D4 and AUTH-D2 ([README](../README.md#decisions)). Cases that check an accepted risk say so. No case is blocked.

## Fixtures

The AUTH fixtures apply ([AUTH test README](../../auth/test-cases/README.md#fixtures)): REST-A (owner OWN-A, active waiter WTR-A, active chef CHF-A), REST-B / OWN-B, REST-P / OWN-P, WTR-A-OFF, NOREST, PH-NEW, DEV-1 / DEV-2. Additional:

| Fixture | State |
|---|---|
| REST-S / OWN-S | Approved restaurant with **no** staff; owner PIN known |
| WTR-B | Active waiter of REST-B |
| PH-NEW2 … | More unused valid mobile numbers (one per create case) |
| PH-USED | OWN-B's phone (in use in another restaurant) |
| ADMIN-1 | Active super admin (as in ONB) |
| DEVICE-PHONE / DEVICE-TAB | Phone and tablet with the app |

Reset REST-A's staff (names, roles `waiter` / `chef`, both active) before each case. Delete users created by a case afterwards.

## Panel requests (use case 04)

As in ONB ([ONB test README → Panel requests](../../onboarding/test-cases/README.md#panel-requests-use-case-03)). Sign in as ADMIN-1 with a cookie jar; `GET /admin/user-management/<REST-A id>`; take the token from the page script (`"X-CSRF-TOKEN": "…"`). Send `POST /admin/user/update` as JSON with that header.

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 owner sees staff, not self | T01, T02 | D01 |
| 01 | AC2 waiter / chef see own row | T03, T04 | D02 |
| 01 | AC3 limit reached | T05 | — |
| 01 | AC4 no staff | T06 | — |
| 01 | AC5 own restaurant only | — | D04, D05 |
| 01 | Alt: role changed, inactive faded, errors | T07, T08 | D03, D06–D12 |
| 02 | AC1 create waiter | T01 | D01 |
| 02 | AC2 first sign-in with 123456 (STAFF-D4) | T02, T11 | D01 |
| 02 | AC3–AC4 taken roles, button greyed | T03, T04 | — |
| 02 | AC5 field checks | T05, T07 | D02–D04 |
| 02 | AC6 phone in use | T06 | D05, D12, D14 |
| 02 | AC7 any role / any caller (STAFF-D3) | — | D06, D07, D08 |
| 02 | Alt: no network, double tap, close, Unicode | T08, T09, T10, T12 | D09–D11, D13, D15, D16 |
| 03 | AC1 app switch fails (STAFF-D1) | T01, T02 | D02 |
| 03 | AC2 API switch-off and its effect | T05 | D01, D11 |
| 03 | AC3 cross-restaurant (STAFF-D2) | — | D03, D04, D05 |
| 03 | AC4 cancel | T03 | — |
| 03 | Alt: waiter view, bad IDs, validation | T04 | D06–D10, D12–D16 |
| 04 | AC1 list | T01 | D10 |
| 04 | AC2 rename | T02 | D01 |
| 04 | AC3 Inactive frees a slot | T03 | D01 |
| 04 | AC4 `+ Add User` | T04 | — |
| 04 | AC5 table switch not saved | T05 | — |
| 04 | AC6 role `owner` | T06 | D03 |
| 04 | AC7 no session | T08 | D08 |
| 04 | Alt: unknown role, odd values, HTML | T07, T09 | D02, D04–D07, D09, D11, D12 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `getStaffDetails` (01) | D07 | D08 | D04, D05 | D03 | D11 | D06 | n/a¹ | n/a² | n/a¹ | D12 | D06 | D06, D09 |
| `create` (02) | D10 | D11 | D09 | D07 | D16 | D02–D04 | D06, D08 | D09 | D12 | D15 | D13 | D14 |
| `update` (03) | D10 | D11 | D03 | D04 | D16 | D07, D08 | D02, D05 | D09 | D12 | D15 | D13 | D07, D14 |
| panel user update (04) | D08 | n/a³ | D09 | n/a³ | D08 | D02, D04–D06 | D03 | D07 | n/a⁴ | D12 | D11 | D02, D06 |

1. Read only.
2. The only input is `userId`.
3. One super-admin role; the account state belongs to ADM.
4. One field set per user; last write wins like any form.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api-dev_guid/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens: `POST /api/auth/loginPin` as the fixture user (AUTH test README). Check the database with a SQL client; use `HEX(userId)`.
