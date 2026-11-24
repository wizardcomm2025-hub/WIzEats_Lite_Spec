# Public QR ordering — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api/api/specs/public-order/test-results.md`, never here.

**Expected results describe the target** (PUB-D1 to PUB-D3 are *To fix* — [README](../README.md#decisions)); each case tagged *Fails until PUB-Dn is fixed* says what happens today in "(today: …)". Re-synced 2026-11-24. **Developer cases only:** there is no customer screen ([ui-ux.md](../ui-ux.md)), so there are no tester files. Staff screens that show public orders are tested in ORD, KITCH, HIST and DLV.

## Fixtures

The AUTH, MENU and ORD fixtures apply ([AUTH](../../auth/test-cases/README.md#fixtures), [MENU](../../menu/test-cases/README.md#fixtures), [ORD](../../orders/test-cases/README.md#fixtures)): REST-A (tax 5 %, CODE-A, all four channels on, `Table 1`–`Table 4`, rooms `101`, `102`), REST-B, REST-P, REST-R, REST-OFF; ITEM-1 `Paneer Tikka` 250.00, ITEM-2 `Chicken 65` 300.00, ITEM-3 inactive, ITEM-DEL deleted, CAT-D / ITEM-4 `Lime Soda` 60.00, ITEM-B1. Since 2026-11-24 the items can be created in the app (MENU-D1 / D2 fixed). Additional:

| Fixture | State |
|---|---|
| PUB-BODY | JSON `{ "restaurantId": "<REST-A hex>", "channelCode": "TAKEAWAY", "orderChannelId": 3, "tableId": null, "roomId": null, "kotNote": "less spicy", "items": [ { "menuItemId": "<ITEM-1 hex>", "menuItemName": "Paneer Tikka", "quantity": 2, "price": 250 } ], "customerName": "Cust One", "customerPhone": "9990001111", "customerAddress": "5 B Road" }`. `orderChannelId`, `menuItemName` and `price` are required today and ignored in the target |
| PUB-ORD | A REST-A Takeaway order created by PUB-02-D01 (target): `createdBy` `customer`, status 0 |
| PUB-ORD-BILLED | PUB-ORD after OWN-A runs BILL `generateBill` (status 1) |
| RAWB(r) | *Today only.* A form-encoded body whose `restaurantId` is restaurant `r`'s 16 raw bytes, percent-encoded (`%` before every pair of `HEX(restaurantId)`); other fields as PUB-BODY in form syntax. If `r`'s ID starts or ends with byte `00`, `09`, `0A`, `0B`, `0D` or `20`, re-create the restaurant (Laravel trims them). Used only to show today's results |
| ORDER-L / ORDER-L-BILLED / ORDER-B-RUN | *Today's results only.* An ORD-created REST-A in-progress order (REST-A's only order; `HEX(orderId)` starts with `A`–`F`), the same after `generateBill`, and REST-B's only running order |
| ROW-SNAP | Before each order case, save the row counts of `orders`, `kots`, `order_items` (all restaurants), every `tables.status` / `rooms.status`, and any fixture order's `subTotal`, `grandTotal`, `updated_at`. "Unchanged" = all equal after |
| SHEET-CATCH | `GOOGLE_SHEET_LOG_URL` points at a local request catcher, `GOOGLE_SHEET_LOG_SECRET` is a dummy |

`APP_DEBUG=false` unless a case says otherwise. No token is sent unless a case says so. Run against `php artisan serve`. Clear the rate limiter (`php artisan cache:clear`) before each case except the rate-limit ones.

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 hex ID → menu (target PUB-D1) | — | D01, D02 |
| 01 | AC2 only active items in active categories (target PUB-D2) | — | D03 |
| 01 | AC3 pending / rejected / off → 404 (target PUB-D2) | — | D04 |
| 01 | AC4 validation | — | D06 |
| 01 | Alt: isolation, over-posting, auth, rate (target PUB-D3), injection, DB failure | — | D05, D07–D11 |
| 02 | AC1 created with menu prices, `CODE-A/<n>` (target PUB-D1, D2) | — | D01, D05 |
| 02 | AC2 Delivery reaches the board and kitchen (target PUB-D1) | — | D13 |
| 02 | AC3 Dine In table `Ordered`; one order per table (target PUB-D1) | — | D12, D14 |
| 02 | AC4 no SQL / path / customer data in replies (target PUB-D3) | — | D03, D10 |
| 02 | AC5 validation | — | D02, D04 |
| 02 | Alt: channel / table / room checks, item checks, over-posting, auth, rate, idempotency, injection, isolation | — | D06–D09, D11, D12, D15 |
| 03 | AC1 KOT added to a public order (target PUB-D1) | — | D01 |
| 03 | AC2 staff-created / other restaurant's order → 404 (target PUB-D1) | — | D02, D03, D07 |
| 03 | AC3 billed → 409 | — | D06 |
| 03 | Alt: `kotNote` optional, `orderId` variants, validation, auth / rate / injection, error text | — | D04, D05, D08–D11 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `public/menu` (01) | D08 | D04 | D05 | n/a¹ | D08 | D06 | D03 | D07 | n/a² | D09 | D10 | D11 |
| `public/order` create (02) | D07 | D07 | D11 | n/a¹ | D07 | D02, D04 | D05, D12, D15 | D06 | D08, D14 | D08 | D09 | D03, D10 |
| `public/order` add (03) | D10 | D06 | D07 | n/a¹ | D10 | D05, D08, D09 | D02, D04 | D08 | D10 | D10 | D10 | D11 |

1. Public endpoint: no user, so no role.
2. Read only.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api/api/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. No token is needed. Check the database with a SQL client (`HEX()` for IDs), and check SHEET-CATCH for error rows. Cases tagged *Fails until …* are expected to fail until the decision is fixed in code.
