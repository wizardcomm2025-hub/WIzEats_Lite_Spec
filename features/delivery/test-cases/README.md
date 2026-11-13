# Delivery board — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api-dev_guid/specs/delivery/test-results.md`, never here.

**Expected results describe today's behaviour**, as fixed by DLV-D1 to DLV-D3 and the inherited decisions ([README](../README.md#decisions)). No case is blocked. Because no delivery card is ever shown (DLV-D1), the tester cases check the board as it is today; the status endpoint is covered by developer cases.

## Fixtures

The AUTH, ORD and KITCH fixtures apply ([AUTH](../../auth/test-cases/README.md#fixtures), [ORD](../../orders/test-cases/README.md#fixtures), [KITCH](../../kitchen/test-cases/README.md#fixtures)). Additional:

| Fixture | State |
|---|---|
| CH-DLV / CH-B-DLV | REST-A's / REST-B's Delivery channel ID (hex) |
| KOT-L | A REST-A KOT (status 0) whose `HEX(kotId)` starts with a letter `A`–`F` |
| KOT-D | A REST-A KOT (status 0) whose `HEX(kotId)` starts with a digit `1`–`9` |
| WA-CATCH | In the test environment, `WHATSAPP_API_URL` points at a request catcher (e.g. a local mock server) and `WHATSAPP_ACCESS_TOKEN` is a dummy, so any WhatsApp send is visible and nothing reaches a real phone |
| SNAP | Before each update case, save `SELECT HEX(kotId), status, updated_at FROM kots` (all restaurants) to compare after |

`APP_DEBUG=false` unless a case says otherwise.

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 first visit: toast + empty (DLV-D1) | T01, T02 | D01 |
| 01 | AC2 no toast on later visits | T03, T04 | — |
| 01 | AC3 real channel ID: wrapped failure (DLV-D1) | — | D02, D03 |
| 01 | Alt: bad path, role, auth, rate | T05 | D04–D08 |
| 02 | AC1 non-zero cast: KOT not found (DLV-D2) | T01 | D01 |
| 02 | AC2 zero cast: SQL error, nothing changed, no WhatsApp (DLV-D2/D3) | — | D02, D03, D10 |
| 02 | AC3 validation | — | D04 |
| 02 | Alt: role, over-posting, auth, rate, injection | — | D05–D09 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `orders/channel/{id}` (01) | D06 | D06 | D03 | D05 | D06 | D04 | n/a¹ | D07 | n/a¹ | D08 | D04 | D01, D04 |
| `orders/update-status/{kotId}` (02) | D07 | D07 | D02 | D05 | D07 | D04 | D03 | D06 | n/a² | D08 | D09 | D02, D10 |

1. Read only.
2. No update ever runs today (DLV-D2).

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api-dev_guid/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. Tokens via `POST /api/auth/loginPin`. Check the database with a SQL client, and check WA-CATCH for WhatsApp requests.
