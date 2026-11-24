# Kitchen (chef) — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api/api/specs/kitchen/test-results.md`, never here.

**Expected results follow the decision statuses** in the [README](../README.md#decisions) (re-synced 2026-11-24, rules in [conventions.md → Decision status](../../../conventions.md#decision-status)). All three decisions are *To fix* and no KITCH code has changed, so most cases assert the **target** and are tagged `Fails until <ID> is fixed`, with today's result in brackets. Tester cases that move a KOT need KITCH-D1 fixed first; until then they fail at the first step. No case is blocked.

## Fixtures

The AUTH fixtures apply ([AUTH test README](../../auth/test-cases/README.md#fixtures)): REST-A (OWN-A, WTR-A, CHF-A), REST-B (OWN-B), REST-P / OWN-P, WTR-A-OFF, NOREST, DEV-1 / DEV-2. The ORD fixtures apply too ([ORD test README](../../orders/test-cases/README.md#fixtures)). Additional:

| Fixture | State |
|---|---|
| CHF-A | REST-A chef, `role` exactly `Chef` (as the app saves it) |
| CHF-A-LC | REST-A chef with `role` `chef`, created through the panel or the staff API (deactivate CHF-A first — STAFF limit) |
| KOT-P | A REST-A KOT on `Table 1`, status 0, created today through `POST /api/orders/createOrder` (`kotNote` `"less spicy"`, `Paneer Tikka` × 2) |
| KOT-R | A REST-A KOT set to status 4 by SQL, `isPickedUp` 0 |
| KOT-X | A REST-A KOT set to status 2 (Rejected) by SQL |
| KOT-B | A REST-B KOT, status 0 |
| RAW(x) | KOT `x`'s 16 raw bytes, percent-encoded for the path: take `HEX(kotId)` and put `%` before every pair (`%3A%9F…`) — only for the "today" cases that show the raw-ID hole |

Unless a case says otherwise, REST-A has no KOTs other than the ones named.

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 empty list | T01, T02 | D01 |
| 01 | AC2 KOTs listed with number and place (target KITCH-D1) | T03, T04, T05, T07 | D02, D03, D12, D13 |
| 01 | AC3 restaurant scope | — | D04, D10 |
| 01 | AC4 chef only (target KITCH-D2) | — | D07 |
| 01 | Alt: validation, chef shell, account state | T06 | D05, D06, D08, D09, D11 |
| 02 | AC1 update by hex ID (target KITCH-D1) | T01 | D01, D07, D14 |
| 02 | AC2 stage times kept (target KITCH-D2) | T02 | D02 |
| 02 | AC3 other restaurant → 404 (target) | — | D04 |
| 02 | AC4 stage path and values enforced (target KITCH-D2) | T03 | D03, D06, D13 |
| 02 | AC5 card moves between tabs (target) | T01, T02 | — |
| 02 | Alt: role, over-posting, concurrency, auth | — | D05, D08–D12 |
| 03 | AC1 zero counts | T01 | D01 |
| 03 | AC2 restaurant counts (target KITCH-D3) | T02–T05 | D02, D03, D04, D05 |
| 03 | AC3 lowercase role (target KITCH-D3) | T06 | — |
| 03 | Alt: role, auth, over-posting, rate | — | D06–D09 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `pending-kots` (01) | D08 | D08 | D04, D10 | D07 | D08 | D05 | D06 | D10 | n/a¹ | D11 | D09 | D12 |
| `update-status` (02) | D10 | D10 | D04 | D05 | D10 | D06, D07 | D03, D13 | D08 | D09 | D11 | D07 | D12 |
| `dashboard` (03) | D07 | D07 | D05 | D06 | D07 | n/a² | n/a¹ | D08 | n/a¹ | D09 | D08 | D07 |

1. Read only.
2. No input.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api/api/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin`. Check the database with a SQL client (`HEX(kotId)`).
