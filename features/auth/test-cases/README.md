# Sign-in & PIN — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api-dev_guid/specs/auth/test-results.md`, never here.

**Expected results describe today's behaviour**, as fixed by decisions AUTH-D1 to AUTH-D11 in the [README](../README.md#decisions). Cases that check an accepted risk say so. The one behaviour change, AUTH-D11, is tested by `AUTH-07-D18`, which fails until it is built. No case is blocked.

## Fixtures

Describe test data by name and state, never by ID. Every user below has a known 6-digit PIN unless stated.

| Fixture | State |
|---|---|
| REST-A | Restaurant: approved (`isApproved = 1`), active, onboarding done, has a restaurant e-mail the tester can read |
| REST-B | A second restaurant like REST-A (isolation) |
| REST-P | Restaurant pending approval (`isApproved = 0`), onboarding done |
| REST-R | Restaurant rejected (`isApproved = 2`) |
| REST-OFF | Restaurant approved but switched off (`status = 0`) |
| REST-NOMAIL | Restaurant approved, `restaurantEmail` empty |
| OWN-A, WTR-A, CHF-A | Active owner, waiter, chef of REST-A, `mustChangePin = 0` |
| MCP-A | Active waiter of REST-A with `mustChangePin = 1` |
| WTR-A-OFF | Waiter of REST-A, switched off (`status = 0`) |
| NOPIN-A | Active user of REST-A with no PIN (`pin` null) |
| OWN-B | Active owner of REST-B |
| OWN-P, OWN-R, OWN-OFF, OWN-NM | Active owners of REST-P, REST-R, REST-OFF, REST-NOMAIL |
| NOREST | User row with `restaurantId` null and a known PIN (signs in, then gets 403 on protected calls) |
| UNV-A | Active user of REST-A with `phone_verified_at` null (legacy register `type: verify`) |
| PH-NEW | A valid mobile number (starts 6–9) that is in no `users` row |
| MAIL-OK | Mail transport the tester can read (for example Mailpit / log driver) |
| MAIL-DOWN | Mail transport pointed at an unreachable host |
| DEV-1, DEV-2 | Two phones / tablets with the POS app installed (tester cases) |

Tokens for developer cases: call `POST /api/auth/loginPin` as the fixture user and use `data.token`.

## Standard request

- Base URL: the API's `APP_URL` + `/api`.
- Headers on every developer case: `Accept: application/json`, `Content-Type: application/json`, plus `Authorization: Bearer <token>` when the caller has one.
- Rate-limit cases: run from one IP and wait for the window to reset (1 minute; 10 minutes for `forgotPin` per phone) before other cases, or clear the cache between runs.

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 known number → PIN step | T01 | D11 |
| 01 | AC2 new number → onboarding | T02 | D12 |
| 01 | AC3–AC5 format errors | T03, T04, T05 | D01–D07, D09 |
| 01 | AC6 switched-off user still "exists" | T06 | D11 |
| 01 | AC7 rate limit | — | D14 |
| 01 | AC8 nothing beyond `exists` returned | — | D11, D13 |
| 01 | AC9 error clears on edit; double tap; no network | T07, T08, T09 | — |
| 02 | AC1–AC2 landing by role | T01, T02, T03 | D01 |
| 02 | AC3 forced change | T04 | D01 |
| 02 | AC4 pending / rejected / switched-off restaurant | T05, T06 | D12 |
| 02 | AC5 wrong PIN; no PIN | T07 | D02, D03 |
| 02 | AC6 switched-off user | T08 | D04 |
| 02 | AC7 rate limit | T09 | D09, D10 |
| 02 | AC8 button / digits only | T10 | D06, D07 |
| 02 | AC9 PIN never shown / returned | T11 | D01, D17 |
| 02 | AC10 multiple sessions | T12 | D13 |
| 02 | Alt: change number; no network | T13, T14 | D05 |
| 03 | AC1 reset sent, masked e-mail | T01 | D01 |
| 03 | AC2–AC3 old PIN dead, new PIN → Change PIN | T02 | D01 |
| 03 | AC4 no e-mail | T03 | D03 |
| 03 | AC5 mail down | T04 | D04 |
| 03 | AC6 rate limit | T05 | D06, D07 |
| 03 | AC7 masking | T01 | D01 |
| 03 | AC8 PIN never in app / response | T01 | D01 |
| 03 | Alt: switched-off user; other sessions; no network | T06, T07 | D08, D11 |
| 04 | AC1 forced screen only | T01 | — |
| 04 | AC2–AC3 success | T02, T03 | D01 |
| 04 | AC4 wrong current PIN | T04 | D04 |
| 04 | AC5–AC6 inline validation | T05, T06 | D05 |
| 04 | AC7 logout from Change PIN | T07 | — |
| 04 | AC8 app restart | T08 | — |
| 04 | Alt: other sessions; no limit; no network | T09, T10 | D01, D08 |
| 05 | AC1–AC3 logout | T01, T02, T03 | D01 |
| 05 | AC4 token dead | — | D01 |
| 05 | AC5 other device stays | T04 | D02 |
| 05 | AC6 offline logout | T05 | — |
| 05 | AC7 next user starts clean | T06 | — |
| 06 | AC1 resume offline | T01 | — |
| 06 | AC2–AC3 session ended by server | T02 | D06 |
| 06 | AC4 401 on `/auth/` doesn't end session | T03 | 04-D04 |
| 06 | AC5 switched-off user keeps working | T04 | D02 |
| 06 | AC6 no expiry | — | D01 |
| 06 | Alt: rejected / switched-off restaurant; removed user | T05 | D03, D04, D05 |
| 07 | AC1–AC7 | n/a — no UI | D01–D20 |

