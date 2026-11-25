# Printer settings — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `POS_APP-kot/specs/printer/test-results.md`, never here.

**Expected results follow the decision statuses** in the [README](../README.md#decisions) (re-synced 2026-11-24, rules in [conventions.md → Decision status](../../../conventions.md#decision-status)):

- *To fix* decisions (PRN-D1, PRN-D2, PRN-D4): the case asserts the **target** and is tagged `Fails until <ID> is fixed`; today's result is in brackets.
- Accepted (PRN-D3 and the accepted parts of D1, D2, D4): asserted as they are.

No case is blocked. **Tester cases only:** everything happens on the device and the printer; no server endpoint is involved ([api-contract.md](../api-contract.md)).

## Fixtures

The AUTH and ORD fixtures apply ([AUTH](../../auth/test-cases/README.md#fixtures), [ORD](../../orders/test-cases/README.md#fixtures)). Additional:

| Fixture | State |
|---|---|
| WIFI-T | A 2.4 GHz test Wi-Fi network (a normal home-style `x.x.x.0/24` network) with no other device listening on port 9100 |
| DEV-P / DEV-T | An Android phone / an Android tablet with the app, on WIFI-T. Neither has a built-in or USB printer |
| SUNMI-T | A Sunmi terminal with a built-in printer (e.g. D3 MINI), paper loaded, with the app, on WIFI-T |
| USB-T | An Android POS terminal whose receipt printer is a USB printer-class device (or an Android tablet with an external USB receipt printer on OTG), with the app; the app has **not** been allowed to use it yet |
| USB-2 | A second external USB receipt printer that can be plugged into USB-T |
| PRN-1 | An SPRT 80 mm Wi-Fi thermal printer (e.g. SP-POS892) on WIFI-T, port 9100, paper loaded. Its IP is known from its self-test page |
| PRN-2 | A second such printer on WIFI-T. **Off** unless a case says otherwise |
| DECOY | A laptop on WIFI-T that a developer sets to listen on TCP port 9100 (e.g. `ncat -lk 9100`). It is not a printer. **Stopped** unless a case says otherwise |
| UNUSED-IP | An address on WIFI-T with no device on it (e.g. `.250`) |
| ORDER-H | A REST-A order paid in the app (BILL, fixed 2026-11-24), so Order History shows `Print Bill No. <n>` for it |
| CLEAN | Before each case: open the Printer screen; if `Remove printer` shows, remove the printer (confirm if asked); clear the IP field; set the port to `9100`. PRN-1 on, PRN-2 off, DECOY stopped |

Signed in as OWN-A on DEV-P unless a case says otherwise. "About 15 s" means 10–20 s. An unreachable printer takes three 5-second attempts.

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | Entry points per role (PRN-D3) | T01 | — |
| 01 | AC1 type and save | T02, T03 | — |
| 01 | AC2 / AC3 search and pick (accepted part of PRN-D1) | T05, T08 | — |
| 01 | AC4 invalid IP | T04 | — |
| 01 | AC6 port 1–65535 (target PRN-D2) | T10 | — |
| 01 | AC7 remove asks first (target PRN-D2) | T09 | — |
| 01 | Alt: nothing found, no Wi-Fi, reopen | T06, T07, T12 | — |
| 01 | AC5 survives sign-out (PRN-D3) | T11 | — |
| 01 | AC8 / AC9 built-in tab, USB pick, switching kind | T13, T14, T15 | — |
| 02 | AC1 / AC2 / AC5 check connection | T01, T02, T03 | — |
| 02 | AC3 test print saves; AC6 no stray text (target PRN-D4) | T04, T05 | — |
| 02 | AC4 out of paper; cover open; unreachable | T06, T07, T08 | — |
| 02 | Alt: chip cleared, buttons locked | T09 | — |
| 02 | AC7 built-in check and test (Sunmi, USB) | T10–T13 | — |
| 03 | AC1 silent Wi-Fi print (KOT, bill, tablet) | T01, T02, T04 | — |
| 03 | AC2 no printer → print screen | T03, T04 | — |
| 03 | AC3 unreachable → alert, then print screen (target PRN-D2) | T05 | — |
| 03 | AC4 / AC5 no silent switch; offer found printers (target PRN-D1) | T06, T07, T08 | — |
| 03 | Alt: bad port, long receipt, double tap, iOS, bill print error title | T09–T13 | — |
| 03 | AC6 built-in / USB printing and failures | T14–T17 | — |

## How to run

On real devices with real printers; emulators can't reach the printers. A developer starts and stops DECOY. Check the printed paper as well as the screen.
