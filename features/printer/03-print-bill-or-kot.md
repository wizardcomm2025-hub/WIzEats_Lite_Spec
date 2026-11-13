# 03 — Print a bill or KOT

## Summary

Every print button goes through one routine:
- the KOT card's `Print KOT` (phone) or `Print` (tablet);
- `Print Bill`, and `Print Bill No. <n>` in Order History.

With a saved printer on Android, the receipt prints silently on it. When that fails, the app looks for the printer once more. If it finds a single other device, it prints there and saves it, without asking (PRN-D1). Otherwise it shows an alert and opens the Android print screen.

## Prerequisites

- **Backend:** none for printing. The data comes from ORD / BILL / HIST.
- **Native:** `WifiPrinter.printPdf`, `WifiPrinter.scanPrinters`, `PdfPrint.printPdf` (Android print screen); `react-native-share` (iOS)

## Actors & preconditions

- **Actor:** owner or waiter.
- **Preconditions:** a KOT card on screen (after sending a KOT, ORD), or an Order History card with a saved bill (HIST; orders paid through the API, BILL-D2).

## Main flow

1. The user taps the print button. It shows a spinner (`Printing…` on bill buttons).
2. The app builds the receipt PDF: 80 mm wide, as long as the content, split into equal pages when it is longer than about 134 mm. The print job is named:
   - KOT: `<KOT label> #<KOT number>`;
   - bill: `Bill No. <bill number>`, or `Bill No. <order ID>` when there is no bill number.
3. **Saved printer, Android:** the app sends it to the saved printer. It prints and is cut; no screen or message appears.
4. **No saved printer (Android):** the Android print screen opens with the PDF; the user picks a printer or backs out.
5. **iOS:** the share sheet opens with the PDF.

## Alternative flows

- **Saved printer fails** (switched off, new IP, out of paper, cover open):
  1. The failure takes up to about 15 s when the printer can't be reached.
  2. The app searches the network once on the same port. Then:
     - **exactly one address found, and it differs from the saved IP:** the app prints there and saves it as the printer, with no message (PRN-D1);
     - **several addresses found:** alert `Printer not reachable` / `Several printers were found. Choose one in Setup → Printer.` + `Opening the print screen instead.`, and the Android print screen opens;
     - **otherwise** (none found, or only the saved IP, e.g. when it's out of paper): alert `Printer not reachable` / `<reason>` + `Opening the print screen instead.`, and the Android print screen opens. `<reason>` is e.g. `Printer not reachable at <ip>:<port>.` or `The printer is out of paper.`
  3. If the fallback print to the single found address also fails, the alert and print screen follow as in "otherwise".
- **Print screen can't open / PDF can't be built:** a KOT gives the alert `Print Failed`, a bill `Share Failed`, with the reason, or `Something went wrong. Please try again.` (PRN-D2).
- **Two taps in a row:** a KOT button ignores taps while busy. Wi-Fi jobs are queued one at a time on the device.
- **Saved port above 65535:** every print fails as not reachable, and the alert and print screen follow (PRN-D2).

## Acceptance criteria

1. Given PRN-1 saved and on, when `Print KOT` is tapped, then the KOT prints on PRN-1 and is cut, and no screen or alert appears.
2. Given nothing saved, when `Print KOT` is tapped, then the Android print screen opens with a job named `KOT #<KOT number>`.
3. Given PRN-1 saved but off, and nothing else on port 9100, when `Print Bill No. <n>` is tapped, then after about 15 s the alert `Printer not reachable` / `Printer not reachable at <PRN-1 ip>:9100.` + `Opening the print screen instead.` shows and the Android print screen opens.
4. Given PRN-1 saved but off and PRN-2 on, when `Print KOT` is tapped, then the KOT prints on PRN-2 with no message, and the Printer screen then reads `Printing to <PRN-2 ip>` (PRN-D1).
5. Given PRN-1 saved but off, and PRN-2 and DECOY on, when `Print KOT` is tapped, then the alert says `Several printers were found. Choose one in Setup → Printer.` and the print screen opens. The saved printer stays PRN-1.

## Platform notes

### POS app

`printPdf.ts` (`printPdf`, `printPdfOverWifi`, `openPrintDialog`), `useShareKot` (`printKot`), `usePrintBill` (`printBill`), `KOTModal`, `PanelKotBanner`, `PaymentSuccessModal`, `ShareBillActions`, `PrintPastBillButton`.

### Android native

`WifiPrinterModule.printPdf` runs one job at a time and clears the SDK's cached printer after each job. `PdfPrintModule.printPdf` opens `PrintManager` and resolves as soon as the screen is up.

### POS API, Super-admin panel, Data — MySQL

None.

## Configuration

None.

## Out of scope

- Automatic printing (nothing prints without a tap). Reprinting a KOT later from the kitchen screen.

## Open questions

- None open.
