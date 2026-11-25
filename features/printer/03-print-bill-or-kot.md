# 03 — Print a bill or KOT

## Summary

Every print button goes through one routine: the KOT card's `Print KOT` (phone) or `Print` (tablet), `Print Bill` on the payment success screen, and `Print Bill No. <n>` in Order History. With a saved printer on Android, the receipt prints silently on it. When that fails, the user is told why and the Android print screen opens. **Today a failed Wi-Fi print can switch silently to another device on the network (PRN-D1), and the alert and print screen appear together (PRN-D2) — both To fix.**

## Prerequisites

- **Backend:** none for printing. The data comes from ORD / BILL / HIST.
- **Native:** `WifiPrinter.printPdf` / `scanPrinters`, `SunmiPrinter.printPdf`, `UsbPrinter.printPdf`, `PdfPrint.printPdf` (Android print screen); `react-native-share` (iOS)

## Actors & preconditions

- **Actor:** owner or waiter.
- **Preconditions:** a KOT card on screen (after sending a KOT, ORD), the payment success screen (BILL), or an Order History card with a saved bill (HIST).

## Main flow

1. The user taps the print button. It shows a spinner (`Printing…` on bill buttons).
2. The app builds the receipt PDF: 80 mm wide, as long as the content, split into equal pages when it is longer than about 134 mm. The print job is named:
   - KOT: `<KOT label> #<KOT number>`;
   - bill: `Bill No. <bill number>`, or `Bill No. <order ID>` when there is no bill number.
3. **Saved printer, Android:** the app sends it to the saved Wi-Fi, Sunmi or USB printer. It prints and is cut; no screen or message appears.
4. **No saved printer (Android):** the Android print screen opens with the PDF; the user picks a printer or backs out.
5. **iOS:** the share sheet opens with the PDF (a saved printer is ignored).

## Alternative flows

- **Saved Wi-Fi printer fails — target (PRN-D1, PRN-D2):**
  1. Out of paper, cover open or another error: alert `Printer not reachable` / `<reason>` + `Opening the print screen instead.`; after `OK` the print screen opens. No search.
  2. Can't be reached (about 15 s) or won't connect: the app searches the network once on the same port.
     - **1–3 addresses found other than the saved one:** the alert also says `Another printer answered on this Wi-Fi.` and offers `Print on <ip>` per address and `Open print screen`. A chosen address that prints is saved (`Printer saved` / `Bills and KOTs will now print to <ip>.`); one that fails leads to the alert and the print screen.
     - **more than 3:** `Several printers were found. Choose one on the Printer screen.`, `Open print screen`.
     - **none (or only the saved IP):** the reason alert, then the print screen after `OK`.
- **Saved Wi-Fi printer fails — today:** the app always searches. Exactly one address other than the saved IP → it prints there and **saves it, with no message**; several → `Several printers were found. Choose one in Setup → Printer.`; otherwise the reason. The alert and the print screen open at the same time.
- **Saved Sunmi / USB printer fails** (out of paper, cover open — Sunmi only —, not attached, permission denied, no answer): alert `Printer not available` / `<reason>` + `Opening the print screen instead.`, and the print screen (today at the same time; target after `OK`). No search.
- **USB without permission:** Android's permission dialog appears during the print; the print waits up to 30 s for an answer.
- **Print screen can't open / PDF can't be built:** a KOT gives the alert `Print Failed`, a bill today `Share Failed` (target `Print Failed` — PRN-D2), with the reason or `Something went wrong. Please try again.`
- **Two taps in a row:** a KOT button ignores taps while busy. Jobs are queued one at a time per printer kind.
- **Saved Wi-Fi port above 65535 (today only):** every print fails as not reachable. Target: such a port can't be saved (PRN-D2).

## Acceptance criteria

1. Given PRN-1 saved and on, when `Print KOT` is tapped, then the KOT prints on PRN-1 and is cut, and no screen or alert appears.
2. Given nothing saved, when `Print KOT` is tapped, then the Android print screen opens with a job named `KOT #<KOT number>`.
3. *(Target, PRN-D2)* Given PRN-1 saved but off, and nothing else on port 9100, when `Print Bill No. <n>` is tapped, then after about 15 s the alert `Printer not reachable` / `Printer not reachable at <PRN-1 ip>:9100.` + `Opening the print screen instead.` shows, and the print screen opens after `OK`.
4. *(Target, PRN-D1)* Given PRN-1 saved but off and PRN-2 on, when `Print KOT` is tapped, then nothing prints until the user taps `Print on <PRN-2 ip>`; then the KOT prints on PRN-2 and PRN-2 is saved with a message.
5. *(Target, PRN-D1)* Given PRN-1 saved, out of paper, and PRN-2 on, when `Print KOT` is tapped, then the alert says `The printer is out of paper.`, nothing prints on PRN-2, and the saved printer stays PRN-1.
6. Given SUNMI-T with the built-in printer saved, when `Print Bill` is tapped, then the bill prints on it in one strip and is cut.

## Platform notes

### POS app

`printPdf.ts` (`printPdf`, `printPdfOverWifi`, `printPdfOnSunmi`, `printPdfOnUsb`, `openPrintDialog`), `useShareKot` (`printKot`), `usePrintBill` (`printBill`), `KOTModal`, `PanelKotBanner`, `PaymentSuccessModal`, `ShareBillActions`, `PrintPastBillButton`.

### Android native

`WifiPrinterModule.printPdf` clears the SDK's cached printer after each job. `SunmiPrinterModule` / `UsbPrinterModule` widen narrow pages and join them into one strip. `PdfPrintModule.printPdf` opens `PrintManager` and resolves as soon as the screen is up.

### POS API, Super-admin panel, Data — MySQL

None.

## Configuration

None.

## Out of scope

- Automatic printing (nothing prints without a tap). Reprinting a KOT later from the kitchen screen. The receipt layout (ORD, BILL).

## Open questions

- None open. Related decisions: PRN-D1, PRN-D2, PRN-D4.
