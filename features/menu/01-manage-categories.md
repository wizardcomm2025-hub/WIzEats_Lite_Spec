# 01 — Manage categories

## Summary

The owner opens Category Management and sees each category with its item count. Tapping a category shows its items. The owner can add a category, edit it, or switch it on or off; switching a category switches all its items. Edit and switch work since 2026-11-18 (MENU-D2 fixed). Today an edit from the app clears the category's description (MENU-D6).

## Prerequisites

- **Backend:**
  - `POST /api/item/getItemsCategory`
  - `POST /api/item/categorySave`

  See [api-contract.md](./api-contract.md).
- **Data:** `item_categories`, `menu_items` — [data-model.md](./data-model.md)

## Actors & preconditions

- **Actor:** owner. Today any signed-in user through the API; target owner only for `categorySave` (MENU-D4 / X-D1).
- **Trigger:** menu → `Category Management` (phone), or Setup (tablet).

## Main flow

1. The header shows `CATEGORY MANAGEMENT` / restaurant name / `Manage food categories & items` and a green `+`.
2. The app posts `{}` to `getItemsCategory`. Each card shows the category name and `Items: <n>`, an edit button and a switch. Inactive categories are faded.
3. Tapping a card expands it and lists each item with its name, description and `₹<price>`. With no items it shows `No menu items`.
4. **Add:** `+` opens `Add Category` with the fields `Category name` and `Description (optional)`, and the button `Add Category`. An empty name shows `Category name is required`. The server checks the name is not already used in the restaurant (any case, trimmed) and answers `Item category created successfully`. The sheet closes, the toast says `Category added`, and the category appears with `Items: 0`.
5. **Edit:** `Edit Category` shows the name filled in and, today, the description **empty** (*target, MENU-D6:* the stored description), with an `Update Category` button. The app sends `{ itemCategoryId, itemCategoryName, itemCategoryDescription }`; the server answers `Item category updated successfully`. The sheet closes and the toast says `Category updated`. Today the stored description is cleared by this save.
6. **Switch:** the dialog `Deactivate category?` / `Are you sure you want to deactivate "<name>"?`. On confirm the card switches at once and the app sends `{ itemCategoryId, isActive }`. The server sets the category **and all its items**; the toast says `Category "<name>" deactivated` / `… activated`. The ordering screen stops (or starts) offering those items.

## Alternative flows

- **No categories.** `No Categories Found` / `Create menu categories to organize your food and beverage items.`
- **Duplicate name** (add or rename, any case). 422 `Category name already exists`; the sheet stays open and the toast shows the message. Today a deleted category's name counts too; *target (MENU-D4)* it doesn't.
- **Save fails otherwise** (no network, name over 255 characters). The sheet stays open; the toast shows the server message or `Failed to add category` / `Failed to update category`. *Target (MENU-D4):* a name over 255 characters gives `The item category name field must not be greater than 255 characters.`
- **Switch fails.** The card flips back; the toast shows the server message.
- **Waiter or chef through the API.** Today allowed. *Target:* `categorySave` 403; `getItemsCategory` stays open to waiters (the ordering screen needs it) and 403 for chefs.

## Acceptance criteria

1. Given REST-A with `Starters` (3 items, 1 of them inactive) and `Drinks`, then the list shows `Starters` with `Items: 3`.
2. Given `Starters` is tapped, then its three items show with their prices.
3. Given `Desserts` is added, then the toast `Category added` shows and it appears with `Items: 0`.
4. Given an empty name, then `Category name is required` shows and nothing is sent.
5. Given `Starters` is renamed to `Starters & Snacks`, then the toast `Category updated` shows and the card shows the new name (MENU-D2 fixed).
6. Given `Starters` is switched off and confirmed, then the toast `Category "Starters" deactivated` shows, the card fades, and the ordering screen no longer offers its items (MENU-D2 fixed).
7. Given `drinks` is added, then the toast `Category name already exists` shows and nothing is added.
8. *(Target, MENU-D6)* Given `Starters` has the description `Hot starters`, when it is renamed, then the edit sheet shows `Hot starters` and the description is kept.

## Platform notes

### POS API

`ItemController::itemCategorySave` / `getItemCategory` → `ItemService::saveCategory` / `getItemsCategory`. The duplicate check runs before the update / create.

### Data — MySQL

`item_categories` insert or update; when `isActive` is sent, `menu_items.isActive` for every item of the category.

### POS app

- `MenuCategoryScreen`, `useMenuCategory`, `menuCategory.slice`.
- The sheet closes only when the save succeeds.
- A switch is applied optimistically and reverted on failure.

### Super-admin panel

See [03](./03-menu-in-panel.md).

### Integrations

None.

## Configuration

None.

## Out of scope

- Deleting a category — no screen.

## Open questions

- None open. Related decisions: MENU-D2 (fixed), MENU-D4, MENU-D6.
