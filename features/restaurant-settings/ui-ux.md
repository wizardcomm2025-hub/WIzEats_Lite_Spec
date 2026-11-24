# Restaurant profile & settings — UI / UX

Screen text exactly as in the code on 2026-11-06; app strings re-checked in `POS_APP-kot` 2026-11-24 (unchanged). `\n` = line break.

## 1. Profile screen

**Where:**

- **Phone:** menu → `Account` → `Profile`, shown today for every role. *Target (RST-D3):* owner only. The header's profile button also opens it.
- **Tablet:** the `Profile` button at the bottom of the rail, owner only.

| Part | Content |
|---|---|
| Header | `PROFILE MANAGEMENT` (small, coloured) · restaurant name (large) · `Manage your restaurant information` |
| Card 1 `Personal Information` | `Restaurant Name` · `Address` · `Owner Name` · `Role` (greyed, not editable — the signed-in user's role) · `Primary Contact` (greyed, not editable) · `Alternate Contact` · `Restaurant Email` |
| Card 2 `Tax Details` | `GSTIN Number` (max 15, capitals) · `FSSAI ID` · `Tax Rate (%)` · `QR ID` |
| Card 3 `Restaurant Settings` | Switch `Enable Takeaway` · switch `Enable Discount` · switch `Show KOT Number on Bill` with `Prints the order's KOT numbers in the bill header.` · text `KOT Name on Bill` (placeholder `KOT`, max 20, hint `Printed before the number on the bill, e.g. "Token" prints "Token : 12". Blank uses "KOT".`) — only while the KOT switch is on |
| Bottom bar | `Save Changes` — grey and disabled until a field differs from the saved values; a spinner while saving |

Labels are shown in capitals. When Room Service is on (set in the panel), Card 3 also shows an info box: `Modules unlocked` · `Floor Management` · `Room Management`.

### States

| State | What the user sees |
|---|---|
| First load, nothing held yet | Full-screen pulse loader |
| Loaded | Form filled ([01](./01-view-restaurant-profile.md)) |
| Saving | Spinner in `Save Changes`; button disabled |
| Saved | Toast `Profile updated successfully`; form refilled; button grey |
| Invalid GSTIN on Save | Field red with `Enter a valid 15-character GSTIN, e.g. 22AAAAA0000A1Z5`; toast `Please fix the highlighted fields` |
| Blank name / owner on Save | Toast `Restaurant name is required` / `Owner name is required` |
| Server error | Toast with the server's `message`, e.g. `The restaurant email field must be a valid email address.` or `Failed to update restaurant details`. Target (RST-D4): validation messages such as `The tax percent field must be at least 0.` instead of the generic failure |
| No network | Toast `Failed to update restaurant profile` |

Toasts disappear by themselves. The form scrolls above the keyboard; the bottom bar stays.

### Field values shown

| Field | Shows |
|---|---|
| Tax Rate (%) | Stored value as `"5.00"`; `0` when empty |
| KOT Name on Bill | Stored value; new restaurants show `kot` |
| Enable Discount / Show KOT Number on Bill | Stored value; on when the record has none |
| Any 16-byte non-ASCII value | Hex string (finding F2) |

## 2. Effects of settings in other screens

| Setting | Where it shows |
|---|---|
| Restaurant name | Profile header; tablet rail initial; bill header (BILL) |
| Tax Rate | Tax on orders created after the change (ORD / BILL) |
| QR ID | The UPI QR on Bill Payment (BILL) |
| Enable Discount | Off hides the discount section on Bill Payment |
| Show KOT Number on Bill / KOT Name on Bill | Bill header `<name> : <numbers>`, or no KOT numbers when off |
| Enable Takeaway | Home shows the `TAKEAWAY` tile only while on (RST-D1 fixed 2026-11-16) |
| Enable Delivery / Enable Room Service (panel only) | Today Home doesn't change (RST-D5). Target: the `DELIVERY` / `ROOM SERVICE` tile shows only while on |
| Enable Room Service | Owner menu: `Floor Management`, `Room Management` |

Restaurants created before 2026-11-16 show all four tiles until their next Profile save, whatever the settings (target: synced by script — RST-D6).

## 3. Panel — restaurant management page

`/admin/restaurant-management/<id>`. Browser tab title `Analytics`; page title `Category Overview`, subtitle `Category management` (copy as in the code).

| Card | Content |
|---|---|
| `Restaurant Details` | `Restaurant Name` · `Contact Number` · `Owner Name` · `GST Number` · `FSSAI Number` · `Address` (multi-line) · button `Save Restaurant Details` |
| `Configuration` | Checkboxes `Enable Restaurant` · `Enable Delivery` · `Enable KOT` · `Enable Room Service`; inputs with placeholders `Tax %` · `Service Charge %` · `Invoice Prefix` (no labels); button `Save Configuration` |

| Event | What the admin sees |
|---|---|
| Restaurant saved | Browser alert `Restaurant Updated` |
| Configuration saved | Browser alert `Config Updated` |
| Server error, session expired, CSRF expired | Nothing — no alert, no message |
| *Target (RST-D4)* validation refused (422) or unknown restaurant (404) | Nothing — no alert, no message (the page's script is not changed) |

The page doesn't reload after saving. Reload to see the stored values; for example, tax `5` shows as `5.00`.
