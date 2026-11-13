# Floors & rooms — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api-dev_guid/specs/rooms/test-results.md`, never here.

**Expected results describe today's behaviour**, as fixed by ROOM-D1 to ROOM-D3 and AUTH-D2 ([README](../README.md#decisions)). Most app actions fail today; the cases record that. No case is blocked.

## Fixtures

The AUTH fixtures apply ([AUTH test README](../../auth/test-cases/README.md#fixtures)): REST-A (OWN-A, WTR-A, CHF-A), REST-B, REST-P / OWN-P, NOREST. Additional:

| Fixture | State |
|---|---|
| RS-ON | REST-A's Room Service switched on in the panel (RST 03) |
| FLOOR-G | REST-A floor `Ground Floor` (active), created through the app or API |
| ROOMS-IMP | Rooms `101`, `102` (active, capacity 2) and `103` (**inactive**) on FLOOR-G — created by Import (IMPEX) or inserted by a developer with correct binary IDs |
| RUN-R | A running Room Service order on room `101` |
| FLOOR-B / ROOM-B1 | REST-B's floor and room |

Reset REST-A's floors and rooms to these fixtures before each case; delete rows a case creates.

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 menu needs Room Service | T01 | D13 |
| 01 | AC2 empty | T02 | — |
| 01 | AC3 add floor | T03 | D02 |
| 01 | AC4 empty name | T04 | D05 |
| 01 | AC5 rename fails (ROOM-D1) | T05 | D03 |
| 01 | AC6 switch fails (ROOM-D1) | T06 | D04 |
| 01 | AC7 room count | T07 | D01 |
| 01 | Alt: silent add failure, duplicates, role, isolation | T08, T09 | D06–D12 |
| 02 | AC1 list empty (ROOM-D1) | T01, T05 | D02 |
| 02 | AC2 add fails (ROOM-D1) | T03 | D04 |
| 02 | AC3 field checks | T04 | D07 |
| 02 | AC4 no floors | T02 | — |
| 02 | AC5 API list | — | D01, D14 |
| 02 | Alt: orphan room, edit fails, inactive hidden, role, isolation | — | D03, D05, D06, D08–D13, D15 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `getFloorDetails` (01) | D10 | D10 | D09 | D08 | D12 | D09 | n/a¹ | n/a¹ | n/a¹ | D12 | D09 | D06 |
| `floorSave` (01) | D10 | D10 | D09 | D08 | D12 | D05, D06 | D13 | D07 | n/a² | D12 | D11 | D06 |
| `getRoomsDetails` (02) | D12 | D12 | D11 | D10 | D12 | D02, D03 | n/a¹ | n/a¹ | n/a¹ | D12 | D03 | D15 |
| `roomSave` (02) | D12 | D12 | D11 | D10 | D12 | D07, D09 | D05, D08 | D13 | n/a² | D12 | D05 | D15 |

1. Read only.
2. Updates can't succeed today (ROOM-D1); creates don't conflict.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api-dev_guid/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin`. Check the database with a SQL client (`HEX(floorId)`, `HEX(roomId)`).
