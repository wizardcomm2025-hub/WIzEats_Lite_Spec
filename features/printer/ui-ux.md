# Printer settings — UI / UX

Texts exactly as shown in the app (re-checked 2026-11-24). *Target* marks text from PRN-D1, PRN-D2 or PRN-D4 that the app doesn't show yet.

## 1. Printer screen

| Part | Text / behaviour |
|---|---|
| Title | `Printer` |
| Intro | `Bills and KOTs print straight to the saved printer, with no print screen. Without one, the Android print screen opens instead.` |
| Tabs | Only when a built-in printer is detected: `Wi-Fi printer` / `Built-in printer` (Sunmi) or `USB printer`. Opens on the built-in tab when a Sunmi / USB printer is saved, or nothing is saved; otherwise on `Wi-Fi printer` |
| Card title | `Wi-Fi receipt printer` / `Built-in printer (Sunmi)` / `USB receipt printer` (by tab) |
| Card subtitle (by what is saved, on either tab) | `Printing to <ip>`, `Printing on the built-in printer`, `Printing on the USB printer` (grey); nothing saved: `Not saved — prints open the Android print screen` (amber, bold) |
| Status chip, Wi-Fi tab | `Online` (green, Wi-Fi icon) / `Not reachable` (red, Wi-Fi-off icon); hidden until a check, test or search pick; cleared when the IP is edited |
| Status chip, built-in tab | `Ready` (green) / `Out of paper`, `Cover open`, `Permission needed`, `Not available` (red); printer icon; hidden until a check or test; cleared when another USB printer is picked |
| Footer | Wi-Fi tab: `The printer and this device must be on the same Wi-Fi network. Print the printer's self-test page to see its IP address.` Built-in tab: `Make sure the paper roll is in and the cover is closed.` |
| Remove | `Remove printer` (red link, whenever anything is saved, on either tab). Today: no confirmation. *Target (PRN-D2):* alert `Remove printer?` / `Bills and KOTs will open the print screen until a printer is saved.` — `Cancel` / `Remove` |

### 1a. Wi-Fi tab

| Part | Text / behaviour |
|---|---|
| Fields | `Printer IP address` (placeholder `192.168.0.200`, decimal keypad, red border when invalid); `Port` (number keypad, default `9100`). *Target (PRN-D2):* port outside 1–65535 or blank → red border and `Enter a port from 1 to 65535.` |
| Buttons | `Search printers`; `Check connection`, `Test print` (disabled while the IP is invalid); `Save printer` / `Saved` (primary; enabled only when a valid IP or port differs from the saved Wi-Fi printer — so with a built-in printer saved, any valid IP enables it and saving replaces the built-in printer). All disabled while one runs; the running one spins |
| Search results | One row per address: printer icon, IP, `Tap to connect`, or `Connected` (highlighted) for the saved one. None: `No printer found on this Wi-Fi. Check it is on and on the same network (2.4 GHz).` |
| Unsaved warning | `Not saved yet. Tap Save printer so bills and KOTs print here.` (amber) |

### 1b. Built-in tab

| Part | Text / behaviour |
|---|---|
| Intro | Sunmi: `This terminal has its own receipt printer. No address or Wi-Fi is needed.` USB: `Pick a receipt printer connected by USB, built-in or external. No address or Wi-Fi is needed. If Android asks to allow the USB printer, tap OK.` |
| USB list (USB only) | Label `USB printer`; one row per attached printer: product name and `<vendor>:<product>`; the picked one highlighted (the first until the user picks, or the saved one). None: `No USB printer found. Plug one in, then tap Check printer.` (amber) |
| Buttons | `Check printer`, `Test print` (both outline); `Save printer` / `Saved` (enabled when the built-in printer, or the picked USB printer, isn't the saved one). All disabled while one runs |
| Unsaved warning | `Not saved yet. Tap Save printer so bills and KOTs print here.` |
| Android permission | USB only: Android's own USB permission dialog (allow / deny) on the first check, test or print for that printer |

## 2. Alerts

| Trigger | Title | Message |
|---|---|---|
| `Save printer`, Wi-Fi | `Saved` | `Bills and KOTs will now print to this printer.` |
| `Save printer`, built-in | `Saved` | `Bills and KOTs will now print on the built-in printer.` / `… on the USB printer.` |
| `Test print` succeeds on an unsaved printer | `Printer saved` | `Test page sent. Bills and KOTs will now print to this printer.` / `Test page sent. Bills and KOTs will now print on the built-in printer.` (`USB printer`) |
| `Test print` fails | `Test print failed` | the native message ([api-contract.md](./api-contract.md)) or `Could not print.` |
| Search fails | `Search failed` | `Connect to a Wi-Fi network first.` or the error text, or `Could not search the network.` |
| Wi-Fi print fails, several found (today) | `Printer not reachable` | `Several printers were found. Choose one in Setup → Printer.` / `Opening the print screen instead.` — *target:* `… Choose one on the Printer screen.` (PRN-D2) |
| Wi-Fi print fails, otherwise | `Printer not reachable` | `<native message>` / `Opening the print screen instead.` |
| *Target (PRN-D1):* Wi-Fi unreachable, 1–3 other addresses found | `Printer not reachable` | `<native message>` / `Another printer answered on this Wi-Fi.` — buttons `Print on <ip>` (each) and `Open print screen` |
| *Target (PRN-D1):* printed on a chosen address | `Printer saved` | `Bills and KOTs will now print to <ip>.` |
| Built-in / USB print fails | `Printer not available` | `<native message>` / `Opening the print screen instead.` |
| KOT print error | `Print Failed` | the error text, or `Something went wrong. Please try again.` |
| Bill print error | `Share Failed` today (PRN-D2); *target* `Print Failed` | same |
| `Invalid IP` | — | in the code, but never shown (accepted) |

Today each `Printer not reachable` / `Printer not available` alert and the print screen open at the same time; *target (PRN-D2):* the print screen opens after `OK` / `Open print screen`.

## 3. Print buttons (other features)

| Where | Label |
|---|---|
| KOT card after sending a KOT, phone | `Print KOT` (next to `Share`) |
| KOT banner, tablet | `Print` |
| Payment success | `Print Bill` / `Printing…` |
| Order History card | `Print Bill No. <n>` / `Printing…` |

## 4. Test page

80 mm, centred: `WhizEats Lite` (bold), `Printer test page`, then `<ip>` / `Built-in printer` / `USB printer`, `<date, time>` (en-IN), `If you can read this, Wi-Fi printing works.` / `… built-in printing works.` / `… USB printing works.` Today a last line follows: `false` (Wi-Fi) or `true` (built-in / USB) — *target:* nothing (PRN-D4).
