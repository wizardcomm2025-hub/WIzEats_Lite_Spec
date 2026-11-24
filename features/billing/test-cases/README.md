# Bill & payment — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api/api/specs/billing/test-results.md`, never here.

**Expected results follow the decision statuses** in the [README](../README.md#decisions) (re-synced 2026-11-24, rules in [conventions.md → Decision status](../../../conventions.md#decision-status)):

- BILL-D2 is *fixed in the app*: the app paths into the bill pages and the freeing of the table are asserted as working and pass today.
- *To fix* decisions (BILL-D1, BILL-D3, AUTH-D2): the case asserts the **target** and is tagged `Fails until <ID> is fixed`; today's result is in brackets.

No case is blocked.

## Fixtures

The AUTH and ORD fixtures apply ([AUTH](../../auth/test-cases/README.md#fixtures), [ORD](../../orders/test-cases/README.md#fixtures)): REST-A (OWN-A, WTR-A, CHF-A; tax 5 %; discount switch **on**), REST-B, WTR-A-OFF, NOREST. Additional:

| Fixture | State |
|---|---|
| ORDER-RUN | REST-A order created through the API with the ORD basic body: `Paneer Tikka` × 2 at 250.00; status 0; tax 5.00 % |
| ORDER-BILLED | ORDER-RUN after `generateBill` with TOT-525: status 1, bill `grandTotal` 525.00, `billStatus` 0 |
| ORDER-T1 | REST-A order on Dine In, Table 1 (created in the app or through the API with Table 1's hex); Table 1 `status` 2 |
| ORDER-NONAME | REST-A order created through the API whose line has `menuItemName: null` (possible only until ORD-D1 is fixed) |
| ORDER-B / ORDER-B-BILLED | REST-B running order / REST-B billed order |
| TOT-525 | `{ "subTotal": 500, "taxPercent": 5, "taxAmount": 25, "isDiscountPercent": 0, "discountPercent": 0, "discountAmount": 0, "roundOff": 0, "grandTotal": 525 }` (what the app sends today; the target uses only the discount fields) |
| DISC-10 | `{ "isDiscountPercent": true, "discountPercent": 10 }` (plus, for today's server, the matching totals: subTotal 500, discountAmount 50, taxAmount 22.5, roundOff 0.5, grandTotal 473) |
| PAY-525 | `{ "orderId": "<order hex>", "payments": [ { "paymentType": "CASH", "amount": 525, "receivedAmount": 600, "changeAmount": 75, "paymentNote": "" } ] }` |
| N | The highest REST-A `billNumber` of the current financial year before the case |

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 bill generated | T02 | D01, D13 |
| 01 | AC2 billed twice (target 409) | — | D02 |
| 01 | AC3–AC4 server totals and discount (target BILL-D1) | T04 | D04, D14, D15 |
| 01 | AC5 app opens the bill (BILL-D2 fixed) | T01–T03 | — |
| 01 | Alt: isolation, validation, bad ID, role, concurrency, over-posting, auth, idempotency | — | D03, D05–D12 |
| 02 | AC1 cash payment | T01 | D01, D02 |
| 02 | AC2 amounts must add up (target BILL-D1) | T04 | D03, D18, D19 |
| 02 | AC3 already paid (target 409) | — | D06, D12 |
| 02 | AC4 pay-and-bill | — | D04, D05 |
| 02 | AC5 table freed (BILL-D2 fixed) | T02 | D10 |
| 02 | Alt: isolation, validation, role, over-posting, auth, legacy, rate, injection, customer display | T03 | D07–D09, D11, D13–D17 |
| 03 | AC1–AC3 | — | D01–D06 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `generateBill` (01) | D11 | D11 | D03 | D08 | D11 | D05, D07, D15 | D04, D06, D14 | D10 | D09, D12 | D12 | D07 | D05, D07 |
| `payments/create2` (02) | D14 | D14 | D07 | D11 | D14 | D08, D09 | D03, D18, D19 | D13 | D06, D12 | D16 | D17 | D05, D09 |
| `payments/create` (02) | D15 | via D14 | via D07 | via D11 | via D14 | D15 | D15 | via D13 | via D12 | via D16 | via D17 | via D09 |
| `payments/bill/{id}` (03) | D04 | D04 | D02 | D04 | D04 | D03 | n/a¹ | n/a¹ | n/a¹ | D06 | D03 | D03 |

1. Read only.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api/api/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin`. Check the database with a SQL client (`HEX(...)` for IDs).
