# Import & export — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api-dev_guid/specs/import-export/test-results.md`, never here.

**Expected results describe today's behaviour**, as fixed by IMPEX-D1 to IMPEX-D3 and AUTH-D2 ([README](../README.md#decisions)). There are **no tester cases**: the feature has no UI (IMPEX-D1). No case is blocked.

## Fixtures

The AUTH fixtures apply ([AUTH test README](../../auth/test-cases/README.md#fixtures)): REST-A (OWN-A, WTR-A, CHF-A), REST-B (OWN-B), REST-P / OWN-P, NOREST. REST-A starts with zone `Main Hall` (Table 1–4), category `Starters` (active), and an **inactive** zone `Patio`. Files (UTF-8, comma-separated, first line = header):

| Fixture | Content after the header |
|---|---|
| ZONE-OK | `Main Hall,Table 5,4` · `Garden,G1,2` · `garden,G2,2` |
| ZONE-BAD | `,Table X,4` · `Lobby,,2` · `Lobby,L1,four` · `Lobby,L2,-3` · `Patio,P9,2` · `Lobby,<256 × a>,2` |
| MENU-OK | `Starters,Veg Roll,Crispy,1,120,` · `Mains,Butter Chicken,,0,350,x.png` |
| MENU-BAD | `Mains,Free Item,,yes,abc,` · `Mains,Neg Item,,1,-50,` · `Mains,Huge,,1,1e9,` · `,No Cat,,1,10,` · `Mains,'); DROP TABLE menu_items; --,,1,10,` · `Mains,=1+1,,1,10,` |
| FLOOR-OK | `Ground Floor,101,2` · `Ground Floor,102,2` |
| HEADER-ONLY | only the header line |
| NOT-CSV | a PNG image renamed `zone.csv` |
| BIG | a valid zone CSV over 10 MB |

Reset REST-A's zones, tables, menu, floors and rooms before each case.

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Developer IDs |
|---|---|---|
| 01 | AC1 zones and tables | D01 |
| 01 | AC2 menu | D02 |
| 01 | AC3 floors and rooms | D03 |
| 01 | AC4 skipped rows | D05, D06 |
| 01 | AC5 duplicates (IMPEX-D2) | D07 |
| 01 | AC6 no file | D08 |
| 01 | Alt: type, size, header only, inactive parent, all three files, role, isolation, errors | D04, D09–D18 |
| 02 | AC1–AC3 | 02-D01–D05 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `data/import` (01) | D13 | D14 | D12 | D11 | D13 | D08–D10, D05, D06 | D07, D06 | D15 | D16 | D17 | D18 | D10, D04 |
| `export/*` (02) | 02-D03 | 02-D03 | n/a¹ | 02-D04 | 02-D03 | n/a² | n/a¹ | n/a² | n/a¹ | 02-D05 | n/a² | 02-D02 |

1. Returns a fixed template; no restaurant data.
2. Takes no input.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api-dev_guid/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin`. Send imports as `multipart/form-data`. Check the database with a SQL client, and Home / ordering data with `showDetails` and `getMenuItems`. Read the Laravel log for skipped rows.
