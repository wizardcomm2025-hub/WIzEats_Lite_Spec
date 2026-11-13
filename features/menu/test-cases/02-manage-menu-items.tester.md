# 02 — Manage menu items — tester cases

Use case: [02-manage-menu-items.md](../02-manage-menu-items.md). Screen text: [ui-ux.md §2](../ui-ux.md#2-item-management-app). Fixtures: [README.md](./README.md#fixtures).

Signed in as OWN-A; Item Management open (phone: menu → `Item Management`; tablet: Setup) unless stated.

| ID | Covers | Steps | Expected |
|---|---|---|---|
| MENU-02-T01 | AC1 (accepted MENU-D2) | Look at the screen with `Starters` selected. | `Loading menu items...` then `No Items Found` / `Add menu items to start building your restaurant's offerings.` |
| MENU-02-T02 | AC1 | Pick `Drinks`; pull down to refresh. | Same empty state each time. |
| MENU-02-T03 | AC2 (accepted MENU-D1) | Add; `Paneer Butter Masala`, `280`, a description, `Veg`; `Save Item`. | `Saving...`; toast `The menu item id field must be a string.`; the sheet stays open with the values; nothing new on the ordering screen. |
| MENU-02-T04 | AC3 | Add; `Save Item` with name and price empty. | `Name is required`, `Price is required` in red. |
| MENU-02-T05 | AC4 | In the sheet tap `Tap to upload image`; choose the gallery; pick a large photo. | Chooser sheet; `Compressing image...`; the preview fills the box; toast `Image updated successfully`. |
| MENU-02-T06 | AC4 | Deny camera access permanently in system settings; tap the image box; choose the camera. | Dialog `Camera access is off` / `Allow Camera for this app in Settings to take photos.` with `Open Settings`, which opens the app's settings. |
| MENU-02-T07 | AC5 | Open the ordering screen (Home → Dine In → a table, or Takeaway). | `Paneer Tikka`, `Chicken 65`, `Lime Soda` offered; `Old Dish` (inactive) not offered. |
| MENU-02-T08 | Alt: API-created item | A developer creates `Masala Dosa` in `Starters` through the API. Check Category Management (expand Starters), the ordering screen, Item Management. | Shown in the first two; not in Item Management. |
