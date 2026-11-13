# 02 — Check the connection and test print — tester cases

Use case: [02-check-and-test-print.md](../02-check-and-test-print.md). Screen text: [ui-ux.md](../ui-ux.md). Fixtures: [README.md](./README.md#fixtures) (CLEAN before each case).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| PRN-02-T01 | AC1 | Type PRN-1's IP. Tap `Check connection`. | The button spins briefly; chip `Online` (green). Nothing saved: subtitle still `Not saved — …`. |
| PRN-02-T02 | AC2 | Type UNUSED-IP. Tap `Check connection`. | All buttons disabled while it runs; after about 15 s the chip reads `Not reachable` (red). No alert. |
| PRN-02-T03 | AC5 (PRN-D1) | Start DECOY. Type DECOY's IP. Tap `Check connection`. | Chip `Online`, although DECOY is not a printer. |
| PRN-02-T04 | AC3 | Type PRN-1's IP (not saved). Tap `Test print`. | PRN-1 prints and cuts a slip: `WhizEats Lite`, `Printer test page`, PRN-1's IP, today's date and time, `If you can read this, Wi-Fi printing works.` Alert `Printer saved` / `Test page sent. Bills and KOTs will now print to this printer.`; subtitle `Printing to <PRN-1 ip>`; chip `Online`. |
| PRN-02-T05 | Main flow 3 | After T04, tap `Test print` again. | A second test slip prints; no alert; chip `Online`. |
| PRN-02-T06 | AC4 | Take the paper out of PRN-1. Type its IP; tap `Test print`. | Nothing prints; alert `Test print failed` / `The printer is out of paper.`; chip `Not reachable`. |
| PRN-02-T07 | Alt: cover open | Paper in, cover open. `Test print` to PRN-1. | Alert `Test print failed` / `The printer cover is open.`; chip `Not reachable`. |
| PRN-02-T08 | Alt: unreachable | Type UNUSED-IP. Tap `Test print`. | After about 15 s: alert `Test print failed` / `Printer not reachable at <UNUSED-IP>:9100.`; chip `Not reachable`; nothing saved. |
| PRN-02-T09 | Alt | Do T01 so the chip shows `Online`, then change one digit of the IP. Then tap `Check connection` on UNUSED-IP and, while it runs, try the other buttons. | The chip disappears as soon as the IP changes. While the check runs, `Search printers`, `Test print` and `Save printer` don't respond. |
