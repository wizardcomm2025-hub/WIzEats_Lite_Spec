# Printer settings — data model

Sources: `POS_APP-kot/src/shared/utils/printerSettings.ts`, `printPdf.ts`, `android/.../print/WifiPrinterModule.kt`.

No server data. One device-local value:

| Store | Key | Value | Written by | Cleared by |
|---|---|---|---|---|
| AsyncStorage | `WIFI_PRINTER` | JSON `{ "ip": "<trimmed IPv4>", "port": <number> }` | `Save printer`; tapping a search result; a successful `Test print` of an unsaved IP; the silent switch after a failed print (PRN-D1) | `Remove printer`; uninstalling the app or clearing its data. **Not** sign-out (PRN-D3) |

Reading rules (`getPrinterSettings`):
- a missing key, invalid JSON or an invalid IP counts as "no printer";
- a port of `0` or a non-number reads as 9100.

The IP rule is IPv4 with each part from 0 to 255 (leading zeros allowed by the pattern, e.g. `192.168.000.1`). The port is stored as typed when it is a non-zero number, with no upper limit (PRN-D2).

## Printer constants (native)

| Constant | Value |
|---|---|
| Paper | 576 dots (72 mm printable on an 80 mm roll, 203 dpi) |
| Connect timeout × attempts | 5 s × 3 |
| Search timeout / threads | 0.5 s / 64; addresses `.1`–`.254` of the first private IPv4 network of the device |
| Status codes | `-2` out of paper, `-4` cover open |
| Page length cap (PDF) | 380 pt (about 134 mm); longer receipts are split into equal pages |
