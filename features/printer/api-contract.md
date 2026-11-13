# Printer settings — API contract

**No server endpoints.** Printing and the printer setting are entirely on the device. The receipt data comes from endpoints that other features specify (ORD for KOTs, BILL / HIST for bills).

The "contract" here is between the JavaScript code and the Android native modules:

| Module.method | Arguments | Resolves | Rejects (code: message) |
|---|---|---|---|
| `WifiPrinter.checkConnection` | `ip`, `port` | `true` / `false` | never |
| `WifiPrinter.scanPrinters` | `port` | array of IP strings, in address order | `E_NO_WIFI`: `Connect to a Wi-Fi network first.`; `E_SCAN`: the error text or `Search failed.` |
| `WifiPrinter.printPdf` | `ip`, `port`, `filePath` | `null` after printing and cutting | `E_NO_FILE`: `The document to print could not be found.`; `E_UNREACHABLE`: `Printer not reachable at <ip>:<port>.`; `E_CONNECT`: `Could not connect to the printer at <ip>.`; `E_NO_PAPER`: `The printer is out of paper.`; `E_COVER_OPEN`: `The printer cover is open.`; `E_PRINT`: the error text or `Printing failed.` |
| `PdfPrint.printPdf` | `filePath`, `jobName` | `null` as soon as the print screen is up | `E_NO_ACTIVITY`: `The app is not in the foreground.`; `E_NO_FILE`; `E_PRINT`: `Could not open the print dialog.` |

On iOS neither module exists:
- `printPdfOverWifi` throws `Wi-Fi printing is not available.`;
- search returns nothing;
- printing uses the share sheet.
