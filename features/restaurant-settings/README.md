# Restaurant profile & settings

After onboarding, a restaurant's details (name, address, owner, contacts, tax IDs) and its settings (tax rate, UPI ID for the payment QR, takeaway, discount, KOT number on the bill) can be viewed and changed in two places: the app's **Profile** screen and the super-admin panel's **restaurant management** page. The same details endpoint also feeds Home and the table screens (zones and tables with their running order). The spec records today's behaviour plus targets (re-synced 2026-11-24 against API commit d54d6c8 and `POS_APP-kot`): RST-D1 is fixed in code, RST-D2 to RST-D6 are *To fix*.

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
- Implementation: `laravel-pos-api/api/specs/restaurant-settings/` (created with the first plan)

## Depends on

- **AUTH** — a signed-in token; AUTH-D2 (target X-D2: switched-off users and restaurants, and unapproved restaurants, are refused with 403; `showDetails` is exempt from the approval check only) applies here too.
- **ONB** — creates the restaurant, the config row and the four order channels this feature edits (channel `isActive` set from the registration switches since 2026-11-16).
- **TBL / ROOM** — zones and tables in the details response; the Room Service switch shows the Floor / Room menus.
- **ORD / BILL** — tax rate, takeaway, discount, KOT-on-bill, KOT name and UPI ID take effect there.
- **ADM** — panel sign-in and navigation.

## Decisions

