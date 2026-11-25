# Printer settings — API contract

**No server endpoints.** Printing and the printer setting are entirely on the device. The receipt data comes from endpoints that other features specify (ORD for KOTs, BILL / HIST for bills).

The "contract" here is between the JavaScript code and the Android native modules (re-checked 2026-11-24). Every method runs on that module's single job thread.

## Wi-Fi — `WifiPrinter`

| Method | Arguments | Resolves | Rejects (code: message) |
|---|---|---|---|
| `checkConnection` | `ip`, `port` | `true` / `false` | never |
| `scanPrinters` | `port` | array of IP strings, in address order | `E_NO_WIFI`: `Connect to a Wi-Fi network first.`; `E_SCAN`: the error text or `Search failed.` |
| `printPdf` | `ip`, `port`, `filePath` | `null` after printing and cutting | `E_NO_FILE`: `The document to print could not be found.`; `E_UNREACHABLE`: `Printer not reachable at <ip>:<port>.`; `E_CONNECT`: `Could not connect to the printer at <ip>.`; `E_NO_PAPER`: `The printer is out of paper.`; `E_COVER_OPEN`: `The printer cover is open.`; `E_PRINT`: the error text or `Printing failed.` |

## Sunmi built-in — `SunmiPrinter`

| Method | Arguments | Resolves | Rejects |
|---|---|---|---|
| `isAvailable` | — | `true` when Sunmi's printer service binds within 3 s and reports a printer | never |
| `checkStatus` | — | `ready` / `no_paper` / `cover_open` / `offline` | never |
| `printPdf` | `filePath` | `null` when the printer reports the job done, or accepted it (some firmware never confirms) | `E_NO_FILE`; `E_NO_PRINTER`: `The built-in printer is not available on this device.`; `E_NO_PAPER`; `E_COVER_OPEN`; `E_OFFLINE`: `The built-in printer is not ready.`; `E_PRINT`: the printer's message, `The printer rejected the job.`, `The printer reported an error (<code>).`, `Could not prepare the receipt.` or `Printing failed.`; `E_TIMEOUT`: `The printer did not respond.` (30 s) |

## USB — `UsbPrinter`

| Method | Arguments | Resolves | Rejects |
|---|---|---|---|
| `isAvailable` | — | `true` when any attached USB device has a printer-class interface | never |
| `listPrinters` | — | `[ { id: "<vendorId>:<productId>", name: <product name or "USB printer"> } ]` | never |
| `checkStatus` | `deviceId` or `null` | `ready` (permission held or granted) / `no_permission` / `offline` (not attached). Asks Android for permission if needed and waits up to 30 s | never |
| `printPdf` | `filePath`, `deviceId` or `null` (= the first printer) | `null` after the data is written (no paper / cover check — PRN-D4 accepted) | `E_NO_FILE`; `E_NO_PRINTER`: `No USB printer was found on this device.` / `The selected USB printer is not connected.`; `E_NO_PERMISSION`: `Allow this app to use the USB printer and try again.`; `E_PRINT`: `The USB printer has no data endpoint.`, `Could not open the USB printer.`, `The USB printer is in use by another app.`, `The USB printer stopped responding.`, `Could not prepare the receipt.` or `Printing failed.` |

## Print screen — `PdfPrint`

| Method | Arguments | Resolves | Rejects |
|---|---|---|---|
| `printPdf` | `filePath`, `jobName` | `null` as soon as the print screen is up | `E_NO_ACTIVITY`: `The app is not in the foreground.`; `E_NO_FILE`; `E_PRINT`: `Could not open the print dialog.` |

On iOS none of these modules exist:
- `printPdfOverWifi` throws `Wi-Fi printing is not available.`;
- search returns nothing; no built-in tab appears;
- printing ignores any saved printer and uses the share sheet.
