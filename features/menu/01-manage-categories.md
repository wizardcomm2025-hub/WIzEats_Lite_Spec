# 01 — Manage categories

## Summary

The owner opens Category Management and sees each category with its item count. Tapping a category shows its items. The owner can add a category. Editing a category or switching it on or off fails today (MENU-D2).

## Prerequisites

- **Backend:**
  - `POST /api/item/getItemsCategory`
  - `POST /api/item/categorySave`

  See [api-contract.md](./api-contract.md).
- **Data:** `item_categories`, `menu_items` — [data-model.md](./data-model.md)

## Actors & preconditions

- **Actor:** owner. Through the API, any signed-in user (MENU-D4).
- **Trigger:** menu → `Category Management` (phone), or Setup (tablet).

## Main flow

1. The header shows `CATEGORY MANAGEMENT` / restaurant name / `Manage food categories & items` and a green `+`.
2. The app posts `{}` to `getItemsCategory`. Each card shows the category name and `Items: <n>`, an edit button and a switch. Inactive categories are faded.
3. Tapping a card expands it and lists each item with its name, description and `₹<price>`. With no items it shows `No menu items`.
4. **Add:** `+` opens `Add Category` with the fields `Category name` and `Description (optional)`, and the button `Add Category`. With an empty name nothing happens and no message shows. On submit the sheet closes. The server answers `Item category created successfully`, and the category appears first with `Items: 0`.
5. **Edit:** `Edit Category` shows the name filled in and the description **empty**, with an `Update Category` button. On submit the sheet closes. The server answers 500 (MENU-D2); no message shows and nothing changes.
6. **Switch:** the dialog `Deactivate category?` / `Are you sure you want to deactivate "<name>"?`. On confirm the card switches at once, then flips back when the server answers 500 (MENU-D2). The toast shows `Failed to save item category`.

## Alternative flows

- **No categories.** `No Categories Found` / `Create menu categories to organize your food and beverage items.`
- **Add fails** (no network, name over 255 characters). There is no message, and the category isn't added.
- **Duplicate name.** Allowed (MENU-D4).

## Acceptance criteria

1. Given REST-A with `Starters` (3 items, 1 of them inactive) and `Drinks`, then the list shows `Starters` with `Items: 3`.
2. Given `Starters` is tapped, then its three items show with their prices.
3. Given `Desserts` is added, then it appears first with `Items: 0`.
4. Given an empty name, then tapping `Add Category` does nothing.
5. Given `Starters` is renamed, then the name stays `Starters` and no message shows (MENU-D2).
6. Given `Starters` is switched off and confirmed, then the toast `Failed to save item category` shows and the card is active again (MENU-D2).

## Platform notes

### POS API

`ItemController::itemCategorySave` / `getItemCategory` → `ItemService::saveCategory` / `getItemsCategory`.

### Data — MySQL

`item_categories` insert. The update path is unreachable with hex IDs.

### POS app

- `MenuCategoryScreen`, `useMenuCategory`, `menuCategory.slice`.
- A switch is applied optimistically and reverted on failure.
- `state.error` is shown only through the switch toast.

### Super-admin panel

See [03](./03-menu-in-panel.md).

### Integrations

None.

## Configuration

None.

## Out of scope

- Deleting a category — no screen.

## Open questions

- None open. Related decisions: MENU-D2, MENU-D4.
