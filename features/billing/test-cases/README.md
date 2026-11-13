# Bill & payment — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api-dev_guid/specs/billing/test-results.md`, never here.

**Expected results describe today's behaviour**, as fixed by BILL-D1 to BILL-D3 and the inherited decisions ([README](../README.md#decisions)). No case is blocked. Because the app can't open the bill page (BILL-D2), the tester cases check the entry points; billing and payment are covered by developer cases. Use case 03 has no UI path, so it has no tester file.

## Fixtures

The AUTH and ORD fixtures apply ([AUTH](../../auth/test-cases/README.md#fixtures), [ORD](../../orders/test-cases/README.md#fixtures)). Each case starts from fresh copies unless it says otherwise. Additional / refined:

| Fixture | State |
|---|---|
| ORDER-RUN | REST-A order created through the API with the ORD basic body: `Paneer Tikka` × 2 at 250.00; status 0; tax 5.00 % |
| ORDER-BILLED | ORDER-RUN after `generateBill` with TOT-525: status 1, bill `grandTotal` 525.00, `billStatus` 0 |
| ORDER-T1 | REST-A order created through the API with `orderChannelId` CH-DINE and `tableId` = Table 1 hex; Table 1 `status` 1 (occupied) |
| ORDER-NONAME | REST-A order created through the API whose line has `menuItemName: null` |
| ORDER-B / ORDER-B-BILLED | REST-B running order / REST-B billed order |
| TOT-525 | `{ "subTotal": 500, "taxPercent": 5, "taxAmount": 25, "isDiscountPercent": 0, "discountPercent": 0, "discountAmount": 0, "roundOff": 0, "grandTotal": 525 }` |
| N | `MAX(billNumber)` of REST-A's bills with `financialYear` `2026-27` before the case (0 if none) |
| PAY-525 | `{ "orderId": "<order hex>", "payments": [ { "paymentType": "CASH", "amount": 525, "receivedAmount": 600, "changeAmount": 75, "paymentNote": "" } ] }` |

Dates assume the current financial year is 2026-27 (1 Apr 2026 – 31 Mar 2027).

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | Entry points do nothing (BILL-D2) | T01, T02, T03 | — |
| 01 | AC1 bill generated | — | D01, D13 |
| 01 | AC2 already billed | — | D02, D09 |
| 01 | AC3 client totals (BILL-D1) | — | D04 |
| 01 | AC4 unnamed line (BILL-D1) | — | D06 |
| 01 | Alt: isolation, validation, bad ID, role, over-posting, auth, rate | — | D03, D05, D07, D08, D10–D12 |
| 02 | AC1 payment | — | D01, D02, D15 |
| 02 | AC2 any amount (BILL-D1) | — | D03 |
| 02 | AC3 already paid | — | D06, D12 |
| 02 | AC4 pay-and-bill | — | D04, D05 |
| 02 | AC5 table freed only when linked (BILL-D2) | T01 | D10 |
| 02 | Alt: isolation, validation, old payload, role, over-posting, auth, rate, injection | — | D07–D09, D11, D13, D14, D16, D17 |
| 03 | AC1–AC3 | — | D01–D06 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `orders/generateBill/{id}` (01) | D11 | D11 | D03 | D08 | D11 | D05, D07 | D04, D06 | D10 | D09 | D12 | D07 | D05, D07 |
| `payments/create2` (02) | D14 | D14 | D07 | D11 | D14 | D08, D09 | D03, D05 | D13 | D12 | D16 | D17 | D05, D09 |
| `payments/create` (02) | D14 | D14 | D07 | D11 | D14 | D15 | D15 | D13 | D12 | D16 | D17 | D15 |
| `payments/bill/{id}` (03) | D04 | D04 | D02 | D04 | D04 | D03 | n/a¹ | n/a¹ | n/a¹ | D06 | D03 | D03 |

1. Read only.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api-dev_guid/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin`. Check the database with a SQL client (`bills`, `bill_items`, `payments`, `orders`, `tables`).
