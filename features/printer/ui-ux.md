# Printer settings — UI / UX

Texts exactly as shown in the app (2026-11-12).

## 1. Printer screen

| Part | Text / behaviour |
|---|---|
| Title | `Printer` |
| Intro | `Bills and KOTs print straight to this Wi-Fi printer, with no print screen. Without one, the Android print screen opens instead.` |
| Card title | `Wi-Fi receipt printer` |
| Card subtitle | Saved: `Printing to <ip>` (grey). Not saved: `Not saved — prints open the Android print screen` (amber, bold) |
| Status chip | `Online` (green, Wi-Fi icon) / `Not reachable` (red, Wi-Fi-off icon); hidden until a check, test or search pick; cleared when the IP is edited |
| Fields | `Printer IP address` (placeholder `192.168.0.200`, decimal keypad, red border when invalid); `Port` (number keypad, default `9100`) |
| Buttons | `Search printers`; `Check connection`, `Test print` (disabled while the IP is invalid); `Save printer` / `Saved` (primary; enabled only when a valid IP differs from the saved one). All disabled while one runs; the running one spins |
| Search results | One row per address: printer icon, IP, `Tap to connect`, or `Connected` (highlighted) for the saved one. None: `No printer found on this Wi-Fi. Check it is on and on the same network (2.4 GHz).` |
| Unsaved warning | `Not saved yet. Tap Save printer so bills and KOTs print here.` (amber) |
| Remove | `Remove printer` (red link, only when saved; no confirmation) |
| Footer | `The printer and this device must be on the same Wi-Fi network. Print the printer's self-test page to see its IP address.` |

## 2. Alerts

| Trigger | Title | Message |
|---|---|---|
| `Save printer` | `Saved` | `Bills and KOTs will now print to this printer.` |
| `Test print` succeeds on an unsaved IP | `Printer saved` | `Test page sent. Bills and KOTs will now print to this printer.` |
| `Test print` fails | `Test print failed` | the native message ([api-contract.md](./api-contract.md)) or `Could not print.` |
| Search fails | `Search failed` | `Connect to a Wi-Fi network first.` or the error text, or `Could not search the network.` |
| Wi-Fi print fails, several found | `Printer not reachable` | `Several printers were found. Choose one in Setup → Printer.` / `Opening the print screen instead.` |
| Wi-Fi print fails, otherwise | `Printer not reachable` | `<native message>` / `Opening the print screen instead.` |
| KOT print error | `Print Failed` | the error text, or `Something went wrong. Please try again.` |
| Bill print error | `Share Failed` (PRN-D2) | same |
| `Invalid IP` | — | in the code, but never shown (PRN-D2) |

## 3. Print buttons (other features)

| Where | Label |
|---|---|
| KOT card after sending a KOT, phone | `Print KOT` (next to `Share`) |
| KOT banner, tablet | `Print` |
| Payment success | `Print Bill` / `Printing…` |
| Order History card | `Print Bill No. <n>` / `Printing…` |

## 4. Test page

80 mm, centred: `WhizEats Lite` (bold), `Printer test page`, `<ip>`, `<date, time>` (en-IN), `If you can read this, Wi-Fi printing works.`
