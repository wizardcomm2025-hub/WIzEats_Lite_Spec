# Waiter — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api-dev_guid/specs/waiter/test-results.md`, never here.

**Expected results describe today's behaviour**, as fixed by WAIT-D1 to WAIT-D3 and the inherited decisions ([README](../README.md#decisions)). No case is blocked. Because no ready card is ever shown (WAIT-D1), the tester cases check the screens as they are today; pick-up is covered by developer cases.

## Fixtures

The AUTH, ORD and KITCH fixtures apply ([AUTH](../../auth/test-cases/README.md#fixtures), [ORD](../../orders/test-cases/README.md#fixtures), [KITCH](../../kitchen/test-cases/README.md#fixtures) — including KOT-P, KOT-R, KOT-B and the RAW(x) path form). Additional:

| Fixture | State |
|---|---|
| KOT-BR | A REST-B KOT set to status 4 by SQL, `isPickedUp` 0 |
| PICK-NONE | No KOT in the **whole** database has `isPickedUp` 1 (`UPDATE kots SET isPickedUp = 0, pickedUp_at = NULL` before the case). The pick-up reply spans all restaurants (WAIT-D2), so other data in a shared database changes the result. |
| DASH-A | WTR-A created five REST-A orders through the API: ORD-Y1–Y3 (`created_at` moved to yesterday by SQL) and ORD-T1, ORD-T2 today. ORD-T1 has `Paneer Tikka` × 2 (500.00) and is paid in full (status 2, `grandTotal` 525.00). ORD-T2 has `Chicken 65` × 1 and is unpaid. OWN-A created one order today. REST-A has 4 tables. |

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 empty list | T01, T02, T05 | D01 |
| 01 | AC2 non-empty list fails (WAIT-D1) | T03, T04 | D02, D09 |
| 01 | AC3 restaurant scope | — | D04, D07 |
| 01 | Alt: picked-up / not-ready KOTs, role, auth, rate | — | D03, D05, D06, D08 |
| 02 | AC1 hex ID no-op (WAIT-D1) | T01 | D01, D07 |
| 02 | AC2 raw ID marks it; reply 500 | — | D02, D12 |
| 02 | AC3 any restaurant / any stage (WAIT-D2) | — | D03, D04, D05 |
| 02 | Alt: Delivered tab not kept (WAIT-D2) | T02 | — |
| 02 | Alt: role, over-posting, concurrency, auth, rate | — | D06, D08–D11 |
| 03 | AC1 all-time Total Orders (WAIT-D3) | T01, T04 | D01, D02 |
| 03 | AC2 top items: today's paid orders only | T02, T03 | D01, D02, D05 |
| 03 | AC3 only the caller's orders | T05 | D03, D04 |
| 03 | Alt: auth, over-posting, rate | — | D06–D08 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `waiter/ready-kots` (01) | D06 | D06 | D04 | D05 | D06 | n/a² | D03 | D07 | n/a¹ | D08 | D07 | D02, D09 |
| `waiter/updatePickedUpStatus/{kotId}` (02) | D10 | D10 | D03, D04 | D06 | D10 | D07 | D05 | D08 | D09 | D11 | D07 | D02, D12 |
| `waiter/dashboard` (03) | D06 | D06 | D03 | D04 | D06 | n/a² | D05 | D07 | n/a¹ | D08 | D07 | D06 |

1. Read only.
2. No input is read.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api-dev_guid/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin`. Check the database with a SQL client (`SELECT HEX(kotId), status, isPickedUp, pickedUp_at FROM kots`). Run RAW(x) calls against `php artisan serve`.
