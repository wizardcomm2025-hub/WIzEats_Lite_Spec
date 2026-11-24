# Take an order & send KOTs — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api/api/specs/orders/test-results.md`, never here.

**Expected results follow the decision statuses** in the [README](../README.md#decisions) (re-synced 2026-11-24, rules in [conventions.md → Decision status](../../../conventions.md#decision-status)):

- ORD-D2 is *fixed in code and the app*: table / room linking, the `Ordered` status and re-opening an order are asserted as working and pass today.
- *To fix* decisions (ORD-D1, ORD-D3, ORD-D4, AUTH-D2): the case asserts the **target** and is tagged `Fails until <ID> is fixed`; today's result is in brackets.

No case is blocked.

## Fixtures

The AUTH fixtures apply ([AUTH test README](../../auth/test-cases/README.md#fixtures)): REST-A (OWN-A, WTR-A, CHF-A), REST-B (OWN-B), REST-P / OWN-P, WTR-A-OFF, NOREST, DEV-1 / DEV-2. Additional:

| Fixture | State |
|---|---|
| REST-A setup | Tax 5.00 %, `restaurantCode` known (CODE-A); all four channels switched **on** (Takeaway, Delivery, Room Service on in the settings — RST); zone `Main Hall` with `Table 1`–`Table 4`, all status 0; floor `Ground Floor` with rooms `101`, `102`, status 0; no orders today |
| CH-DINE / CH-TAKE / CH-ROOM / CH-DLV | REST-A's channel IDs (hex) |
| ITEM-1 / ITEM-2 | `Paneer Tikka` 250.00, `Chicken 65` 300.00 (active) |
| ITEM-3 | `Old Dish` 100.00 (inactive) |
| ITEM-B1 / CH-B / TABLE-B1 | REST-B's item, Dine In channel and table |
| ORDER-RUN | A REST-A order created through the API (status 0) |
| ORDER-BILLED | A REST-A order with its bill generated (status 1) |
| ORDER-B | A REST-B running order |

Body "**basic**" = `{ "orderChannelId": "<CH-TAKE>", "tableId": null, "roomId": null, "kotNote": "", "items": [ { "menuItemId": "<ITEM-1>", "menuItemName": "Paneer Tikka", "quantity": 2, "price": 250 } ] }`.

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 active channels | T01 | D01, D09 |
| 01 | AC2–AC4 grid, occupy, leave | T02, T03, T04 | D02, D03 |
| 01 | AC5 takeaway | T05 | — |
| 01 | AC6 filter ignored | — | D04 |
| 01 | AC7 ordered table re-opens its order (ORD-D2 fixed) | T06, T08 | — |
| 01 | Alt: channel switch, roles, account state | T07 | D05–D08 |
| 02 | AC1 first KOT | T01 | D01, D22 |
| 02 | AC2 linked to table / room (ORD-D2 fixed) | T02, T10 | D02, D03 |
| 02 | AC3 held items | T03 | — |
| 02 | AC4 nothing ticked | T04 | — |
| 02 | AC5 customer captured (target ORD-D3) | T05 | D04 |
| 02 | AC6 menu price and items (target ORD-D1) | — | D05, D06, D10, D11, D21 |
| 02 | AC7 idempotency (target ORD-D4) | T08 | D18 |
| 02 | Alt: numbering, one order per table, errors, role, isolation | T06, T07, T09 | D07–D09, D12–D17, D19, D20 |
| 03 | AC1 add KOT | T01, T02 | D01 |
| 03 | AC2 billed | T03 | D02 |
| 03 | AC3 isolation (target 404) | — | D03 |
| 03 | AC4 re-open adds to the same order (ORD-D2 fixed) | T04 | — |
| 03 | Alt: bad IDs, note missing, tax snapshot, lock, idempotency | — | D04–D11 |
| 04 | AC1–AC3 server | — | D01–D04 |
| 04 | AC4 app shows the order's KOTs (ORD-D2 fixed) | T01, T02, T03 | D05, D06 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `getOrderChannels` (01) | D05 | D06 | D07 | D08 | D05 | D04 | n/a¹ | n/a¹ | n/a¹ | D08 | D04 | D07 |
| `createOrder` (02) | D16 | D17 | D10, D11 | D12 | D16 | D08, D09, D21 | D05, D06, D13 | D14 | D15, D18 | D19 | D20 | D07, D09 |
| `kots/{orderId}` (03) | D08 | D08 | D03 | D08 | D08 | D04, D05 | D02, D06 | D07 | D09, D11 | D10 | D04 | D04, D05 |
| `orders/{orderId}` (04) | D04 | D04 | D03 | D04 | D04 | D05 | n/a¹ | n/a¹ | n/a¹ | D06 | D05 | D05 |

1. Read only, or no input.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api/api/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin`. Check the database with a SQL client (`HEX(orderId)` etc.). Check the table grid data with `POST /api/restaurants/showDetails`, the room grid with `POST /api/rooms/getRoomsDetails`.
