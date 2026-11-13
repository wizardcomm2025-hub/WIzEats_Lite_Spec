# 01 — Find and save a printer — tester cases

Use case: [01-find-and-save-printer.md](../01-find-and-save-printer.md). Screen text: [ui-ux.md §1](../ui-ux.md#1-printer-screen). Fixtures: [README.md](./README.md#fixtures) (CLEAN before each case).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| PRN-01-T01 | Entry points (PRN-D3) | (a) DEV-P as OWN-A: open the drawer. (b) DEV-P as WTR-A: open the drawer. (c) DEV-P as CHF-A: open the drawer. (d) DEV-T as OWN-A, then WTR-A, then CHF-A: look at the bottom of the side rail. | (a) `Account` section has `Printer`. (b) `Operations` section has `Printer`. (c) No `Printer` item. (d) Owner and waiter: a `Printer` item at the bottom; chef: none. Each `Printer` item opens the Printer screen. |
| PRN-01-T02 | Main flow 1 | Open the Printer screen. | Title `Printer`; card `Wi-Fi receipt printer` with `Not saved — prints open the Android print screen` in amber; IP field empty with placeholder `192.168.0.200`; Port `9100`; no status chip; `Check connection`, `Test print` and `Save printer` disabled; no `Remove printer`. |
| PRN-01-T03 | AC1 | Type PRN-1's IP. Tap `Save printer`. | While typing a valid IP: the amber warning `Not saved yet. Tap Save printer so bills and KOTs print here.` and `Save printer` enabled. After the tap: alert `Saved` / `Bills and KOTs will now print to this printer.`; subtitle `Printing to <PRN-1 ip>`; button `Saved` (disabled); warning gone; `Remove printer` shown. |
| PRN-01-T04 | AC4 (PRN-D2) | Type `192.168.0`, then `300.1.1.1`, then `1.2.3.4.5`. | Each time: the field border turns red; `Check connection`, `Test print`, `Save printer` stay disabled; no alert appears; `Search printers` stays enabled. |
| PRN-01-T05 | AC2, AC3 | Start DECOY. Tap `Search printers`. Tap PRN-1's row. | The button spins for a few seconds; then two rows: PRN-1's IP and DECOY's IP, each `Tap to connect`. After tapping PRN-1: no alert; the IP field shows PRN-1's IP; subtitle `Printing to <PRN-1 ip>`; chip `Online`; PRN-1's row highlighted with `Connected`. |
| PRN-01-T06 | Alt: nothing found | Switch PRN-1 off. Tap `Search printers`. | `No printer found on this Wi-Fi. Check it is on and on the same network (2.4 GHz).` |
| PRN-01-T07 | Alt: no Wi-Fi | Turn on airplane mode. Tap `Search printers`. | Alert `Search failed` / `Connect to a Wi-Fi network first.` |
| PRN-01-T08 | Alt (PRN-D1) | Switch PRN-1 off; start DECOY. Search; tap DECOY's row. | DECOY is listed and, once tapped, saved: subtitle `Printing to <DECOY ip>`, chip `Online`, row `Connected`. No warning that it isn't a printer. |
| PRN-01-T09 | Main flow 6 (PRN-D2) | Save PRN-1 (T03). Tap `Remove printer`. | No confirmation. Subtitle back to `Not saved — …`; chip gone; `Remove printer` gone; the IP stays in the field and the `Not saved yet…` warning shows with `Save printer` enabled. |
| PRN-01-T10 | Alt: port (PRN-D2) | (a) Type PRN-1's IP, clear the port, tap `Save printer`; leave the screen and reopen it. (b) Type PRN-1's IP, port `70000`, save; reopen. | (a) Saved with no message about the port; on reopening the port shows `9100`. (b) Saved; on reopening the port shows `70000` (see PRN-03-T09 for printing). |
| PRN-01-T11 | AC5 (PRN-D3) | Save PRN-1. Sign out. Sign in on the same device as a REST-B user (owner or waiter). Open the Printer screen. | `Printing to <PRN-1 ip>`: the setting stayed through sign-out and applies to the other restaurant. |
| PRN-01-T12 | Alt: reopen | Save PRN-1. Go to Home and back to the Printer screen. | IP and port shown as saved; button `Saved`; no status chip; no search results. |