Statuses per [conventions.md → Decision status](../../conventions.md#decision-status), set 2026-11-24. Cross-cutting items: [project-overview.md → Known gaps and targets](../../project-overview.md#known-gaps-and-targets).

| ID | Status | Short | Tests |
|---|---|---|---|
| RST-D1 | **Fixed in code 2026-11-16** | Takeaway / Delivery / Room Service switches now switch the order channel | 02-D06, 02-T08 |
| RST-D2 | **To fix** | App save clears service charge and company address; API ignores the KOT switch and invoice prefix | 02-D04, 02-D05, 02-T12 |
| RST-D3 | **To fix** (X-D1) | Any role may save the profile, including the UPI ID | 02-D18, 01-T04, 02-T13 |
| RST-D4 | **To fix** | Loose validation in the API and the panel; 500s instead of 422 / 404 | 02-D08–D11, 02-D16, 02-D25, 03-D03–D08, 03-D10, 03-D15; 02-T07, 02-T16, 03-T03, 03-T07 |
| RST-D5 | **To fix** (new 2026-11-24) | Panel configuration save doesn't switch the order channels | 03-D13; 03-T08, 03-T09 |
| RST-D6 | **To fix** (new 2026-11-24) | Restaurants created before 2026-11-16 have all channels on whatever their settings | 02-D28; 02-T22 |
| AUTH-D2 (inherited) | **To fix** (X-D2) | Switched-off / unapproved callers keep access | 01-D06, 01-D07, 02-D19 |

- **RST-D1 — Fixed in code 2026-11-16.** `updateDetails` now switches the channel by `channelCode` (`TAKEAWAY`, `DELIVERY`, `ROOM_SERVICE`) whenever the matching flag is sent, and `orders/getOrderChannels` returns only active channels, so Home offers exactly the switched-on channels. Dine In is never switched by RST.
- **RST-D2 — To fix (2026-11-24, was accepted 2026-11-06).** *Target:* API — `enableKOT` (`sometimes|boolean`) and `invoicePrefix` (`nullable|string|max:255`) are validated and saved; `enableKot` stays accepted as an alias of `enableKOT` (if both are sent, `enableKOT` wins); `invocePrefix` is no longer a rule (dropped like any unknown field). App — the Profile save **leaves out** `serviceChargePercent` and `companyAddress`, so values set in the panel survive an app save. A field left out is unchanged (as today); a field sent as `null` is still cleared.
- **RST-D3 — To fix (2026-11-24, was accepted 2026-11-06), via [X-D1](../../project-overview.md#known-gaps-and-targets).** *Target:* `updateDetails` is owner-only — a waiter or chef gets 403 `You do not have permission to perform this action`, nothing written. `showDetails` stays open to every role (Home and the table screens need it). The phone menu shows `Profile` to owners only, as the tablet rail already does.
- **RST-D4 — To fix (2026-11-24, was accepted 2026-11-06).** *Target, API* (`RestaurantProfileRequest`, 422 with Laravel's default messages, never 500): `restaurantName`, `restaurantOwnerName` `sometimes|filled|string|max:255`; other text fields `nullable|string|max:255` (`restaurantAddress` `max:65535`, `restaurantGstId` `max:15`, `kotDisplayName` `max:100`); switches `sometimes|boolean` (`null` refused); `taxPercent`, `serviceChargePercent` `nullable|numeric|min:0|max:100`, any number of decimals (stored rounded to 2). *Target, panel* (JSON 422 `{ message, errors }`; the page still shows nothing on failure): details — `restaurantId` required, `restaurantName` and `restaurantOwnerName` required, the same max lengths; configuration — the four switches `required|boolean`, tax and service charge as the API; an unknown `restaurantId` → 404 on all three panel routes (X-D4); a restaurant with no config row gets one created instead of a 500.
- **RST-D5 — To fix (new 2026-11-24).** *Today:* the panel's `Save Configuration` stores `enableDelivery` / `enableRoomService` but leaves `order_channels` alone. Since restaurants now register with Delivery and Room Service channels off and the app has no switch for them, ticking them in the panel shows the Floor / Room menus but Home never offers Room Service or Delivery. *Target:* `updateConfig` also sets `isActive` of the `DELIVERY` and `ROOM_SERVICE` channels, by `channelCode`, in the same way as `updateDetails`, in one transaction with the config write.
- **RST-D6 — To fix (new 2026-11-24).** *Today:* restaurants created before 2026-11-16 have all four channels `isActive = 1` (the old code never switched them), even where the settings say Takeaway / Delivery / Room Service off; each one corrects itself only at the owner's next Profile save. *Target:* a one-off script `db_script/sync_order_channels.sql` sets, for every restaurant, `TAKEAWAY` = `enableTakeAway`, `DELIVERY` = `enableDelivery`, `ROOM_SERVICE` = `enableRoomService`, `DINE_IN` = 1. Safe to run more than once.
- **Inherited:** AUTH-D2 (To fix, X-D2); AUTH-D5 / X-D5 (To fix — here only the details controller's outer catch, which the service never reaches, so no test can trigger it).

Original decision texts (history):

- **RST-D1 (2026-11-06):** The Takeaway, Delivery and Room Service switches are stored in the settings only. The code tries to switch the matching order channel on or off by the numbers 1 / 2 / 3, but channel IDs are GUIDs, so nothing matches. All four channels stay active and Home always offers Dine In, Takeaway, Room Service and Delivery. (The Room Service switch still shows the Floor / Room menus in the app.) Accepted risk.
- **RST-D2 (2026-11-06):** The app's save keeps its current payload. It always sends `serviceChargePercent` and `companyAddress` as empty, so every app save clears them, including a service charge set in the panel. It also sends `enableKot` (the server only saves `enableKOT`), and the server's rule is spelled `invocePrefix`. As a result, the KOT switch and the invoice prefix can't be changed through the API; only the panel changes them. Accepted.
- **RST-D3 (2026-11-06):** No role check. On a phone, waiters and chefs see Profile in the menu and can save. The API accepts a save from any signed-in user of the restaurant. That includes the UPI ID that customers pay to. Accepted risk.
- **RST-D4 (2026-11-06):** Validation stays as it is:
  - API: tax accepts any value. A negative value is stored; text, or 999.995 and above, gives a 500. The name and owner name can be blanked. Text longer than the column gives a 500. The service charge must have exactly two decimals.
  - Panel: nothing is checked, and a crafted panel request can blank any field.
  
  Accepted risk.
- **Inherited (2026-11-06):** AUTH-D2 (access survives switching off), AUTH-D5 (exception text in some 500 responses — here only the details controller's outer catch, which the service never reaches).

## Findings to watch (not decisions)

These are facts from reading the code. Tests record them. If a run shows otherwise, update the spec.

- **F1 — resolved 2026-11-24.** The app (`restaurantSync.ts`) now compares restaurant IDs with `String(...)`; `belongsToSession` no longer exists. `RST-01-T01` still covers it. Original text:
  **F1 — restaurant IDs compared as numbers in the app.** `belongsToSession`, `mergeRestaurant` and `loadRestaurantCache` compare restaurant IDs with `Number(...)`, and a 32-character hex ID turns into `NaN`. If that holds on a device, the app throws away every details response, and Profile and Home never fill. `RST-01-T01` catches it. If it fails, look here first.
- **F2 — 16-byte non-ASCII values come back as hex.** `HasBinaryUuids` turns any 16-byte value that isn't plain printable ASCII into hex, assuming it is a GUID. For example, the name `Café Délice 12` (16 bytes in UTF-8) is returned as `436166c3a92044c3a96c696365203132`. The app shows that hex, and the next Profile save writes it back as the real name. Tested by `RST-01-T07`, `RST-02-T18` and `RST-01-D09`.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Registering the restaurant — ONB.
- Switching the restaurant on / off (`restaurants.status`), approval — ADM / ONB.
- Zones, tables, floors, rooms — TBL / ROOM; order channels' own screens — ORD.
- Importing / exporting menu and zones (the hidden Import section of Profile) — IMPEX.
- Printer settings — PRN.
