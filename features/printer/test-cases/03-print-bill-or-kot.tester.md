# 03 — Print a bill or KOT — tester cases

Use case: [03-print-bill-or-kot.md](../03-print-bill-or-kot.md). Screen text: [ui-ux.md](../ui-ux.md). Fixtures: [README.md](./README.md#fixtures) (CLEAN before each case).

"Send a KOT" = Home → Takeaway → add `Paneer Tikka` → send the KOT (ORD). The KOT card then shows `Print KOT` (phone). "Print screen" = Android's own print dialog.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| PRN-03-T01 | AC1 | Save PRN-1. Send a KOT. Tap `Print KOT`. | The button spins; PRN-1 prints the KOT at full width and cuts it. No print screen, no alert. |
| PRN-03-T02 | AC1 (tablet) | On DEV-T: save PRN-1, send a KOT, tap `Print` on the KOT banner. | As T01. |
| PRN-03-T03 | AC2 | Nothing saved. Send a KOT; tap `Print KOT`. Then press Back on the print screen. | The print screen opens with a job named `KOT #<KOT number>`. After Back: no alert; nothing printed. |
| PRN-03-T04 | AC1, AC2 | ORDER-H in Order History → History tab, expand the card. (a) With PRN-1 saved, tap `Print Bill No. <n>`. (b) With nothing saved, tap it again. | (a) `Printing…`, then PRN-1 prints the bill and cuts it; no screen. (b) The print screen opens with the job `Bill No. <n>`. |
| PRN-03-T05 | AC3 (PRN-D2) | Save PRN-1, then switch it off. Send a KOT; tap `Print KOT`. | After about 15 s: alert `Printer not reachable` / `Printer not reachable at <PRN-1 ip>:9100.` / `Opening the print screen instead.`, and the print screen opens straight away (both at once). The Printer screen still shows PRN-1. |
| PRN-03-T06 | AC4 (PRN-D1) | Save PRN-1, switch it off, switch PRN-2 on. Send a KOT; tap `Print KOT`. Open the Printer screen. | After about 15 s the KOT prints on **PRN-2**, with no alert or print screen. The Printer screen reads `Printing to <PRN-2 ip>`. |
| PRN-03-T07 | AC5 | Save PRN-1, switch it off; PRN-2 on; start DECOY. Send a KOT; tap `Print KOT`. | Alert `Printer not reachable` / `Several printers were found. Choose one in Setup → Printer.` / `Opening the print screen instead.`, and the print screen opens. The saved printer is still PRN-1. |
| PRN-03-T08 | Alt: out of paper | Save PRN-1; take its paper out. Send a KOT; tap `Print KOT`. | Alert `Printer not reachable` / `The printer is out of paper.` / `Opening the print screen instead.`, and the print screen opens. Saved printer unchanged. |
| PRN-03-T09 | Alt: bad port (PRN-D2) | Save PRN-1 with port `70000` (PRN-01-T10 b). Send a KOT; tap `Print KOT`. | Almost at once: alert `Printer not reachable` / `Printer not reachable at <PRN-1 ip>:70000.` / `Opening the print screen instead.`, and the print screen opens. |
| PRN-03-T10 | Alt: long receipt | Save PRN-1. Send a KOT with 30 different items. Tap `Print KOT`. | The whole KOT prints at full width (not shrunk), in one strip, with one cut at the end. |
| PRN-03-T11 | Alt: double tap | Save PRN-1. Send a KOT; tap `Print KOT` twice quickly. | One KOT prints (the second tap is ignored while the first runs). |
| PRN-03-T12 | Alt: iOS (if an iPhone build exists) | On an iPhone, send a KOT; tap `Print KOT`. | The share sheet opens with the KOT PDF. On the Printer screen, `Search printers` does nothing (no list, no message). |
