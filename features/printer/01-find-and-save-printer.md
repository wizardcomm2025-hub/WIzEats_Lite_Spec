# 01 — Find and save a printer

## Summary

On the Printer screen the user picks the printer this device prints to: a Wi-Fi printer (typed IP or `Search printers`), or — on a terminal that has one — its built-in Sunmi or USB printer. From then on, bills and KOTs on this device print straight to it. `Remove printer` forgets it.

## Prerequisites

- **Backend:** none.
- **Data:** device AsyncStorage key `WIFI_PRINTER`; see [data-model.md](./data-model.md)
- **Native:** `WifiPrinter.scanPrinters`, `SunmiPrinter.isAvailable`, `UsbPrinter.isAvailable` / `listPrinters` (Android)

## Actors & preconditions

- **Actor:** owner or waiter, signed in. Chefs have no entry (PRN-D3).
- **Preconditions:** for Wi-Fi, the device is on the same Wi-Fi as the printer (2.4 GHz).

## Main flow

1. The user opens the Printer screen ([README](./README.md#where-the-screen-is)). The app reads the saved printer and checks for a built-in printer (Sunmi first, then USB).
   - **No built-in printer:** no tabs; the card `Wi-Fi receipt printer`. With nothing saved the subtitle reads `Not saved — prints open the Android print screen`; the `Printer IP address` field is empty (placeholder `192.168.0.200`) and `Port` holds `9100`.
   - **Built-in printer:** tabs `Wi-Fi printer` / `Built-in printer` (or `USB printer`). The screen opens on the built-in tab when nothing or a built-in printer is saved.
2. **Wi-Fi — search:** the user taps `Search printers`. The button spins while every address on the network is tried (a few seconds). Each address that answers is listed with `Tap to connect`.
3. Tapping a listed address saves it at once (an explicit choice; any open port is listed — accepted, PRN-D1): the IP field fills in, the subtitle becomes `Printing to <ip>`, the chip reads `Online`, the row reads `Connected`.
4. **Wi-Fi — type:** the user types the IP. Once it's a valid IPv4 address and the IP or port differs from the saved Wi-Fi printer, the warning `Not saved yet. Tap Save printer so bills and KOTs print here.` appears and `Save printer` is enabled.
5. The user taps `Save printer`. The alert `Saved` reads `Bills and KOTs will now print to this printer.`, the subtitle reads `Printing to <ip>`, and the button reads `Saved` and is disabled.
6. **Built-in tab:** Sunmi shows `This terminal has its own receipt printer. No address or Wi-Fi is needed.` USB shows its intro and the list of attached USB printers (name and `<vendor>:<product>`), the first one picked. Until it is saved, the "Not saved yet" warning shows and `Save printer` is enabled. Tapping it: alert `Saved` / `Bills and KOTs will now print on the built-in printer.` (`… on the USB printer.`); subtitle `Printing on the built-in printer` / `Printing on the USB printer`.
7. **Remove:** with anything saved, `Remove printer` (red link) clears it. Today at once, with no confirmation. *Target (PRN-D2):* after `Remove printer?` / `Bills and KOTs will open the print screen until a printer is saved.` → `Remove`. The subtitle returns to `Not saved — …` and the chips disappear; a typed IP stays, so the "Not saved yet" warning shows again.

## Alternative flows

- **Invalid IP** (e.g. `192.168.0`, `300.1.1.1`): the field border turns red; `Save printer`, `Check connection` and `Test print` stay disabled. No alert (accepted).
- **Port:** today blank, `0` or letters → 9100 is used without a message, and a port above 65535 is saved as typed (printing to it then always fails). *Target (PRN-D2):* outside 1–65535 or blank → red field, `Enter a port from 1 to 65535.`, the four buttons disabled.
- **Search finds nothing:** `No printer found on this Wi-Fi. Check it is on and on the same network (2.4 GHz).`
- **No Wi-Fi:** alert `Search failed` / `Connect to a Wi-Fi network first.`
- **Search finds a non-printer:** listed and can be saved (accepted, PRN-D1).
- **Switching kind:** saving on the other tab replaces the saved printer (one per device). With a built-in printer saved, any valid IP on the Wi-Fi tab shows the warning.
- **USB: no printer attached:** `No USB printer found. Plug one in, then tap Check printer.` `Save printer` still saves "the first USB printer".
- **USB: another printer picked:** the chip clears; `Save printer` enabled.
- **Signed out / another user signs in:** the saved printer stays (PRN-D3).
- **Reopening the screen:** the saved printer is shown, with no chip until a check or print.

## Acceptance criteria

1. Given nothing saved, when the user types PRN-1's IP and taps `Save printer`, then the alert `Saved` shows and the subtitle reads `Printing to <PRN-1 ip>`.
2. Given PRN-1 and DECOY on the network, when the user taps `Search printers`, then both addresses are listed.
3. Given a listed address, when the user taps it, then it is saved without a further tap, and the row reads `Connected`.
4. Given the IP `192.168.0`, then the field is red and `Save printer` is disabled.
5. Given a saved printer, when the user signs out and a waiter of REST-B signs in on the same device, then the Printer screen still shows it as saved.
6. *(Target, PRN-D2)* Given the port `70000` or blank, then the port field is red with `Enter a port from 1 to 65535.` and `Save printer` is disabled.
7. *(Target, PRN-D2)* Given a saved printer, when `Remove printer` is tapped and then `Cancel`, then it stays saved.
8. Given SUNMI-T with nothing saved, when the Printer screen opens, then it shows the `Built-in printer` tab selected, and `Save printer` saves it (`Printing on the built-in printer`).
9. Given USB-T with two USB printers, when the user picks the second and saves, then prints go to that one.

## Platform notes

### POS app

`PrinterSettingsScreen`, `printerSettings.ts` (`isValidIp`, `getPrinterSettings`, `savePrinterSettings`, `clearPrinterSettings`), `printPdf.ts` (`scanForPrinters`, `isSunmiPrinterAvailable`, `isUsbPrinterAvailable`, `listUsbPrinters`).

### Android native

`WifiPrinterModule.scanPrinters`: takes the device's first private IPv4 address, tries `.1`–`.254` on 64 threads with a 0.5 s timeout, and returns the addresses that answer, in address order. `SunmiPrinterModule.isAvailable`: binds Sunmi's printer service (3 s) and asks whether it has a printer. `UsbPrinterModule.listPrinters`: every attached device with a printer-class interface.

### POS API, Super-admin panel, Data — MySQL

None.

## Configuration

None. The port default (9100) is in code.

## Out of scope

- More than one printer per device (e.g. separate kitchen and counter printers). Bluetooth printers.

## Open questions

- None open. Related decisions: PRN-D1, PRN-D2, PRN-D3.
