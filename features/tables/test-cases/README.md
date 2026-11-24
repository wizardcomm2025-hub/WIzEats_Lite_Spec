# Zones & tables — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api/api/specs/tables/test-results.md`, never here.

**Expected results follow the decision statuses** in the [README](../README.md#decisions) (re-synced 2026-11-24, rules in [conventions.md → Decision status](../../../conventions.md#decision-status)):

- TBL-D1 is *fixed in the app*: Table Management cases assert the working list and pass today.
- *To fix* decisions (TBL-D2 to TBL-D5, AUTH-D2): the case asserts the **target** and is tagged `Fails until <ID> is fixed`; today's result is in brackets.

No case is blocked.

## Fixtures

The AUTH fixtures apply ([AUTH test README](../../auth/test-cases/README.md#fixtures)): REST-A (OWN-A, WTR-A, CHF-A), REST-B (OWN-B), REST-P / OWN-P, WTR-A-OFF, NOREST, DEV-1. Additional:

| Fixture | State |
|---|---|
| MAIN-A | REST-A's zone `Main Hall` (active) with `Table 1`–`Table 4` (capacity 4, active, status 0) |
| PATIO-A | REST-A zone `Patio`, **inactive**, with table `P1` (inactive) |
| RUN-1 | A running order (status 0) on REST-A `Table 1` |
| ZONE-B / TABLE-B1 | REST-B's zone `B Hall` and its table `B1` |
| ADMIN-1 | Active super admin |

Reset REST-A to these fixtures before each case. Delete rows a case creates.

## Panel requests (use case 03)

As in ONB ([ONB test README → Panel requests](../../onboarding/test-cases/README.md#panel-requests-use-case-03)): sign in as ADMIN-1, `GET` the zones or tables page, and take the CSRF token from the page script. Send the POSTs as JSON with `X-CSRF-TOKEN`. IDs are the upper-case hex the page shows.

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 list with count | T01 | D01 |
| 01 | AC2 add | T02 | D02 |
| 01 | AC3 rename | T03 | D03 |
| 01 | AC4–AC5 switch off / on cascades | T04, T05 | D04, D05 |
| 01 | AC6 empty name | T06 | D06 |
| 01 | AC7 owner only (target TBL-D3) | — | D09 |
| 01 | AC8 duplicate zone name refused and shown (target TBL-D2, TBL-D5) | T09, T10 | D17 |
| 01 | Alt: add failure shown (target TBL-D5), count after switch, delete, not found | T07, T08 | D07, D08, D10–D16 |
| 02 | AC1 list shows the zone's tables (TBL-D1 fixed) | T01, T02 | D01 |
| 02 | AC2 add shows on Home | T03 | D02 |
| 02 | AC3 duplicate on create | T04 | D03 |
| 02 | AC4 field checks | T05 | D04 |
| 02 | AC5 switch in the app hides on Home | T07 | D05 |
| 02 | Main flow: edit | T11 | — |
| 02 | AC6 rename to a duplicate refused (target TBL-D2) | T08 | D06 |
| 02 | AC7 one row per table (target TBL-D4) | T10 | D20 |
| 02 | Alt: no zones, inactive zone, server error, cross-zone, filters | T06, T09 | D07–D19 |
| 03 | AC1–AC3 zones page | T01, T02, T03 | D01, D02, D03 |
| 03 | AC4–AC6 tables page | T04, T05, T06 | D04, D05 |
| 03 | AC7 table switch not saved | T07 | — |
| 03 | Alt: empty values, duplicates (target TBL-D2), session | T08, T09 | D06–D12 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `getTablesZoneDetails` (01) | D11 | D12 | D13 | D09 | D16 | D14 | n/a¹ | n/a¹ | n/a¹ | D15 | D14 | D14 |
| `ZoneSave` (01) | D11 | D12 | D07 | D09 | D16 | D06, D08 | D04, D10, D17 | D02 | D05 | D15 | D03 | D07, D08 |
| `getTablesDetails` (02) | D17 | D17 | D13 | D17 | D17 | D15 | n/a¹ | n/a¹ | n/a¹ | D17 | D15 | D15 |
| `tableSave` (02) | D17 | D17 | D07, D08 | D09 | D17 | D04, D10, D11 | D06, D12 | D14 | D16 | D17 | D18 | D19 |
| panel zone / table routes (03) | D10 | n/a² | D11 | n/a² | D10 | D06, D07 | D03, D08 | D09 | n/a³ | D12 | D12 | D06, D07 |

1. Read only.
2. One super-admin role; its account state belongs to ADM.
3. Plain last-write-wins updates.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api/api/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin` (AUTH test README). Check the database with a SQL client (`HEX(zoneId)`, `HEX(tableId)`). Check Home's data with `POST /api/restaurants/showDetails` (RST 01).
