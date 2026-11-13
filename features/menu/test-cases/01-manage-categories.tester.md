# 01 — Manage categories — tester cases

Use case: [01-manage-categories.md](../01-manage-categories.md). Screen text: [ui-ux.md §1](../ui-ux.md#1-category-management-app). Fixtures: [README.md](./README.md#fixtures).

Signed in as OWN-A; Category Management open (phone: menu → `Category Management`; tablet: Setup).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| MENU-01-T01 | AC1 | Look at the list. | `CATEGORY MANAGEMENT` / `Rest A` / `Manage food categories & items`. `Starters` with `Items: 3` (the inactive `Old Dish` counted); `Drinks` with `Items: 1`. |
| MENU-01-T02 | AC2 | Tap `Starters`; tap it again. | Expands: `Paneer Tikka` `₹250.00`, `Chicken 65` `₹300.00`, `Old Dish` `₹100.00`; second tap collapses. |
| MENU-01-T03 | AC3 | `+`; `Desserts`; `Add Category`. Open Item Management's picker. | `Desserts` first with `Items: 0`; the picker lists it. |
| MENU-01-T04 | AC4 | `+`; leave the name empty; `Add Category`. | Nothing happens; no message; the sheet stays. |
| MENU-01-T05 | AC5 (accepted MENU-D2) | Edit `Starters`; `Starters & Snacks`; `Update Category`. | Sheet closes; no message; still `Starters`. |
| MENU-01-T06 | AC6 (accepted MENU-D2) | Switch `Starters` off; `Deactivate`. | Dialog `Deactivate category?` / `Are you sure you want to deactivate "Starters"?`; the card fades, then returns; toast `Failed to save item category`; the ordering screen still shows its items. |
| MENU-01-T07 | Alt: empty | An owner with no categories opens the screen. | `No Categories Found` / `Create menu categories to organize your food and beverage items.` |
| MENU-01-T08 | Alt: silent failure / duplicate (accepted MENU-D4) | Airplane on; add `Soups` → then airplane off; add `Drinks`. | Offline: sheet closes, no message, nothing added. `Drinks` again: added; two `Drinks`. |
