# Order history — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api-dev_guid/specs/order-history/test-results.md`, never here.

**Expected results describe today's behaviour**, as fixed by HIST-D1 to HIST-D3 and the inherited decisions ([README](../README.md#decisions)). No case is blocked. Paid orders can't be made in the app (BILL-D2), so a developer prepares them through the BILL API for the tester cases.

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

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 running order listed | T01, T02 | D01 |
| 01 | AC2 billed still `Sub Total` (HIST-D2) | T03 | D01 |
| 01 | AC3 paid leaves Live | T06 | D01 |
| 01 | AC4 isolation | — | D04 |
| 01 | Alt: UTC day, old orders, ignored filters, bill button, role, auth | T04, T05 | D02, D03, D05–D09 |
| 02 | AC1 paid order details | T02, T03 | D01 |
| 02 | AC2 paging | T04 | D02 |
| 02 | AC3 only paid / empty | T01 | D03, D04 |
| 02 | Alt: IST day, garbled payment, role, auth, validation, rate | — | D05–D10 |
| 03 | AC1 History search | T01 | D01, D02 |
| 03 | AC2 Live ignores search (HIST-D2) | T02 | 01-D03 |
| 03 | AC3 channel filter errors (HIST-D1) | T03, T04 | D04, 01-D05 |
| 03 | AC4 payment mode | T05 | D03 |
| 03 | Alt: dates, injection, over-posting, length | T06, T07 | D05–D08 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `orders/orderHistoryLive` (01) | D08 | D08 | D04 | D07 | D08 | D05, D06 | D03 | D03 | n/a¹ | D09 | D09 | D05 |
| `orders/orderHistory` (02, 03) | 02-D08 | 02-D08 | 02-D04, 03-D07 | 02-D07 | 02-D08 | 02-D09, 03-D04, 03-D08 | 02-D06, 03-D02 | 03-D07 | n/a¹ | 02-D10 | 03-D06 | 03-D04 |

1. Read only.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api-dev_guid/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin`. Prepare paid orders with `orders/generateBill` and `payments/create2` (BILL). Move `created_at` by SQL where a fixture says so.
