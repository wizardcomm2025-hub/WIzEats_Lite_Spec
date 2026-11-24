# 03 — Menu in the panel — tester cases

Use case: [03-menu-in-panel.md](../03-menu-in-panel.md). Screen text: [ui-ux.md §3](../ui-ux.md#3-panel-pages). Fixtures: [README.md](./README.md#fixtures).

Desktop browser, signed in to the panel as ADMIN-1, on REST-A's Categories or Items page. A second tester holds the app as OWN-A where needed.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| MENU-03-T01 | AC1 | Open the Categories page. | Tab `Analytics`; `Category Overview` / `Category management`; `Starters`, `Drinks` with their switches on. |
| MENU-03-T02 | AC2 | `+ Add Category`; `Soups`, Active; Save. App: reopen Category Management. | Page reloads with `Soups`; the app lists `Soups`. |
| MENU-03-T03 | AC3 — *Fails until MENU-D3 is fixed* | Switch `Starters` off; reload. App: ordering screen. | Off after reload; the ordering screen no longer offers `Paneer Tikka` and `Chicken 65` (the app hides items of an inactive category today too); Item Management shows them faded (today: still active). |
| MENU-03-T04 | AC4 | Open the Items page. | `Items Overview` / `Items management`; rows with category, item name and price for ITEM-1–4. |
| MENU-03-T05 | AC5 — *Fails until MENU-D3 is fixed* | `+ Add Item`; `Starters`, `Samosa`, `40`, Active; Save. Then edit `Paneer Tikka`'s price to `260`; Update. App: Item Management. | Both saved; after the reload `Samosa` is listed and `Paneer Tikka` shows `260.00`; the app shows both changes. (Today: alert `Something went wrong` each time, nothing changes.) |
| MENU-03-T06 | AC6 — *Fails until MENU-D3 is fixed* | Switch `Lime Soda` off; reload. App: ordering screen. | Off after reload; the ordering screen no longer offers `Lime Soda`. (Today: on again after reload; still offered.) |
| MENU-03-T07 | Alt: empty name | `+ Add Category` with an empty name; Save. | No reload, no message; nothing added. |
