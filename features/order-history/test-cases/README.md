# Order history — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api/api/specs/order-history/test-results.md`, never here.

**Expected results follow the decision statuses** in the [README](../README.md#decisions) (re-synced 2026-11-24, rules in [conventions.md → Decision status](../../../conventions.md#decision-status)):

- HIST-D1 (fixed in code 2026-11-18) and BILL-D2 (fixed in the app): the channel filter, the `Bill Payment` button and a filling History tab are asserted as working and pass today.
- *To fix* decisions (HIST-D2, HIST-D3, AUTH-D2): the case asserts the **target** and is tagged `Fails until <ID> is fixed`; today's result is in brackets.

No case is blocked.

## Fixtures

The AUTH, ORD and BILL fixtures apply ([AUTH](../../auth/test-cases/README.md#fixtures), [ORD](../../orders/test-cases/README.md#fixtures), [BILL](../../billing/test-cases/README.md#fixtures)). Times are IST; "today" is the test day. Additional:

| Fixture | State |
|---|---|
| ORDER-LIVE | REST-A Takeaway order sent by WTR-A today at about 14:00 (`Paneer Tikka` × 2, subtotal 500.00), status 0 |
| ORDER-LIVE-BILLED | Like ORDER-LIVE, then billed with TOT-525 (status 1) |
| ORDER-PAID | REST-A order, today, billed with TOT-525 and paid with PAY-525 (CASH, received 600, change 75); its number noted (e.g. `CODE-A/7`) |
| ORDER-UPI | Like ORDER-PAID, paid with one `UPI` entry |
| ORDER-OLD | REST-A running order with `created_at` moved to yesterday 14:00 by SQL |
| ORDER-EARLY | REST-A running order with `created_at` = today 00:30 IST (19:00 UTC yesterday) by SQL |
| ORDER-B-RUN / ORDER-B-PAID | REST-B running / paid orders today |
| PAID-25 | 25 REST-A paid orders today (API) |
| BODY-TODAY | `{ "page": 1, "perPage": 10, "search": null, "paymentType": null, "startDate": "<today>", "endDate": "<today>" }` |
| ORDER-DINE | REST-A Dine In running order today (Table 1) |

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 running order listed | T01, T02 | D01 |
| 01 | AC2 billed still `Sub Total` (accepted) | T03 | D01 |
| 01 | AC3 paid leaves Live | T06 | D01 |
| 01 | AC4 isolation | — | D04 |
| 01 | AC5 every running order, any date (target HIST-D2) | T05 | D02 |
| 01 | AC6 `Bill Payment` opens the bill (BILL-D2 fixed) | T04 | — |
| 01 | Alt: search and ignored filters (target HIST-D2), channel, role, auth | — | D03, D05–D10 |
| 02 | AC1 paid order details | T02, T03 | D01 |
| 02 | AC2 paging | T04 | D02 |
| 02 | AC3 only paid / empty | T01 | D03, D04 |
| 02 | AC4 app-paid order listed (BILL-D2 fixed) | T05 | — |
| 02 | Alt: IST day, payments as stored (target HIST-D3), Partial, role, auth, validation, rate | — | D05–D12 |
| 03 | AC1 History search | T01 | D01, D02 |
| 03 | AC2 search on Live (target HIST-D2) | T02 | 01-D03, 01-D10 |
| 03 | AC3 channel filter (HIST-D1 fixed) | T03, T04 | D04, 01-D05 |
| 03 | AC4 payment mode | T05 | D03 |
| 03 | Alt: dates, injection, over-posting, length | T06, T07 | D05–D08 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `orders/orderHistoryLive` (01) | D08 | D08 | D04 | D07 | D08 | D05, D06 | D02, D03, D10 | D03 | n/a¹ | D09 | D09 | D05 |
| `orders/orderHistory` (02, 03) | 02-D08 | 02-D08 | 02-D04, 03-D07 | 02-D07 | 02-D08 | 02-D09, 03-D04, 03-D08 | 02-D06, 02-D11, 02-D12, 03-D02 | 03-D07 | n/a¹ | 02-D10 | 03-D06 | 03-D04 |

1. Read only.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api/api/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin`. Prepare paid orders with `orders/generateBill` and `payments/create2` (BILL). Move `created_at` by SQL where a fixture says so.
