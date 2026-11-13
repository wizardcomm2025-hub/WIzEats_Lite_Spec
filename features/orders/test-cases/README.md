# Take an order & send KOTs — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api-dev_guid/specs/orders/test-results.md`, never here.

**Expected results describe today's behaviour**, as fixed by ORD-D1 to ORD-D4 and the inherited decisions ([README](../README.md#decisions)). No case is blocked.

## Fixtures

The AUTH fixtures apply ([AUTH test README](../../auth/test-cases/README.md#fixtures)): REST-A (OWN-A, WTR-A, CHF-A), REST-B (OWN-B), REST-P / OWN-P, WTR-A-OFF, NOREST, DEV-1 / DEV-2. Additional:

| Fixture | State |
|---|---|
| REST-A setup | Tax 5.00 %, `restaurantCode` known (CODE-A); zone `Main Hall` with `Table 1`–`Table 4`, all status 0; no orders today |
| CH-DINE / CH-TAKE / CH-ROOM / CH-DLV | REST-A's channel IDs (hex) |
| ITEM-1 / ITEM-2 | `Paneer Tikka` 250.00, `Chicken 65` 300.00 (active) |
| ITEM-3 | `Old Dish` 100.00 (inactive) |
| ITEM-B1 / CH-B | REST-B's item and Dine In channel |
| ORDER-RUN | A REST-A order created through the API (status 0) |
| ORDER-BILLED | A REST-A order with its bill generated (status 1) |
| ORDER-B | A REST-B running order |

Body "**basic**" = `{ "orderChannelId": "<CH-TAKE>", "tableId": null, "roomId": null, "kotNote": "", "items": [ { "menuItemId": "<ITEM-1>", "menuItemName": "Paneer Tikka", "quantity": 2, "price": 250 } ] }`.

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 channels | T01 | D01 |
| 01 | AC2–AC4 grid, occupy, leave | T02, T03, T04 | D02, D03 |
| 01 | AC5 takeaway | T05 | — |
| 01 | AC6 filter ignored | — | D04 |
| 01 | Alt: re-open occupied table, channel switch | T06, T07 | D05–D08 |
| 02 | AC1 first KOT | T01 | D01 |
| 02 | AC2 not linked to table (ORD-D2) | T02 | D02, D03 |
| 02 | AC3 held items | T03 | — |
| 02 | AC4 nothing ticked | T04 | — |
| 02 | AC5 customer dropped (ORD-D3) | T05 | D04 |
| 02 | AC6 client price (ORD-D1) | — | D05, D06, D07 |
| 02 | Alt: numbering, duplicates, timeout, errors, role, isolation | T06–T10 | D08–D20 |
| 03 | AC1 add KOT | T01, T02 | D01 |
| 03 | AC2 billed | T03 | D02 |
| 03 | AC3 isolation | — | D03 |
| 03 | Alt: bad IDs, note missing, tax snapshot, lock | T04 | D04–D10 |
| 04 | AC1–AC3 server | — | D01–D04 |
| 04 | AC4 app can't load (ORD-D2) | T01, T02, T03 | D05, D06 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `getOrderChannels` (01) | D05 | D06 | D07 | D08 | D05 | D04 | n/a¹ | n/a¹ | n/a¹ | D08 | D04 | D07 |
| `createOrder` (02) | D16 | D17 | D10, D11 | D12 | D16 | D08, D09 | D05–D07, D13 | D14 | D15, D18 | D19 | D20 | D09, D10 |
| `kots/{orderId}` (03) | D08 | D08 | D03 | D08 | D08 | D04, D05 | D02, D06 | D07 | D09 | D10 | D04 | D04, D05 |
| `orders/{orderId}` (04) | D04 | D04 | D03 | D04 | D04 | D05 | n/a¹ | n/a¹ | n/a¹ | D06 | D05 | D05 |

1. Read only, or no input.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api-dev_guid/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin`. Check the database with a SQL client (`HEX(orderId)` etc.). Check the table grid data with `POST /api/restaurants/showDetails`.
