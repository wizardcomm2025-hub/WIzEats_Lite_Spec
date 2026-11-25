# 02 — Check the printer and test print

## Summary

On the Wi-Fi tab, `Check connection` shows whether something answers on the IP and port, and `Test print` prints a small test page. On the built-in tab, `Check printer` shows the printer's state and `Test print` prints the test page there. A test page that prints on an unsaved printer saves it. **Today the test page ends with a stray `false` / `true` (PRN-D4 — To fix).**

## Prerequisites

- **Backend:** none.
- **Native:** `WifiPrinter.checkConnection` / `printPdf`, `SunmiPrinter.checkStatus` / `printPdf`, `UsbPrinter.checkStatus` / `printPdf` (Android); `react-native-html-to-pdf`

## Actors & preconditions

- **Actor:** owner or waiter.
- **Preconditions:** Wi-Fi: a valid IP in the field (the buttons are disabled otherwise). Built-in: a built-in printer detected.

## Main flow — Wi-Fi

1. **Check:** the user taps `Check connection`. The button spins. If something answers, the chip reads `Online` (green). If not, it reads `Not reachable` (red), after about 15 s. Nothing is saved.
2. **Test print:** the user taps `Test print`. The app builds an 80 mm page — `WhizEats Lite`, `Printer test page`, the IP, the date and time (Indian format), `If you can read this, Wi-Fi printing works.` — prints it and cuts the paper. The chip reads `Online`. *Today* a last line `false` is printed too; *target:* nothing (PRN-D4).
3. If the IP or port differs from the saved one, the app saves it. The alert `Printer saved` reads `Test page sent. Bills and KOTs will now print to this printer.` If it was already saved, no alert appears.

## Main flow — built-in tab

1. **Check:** `Check printer` spins, then the chip shows the state: Sunmi `Ready`, `Out of paper`, `Cover open` or `Not available`; USB `Ready`, `Permission needed` (the user denied Android's dialog) or `Not available` (not attached). For USB the list of attached printers is refreshed first, and Android's permission dialog appears if the app doesn't have permission yet.
2. **Test print:** the page reads `Built-in printer` / `USB printer` instead of the IP and `If you can read this, built-in printing works.` / `… USB printing works.`; *today* a last line `true` follows (PRN-D4). It is widened if the page came out narrow, printed in one strip and cut. The chip reads `Ready`.
3. If this printer isn't the saved one, the app saves it: `Printer saved` / `Test page sent. Bills and KOTs will now print on the built-in printer.` (`USB printer`).

## Alternative flows

- **Wi-Fi not reachable:** after about 15 s the chip reads `Not reachable` and the alert `Test print failed` reads `Printer not reachable at <ip>:<port>.` Nothing is saved.
- **Wi-Fi / Sunmi out of paper or cover open:** `Test print failed` / `The printer is out of paper.` / `The printer cover is open.`; the Wi-Fi chip reads `Not reachable` (accepted), the built-in chip `Not available`.
- **Wi-Fi port answers but the print service won't connect:** `Could not connect to the printer at <ip>.`
- **A non-printer device answers:** `Check connection` shows `Online` (accepted, PRN-D1).
- **USB without paper:** the test "succeeds" — no paper check (accepted, PRN-D4).
- **USB permission denied on test:** `Test print failed` / `Allow this app to use the USB printer and try again.`
- **Sunmi doesn't answer in 30 s:** `Test print failed` / `The printer did not respond.`
- **Typing in the IP field** clears the Wi-Fi chip; **picking another USB printer** clears the built-in chip.
- While one action runs, all buttons are disabled.

## Acceptance criteria

1. Given PRN-1's IP, when `Check connection` is tapped, then the chip reads `Online`.
2. Given an unused IP on the network, when `Check connection` is tapped, then after about 15 s the chip reads `Not reachable`.
3. Given PRN-1's IP typed but not saved, when `Test print` is tapped, then the test page prints and is cut, the alert `Printer saved` appears, and the subtitle reads `Printing to <PRN-1 ip>`.
4. Given PRN-1 out of paper, when `Test print` is tapped, then the alert `Test print failed` / `The printer is out of paper.` shows.
5. Given DECOY's IP, when `Check connection` is tapped, then the chip reads `Online`.
6. *(Target, PRN-D4)* Given any test page, then nothing is printed after `If you can read this, … printing works.`
7. Given SUNMI-T with the cover open, when `Check printer` is tapped, then the chip reads `Cover open`.

## Platform notes

### POS app

`PrinterSettingsScreen` (`check`, `checkBuiltIn`, `testPrint`, `testPrintBuiltIn`, `testPageHtml`), `receiptPaper.ts` (`receiptPageOptions`).

### Android native

`WifiPrinterModule.checkConnection`: up to 3 TCP attempts, 5 s each. `printPdf`: the reachability check; the SDK connection; a status check (`-2` out of paper, `-4` cover open); the pages rendered to 576-dot images; a cut. `SunmiPrinterModule`: state check, one buffered job, 3-line feed, cut, 30 s wait. `UsbPrinterModule`: permission, ESC/POS raster, feed-and-cut. Error texts are in [api-contract.md](./api-contract.md).

### POS API, Super-admin panel, Data — MySQL

None.

## Configuration

None.

## Out of scope

- Printer self-test and network configuration (done on the printer).

## Open questions

- None open. Related decisions: PRN-D1, PRN-D4.
