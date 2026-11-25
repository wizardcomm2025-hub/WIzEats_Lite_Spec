# Printer settings & printing

Each device can be linked to one **receipt printer** of one of three kinds:

- a **Wi-Fi printer** — an SPRT 80 mm thermal printer on the local network, port 9100 by default;
- the **built-in printer** of a Sunmi terminal (e.g. D3 MINI);
- a **USB printer** — any printer-class USB device: the built-in printer of a generic Android POS terminal, or an external one.

When one is saved, bills and KOTs print straight to it with no print screen. Otherwise, or when it fails, the Android print screen opens. Everything lives on the device: the server is not involved, so this feature has **tester cases only**.

The spec records today's behaviour plus targets (re-synced 2026-11-24 against `POS_APP-kot`). Since 2026-11-12 the app added Sunmi built-in and USB printers (spec'd as they are), and the Wi-Fi module now checks paper and cover. PRN-D1, PRN-D2 and the new PRN-D4 are *To fix*; PRN-D3 stays accepted.

**Feature code:** `PRN` (test IDs `PRN-01-T01`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Find and save a printer](./01-find-and-save-printer.md) | Owner, waiter | [tester](./test-cases/01-find-and-save-printer.tester.md) | — (device only) |
| 02 | [Check the printer and test print](./02-check-and-test-print.md) | Owner, waiter | [tester](./test-cases/02-check-and-test-print.tester.md) | — |
| 03 | [Print a bill or KOT](./03-print-bill-or-kot.md) | Owner, waiter | [tester](./test-cases/03-print-bill-or-kot.tester.md) | — |

## Key concepts

- **Saved printer:** one value in the device's AsyncStorage under `WIFI_PRINTER` (the key name predates the other kinds): `{ type: "wifi", ip, port }`, `{ type: "sunmi" }` or `{ type: "usb", deviceId? }`. It belongs to the device, not to the user, and survives sign-out (PRN-D3).
- **Silent print:** the app turns the receipt into a PDF and renders it as an image 576 dots wide. Wi-Fi: sent through the SPRT SDK after a paper / cover check, then cut. Sunmi: widened if the page came out narrow, joined into one strip, sent to Sunmi's printer service after a paper / cover check, fed 3 lines and cut. USB: widened and joined the same way, sent as ESC/POS raster commands, then a feed-and-cut command (no paper / cover check). One job at a time per kind.
- **Built-in printer detected:** on opening the Printer screen the app asks whether a Sunmi printer service with a printer is present, else whether any printer-class USB device is attached. If so, the screen shows two tabs, `Wi-Fi printer` and `Built-in printer` (Sunmi) or `USB printer`.
- **Reachable:** something accepts a TCP connection on `ip:port`. The app tries 3 times, waiting up to 5 s each time, so an unreachable printer takes about 15 s to fail. Any device with that port open counts as reachable (PRN-D1).
- **Search:** the app tries every address `x.x.x.1`–`254` on the device's own network with a 0.5 s timeout. It lists each address that accepts a connection on the port.
- **Print screen:** Android's own print dialog. The app can't tell whether the user printed or backed out (PRN-D2, accepted). On iOS the share sheet opens instead, and a saved printer is ignored.

## Where the screen is

- **Phone, owner:** drawer → `Account` → `Printer`.
- **Phone, waiter:** drawer → `Operations` → `Printer`.
- **Tablet, owner or waiter:** the `Printer` item at the bottom of the side rail.
- **Chef:** no entry. Chefs don't print (PRN-D3).

Unchanged since 2026-11-12.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `POS_APP-kot/specs/printer/` (created with the first plan)

## Depends on

- **ORD**: the KOT card's `Print KOT` / `Print` button.
- **BILL / HIST**: `Print Bill` and `Print Bill No. <n>` buttons. (The Sunmi / SRS810 customer amount display is part of BILL.)
- **RST**: the KOT label and the restaurant details printed on the receipts.

## Decisions

