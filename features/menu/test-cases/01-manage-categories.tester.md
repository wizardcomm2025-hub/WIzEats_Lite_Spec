# 01 — Manage categories — tester cases

Use case: [01-manage-categories.md](../01-manage-categories.md). Screen text: [ui-ux.md §1](../ui-ux.md#1-category-management-app). Fixtures: [README.md](./README.md#fixtures).

Signed in as OWN-A; Category Management open (phone: menu → `Category Management`; tablet: Setup).

| ID | Covers | Steps | Expected |
|---|---|---|---|
| MENU-01-T01 | AC1 | Look at the list. | `CATEGORY MANAGEMENT` / `Rest A` / `Manage food categories & items`. `Starters` with `Items: 3` (the inactive `Old Dish` counted; today also the deleted `Gone Dish` — `Items: 4` — until MENU-D4 is fixed); `Drinks` with `Items: 1`. |
| MENU-01-T02 | AC2 | Tap `Starters`; tap it again. | Expands: `Paneer Tikka` `₹250.00`, `Chicken 65` `₹300.00`, `Old Dish` `₹100.00`; second tap collapses. |
| MENU-01-T03 | AC3 | `+`; `Desserts`; `Add Category`. Open Item Management's picker. | Toast `Category added`; `Desserts` with `Items: 0`; the picker lists it. |
| MENU-01-T04 | AC4 | `+`; leave the name empty (or only spaces); `Add Category`. | `Category name is required`; nothing sent; the sheet stays. |
| MENU-01-T05 | AC5 (MENU-D2 fixed) | Edit `Starters`; `Starters & Snacks`; `Update Category`. Open the ordering screen. | Toast `Category updated`; the card shows `Starters & Snacks`; so does the ordering screen's category list. |
| MENU-01-T06 | AC6 (MENU-D2 fixed) | Switch `Starters` off; `Deactivate`. Open the ordering screen. Then switch it on again. | Dialog `Deactivate category?` / `Are you sure you want to deactivate "Starters"?`; toast `Category "Starters" deactivated`; card faded; the ordering screen no longer offers `Paneer Tikka` / `Chicken 65`. On again: `Category "Starters" activated`; they are back. |
| MENU-01-T07 | Alt: empty | An owner with no categories opens the screen. | `No Categories Found` / `Create menu categories to organize your food and beverage items.` |
| MENU-01-T08 | AC7; Alt: save failure | (a) `+`; `drinks`; `Add Category`. (b) Airplane on; `+`; `Soups`; `Add Category`. | (a) Toast `Category name already exists`; the sheet stays open; one `Drinks`. (b) Toast `Failed to add category`; the sheet stays open with `Soups`; nothing added. |
| MENU-01-T09 | AC8 — *Fails until MENU-D6 is fixed* | Edit `Starters`. Look at the description field. Change only the name to `Starters 2`; `Update Category`. Edit it again. | The description field shows `Hot starters` both times; after saving, the description is still `Hot starters` (a developer can confirm in the database). (Today: the field is empty and the save clears the description.) |
