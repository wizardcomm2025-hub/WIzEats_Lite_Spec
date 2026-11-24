# Delivery board — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api/api/specs/delivery/test-results.md`, never here.

**Expected results follow the decision statuses** in the [README](../README.md#decisions) (re-synced 2026-11-24, rules in [conventions.md → Decision status](../../../conventions.md#decision-status)). DLV-D1 to DLV-D3 are *To fix*: most cases assert the **target** and are tagged `Fails until <ID> is fixed`, with today's result in brackets. DLV-D4 is accepted. App cases that need a card fail at their first step until DLV-D1 is fixed. No case is blocked.

## Fixtures

The AUTH and ORD fixtures apply ([AUTH](../../auth/test-cases/README.md#fixtures), [ORD](../../orders/test-cases/README.md#fixtures)): REST-A (OWN-A, WTR-A, CHF-A, Delivery switched on), REST-B, WTR-A-OFF, NOREST, DEV-1. Additional:

| Fixture | State |
|---|---|
| CH-DLV / CH-B-DLV | REST-A's / REST-B's Delivery channel ID (hex) |
| DLV-ORD | A REST-A order on CH-DLV created through `POST /api/orders/createOrder` (or PUB), Pending KOT DLV-KOT; `customerName` `Ravi`, `customerPhone` `9876543210`, `customerAddress` `12 Park St` (set by SQL until ORD-D3 / PUB save them) |
| DLV-ORD-B | The same for REST-B (KOT DLV-KOT-B) |
| DINE-KOT | A REST-A Dine In KOT (status 0) |
| KOT-L / KOT-D | REST-A delivery KOTs (status 0) whose `HEX(kotId)` starts with a letter `A`–`F` / a digit `1`–`9` — for the "today" results only |
| WA-CATCH | In the test environment, `WHATSAPP_API_URL` points at a request catcher (e.g. a local mock server) and `WHATSAPP_ACCESS_TOKEN` is a dummy, so any WhatsApp send is visible and nothing reaches a real phone |
| SNAP | Before each update case, save `SELECT HEX(kotId), status, updated_at FROM kots` (all restaurants) to compare after |

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 empty board | T01, T02 | D01 |
| 01 | AC2 orders listed (target DLV-D1) | T03, T04 | D02, D09, D10 |
| 01 | AC3 other restaurant's channel (target DLV-D1) | — | D03 |
| 01 | Alt: bad path, role, auth, over-posting, rate, no New Order | T05, T06 | D04–D08 |
| 02 | AC1 accept (target DLV-D2) | T01 | D01, D02 |
| 02 | AC2 stage rules (target DLV-D2) | — | D03, D04 |
| 02 | AC3 WhatsApp to the customer (target DLV-D3) | T02 | D13 |
| 02 | AC4 Complete Payment through Bill Payment (target DLV-D3) | T03 | D14 |
| 02 | AC5 other restaurant / non-delivery KOT (target DLV-D2) | — | D05, D06 |
| 02 | Alt: role, over-posting, auth, rate, injection, errors | — | D07–D12 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `orders/channel/{id}` (01) | D06 | D06 | D03 | D05 | D06 | D04 | n/a¹ | D07 | n/a¹ | D08 | D04 | D04, D09 |
| `orders/update-status/{kotId}` (02) | D07 | D07 | D05, D06 | D08 | D07 | D04, D10 | D03 | D09 | D12 | D11 | D10 | D10, D12 |

1. Read only.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api/api/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin`. Check the database with a SQL client, and check WA-CATCH for WhatsApp requests.
