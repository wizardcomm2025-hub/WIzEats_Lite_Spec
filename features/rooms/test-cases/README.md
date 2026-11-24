# Floors & rooms — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api/api/specs/rooms/test-results.md`, never here.

**Expected results follow the decision statuses** in the [README](../README.md#decisions) (re-synced 2026-11-24, rules in [conventions.md → Decision status](../../../conventions.md#decision-status)):

- ROOM-D1 is *fixed in code* (2026-11-16) and in the app: its cases assert the working add / rename / switch and pass today.
- *To fix* decisions (ROOM-D2 to ROOM-D6, AUTH-D2): the case asserts the **target** and is tagged `Fails until <ID> is fixed`; today's result is in brackets.

No case is blocked.

## Fixtures

The AUTH fixtures apply ([AUTH test README](../../auth/test-cases/README.md#fixtures)): REST-A (OWN-A, WTR-A, CHF-A), REST-B, REST-P / OWN-P, WTR-A-OFF, NOREST. Additional:

| Fixture | State |
|---|---|
| RS-ON | REST-A's Room Service switched on in the panel (RST 03) |
| FLOOR-G | REST-A floor `Ground Floor` (active) |
| FLOOR-1 | REST-A floor `First Floor` (active), no rooms |
| ROOMS-IMP | Rooms `101`, `102` (active, capacity 2) and `103` (**inactive**) on FLOOR-G — created in the app, by Import (IMPEX), or by a developer |
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
| 01 | AC5 rename (ROOM-D1 fixed) | T05 | D03 |
| 01 | AC6 switch cascades to rooms (ROOM-D1 fixed) | T06 | D04 |
| 01 | AC7 room count | T07 | D01 |
| 01 | AC8 duplicate floor name (target ROOM-D3, ROOM-D6) | T09 | D14 |
| 01 | Alt: add failure shown (target ROOM-D6), not found, delete, filter, role, isolation | T08 | D06–D12, D15, D16 |
| 02 | AC1 floor's rooms listed (ROOM-D1 fixed; inactive target ROOM-D5) | T01, T05 | D01, D02, D19 |
| 02 | AC2 add room | T03 | D04 |
| 02 | AC3 field checks (capacity ≥ 1 target ROOM-D3) | T04 | D07, D08 |
| 02 | AC4 no floors | T02 | — |
| 02 | AC5 edit | T06 | D06 |
| 02 | AC6 switch; switched-off room stays listed (target ROOM-D5) | T07 | D06, D19 |
| 02 | AC7 duplicate room name (target ROOM-D3, ROOM-D6) | T08 | D17 |
| 02 | Alt: invalid floor, move, one row per room, roomId filter, role, isolation, order-screen status | — | D03, D05, D09–D16, D18 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `getFloorDetails` (01) | D10 | D10 | D09 | D08 | D12 | D09 | n/a¹ | n/a¹ | n/a¹ | D12 | D09 | D06 |
| `floorSave` (01) | D10 | D10 | D15 | D08 | D12 | D05, D06 | D13, D14, D16 | D07 | n/a² | D12 | D11 | D06 |
| `getRoomsDetails` (02) | D12 | D12 | D11 | D10, D19 | D12 | D02, D03 | n/a¹ | n/a¹ | n/a¹ | D12 | D03 | D15 |
| `roomSave` (02) | D12 | D12 | D05, D11 | D10, D18 | D12 | D07, D09 | D08, D16, D17 | D13 | n/a² | D12 | D05 | D15 |

1. Read only.
2. Plain last-write-wins updates; creates don't conflict.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api/api/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin`. Check the database with a SQL client (`HEX(floorId)`, `HEX(roomId)`).
