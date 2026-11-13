# Kitchen (chef) — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api-dev_guid/specs/kitchen/test-results.md`, never here.

**Expected results describe today's behaviour**, as fixed by KITCH-D1 to KITCH-D3 and the inherited decisions ([README](../README.md#decisions)). No case is blocked. Because the chef's list never loads (KITCH-D1), the tester cases check what the screens show today; KOT movement is covered by developer cases.

## Fixtures

The AUTH fixtures apply ([AUTH test README](../../auth/test-cases/README.md#fixtures)): REST-A (OWN-A, WTR-A, CHF-A), REST-B (OWN-B), WTR-A-OFF, NOREST, DEV-1 / DEV-2. The ORD fixtures apply too ([ORD test README](../../orders/test-cases/README.md#fixtures)). Additional:

| Fixture | State |
|---|---|
| CHF-A | REST-A chef, `role` exactly `Chef` (as the app saves it) |
| CHF-A-LC | REST-A chef with `role` `chef`, created through the panel or the staff API |
| KOT-P | A REST-A KOT, status 0, created today through `POST /api/orders/createOrder` (ORD basic body, `kotNote` `"less spicy"`) |
| KOT-R | A REST-A KOT set to status 4 by SQL, `isPickedUp` 0 |
| KOT-B | A REST-B KOT, status 0 |
| RAW(x) | KOT `x`'s 16 raw bytes, percent-encoded for the path: take `HEX(kotId)` and put `%` before every pair (`%3A%9F…`). Re-create the KOT if its ID has a `00` or `2F` byte (a front web server or the router may reject those). Run against `php artisan serve`. |

Unless a case says otherwise, REST-A has no KOTs other than the ones named.

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 empty list | T01, T02 | D01 |
| 01 | AC2 non-empty list fails (KITCH-D1) | T03, T04, T05, T07 | D02, D03, D12 |
| 01 | AC3 restaurant scope | — | D04, D10 |
| 01 | AC4 any role (KITCH-D2) | — | D07 |
| 01 | Alt: validation, unlabeled status, chef shell | T06 | D05, D06, D08, D09, D11 |
| 02 | AC1 hex ID no-op (KITCH-D1) | T01 | D01, D07 |
| 02 | AC2 raw ID update | T02 | D02, D03 |
| 02 | AC3 other restaurant (KITCH-D2) | — | D04 |
| 02 | AC4 out-of-range status (KITCH-D2) | — | D06 |
| 02 | Alt: role, over-posting, concurrency, picked-up KOT | — | D05, D08–D13 |
| 03 | AC1 zero counts (KITCH-D1) | T01, T02 | D01 |
| 03 | AC2 counts follow the last change (KITCH-D3) | T03, T04, T05 | D02, D03, D04, D05 |
| 03 | AC3 lowercase role (KITCH-D3) | T06 | — |
| 03 | Alt: role, auth, over-posting, rate | — | D06–D09 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `chef/pending-kots` (01) | D08 | D08 | D04 | D07 | D08 | D05 | D06 | D10 | n/a¹ | D11 | D09 | D02, D12 |
| `chef/update-status/{kotId}` (02) | D10 | D10 | D04 | D05 | D10 | D06 | D03, D13 | D08 | D09 | D11 | D07 | D06, D12 |
| `chef/dashboard` (03) | D07 | D07 | D05 | D06 | D07 | n/a² | D03, D04 | D08 | n/a¹ | D09 | D08 | D07 |

1. Read only.
2. No input.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api-dev_guid/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin`. Check the database with a SQL client (`SELECT HEX(kotId), status, HEX(chefId), preparing_at, rejected_at, completed_at FROM kots`).
