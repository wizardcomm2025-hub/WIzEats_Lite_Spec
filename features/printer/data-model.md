# Printer settings — data model

Sources (re-checked 2026-11-24): `POS_APP-kot/src/shared/utils/printerSettings.ts`, `printPdf.ts`, `android/.../print/WifiPrinterModule.kt`, `SunmiPrinterModule.kt`, `UsbPrinterModule.kt`, `ReceiptRaster.kt`.

No server data. One device-local value:

| Store | Key | Value | Written by | Cleared by |
|---|---|---|---|---|
| AsyncStorage | `WIFI_PRINTER` | JSON, one of: `{ "type": "wifi", "ip": "<trimmed IPv4>", "port": <number> }`; `{ "type": "sunmi" }`; `{ "type": "usb", "deviceId": "<vendor>:<product>" }` (or `{ "type": "usb" }` = the first USB printer) | `Save printer` (either tab); tapping a search result; a successful `Test print` of an unsaved printer; today also the silent switch after a failed print (PRN-D1 — target: only after the user taps `Print on <ip>` and it prints) | `Remove printer` (today at once; target after a confirmation — PRN-D2); uninstalling the app or clearing its data. **Not** sign-out (PRN-D3) |

Reading rules (`getPrinterSettings`):
- a missing key or invalid JSON counts as "no printer";
- `type` `sunmi` / `usb` are read as such; a `usb` value without a non-empty `deviceId` means "the first USB printer";
- anything else (including values saved before 2026-11, which have no `type`) is a Wi-Fi printer if `ip` is a valid IPv4, else "no printer";
- a port of `0` or a non-number reads as 9100;
- only Android reads it; iOS ignores it.

The IP rule is IPv4 with each part from 0 to 255 (leading zeros allowed by the pattern, e.g. `192.168.000.1`). The port is stored as typed when it is a non-zero number, with no upper limit (today; target 1–65535 — PRN-D2).

## Printer constants (native)

| Constant | Value |
|---|---|
| Paper | 576 dots (72 mm printable on an 80 mm roll, 203 dpi), all kinds |
| Wi-Fi connect timeout × attempts | 5 s × 3 |
| Wi-Fi search timeout / threads | 0.5 s / 64; addresses `.1`–`.254` of the first private IPv4 network of the device |
| Wi-Fi status codes | `-2` out of paper, `-4` cover open |
| Sunmi | service bind wait 3 s; job wait 30 s; states `1` ready, `4` out of paper, `6` cover open, others offline; 3 lines fed before the cut |
| USB | permission wait 30 s; write timeout 5 s, 16 KB chunks; raster bands of 128 rows; dark below luminance 160; ESC @ … GS V 66 0 (feed and partial cut) |
| Narrow pages (Sunmi, USB) | content narrower than 85 % of the roll is cropped and scaled to about 89 % (at most 1.5×) |
| Page length cap (PDF) | 380 pt (about 134 mm); longer receipts are split into equal pages and joined again for Sunmi / USB |
