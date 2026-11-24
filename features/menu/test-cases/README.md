# Menu — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api/api/specs/menu/test-results.md`, never here.

**Expected results follow the decision statuses** in the [README](../README.md#decisions) (re-synced 2026-11-24, rules in [conventions.md → Decision status](../../../conventions.md#decision-status)):

- MENU-D1 (fixed in the app) and MENU-D2 (fixed in code and the app), and the item rules added 2026-11-18: their cases assert the working behaviour and pass today.
- *To fix* decisions (MENU-D3 to MENU-D6, AUTH-D2): the case asserts the **target** and is tagged `Fails until <ID> is fixed`; today's result is in brackets.

No case is blocked.

## Fixtures

The AUTH fixtures apply ([AUTH test README](../../auth/test-cases/README.md#fixtures)): REST-A (OWN-A, WTR-A, CHF-A), REST-B, REST-P / OWN-P, NOREST. Additional (created through the app, the API or Import):

| Fixture | State |
|---|---|
| CAT-S | REST-A category `Starters` (active), description `Hot starters` |
| ITEM-1, ITEM-2 | In CAT-S: `Paneer Tikka` 250.00 Veg, `Chicken 65` 300.00 Non-Veg — active, created in that order |
| ITEM-3 | In CAT-S: `Old Dish` 100.00 — **inactive** |
| ITEM-DEL | In CAT-S: `Gone Dish` 90.00 — **deleted** (`isDelete = 1`) |
| CAT-D / ITEM-4 | REST-A category `Drinks` with `Lime Soda` 60.00 |
| CAT-X | REST-A category `Specials`, **deleted** |
| CAT-B / ITEM-B1 | REST-B category `B Mains` with `B Curry` |
| IMG-PNG | A small valid PNG as `data:image/png;base64,…` |
| IMG-BIG | A valid PNG of about 3 MB, same form |
| ADMIN-1 | Active super admin |

Reset REST-A's menu to these fixtures before each case; delete rows and image files a case creates.

## Panel requests (use case 03)

As in ONB ([ONB test README → Panel requests](../../onboarding/test-cases/README.md#panel-requests-use-case-03)): sign in as ADMIN-1, `GET` the page, take the CSRF token from the page script, and POST JSON with `X-CSRF-TOKEN` (and `Accept: application/json` for the target 422 / 404 bodies).

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1–AC2 list and expand | T01, T02 | D01 |
| 01 | AC3 add | T03 | D02 |
| 01 | AC4 empty name | T04 | D05 |
| 01 | AC5 rename (MENU-D2 fixed) | T05 | D03 |
| 01 | AC6 switch cascades (MENU-D2 fixed) | T06 | D04 |
| 01 | AC7 duplicate name; deleted names, IDs (target MENU-D4) | T08 | D15 |
| 01 | AC8 description kept (target MENU-D6) | T09 | D16 |
| 01 | Alt: empty, save failure, long name, filter, role, isolation | T07, T08 | D06–D14 |
| 02 | AC1 Item Management lists (MENU-D2 fixed) | T01, T02 | D03, D04, D22 |
| 02 | AC2 add (MENU-D1 fixed) | T03 | D05, D06 |
| 02 | AC3 field checks | T04 | D07 |
| 02 | AC4 image picking | T05, T06 | — |
| 02 | AC5 ordering menu | T07 | D01, D02 |
| 02 | AC6–AC7 edit and switch (MENU-D2 fixed) | T09, T10 | D08 |
| 02 | AC8 duplicate name | T11 | D12, D19 |
| 02 | AC9 price > 0 | T08 | D07 |
| 02 | Alt: category checks, limits and 404s (target MENU-D4), image (target MENU-D5), role, paging | — | D09–D11, D13–D18, D20, D21 |
| 03 | AC1–AC3 categories page (cascade target MENU-D3) | T01, T02, T03 | D01, D02, D03 |
| 03 | AC4–AC6 items page (target MENU-D3) | T04, T05, T06 | D04, D05, D06, D11 |
| 03 | Alt: empty name, unknown restaurant, session | T07 | D07–D10 |
| 04 | AC1–AC4 (target MENU-D5) | n/a — no screen | D01–D09 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `getItemsCategory` (01) | D10 | D11 | D09 | D08 | D13 | D12 | n/a¹ | n/a¹ | n/a¹ | D14 | D12 | D12 |
| `categorySave` (01) | D10 | D11 | D09 | D08 | D13 | D05, D06, D15 | D04, D15, D16 | D07 | n/a² | D14 | D07 | D06 |
| `getMenuItems` / `showCategoryMenuItems` (02) | D17 | D17 | D15 | D16 | D17 | D04, D14, D22 | n/a¹ | n/a¹ | n/a¹ | D17 | D04 | D04, D22 |
| `menuItemSave` (02) | D17 | D17 | D10, D21 | D16 | D17 | D07, D09, D11 | D10, D12 | D13 | D19 | D17 | D18 | D20, D21 |
| `image/upload` (04) | D07 | D07 | n/a³ | D07 | D07 | D02–D05 | D04, D09 | n/a⁴ | n/a⁴ | D08 | D04 | D03, D06 |
| panel category / item routes (03) | D09 | n/a⁵ | D08 | n/a⁵ | D09 | D07, D11 | D03, D11 | D10 | n/a⁶ | D10 | D10 | D07 |

1. Read only.
2. Plain last-write-wins updates.
3. Files aren't tied to a restaurant.
4. One field; every call writes a new file.
5. One super-admin role; its account state belongs to ADM.
6. Plain last-write-wins updates.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api/api/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin`. Check the database with a SQL client (`HEX(...)` for IDs) and the `storage/app/public/image/` folder for files.
