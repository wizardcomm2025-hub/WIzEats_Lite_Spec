# 02 — Check the connection and test print

## Summary

With an IP in the field, `Check connection` shows whether something answers on that address and port. `Test print` prints a small test page. If the test page prints and the IP isn't saved yet, the app saves it.

## Prerequisites

- **Backend:** none.
- **Native:** `WifiPrinter.checkConnection`, `WifiPrinter.printPdf` (Android); `react-native-html-to-pdf`

## Actors & preconditions

- **Actor:** owner or waiter.
- **Preconditions:** a valid IP in the field (the buttons are disabled otherwise).

## Main flow

1. **Check:** the user taps `Check connection`. The button spins. If something answers, the chip reads `Online` (green). If not, it reads `Not reachable` (red), after about 15 s. Nothing is saved.
2. **Test print:** the user taps `Test print`. The app builds an 80 mm page with these lines:
   - `WhizEats Lite`
   - `Printer test page`
   - the IP
   - the date and time (Indian format)
   - `If you can read this, Wi-Fi printing works.`

   It prints the page on the printer and cuts the paper. The chip reads `Online`.
3. If the IP or port differs from the saved one, the app saves it. The alert `Printer saved` reads `Test page sent. Bills and KOTs will now print to this printer.` If it was already saved, no alert appears.

## Alternative flows

- **Not reachable:** after about 15 s the chip reads `Not reachable` and the alert `Test print failed` reads `Printer not reachable at <ip>:<port>.` Nothing is saved.
- **Out of paper:** `Test print failed` / `The printer is out of paper.`
- **Cover open:** `Test print failed` / `The printer cover is open.`
- **The printer's port answers but its print service won't connect:** `Could not connect to the printer at <ip>.`
- **A non-printer device answers:** `Check connection` shows `Online` (PRN-D1). What `Test print` does depends on that device.
- **Typing in the IP field** clears the chip.
- While one action runs, all buttons are disabled.

## Acceptance criteria

1. Given PRN-1's IP, when `Check connection` is tapped, then the chip reads `Online`.
2. Given an unused IP on the network, when `Check connection` is tapped, then after about 15 s the chip reads `Not reachable`.
3. Given PRN-1's IP typed but not saved, when `Test print` is tapped, then the test page prints and is cut, the alert `Printer saved` appears, and the subtitle reads `Printing to <PRN-1 ip>`.
4. Given PRN-1 out of paper, when `Test print` is tapped, then the alert `Test print failed` / `The printer is out of paper.` shows and the chip reads `Not reachable`.
5. Given DECOY's IP, when `Check connection` is tapped, then the chip reads `Online`.

## Platform notes

### POS app

`PrinterSettingsScreen` (`check`, `testPrint`, `testPageHtml`), `receiptPaper.ts` (`receiptPageOptions`).

### Android native

`WifiPrinterModule.checkConnection`: up to 3 TCP attempts, 5 s each. `printPdf`:
- the reachability check;
- the SDK connection;
- a status check (`-2` out of paper, `-4` cover open);
- the pages rendered to 576-dot images;
- a cut.

Error texts are in [ui-ux.md](./ui-ux.md#2-alerts).

### POS API, Super-admin panel, Data — MySQL

None.

## Configuration

None.

## Out of scope

- Printer self-test and network configuration (done on the printer).

## Open questions

- None open.
