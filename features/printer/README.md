# Printer settings & printing

Each device can be linked to one **Wi-Fi receipt printer**: an SPRT 80 mm thermal printer on the local network, port 9100 by default. When one is saved, bills and KOTs print straight to it with no print screen. Otherwise, or when it can't be reached, the Android print screen opens. Everything lives on the device: the server is not involved, so this feature has **tester cases only**.

The spec records today's behaviour. Decisions PRN-D1 to PRN-D3 keep the current rules as accepted risks.

**Feature code:** `PRN` (test IDs `PRN-01-T01`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Find and save a printer](./01-find-and-save-printer.md) | Owner, waiter | [tester](./test-cases/01-find-and-save-printer.tester.md) | — (device only) |
| 02 | [Check the connection and test print](./02-check-and-test-print.md) | Owner, waiter | [tester](./test-cases/02-check-and-test-print.tester.md) | — |
| 03 | [Print a bill or KOT](./03-print-bill-or-kot.md) | Owner, waiter | [tester](./test-cases/03-print-bill-or-kot.tester.md) | — |

## Key concepts

- **Saved printer:** `{ ip, port }` in the device's AsyncStorage under `WIFI_PRINTER`. It belongs to the device, not to the user, and survives sign-out (PRN-D3).
- **Wi-Fi print:** the app turns the receipt into a PDF, renders it as an image 576 dots wide, sends it to the printer and cuts the paper. One job runs at a time.
- **Reachable:** something accepts a TCP connection on `ip:port`. The app tries 3 times, waiting up to 5 s each time, so an unreachable printer takes about 15 s to fail. Any device with that port open counts as reachable (PRN-D1).
- **Search:** the app tries every address `x.x.x.1`–`254` on the device's own network with a 0.5 s timeout. It lists each address that accepts a connection on the port.
- **Print screen:** Android's own print dialog. The app can't tell whether the user printed or backed out (PRN-D2). On iOS the share sheet opens instead.

## Where the screen is

- **Phone, owner:** drawer → `Account` → `Printer`.
- **Phone, waiter:** drawer → `Operations` → `Printer`.
- **Tablet, owner or waiter:** the `Printer` item at the bottom of the side rail.
- **Chef:** no entry. Chefs don't print (PRN-D3).

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `POS_APP-kot/specs/printer/` (created with the first plan)

## Depends on

- **ORD**: the KOT card's `Print KOT` / `Print` button.
- **BILL / HIST**: `Print Bill` and `Print Bill No. <n>` buttons.
- **RST**: the KOT label and the restaurant details printed on the receipts.

## Decisions

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
- **Inherited:** BILL-D2 (the app's own bill page can't be opened, so `Print Bill` is reached through Order History for orders paid through the API), HIST-D2.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Bluetooth or USB printers. Cash-drawer kick. Separate kitchen and counter printers (there is only one saved printer).
- The layout and content of the printed bill and KOT (BILL, ORD); only how they reach the printer is covered here.
