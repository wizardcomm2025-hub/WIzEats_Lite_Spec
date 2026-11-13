# Onboarding & approval — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api-dev_guid/specs/onboarding/test-results.md`, never here.

**Expected results describe today's behaviour**, as fixed by ONB-D1 to ONB-D4 and AUTH-D2 ([README](../README.md#decisions)). Cases that check an accepted risk say so. No case is blocked.

## Fixtures

The AUTH fixtures apply ([AUTH test README](../../auth/test-cases/README.md#fixtures)): REST-A (Accepted, has e-mail), REST-B, REST-R (Rejected), OWN-A, WTR-A, CHF-A, OWN-B, OWN-R, NOREST, PH-NEW, MAIL-OK, MAIL-DOWN, DEV-1 / DEV-2. Additional:

| Fixture | State |
|---|---|
| REST-P, REST-P2 | Pending restaurants, onboarding done, each with an e-mail the tester can read; owners OWN-P, OWN-P2 |
| REST-P-NOMAIL | Pending, onboarding done, `restaurantEmail` empty |
| REST-LEG / OWN-LEG | Restaurant created by legacy register (onboarding not done) and its owner, PIN known |
| PH-NEW2 … | More unused valid mobile numbers (one per registration case) |
| ADMIN-1 | Active super admin with known username and password |
| ADMIN-INBOX | The mailbox behind `ADMIN_EMAIL` |
| PROFILE-OK | A complete valid profile: name `Test Cafe`, owner `Test Owner`, e-mail on the `.com` domain the tester can read, address `1 Test Road` |

Body "**full**" below = PROFILE-OK as API fields with `enableRestaurant: true, enableDelivery: false, enableKot: true, enableRoomService: false, enableTakeAway: false` — what the app sends.

## Panel requests (use case 03)

The approve / reject endpoint is a web form. To call it directly: keep a cookie jar; `GET /` to get the session cookie and the `_token` from the login form; `POST /admin/login` with `username`, `password`, `_token` as ADMIN-1; `GET /admin` and take a fresh `_token` from any Accept form. Send `POST /admin/update-status/<restaurant id>` as form data `isApproved=<n>&_token=<token>`. Probe = restaurant row in the database, the panel list, and the restaurant inbox.

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 success screens | T01 | D01 |
| 01 | AC2 PIN and admin e-mails | T01 | D01, D14 |
| 01 | AC3 Pending + starter data | — | D01, D15 |
| 01 | AC4 sign-in before approval | T13 | 02-D01 |
| 01 | AC5 empty field / spaces | T02, T03 | D02, D09 |
| 01 | AC6 e-mail ending | T04 | D04 |
| 01 | AC7 phone taken meanwhile | T06 | D05, D11 |
| 01 | AC8 PIN mail fails | — | D13 |
| 01 | AC9 double tap | T09 | D11 |
| 01 | AC10 Takeaway switch | T01, T14 | D07 |
| 01 | Alt: long name, no network, back, rate limit | T07, T08, T10 | D08, D12 |
| 02 | AC1 Pending screen | T01 | D01 |
| 02 | AC2 check result | T02, T03 | — |
| 02 | AC3 accepted while waiting | T04, T05 | — |
| 02 | AC4 Rejected screen | T06, T07 | D03 |
| 02 | AC5 no network | T08 | — |
| 02 | AC6 logout | T09 | — |
| 02 | AC7 staff see the same | T10 | — |
| 02 | AC8 no Contact Support | T11 | — |
| 02 | Alt: legacy owner stuck; API not blocked; odd value | T12 | D02, D03, D04, D05 |
| 03 | AC1–AC2 accept / reject + mails | T01, T02 | D01, D02 |
| 03 | AC3 no buttons once decided | T03 | D04, D05, D06 |
| 03 | AC4–AC5 app reacts | 02-T04, 02-T07 | — |
| 03 | AC6 only onboarded listed | T04 | 02-D04 |
| 03 | AC7 mail down | T06 | D13 |
| 03 | AC8 no session | T07, T08 | D10, D11, D12 |
| 03 | Alt: unknown id, bad value, repeat, no e-mail | T09 | D03, D07, D09, D15, D16 |
| 04 | AC1–AC5 | n/a — no UI | D01–D11 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `setup` no token (01) | D17 | n/a¹ | D06 | n/a² | n/a³ | D02–D05, D09 | D04, D07 | D06 | D11 | D12 | D10 | D08 |
| `setup` with token (04) | D05 | D01, D10 | D06 | D02 | n/a³ | D09 | D02, D07 | D03, D06 | D07 | D11 | via 01-D10 | via 01-D08 |
| approval via API (02) | n/a⁴ | D01–D05 | n/a⁴ | n/a² | n/a³ | n/a⁴ | D02, D03 | n/a⁴ | n/a⁴ | n/a⁴ | n/a⁴ | n/a⁴ |
| `update-status` (03) | D10, D11 | D16 | D18 | n/a⁵ | D12 | D06, D07, D09 | D04, D05 | D08 | D17 | D14 | D18 | D07 |

1. No account exists yet.
2. ONB has no role rules (04 records that any role is accepted — ONB-D1).
3. Not a panel endpoint.
4. Uses other features' endpoints only to show that approval isn't enforced; their own categories belong to those features.
5. One super-admin role only.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api-dev_guid/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. API cases: standard headers from the AUTH test README. Panel cases: see *Panel requests* above. Read e-mails with MAIL-OK; check the database with a SQL client.
