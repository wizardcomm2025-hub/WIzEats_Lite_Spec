# Waiter — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api/api/specs/waiter/test-results.md`, never here.

**Expected results follow the decision statuses** in the [README](../README.md#decisions) (re-synced 2026-11-24, rules in [conventions.md → Decision status](../../../conventions.md#decision-status)). All three decisions are *To fix* and no WAIT code has changed, so most cases assert the **target** and are tagged `Fails until <ID> is fixed`, with today's result in brackets. App cases that need a Ready KOT also need KITCH-D1 fixed (or a developer setting the KOT to Ready by SQL). No case is blocked.

## Fixtures

The AUTH, ORD and KITCH fixtures apply ([AUTH](../../auth/test-cases/README.md#fixtures), [ORD](../../orders/test-cases/README.md#fixtures), [KITCH](../../kitchen/test-cases/README.md#fixtures)): REST-A (OWN-A, WTR-A, CHF-A), REST-B, WTR-A-OFF, NOREST, DEV-1 / DEV-2; KOT-P (Pending, Table 1), KOT-R (Ready, not picked up), RAW(x). Additional:

| Fixture | State |
|---|---|
| KOT-BR | A REST-B KOT set to status 4 by SQL, `isPickedUp` 0 |
| KOT-RP | A REST-A KOT with status 4 and `isPickedUp` 1, `pickedUp_at` today |
| PICK-NONE | No KOT in the **whole** database has `isPickedUp` 1 (`UPDATE kots SET isPickedUp = 0, pickedUp_at = NULL`) — needed only for the "today" results, whose reply spans all restaurants |
| DASH-A | WTR-A created five REST-A orders through the API: ORD-Y1–Y3 (`created_at` moved to yesterday by SQL) and ORD-T1, ORD-T2 today. ORD-T1 has `Paneer Tikka` × 2 (500.00) and is paid in full (status 2, `grandTotal` 525.00). ORD-T2 has `Chicken 65` × 1 and is unpaid. OWN-A created one order today. REST-A has 4 tables. |

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 empty list | T01, T02, T05 | D01, D03 |
| 01 | AC2 ready KOT listed with number and place (target WAIT-D1) | T03, T04 | D02, D10 |
| 01 | AC3 restaurant scope | — | D04, D07 |
| 01 | Alt: role, auth, rate, errors | — | D05, D06, D08, D09 |
| 02 | AC1 pick-up by hex ID (target WAIT-D1) | T01 | D01, D07, D13 |
| 02 | AC2 only Ready, only once (target WAIT-D2) | — | D05, D14 |
| 02 | AC3 other restaurant → 404 (target WAIT-D2) | — | D03, D04 |
| 02 | AC4 Delivered tab from the server (target WAIT-D2) | T02, T03 | D15 |
| 02 | Alt: role, over-posting, concurrency, auth, rate, errors | — | D02, D06, D08–D12 |
| 03 | AC1 today's orders (target WAIT-D3) | T01, T04, T05 | D01, D02 |
| 03 | AC2 top selling | T02, T03 | D01, D05 |
| 03 | AC3 own orders only | T05 | D03 |
| 03 | Alt: role, auth, over-posting, rate | — | D04, D06–D08 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `ready-kots` (01, 02) | D06 | D06 | D04, D07 | D05 | D06 | 02-D15 | n/a¹ | D07 | n/a¹ | D08 | D07 | D09 |
| `updatePickedUpStatus` (02) | D10 | D10 | D03, D04 | D06 | D10 | D07, D13 | D05, D14 | D08 | D09 | D11 | D07 | D12 |
| `dashboard` (03) | D06 | D06 | D03 | D04 | D06 | n/a² | D05 | D07 | n/a¹ | D08 | D07 | D06 |

1. Read only.
2. No input.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api/api/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin`. Check the database with a SQL client (`HEX(kotId)`).
