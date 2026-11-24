# Restaurant profile & settings — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api/api/specs/restaurant-settings/test-results.md`, never here.

**Expected results follow the decision statuses** in the [README](../README.md#decisions) (re-synced 2026-11-24, rules in [conventions.md → Decision status](../../../conventions.md#decision-status)):

- RST-D1 is *fixed in code* (2026-11-16): its cases assert the new behaviour and pass today.
- *To fix* decisions (RST-D2 to RST-D6, AUTH-D2): the case asserts the **target** and is tagged `Fails until <ID> is fixed`; the brackets give today's result.
- Finding F2 cases record a known quirk. Finding F1 is resolved; `RST-01-T01` still covers it.

No case is blocked.

**Channel data.** REST-A, if created before 2026-11-16, may show all four Home tiles until it is saved once in Profile or RST-D6's script runs. Save REST-A once in Profile after resetting it to REST-A-BASE so its channels match.

## Fixtures

The AUTH fixtures apply ([AUTH test README](../../auth/test-cases/README.md#fixtures)): REST-A, REST-B, REST-P, OWN-A, WTR-A, CHF-A, WTR-A-OFF, OWN-B, OWN-P, NOREST, MAIL-OK, DEV-1 / DEV-2. Additional:

| Fixture | State |
|---|---|
| REST-A-BASE | The REST-A values to reset to before each case: name `Rest A`, owner `Owner A`, address `1 A Street`, e-mail readable, tax `5.00`, service charge `NULL`, QR `resta@upi`, Takeaway on, Delivery off, Room Service off, Discount on, Show KOT on, KOT name `KOT`, KOT switch on, prefix `NULL`; channels `DINE_IN`, `TAKEAWAY` active, `DELIVERY`, `ROOM_SERVICE` inactive |
| REST-FRESH | A restaurant just registered with Takeaway off and approved (ONB-03-T01); owner OWN-FRESH has changed the PIN; nothing edited since |
| NAME-HEX | The name `Café Délice 12` — exactly 16 bytes in UTF-8 |
| NAME-UNI | The name `রেস্তোরাঁ 🍛 Café` (not 16 bytes) |
| TEXT-256 | A 256-character string of `a` |
| UPI-X | A UPI ID the tester controls, e.g. `tester@upi` |
| ADMIN-1 | Active super admin (as in ONB) |
| DEVICE-PHONE / DEVICE-TAB | A phone and a tablet with the app (phone menu vs tablet rail) |

Body "**app-full**" below = the JSON in [api-contract.md → What the app sends](../api-contract.md#what-the-app-sends), filled with REST-A-BASE values.

## Panel requests (use case 03)

Same as ONB ([ONB test README → Panel requests](../../onboarding/test-cases/README.md#panel-requests-use-case-03)): sign in as ADMIN-1 with a cookie jar. Then `GET /admin/restaurant-management/<REST-A id>` and take the CSRF token from the page script (`"X-CSRF-TOKEN": "…"`). Send the POSTs as JSON with headers `Content-Type: application/json`, `X-CSRF-TOKEN: <token>`.

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 Profile shows current values | T01 | D01 |
| 01 | AC2 fresh restaurant defaults | T02 | D02 |
| 01 | AC3 read-only fields | T03 | — |
| 01 | AC4 who sees Profile (target RST-D3) | T04, T05 | D03 |
| 01 | AC5 panel change reaches the app | T06 | — |
| 01 | AC6 Room Service menus | T08 | — |
| 01 | AC7 own restaurant only | — | D04 |
| 01 | Alt: hex quirk, no network, no token, no restaurant, unsaved edits | T07, T09, T10 | D05, D08–D12 |
| 01 | Account state (target AUTH-D2; `showDetails` approval exemption) | — | D06, D07 |
| 02 | AC1 save name | T01 | D01 |
| 02 | Alt: change and change back | T02 | — |
| 02 | AC2 blank name / owner | T03, T04 | D09 |
| 02 | AC3 GSTIN | T05 | D25 |
| 02 | AC4 e-mail | T06, T17 | D12 |
| 02 | AC5 tax rate; Alt: tax limits (target RST-D4) | T07 | D10, D11, D27 |
| 02 | AC6 discount | T09 | D01 |
| 02 | AC7 KOT on bill | T10 | D01 |
| 02 | AC8 UPI QR | T11 | D18 |
| 02 | AC9 Takeaway switches the channel (RST-D1 fixed) | T08 | D06 |
| 02 | AC10 service charge kept; KOT switch and prefix saved (target RST-D2) | T12 | D04, D05, D07 |
| 02 | AC11 owner only (target RST-D3) | T13 | D18 |
| 02 | AC12 no network | T14 | — |
| 02 | Alt: double tap, long text, hex round-trip, Unicode, layout | T15, T16, T18, T19, T20, T21 | D16, D20, D21, D24 |
| 02 | Old restaurants' channels synced (target RST-D6) | T22 | D28 |
| 02 | Account state (target AUTH-D2) | — | D19 |
| 03 | AC1 page shows values | T01 | D10 |
| 03 | AC2 save details | T02 | D01 |
| 03 | AC3 save configuration | T05 | D02 |
| 03 | AC4 bad tax | T06 | D05 |
| 03 | AC5–AC6 Room Service / Delivery switch the channels (target RST-D5) | T08, T09 | D13 |
| 03 | AC7 empty owner refused (target RST-D4) | T03 | D03 |
| 03 | AC8 no session | T10 | D09 |
| 03 | Alt: contact number, negative tax, Unicode, wipe by app | T04, T07, T11, 02-T12 | D04, D06, D07, D08, D11, D12, D14, D15 |

## Developer endpoint checklist

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `showDetails` (01) | D05 | D06, D07 | D04 | D03 | D11 | n/a¹ | n/a² | D04 | n/a³ | D12 | D04 | D08, D09 |
| `updateDetails` (02) | D19 | D19 | D17 | D18 | D26 | D08–D16, D25 | D04–D07, D28 | D17 | D20 | D23 | D21 | D22, D24 |
| panel restaurant / config (03) | D09 | n/a⁴ | D07, D08 | n/a⁵ | D09 | D03–D06 | D13, D15 | D11 | D14 | D14 | D12 | D05, D10 |

1. No input.
2. Read only.
3. Read only; sharing one request is an app detail.
4. One super-admin account state only (ADM).
5. One super-admin role only.

## How to run the developer cases

PHPUnit feature tests (decided 2026-11-13): `laravel-pos-api/api/tests/Feature/<FeatureFolder>/`, one class per use case, one test method per case ID. API cases: standard headers and tokens from the AUTH test README (`POST /api/auth/loginPin` as the fixture user). Reset REST-A to REST-A-BASE before each case. Check the database with a SQL client; for `binary(16)` IDs use `HEX(...)`.
