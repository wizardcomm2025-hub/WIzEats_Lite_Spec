# 01 — Find and save a printer

## Summary

On the Printer screen the user types the printer's IP address or finds it with `Search printers`, then saves it. From then on, bills and KOTs on this device print straight to it. `Remove printer` forgets it.

## Prerequisites

- **Backend:** none.
- **Data:** device AsyncStorage key `WIFI_PRINTER`; see [data-model.md](./data-model.md)
- **Native:** `WifiPrinter.scanPrinters` (Android)

## Actors & preconditions

- **Actor:** owner or waiter, signed in. Chefs have no entry (PRN-D3).
- **Preconditions:** the device is on the same Wi-Fi as the printer (2.4 GHz).

## Main flow

1. The user opens the Printer screen ([README](./README.md#where-the-screen-is)). It shows the title `Printer` and the card `Wi-Fi receipt printer`. With nothing saved, the card's subtitle reads `Not saved — prints open the Android print screen`. The `Printer IP address` field is empty (placeholder `192.168.0.200`) and `Port` holds `9100`.
2. **Search:** the user taps `Search printers`. The button spins while every address on the network is tried (a few seconds). Each address that answers is listed with `Tap to connect`.
3. Tapping a listed address saves it at once (PRN-D1):
   - the IP field fills in;
   - the subtitle becomes `Printing to <ip>`;
   - the chip reads `Online`;
   - the row reads `Connected`.
4. **Or type:** the user types the IP. Once it's a valid IPv4 address (and differs from the saved one):
   - the warning `Not saved yet. Tap Save printer so bills and KOTs print here.` appears;
   - `Save printer` is enabled.
5. The user taps `Save printer`. The alert `Saved` reads `Bills and KOTs will now print to this printer.`, the subtitle reads `Printing to <ip>`, and the button reads `Saved` and is disabled.
6. **Remove:** with a printer saved, `Remove printer` (red link) clears it at once, with no confirmation. The subtitle returns to `Not saved — …`, the chip disappears, and the IP stays in the field, so the "Not saved yet" warning shows again.

## Alternative flows

- **Invalid IP** (e.g. `192.168.0`, `300.1.1.1`): the field border turns red; `Save printer`, `Check connection` and `Test print` stay disabled. No alert (PRN-D2).
- **Search finds nothing:** `No printer found on this Wi-Fi. Check it is on and on the same network (2.4 GHz).`
- **No Wi-Fi:** alert `Search failed` / `Connect to a Wi-Fi network first.`
- **Search finds a non-printer:** any device with the port open is listed and can be saved (PRN-D1).
- **Port:** blank, `0` or letters → 9100 is used without a message. A port above 65535 is saved as typed, and printing to it always fails (PRN-D2).
- **Signed out / another user signs in:** the saved printer stays (PRN-D3).
- **Reopening the screen:** the saved IP and port are shown, with no chip until a check or print.

## Acceptance criteria

1. Given nothing saved, when the user types PRN-1's IP and taps `Save printer`, then the alert `Saved` shows and the subtitle reads `Printing to <PRN-1 ip>`.
2. Given PRN-1 and DECOY on the network, when the user taps `Search printers`, then both addresses are listed.
3. Given a listed address, when the user taps it, then it is saved without a further tap, and the row reads `Connected`.
4. Given the IP `192.168.0`, then the field is red and `Save printer` is disabled.
5. Given a saved printer, when the user signs out and a waiter of REST-B signs in on the same device, then the Printer screen still shows `Printing to <ip>`.

## Platform notes

### POS app

`PrinterSettingsScreen`, `printerSettings.ts` (`isValidIp`, `savePrinterSettings`, `clearPrinterSettings`), `printPdf.ts` (`scanForPrinters`).

### Android native

`WifiPrinterModule.scanPrinters`: takes the device's first private IPv4 address, tries `.1`–`.254` on 64 threads with a 0.5 s timeout, and returns the addresses that answer, in address order.

### POS API, Super-admin panel, Data — MySQL

None.

## Configuration

None. The port default (9100) is in code.

## Out of scope

- More than one printer per device.

## Open questions

- None open.
