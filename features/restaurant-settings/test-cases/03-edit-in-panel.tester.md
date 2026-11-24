# 03 — Edit restaurant and configuration in the panel — tester cases

Use case: [03-edit-in-panel.md](../03-edit-in-panel.md). Screen text: [ui-ux.md §3](../ui-ux.md#3-panel--restaurant-management-page). Fixtures: [README.md](./README.md#fixtures).

Run in a desktop browser, signed in to the panel as ADMIN-1, on REST-A's restaurant management page. Reset REST-A to REST-A-BASE before each case. A second tester holds the app signed in as OWN-A where needed; "app refresh" = background → foreground, then Home.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| RST-03-T01 | AC1 | Open the page. | Tab title `Analytics`; page title `Category Overview` / `Category management` (copy as-is). `Restaurant Details` shows REST-A's name, contact number, owner, GST, FSSAI, address. `Configuration` shows Enable Restaurant, Enable KOT ticked, Enable Delivery and Enable Room Service as stored, Tax `5.00`, Service Charge empty, Invoice Prefix empty. |
| RST-03-T02 | AC2 | Change Restaurant Name to `Rest A Panel`; `Save Restaurant Details`. Reload. App refresh, open Profile. | Alert `Restaurant Updated`; after reload the new name; the app shows `Rest A Panel`. |
| RST-03-T03 | AC7 — *Fails until RST-D4 is fixed* | Clear Owner Name; `Save Restaurant Details`. Reload. App refresh, open Profile. | No alert; after reload the owner name is unchanged; the app shows it too. (Today: alert `Restaurant Updated` and the app's Owner Name is empty.) |
| RST-03-T04 | Alt: contact | Change Contact Number to another valid number; save. App: Profile; then log out and sign in. | Profile's Primary Contact shows the new number. Sign-in still works only with the **old** number. |
| RST-03-T05 | AC3 | Tax `5`, Service Charge `10`, Invoice Prefix `INV`, untick Enable KOT; `Save Configuration`; reload. | Alert `Config Updated`; after reload `5.00`, `10.00`, `INV`, Enable KOT unticked. |
| RST-03-T06 | AC4 | Tax `abc`; `Save Configuration`; reload. | No alert, no message; after reload Tax still `5.00`. |
| RST-03-T07 | Alt: negative — *Fails until RST-D4 is fixed* | Tax `-5`; save; reload. App refresh, open Profile. | No alert; tax still `5.00` in the panel and the app. (Today: alert `Config Updated`; `-5.00` in both.) |
| RST-03-T08 | AC5, AC6 — Home part *Fails until RST-D5 is fixed* | Tick Enable Room Service; save. App refresh; open the menu; go Home. | Alert; app menu shows `Floor Management` and `Room Management` (passes today). Home shows the `ROOM SERVICE` tile (today: not shown for a restaurant registered after 2026-11-16). |
| RST-03-T09 | AC6 — *Fails until RST-D5 is fixed* | Tick Enable Delivery; save. App refresh; look at Home. Untick it; save. App refresh; look at Home. | Ticked: Home shows the `DELIVERY` tile. Unticked: it is gone. (Today: Home doesn't change either time.) |
| RST-03-T10 | AC8 | Open the page; in another tab log out of the panel; back in the first tab change the name and click `Save Restaurant Details`; sign in again and open the page. | No alert after the click; the name is unchanged. |
| RST-03-T11 | Encoding | Restaurant Name NAME-UNI; save; reload. App refresh. | Shows `রেস্তোরাঁ 🍛 Café` exactly in the panel and the app. |
