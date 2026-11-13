# Restaurant profile & settings

After onboarding, a restaurant's details (name, address, owner, contacts, tax IDs) and its settings (tax rate, UPI ID for the payment QR, takeaway, discount, KOT number on the bill) can be viewed and changed in two places: the app's **Profile** screen and the super-admin panel's **restaurant management** page. The same details endpoint also feeds Home and the table screens (zones and tables with their running order). The spec records today's behaviour; decisions RST-D1 to RST-D4 keep the current rules as accepted risks.

**Feature code:** `RST` (test IDs `RST-01-T01`, `RST-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [View the restaurant profile](./01-view-restaurant-profile.md) | Any signed-in user (owner in the tablet UI; any role on a phone) | [tester](./test-cases/01-view-restaurant-profile.tester.md) | [developer](./test-cases/01-view-restaurant-profile.developer.md) |
| 02 | [Edit profile and settings in the app](./02-edit-profile-and-settings.md) | Owner (tablet); any role on a phone or through the API (RST-D3) | [tester](./test-cases/02-edit-profile-and-settings.tester.md) | [developer](./test-cases/02-edit-profile-and-settings.developer.md) |
| 03 | [Edit restaurant and configuration in the panel](./03-edit-in-panel.md) | Super admin (web panel) | [tester](./test-cases/03-edit-in-panel.tester.md) | [developer](./test-cases/03-edit-in-panel.developer.md) |

## Key concepts

See the [glossary](../../project-overview.md#glossary) (Restaurant, Restaurant config, Zone, Table, Order channel). Specific to this feature:

- **Profile** — the app screen `ProfileScreen` (menu → Account → Profile on a phone; the `Profile` button at the bottom of the tablet rail).
- **Restaurant details** — the `restaurants` row: name, address, owner name, contact, alternate contact, e-mail, GSTIN, FSSAI ID, logo URL, company address.
- **Settings** — the `restaurant_configs` row: switches (`enableRestaurant`, `enableTakeAway`, `enableDelivery`, `enableRoomService`, `enableKOT`, `enableDiscount`, `showKotInBill`), `taxPercent`, `serviceChargePercent`, `invoicePrefix`, `kotDisplayName`, `qr` (the UPI ID shown as a QR on Bill Payment) and the read-only `restaurantCode`.
- **Tax rate** — copied onto each **new** order when it is created (ORD). Changing it doesn't touch existing orders.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api-dev_guid/specs/restaurant-settings/` (created with the first plan)

## Depends on

- **AUTH** — a signed-in token; AUTH-D2 (a switched-off user or a Pending / Rejected restaurant keeps API access) applies here too.
- **ONB** — creates the restaurant, the config row and the four order channels this feature edits.
- **TBL / ROOM** — zones and tables in the details response; the Room Service switch shows the Floor / Room menus.
- **ORD / BILL** — tax rate, takeaway, discount, KOT-on-bill, KOT name and UPI ID take effect there.
- **ADM** — panel sign-in and navigation.

## Decisions

- **RST-D1 (2026-11-06):** The Takeaway, Delivery and Room Service switches are stored in the settings only. The code tries to switch the matching order channel on or off by the numbers 1 / 2 / 3, but channel IDs are GUIDs, so nothing matches. All four channels stay active and Home always offers Dine In, Takeaway, Room Service and Delivery. (The Room Service switch still shows the Floor / Room menus in the app.) Accepted risk.
- **RST-D2 (2026-11-06):** The app's save keeps its current payload. It always sends `serviceChargePercent` and `companyAddress` as empty, so every app save clears them, including a service charge set in the panel. It also sends `enableKot` (the server only saves `enableKOT`), and the server's rule is spelled `invocePrefix`. As a result, the KOT switch and the invoice prefix can't be changed through the API; only the panel changes them. Accepted.
- **RST-D3 (2026-11-06):** No role check. On a phone, waiters and chefs see Profile in the menu and can save. The API accepts a save from any signed-in user of the restaurant. That includes the UPI ID that customers pay to. Accepted risk.
- **RST-D4 (2026-11-06):** Validation stays as it is:
  - API: tax accepts any value. A negative value is stored; text, or 999.995 and above, gives a 500. The name and owner name can be blanked. Text longer than the column gives a 500. The service charge must have exactly two decimals.
  - Panel: nothing is checked, and a crafted panel request can blank any field.
  
  Accepted risk.
- **Inherited:** AUTH-D2 (access survives switching off), AUTH-D5 (exception text in some 500 responses — here only the details controller's outer catch, which the service never reaches).

## Findings to watch (not decisions)

These are facts from reading the code. Tests record them. If a run shows otherwise, update the spec.

- **F1 — restaurant IDs compared as numbers in the app.** `belongsToSession`, `mergeRestaurant` and `loadRestaurantCache` compare restaurant IDs with `Number(...)`, and a 32-character hex ID turns into `NaN`. If that holds on a device, the app throws away every details response, and Profile and Home never fill. `RST-01-T01` catches it. If it fails, look here first.
- **F2 — 16-byte non-ASCII values come back as hex.** `HasBinaryUuids` turns any 16-byte value that isn't plain printable ASCII into hex, assuming it is a GUID. For example, the name `Café Délice 12` (16 bytes in UTF-8) is returned as `436166c3a92044c3a96c696365203132`. The app shows that hex, and the next Profile save writes it back as the real name. Tested by `RST-01-T07`, `RST-02-T18` and `RST-01-D09`.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Registering the restaurant — ONB.
- Switching the restaurant on / off (`restaurants.status`), approval — ADM / ONB.
- Zones, tables, floors, rooms — TBL / ROOM; order channels' own screens — ORD.
- Importing / exporting menu and zones (the hidden Import section of Profile) — IMPEX.
- Printer settings — PRN.