## Developer endpoint checklist

`✓` = covered (case IDs in the file for that use case). `n/a` = not applicable, reason given.

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `checkPhone` (01) | D15 (token ignored) | D11 | n/a¹ | n/a² | n/a³ | D01–D09 | n/a⁴ | D13 | n/a⁵ | D14 | D10 | D16 |
| `loginPin` (02) | D17 | D04, D12 | D11 | n/a² | n/a³ | D05–D08 | D03 | D11 | D13 | D09, D10 | D14 | D16 |
| `forgotPin` (03) | D10 | D08 | D09 | n/a² | n/a³ | D05, D13 | D11 | D09 | D12 | D06, D07 | D13 | D14 |
| `changePin` (04) | D02 | D03, D09 | D06 | n/a² | D13 | D05 | D07 | D06 | D10 | D08 | D11 | D12 |
| `logout` (05) | D03 | D04 | D06 | n/a² | n/a³ | n/a⁶ | n/a⁴ | D05 | D07 | n/a⁷ | n/a⁶ | D08 |
| session (06) | D06, D07 | D02–D05 | D08 | n/a² | n/a³ | n/a⁶ | n/a⁴ | D08 | n/a⁵ | n/a⁷ | n/a⁶ | n/a⁸ |
| `sendOtp` (07) | n/a⁹ | D01 | n/a¹ | n/a² | n/a³ | D03, D04 | D01 | — | n/a⁵ | D09 | D03 | D19 |
| `verifyOtp` (07) | D10 | D08 | n/a¹ | n/a² | n/a³ | D11 | D06, D07 | — | n/a⁵ | D09 | D11 | D19 |
| `register` (07) | n/a⁹ | D13, D14 | D16 | n/a² | n/a³ | D15 | D20 | D16 | D18 | D17 | D15 | D19 |

1. Public endpoint keyed only by phone; no restaurant IDs are accepted.
2. AUTH has no role rules — every role may use every AUTH endpoint.
3. AUTH has no super-admin panel page; the `changePin` case covers the panel session cookie for all token endpoints.
4. The app sends nothing this endpoint could refuse beyond what the other categories cover.
5. Read-only or stateless; no race can corrupt data.
6. No body fields.
7. No limiter and the spec doesn't ask for one.
8. No response of its own.
9. Public endpoint; a token is not used.

## How to run the developer cases

Runner to be decided (GETTING-STARTED item 3). Until then, run them by hand with Postman or curl using the standard request above, and check the database with a SQL client. Each case's "Database after" names the rows to look at.