Statuses per [conventions.md → Decision status](../../conventions.md#decision-status), set 2026-11-24.

| ID | Status | Short | Tests |
|---|---|---|---|
| PRN-D1 | **To fix** | Failed print silently switches to any device found | 03-T06–T08 |
| PRN-D2 | **To fix** (part accepted) | Port unchecked; alert and print screen at once; wrong texts; no remove confirmation | 01-T09, 01-T10, 03-T05, 03-T08, 03-T13, 03-T15 |
| PRN-D3 | Accepted | Setting per device, survives sign-out | 01-T11 |
| PRN-D4 | **To fix** (new) | Test page prints a stray `false` / `true` | 02-T04, 02-T11, 02-T13 |

- **PRN-D1 — To fix (2026-11-24, was accepted 2026-11-12).** *Target:* after a failed Wi-Fi print the app searches the network **only when the printer can't be reached** (`E_UNREACHABLE`, `E_CONNECT`) — not for out of paper, cover open or other errors. It never prints elsewhere or saves anything by itself: if the search finds addresses other than the saved one, the alert `Printer not reachable` / `<reason>` + `Another printer answered on this Wi-Fi.` offers one button `Print on <ip>` per address (at most 3, in address order) and `Open print screen`. Tapping `Print on <ip>` prints there; only if that succeeds is the address saved, with the alert `Printer saved` / `Bills and KOTs will now print to <ip>.` More than 3 addresses: `Several printers were found. Choose one on the Printer screen.` with `Open print screen` only. *Accepted:* `Search printers`, `Check connection` and `Test print` still treat any open port as a printer, and tapping a search result still saves it (an explicit choice).
- **PRN-D2 — To fix (2026-11-24, was accepted 2026-11-12).** *Target:* (a) the port must be a whole number 1–65535: otherwise the field turns red with `Enter a port from 1 to 65535.` under it and `Search printers`, `Check connection`, `Test print` and `Save printer` are disabled; (b) when a silent print fails (Wi-Fi `Printer not reachable`, built-in / USB `Printer not available`), the print screen opens only after the alert's `OK` (or `Open print screen`), not at the same time; (c) the alert text says `Choose one on the Printer screen.`, not `Setup → Printer`; (d) a failed bill **print** is titled `Print Failed` (`Share Failed` stays for sharing); (e) `Remove printer` asks `Remove printer?` / `Bills and KOTs will open the print screen until a printer is saved.` with `Cancel` / `Remove`. *Still accepted:* the unreachable `Invalid IP` alert in the code; the print screen can't report the outcome; a failed Wi-Fi test print shows the chip `Not reachable` whatever the reason; iOS opens the share sheet.
- **PRN-D3 — Accepted (confirmed 2026-11-24).** As 2026-11-12, for all three kinds: the printer is part of the terminal.
- **PRN-D4 — To fix (new 2026-11-24).** The test page builder still inserts its `probe` argument after the layout probe was commented out, so the Wi-Fi test page ends with the word `false` and the built-in / USB test page with `true`. *Target:* nothing after the last line. *Also accepted as they are (new 2026-11-24):* the USB printer has no paper / cover check (a print with no paper "succeeds"); a print waits up to 30 s for the Android USB permission answer; a saved Sunmi / USB printer that is no longer present fails each print with the alert and the print screen.

Original decision texts (history):

- **PRN-D1 (2026-11-12):** Any listening device counts as a printer:
  - when printing to the saved printer fails, the app searches the network once. If exactly **one** address answers and it isn't the saved one, the app prints there and **saves it as the new printer without asking or telling the user**. The search finds any device with the port open, for example the kitchen's printer, which then receives the bills;
  - `Search printers` lists any address that accepts a connection on the port, and `Check connection` shows `Online` for it;
  - tapping a search result saves it straight away, and so does a successful `Test print` of an unsaved address.

  Accepted risk.
- **PRN-D2 (2026-11-12):** Feedback and input checks stay as they are:
  - the port field isn't checked. Blank, `0` or a non-number is used as 9100 without a message. A port above 65535 (e.g. `70000`) is saved, and every print to it then fails as not reachable;
  - the `Invalid IP` alert in the code is never shown: `Save printer` is simply disabled while the IP is invalid, and the field turns red;
  - when a Wi-Fi print fails, the alert `Printer not reachable` appears and the Android print screen opens at the same time;
  - the alert tells the user to "Choose one in Setup → Printer", although the screen isn't under Setup;
  - the print screen can't report whether anything printed;
  - a failed bill print (PDF or print-screen error) is titled `Share Failed`. A failed KOT print is titled `Print Failed`;
  - `Remove printer` asks for no confirmation;
  - iOS has no Wi-Fi printing and opens the share sheet.

  Accepted.
- **PRN-D3 (2026-11-12):** The setting is per device. It survives sign-out and applies to whoever signs in next on that device, even a user of another restaurant. Owners and waiters can change it; chefs have no Printer screen. Accepted.
- **Inherited (2026-11-12):** BILL-D2 (the app's own bill page can't be opened, so `Print Bill` is reached through Order History for orders paid through the API), HIST-D2. *2026-11-24:* BILL-D2 is fixed in the app, so `Print Bill` on the payment success screen is reachable.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Bluetooth printers. Cash-drawer kick. Separate kitchen and counter printers (there is only one saved printer).
- The layout and content of the printed bill and KOT (BILL, ORD); only how they reach the printer is covered here.
