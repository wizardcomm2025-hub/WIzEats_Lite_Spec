# Public QR ordering — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api-dev_guid/specs/public-order/test-results.md`, never here.

**Expected results describe today's behaviour**, as fixed by PUB-D1 to PUB-D3 and the inherited decisions ([README](../README.md#decisions)). No case is blocked. **Developer cases only:** there is no customer screen ([ui-ux.md](../ui-ux.md)), so there are no tester files.

## Fixtures

The AUTH, MENU and ORD fixtures apply ([AUTH](../../auth/test-cases/README.md#fixtures), [MENU](../../menu/test-cases/README.md#fixtures), [ORD](../../orders/test-cases/README.md#fixtures)). Additional:

| Fixture | State |
|---|---|
| PUB-BODY | JSON `{ "restaurantId": "<REST-A hex>", "tableId": null, "roomId": null, "orderChannelId": 3, "kotNote": "less spicy", "items": [ { "menuItemId": "<ITEM-1 hex>", "menuItemName": "Paneer Tikka", "quantity": 2, "price": 250 } ], "customerName": "Cust One", "customerPhone": "9990001111", "customerAddress": "5 B Road" }` |
| RAWB(r) | A form-encoded body (`Content-Type: application/x-www-form-urlencoded`) whose `restaurantId` is restaurant `r`'s 16 raw bytes, percent-encoded: take `HEX(restaurantId)` and put `%` before every pair. Other fields as PUB-BODY in form syntax (`items[0][menuItemId]=…&items[0][quantity]=2…`, `tableId=&roomId=`). Laravel trims whitespace and NUL bytes from the ends of request strings, so if `r`'s ID starts or ends with byte `00`, `09`, `0A`, `0B`, `0D` or `20`, re-create the restaurant |
| ITEM-DEL | REST-A item `Gone Dish` 90.00 in CAT-S, created through the API, then `isDelete = 1` set by SQL |
| ORDER-L | A REST-A in-progress order (status 0) created through the ORD API whose `HEX(orderId)` starts with a letter `A`–`F`. **REST-A has no other order** in the cases that use it |
| ORDER-L-BILLED | ORDER-L after BILL `generateBill` (status 1) |
| ORDER-D | Like ORDER-L, but `HEX(orderId)` starts with a digit `1`–`9` |
| ORDER-B-RUN | A REST-B in-progress order; REST-B's only order, ID starting with a letter |
| ROW-SNAP | Before each order case, save: the row counts of `orders`, `kots`, `order_items` (all restaurants); every `tables.status` and `rooms.status`; and, where an order fixture is used, its `subTotal`, `grandTotal`, `updated_at`. Compare after |
| SHEET-CATCH | `GOOGLE_SHEET_LOG_URL` points at a local request catcher, `GOOGLE_SHEET_LOG_SECRET` is a dummy, so error rows are visible and nothing reaches Google |

`APP_DEBUG=false` unless a case says otherwise. No token is sent unless a case says so. Run against `php artisan serve`.

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 hex ID → empty data (PUB-D1) | — | D02 |
| 01 | AC2 raw ID → full menu incl. inactive / deleted (PUB-D2) | — | D01, D03 |
| 01 | AC3 unapproved / switched-off restaurant served | — | D04 |
| 01 | AC4 validation | — | D06 |
| 01 | Alt: isolation, over-posting, auth, rate, injection, DB failure | — | D05, D07–D11 |
| 02 | AC1 create → 500 missing column, nothing written (PUB-D1) | — | D01 |
| 02 | AC2 `tableId` / `roomId` key missing | — | D02 |
| 02 | Alt: raw ID → `Server Error` | — | D03 |
| 02 | AC3 validation | — | D04 |
| 02 | AC4 customer details in the error (PUB-D3) | — | D10 |
| 02 | Alt: client prices, over-posting, auth, rate, injection, isolation | — | D05–D09, D11 |
| 03 | AC1 hex `orderId` → TypeError (PUB-D1) | — | D01, D02 |
| 03 | AC2 numeric `orderId`, hex restaurant → Order not found | — | D03 |
| 03 | AC3 `orderId` 0 + raw restaurant → 1364, nothing written | — | D04, D05, D06 |
| 03 | Alt: cross-restaurant, `orderId` variants, validation, auth / rate / injection, error text | — | D07–D11 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `public/menu` (01) | D08 | D04 | D05 | n/a¹ | D08 | D06 | D03 | D07 | n/a² | D09 | D10 | D11 |
| `public/order` create (02) | D07 | D07 | D11 | n/a¹ | D07 | D04 | D05 | D06 | n/a³ | D08 | D09 | D10 |
| `public/order` add (03) | D10 | D06 | D07 | n/a¹ | D10 | D09 | D04, D08 | D08 | n/a³ | D10 | D10 | D11 |

1. Public endpoint: no user, so no role.
2. Read only.
3. Nothing is ever written today (PUB-D1). Each failure rolls back the whole transaction.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api-dev_guid/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. No token is needed. Check the database with a SQL client (`HEX()` for IDs), and check SHEET-CATCH for error rows.
