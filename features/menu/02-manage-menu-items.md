# 02 — Manage menu items

## Summary

The owner opens Item Management, picks a category, and sees its items, a page at a time. They can add an item, edit it, or switch it on or off. Since the app fix (MENU-D1) and the server changes of 2026-11-05 / 2026-11-18 (MENU-D2, part of MENU-D4) all of this works, and the server checks the price, the category and duplicate names. The ordering screen loads the full active menu.

## Prerequisites

- **Backend:**
  - `POST /api/item/showCategoryMenuItems`
  - `POST /api/item/getMenuItems`
  - `POST /api/item/menuItemSave`
  - `POST /api/item/getItemsCategory`

  See [api-contract.md](./api-contract.md).
- **Data:** `menu_items`, `item_categories`; image files under `storage/app/public/image/`
- **Depends on:** [01](./01-manage-categories.md)

## Actors & preconditions

- **Actor:** owner. Today any signed-in user through the API; target per [api-contract.md](./api-contract.md#endpoints) (MENU-D4 / X-D1).
- **Trigger:** menu → `Item Management` (phone), or Setup (tablet).
- **Preconditions:** at least one category.

## Main flow

1. The header shows a category picker (the first category is selected) and an add button. While loading it shows `Loading menu items...`.
2. The app posts `{ categoryId, menuItemId: "", page: 1, perPage: 10 }` to `showCategoryMenuItems` and shows the category's items (active and inactive, oldest first). Scrolling to the end loads the next page; pull-to-refresh reloads page 1. A category without items shows `No Items Found` / `Add menu items to start building your restaurant's offerings.`
3. **Add:** the sheet `Add Menu Item` has `Item Name`; `Price` (numeric keypad); `Description (Optional)`; `Veg` / `Non-Veg` (Veg by default); an image box `Tap to upload image`, which opens the camera / gallery chooser and compresses the photo (`Compressing image...`); and `Save Item` (`Saving...`). Empty fields show `Name is required` / `Price is required`.
4. The app sends `{ name, price: Number(price), description, categoryId, isActive: true, isVeg, menuItemId: "", image: <base64 or ""> }` (MENU-D1 fixed).
5. The server validates (price > 0), checks the category belongs to the restaurant and isn't deleted, trims the name, refuses a duplicate name in the category, saves the image (if any) and creates the item: `Menu item created successfully`.
6. The sheet closes, the toast says `Menu item added`, and the item appears in the list.
7. **Edit:** the edit button opens `Edit Menu Item` filled with the item's values and its image; `Update Item` (`Updating...`). The app sends the same fields with the item's `menuItemId` and its current `isActive`; `image` only if a new one was picked. Success: toast `Menu item updated`.
8. **Switch:** the dialog `Deactivate menu item?` / `Activate menu item?`; the app sends `{ menuItemId, isActive }`; toast `Menu item "<name>" deactivated` / `… activated`; the card fades or brightens.

## Main flow (ordering menu)

The ordering screen calls `getMenuItems` with `{ menuItemId: "", page, perPage: 100 }` until it has every page (up to 20 pages). The server returns the restaurant's active, not-deleted items, ordered by `created_at`, with `pagination`. The app also hides items whose category is inactive.

## Alternative flows

- **No categories.** `No Categories Found`.
- **Price `0` or negative.** 422 `The price field must be greater than 0.`; toast; the sheet stays open.
- **Duplicate name in the category** (any case, trimmed, not deleted). 422 `Menu item name already exists in this category`; toast; the sheet stays open. The same name in another category is allowed.
- **Name over 255 characters, or price 100 000 000 or more.** Today 500 `Failed to save menu item`. *Target (MENU-D4):* 422 with the field's message.
- **Bad image** (not PNG / JPEG, or too large). Today 500 `Failed to save menu item` (or, for a non-image file with a png/jpg label, saved). *Target (MENU-D5):* 422 `The image must be a PNG or JPEG of at most 2 MB.`
- **Camera access refused for good.** The dialog `Camera access is off` / `Allow Camera for this app in Settings to take photos.`, with the button `Open Settings`. For photos it reads `Photo access is off`.
- **No camera.** Toast `No camera is available on this device.`
- **Waiter or chef through the API.** Today allowed. *Target:* `menuItemSave` and `showCategoryMenuItems` 403; `getMenuItems` open to waiters, 403 for chefs.

## Acceptance criteria

1. Given `Starters` has `Paneer Tikka`, `Chicken 65` and the inactive `Old Dish`, then Item Management lists all three, `Old Dish` faded (MENU-D2 fixed).
2. Given `Paneer Butter Masala`, price `280`, Veg is saved, then the toast `Menu item added` shows and it is listed (MENU-D1 fixed).
3. Given empty name and price, then `Name is required` and `Price is required` show.
4. Given a photo is picked, then `Compressing image...` shows, then the preview and the toast `Image updated successfully`.
5. Given active and inactive items exist, then the ordering screen shows only the active ones of active categories.
6. Given `Paneer Tikka` is edited to `260`, then the toast `Menu item updated` shows and the card shows `₹260.00`.
7. Given `Chicken 65` is switched off, then the toast `Menu item "Chicken 65" deactivated` shows and the ordering screen no longer offers it.
8. Given `paneer tikka` is added to `Starters`, then the toast `Menu item name already exists in this category` shows and nothing is added.
9. Given price `0`, then the toast `The price field must be greater than 0.` shows and nothing is saved.

## Platform notes

### POS API

- `ItemController::menuItemSave` → `ItemService::saveMenuItem` (rewritten 2026-11-18; the old version remains as `saveMenuItem_old`, unused). Checks run before the image is saved.

### Data — MySQL

`menu_items` insert / update. The image is a file plus `imageUrl`.

### POS app

- `MenuItemManagementScreen` (the `AddMenuItemScreen.tsx` file) uses the `menuItem.lists.manage` paged list (`PER_PAGE = 10`); IDs are compared as strings.
- `fetchFullMenu` loads the menu for ordering.
- `compressImageToBase64` prepares the image.

### Super-admin panel

See [03](./03-menu-in-panel.md).

### Integrations

None. Images are stored on the API server's `public` disk.

## Configuration

`FULL_MENU_PAGE_SIZE = 100`, `FULL_MENU_MAX_PAGES = 20` (app). The server caps `perPage` at 100.

## Out of scope

- Ordering screens — ORD.

## Open questions

- None open. Related decisions: MENU-D1 (fixed), MENU-D2 (fixed), MENU-D4, MENU-D5.
