# 02 — Manage menu items — tester cases

Use case: [02-manage-menu-items.md](../02-manage-menu-items.md). Screen text: [ui-ux.md §2](../ui-ux.md#2-item-management-app). Fixtures: [README.md](./README.md#fixtures).

Signed in as OWN-A; Item Management open (phone: menu → `Item Management`; tablet: Setup) unless stated.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| MENU-02-T01 | AC1 (MENU-D2 fixed) | Look at the screen with `Starters` selected. | `Loading menu items...`, then `Paneer Tikka`, `Chicken 65`, and `Old Dish` faded; no `Gone Dish`. |
| MENU-02-T02 | AC1 | Pick `Drinks`; pull down to refresh. Pick `Desserts` (no items). | `Lime Soda`; the same after refreshing. `Desserts`: `No Items Found` / `Add menu items to start building your restaurant's offerings.` |
| MENU-02-T03 | AC2 (MENU-D1 fixed) | Add; `Paneer Butter Masala`, `280`, a description, `Veg`; `Save Item`. Open the ordering screen. | `Saving...`; sheet closes; toast `Menu item added`; the item is listed; the ordering screen offers it. |
| MENU-02-T04 | AC3 | Add; `Save Item` with name and price empty. | `Name is required`, `Price is required` in red. |
| MENU-02-T05 | AC4 | In the sheet tap `Tap to upload image`; choose the gallery; pick a large photo; save. | Chooser sheet; `Compressing image...`; the preview fills the box; toast `Image updated successfully`; after saving, the card shows the photo. |
| MENU-02-T06 | AC4 | Deny camera access permanently in system settings; tap the image box; choose the camera. | Dialog `Camera access is off` / `Allow Camera for this app in Settings to take photos.` with `Open Settings`, which opens the app's settings. |
| MENU-02-T07 | AC5 | Open the ordering screen (Home → Dine In → a table, or Takeaway). | `Paneer Tikka`, `Chicken 65`, `Lime Soda` offered; `Old Dish` (inactive) and `Gone Dish` (deleted) not offered. |
| MENU-02-T08 | AC9 | Add; `Free Item`, price `0`; `Save Item`. | Toast `The price field must be greater than 0.`; the sheet stays open with the values; nothing added. |
| MENU-02-T09 | AC6 (MENU-D2 fixed) | Edit `Paneer Tikka`; price `260`; `Update Item`. | `Updating...`; toast `Menu item updated`; the card shows `₹260.00`; so does the ordering screen. |
| MENU-02-T10 | AC7 (MENU-D2 fixed) | Switch `Chicken 65` off; `Deactivate`. Open the ordering screen. Switch it on again. | Toast `Menu item "Chicken 65" deactivated`; card faded; the ordering screen no longer offers it. On again: `Menu item "Chicken 65" activated`; offered again. |
| MENU-02-T11 | AC8 | Add `paneer tikka` to `Starters`; save. Then add `Paneer Tikka` to `Drinks`. | First: toast `Menu item name already exists in this category`; the sheet stays open. Second: `Menu item added`. |
