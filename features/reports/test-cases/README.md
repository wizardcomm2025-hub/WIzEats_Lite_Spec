# Reports — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api-dev_guid/specs/reports/test-results.md`, never here.

**Expected results describe today's behaviour**, as fixed by RPT-D1 to RPT-D3 and the inherited decisions ([README](../README.md#decisions)). No case is blocked. Paid orders can't be made in the app (BILL-D2), so a developer prepares them through the BILL API for the tester cases. Each case starts from a REST-A with only the orders it names.

## Fixtures

The AUTH, ORD and BILL fixtures apply ([AUTH](../../auth/test-cases/README.md#fixtures), [ORD](../../orders/test-cases/README.md#fixtures), [BILL](../../billing/test-cases/README.md#fixtures)). REST-A has 4 tables. Additional:

| Fixture | State |
|---|---|
| ORDER-LIVE | Running Takeaway order today: `Paneer Tikka` × 2 (500.00) |
| ORDER-PAID | Takeaway order today, billed with TOT-525, paid CASH (`receivedAmount` 600, `changeAmount` 75) |
| ORDER-UPI | Like ORDER-PAID, paid with one UPI entry, `receivedAmount` 525 |
| ORDER-PAID-T1 | Dine In order today with `tableId` = Table 1 (API), billed with TOT-525 and paid in CASH (`receivedAmount` 525) |
| PAID-AT(x) | A paid order (as ORDER-PAID) whose `created_at` is moved by SQL to time `x` |
| ORDER-B-PAID | A REST-B paid order today |
| ITEM-COMMA | A REST-A menu item named `Tikka, Paneer` (created through the API — MENU-D1 blocks the app) |

Dates below assume today is 12 Nov 2026 (a Thursday); adjust to the test day.

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 zeros with only running orders (RPT-D1) | T01 | D02 |
| 01 | AC2 a paid order counted | T03 | D01, D03, D04 |
| 01 | AC3 periods (RPT-D2) | T02 | D05, D06 |
| 01 | AC4 isolation | — | D07 |
| 01 | Alt: owner-only screen, validation, role, auth, rate, injection | T04 | D08–D12 |
| 02 | AC1 empty trends | T01 | D04 |
| 02 | AC2 daily order (RPT-D2) | T02 | D01 |
| 02 | AC3 week 5 (RPT-D2) | T02 | D02, D03 |
| 02 | Alt: validation, isolation, role, auth | — | D05–D07 |
| 03 | AC1 items CSV only (RPT-D3) | T01 | D01, D02 |
| 03 | AC2 nothing to export | T02 | D02 |
| 03 | AC3 orders endpoint 500 | — | D02 |
| 03 | Alt: unescaped CSV, validation, grouping, isolation, role, auth | T03 | D03–D07 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `dashboard-summary` (01) | D10 | D10 | D07 | D09 | D10 | D08 | D04 | D12 | n/a¹ | D11 | D12 | D08 |
| `top-selling-items` (01) | D10 | D10 | D07 | D09 | D10 | D08 | D02 | D12 | n/a¹ | D11 | D12 | D08 |
| `order-count-by-channel` (01) | D10 | D10 | D07 | D09 | D10 | D08 | n/a² | D12 | n/a¹ | D11 | D12 | D08 |
| `revenue-and-order-trend` (02) | D07 | D07 | D06 | D07 | D07 | D05 | D04 | D05 | n/a¹ | D07 | D05 | D05 |
| `get-all-items` (03) | D05 | D05 | D04 | D05 | D05 | D03 | D06 | D05 | n/a¹ | D05 | D07 | D03 |
| `get-all-orders` (03) | D05 | D05 | D04 | D05 | D05 | D03 | n/a² | D05 | n/a¹ | D05 | D07 | D02 |

1. Read only.
2. No rule to bypass beyond the period.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api-dev_guid/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin`. Prepare paid orders with `orders/generateBill` and `payments/create2` (BILL); move `created_at` by SQL where a fixture says so.
