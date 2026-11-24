# Printer settings — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `POS_APP-kot/specs/printer/test-results.md`, never here.

**Expected results describe today's behaviour**, as fixed by PRN-D1 to PRN-D3 ([README](../README.md#decisions)). No case is blocked. **Tester cases only:** everything happens on the device and the printer; no server endpoint is involved ([api-contract.md](../api-contract.md)).

## Fixtures

The AUTH and ORD fixtures apply ([AUTH](../../auth/test-cases/README.md#fixtures), [ORD](../../orders/test-cases/README.md#fixtures)). Additional:

| Fixture | State |
|---|---|
| WIFI-T | A 2.4 GHz test Wi-Fi network (a normal home-style `x.x.x.0/24` network) with no other device listening on port 9100 |
| DEV-P / DEV-T | An Android phone / an Android tablet with the app, on WIFI-T |
| PRN-1 | An SPRT 80 mm Wi-Fi thermal printer (e.g. SP-POS892) on WIFI-T, port 9100, paper loaded. Its IP is known from its self-test page |
| PRN-2 | A second such printer on WIFI-T. **Off** unless a case says otherwise |
| DECOY | A laptop on WIFI-T that a developer sets to listen on TCP port 9100 (e.g. `ncat -lk 9100`). It is not a printer. **Stopped** unless a case says otherwise |
| UNUSED-IP | An address on WIFI-T with no device on it (e.g. `.250`) |
| ORDER-H | A REST-A order paid through the API, so Order History shows `Print Bill No. <n>` for it (BILL-D2, HIST-D2) |
| CLEAN | Before each case: open the Printer screen; if `Remove printer` shows, tap it; clear the IP field; set the port to `9100`. PRN-1 on, PRN-2 off, DECOY stopped |

Signed in as OWN-A on DEV-P unless a case says otherwise. "About 15 s" means 10–20 s. An unreachable printer takes three 5-second attempts.

## Coverage matrix

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | Entry points per role (PRN-D3) | T01 | — |
| 01 | AC1 type and save | T02, T03 | — |
| 01 | AC2 / AC3 search and pick (PRN-D1) | T05, T08 | — |
| 01 | AC4 invalid IP (PRN-D2) | T04 | — |
| 01 | Alt: nothing found, no Wi-Fi, remove, port, reopen | T06, T07, T09, T10, T12 | — |
| 01 | AC5 survives sign-out (PRN-D3) | T11 | — |
| 02 | AC1 / AC2 / AC5 check connection | T01, T02, T03 | — |
| 02 | AC3 test print saves | T04, T05 | — |
| 02 | AC4 out of paper; cover open; unreachable | T06, T07, T08 | — |
| 02 | Alt: chip cleared, buttons locked | T09 | — |
| 03 | AC1 silent Wi-Fi print (KOT, bill, tablet) | T01, T02, T04 | — |
| 03 | AC2 no printer → print screen | T03, T04 | — |
| 03 | AC3 unreachable → alert + print screen | T05, T09 | — |
| 03 | AC4 silent switch (PRN-D1) | T06 | — |
| 03 | AC5 several found | T07 | — |
| 03 | Alt: out of paper, long receipt, double tap, iOS | T08, T10, T11, T12 | — |

## How to run

On real devices with real printers; emulators can't reach the printers. A developer starts and stops DECOY. Check the printed paper as well as the screen.
