# Menu — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api-dev_guid/specs/menu/test-results.md`, never here.

**Expected results describe today's behaviour**, as fixed by MENU-D1 to MENU-D4 and AUTH-D2 ([README](../README.md#decisions)). Most edit actions fail today; the cases record that. No case is blocked.

## Fixtures

The AUTH fixtures apply ([AUTH test README](../../auth/test-cases/README.md#fixtures)): REST-A (OWN-A, WTR-A, CHF-A), REST-B, REST-P / OWN-P, NOREST. Additional (created through the API create path or Import):

| Fixture | State |
|---|---|
| CAT-S | REST-A category `Starters` (active) |
| ITEM-1, ITEM-2 | In CAT-S: `Paneer Tikka` 250.00 Veg, `Chicken 65` 300.00 Non-Veg — active |
| ITEM-3 | In CAT-S: `Old Dish` 100.00 — **inactive** |
| CAT-D / ITEM-4 | REST-A category `Drinks` with `Lime Soda` 60.00 |
| CAT-B / ITEM-B1 | REST-B category `B Mains` with `B Curry` |
| IMG-PNG | A small valid PNG as `data:image/png;base64,…` |
| ADMIN-1 | Active super admin |

Reset REST-A's menu to these fixtures before each case; delete rows and image files a case creates.

## Panel requests (use case 03)

As in ONB ([ONB test README → Panel requests](../../onboarding/test-cases/README.md#panel-requests-use-case-03)): sign in as ADMIN-1, `GET` the page, take the CSRF token from the page script, and POST JSON with `X-CSRF-TOKEN`.

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1–AC2 list and expand | T01, T02 | D01 |
| 01 | AC3 add | T03 | D02 |
| 01 | AC4 empty name | T04 | D05 |
| 01 | AC5 edit fails (MENU-D2) | T05 | D03 |
| 01 | AC6 switch fails (MENU-D2) | T06 | D04 |
| 01 | Alt: empty, silent failure, duplicates, role, isolation | T07, T08 | D06–D14 |
| 02 | AC1 Item Management empty (MENU-D2) | T01, T02 | D03, D04 |
| 02 | AC2 add fails (MENU-D1) | T03 | D05 |
| 02 | AC3 field checks | T04 | D07 |
| 02 | AC4 image picking | T05, T06 | — |
| 02 | AC5 ordering menu | T07 | D01, D02 |
| 02 | Alt: API create, edit fails, price, category, image, role, paging | T08 | D06, D08–D20 |
| 03 | AC1–AC3 categories page | T01, T02, T03 | D01, D02, D03 |
| 03 | AC4–AC6 items page (MENU-D3) | T04, T05, T06 | D04, D05, D06 |
| 03 | Alt: empty name, session | T07 | D07–D10 |
| 04 | AC1–AC3 | n/a — no screen | D01–D08 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `getItemsCategory` (01) | D10 | D11 | D09 | D08 | D13 | D12 | n/a¹ | n/a¹ | n/a¹ | D14 | D12 | D12 |
| `categorySave` (01) | D10 | D11 | D09 | D08 | D13 | D05, D06 | D04 | D07 | n/a² | D14 | D07 | D06 |
| `getMenuItems` / `showCategoryMenuItems` (02) | D17 | D17 | D15 | D16 | D17 | D03, D04, D14 | n/a¹ | n/a¹ | n/a¹ | D17 | D04 | D04 |
| `menuItemSave` (02) | D17 | D17 | D10 | D16 | D17 | D07, D09, D11 | D08, D10, D12 | D13 | D19 | D17 | D18 | D20 |
| `image/upload` (04) | D07 | D07 | n/a³ | D07 | D07 | D02–D05 | D04 | n/a⁴ | n/a⁴ | D08 | D04 | D03, D06 |
| panel category / item routes (03) | D09 | n/a⁵ | D08 | n/a⁵ | D09 | D07 | D03 | D10 | n/a⁶ | D10 | D10 | D05, D07 |

1. Read only.
2. Updates can't succeed today (MENU-D2); creates don't conflict.
3. Files aren't tied to a restaurant.
4. One field; every call writes a new file.
5. One super-admin role; its account state belongs to ADM.
6. Plain last-write-wins updates.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api-dev_guid/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin`. Check the database with a SQL client (`HEX(...)` for IDs) and the `storage/app/public/image/` folder for files.
