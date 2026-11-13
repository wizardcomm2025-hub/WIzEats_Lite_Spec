# 02 — Manage menu items

## Summary

The owner opens Item Management, picks a category, and would see its items, a page at a time, and add, edit or switch them. Today the list is always empty (MENU-D2), and adding an item always fails (MENU-D1). The ordering screen's full menu (`getMenuItems`) works and shows every active item.

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

- **Actor:** owner. Through the API, any signed-in user (MENU-D4).
- **Trigger:** menu → `Item Management` (phone), or Setup (tablet).
- **Preconditions:** at least one category.

## Main flow (as it is today)

1. The header shows a category picker (the first category is selected) and an add button. While loading it shows `Loading menu items...`.
2. The app posts `{ categoryId, menuItemId: "", page: 1, perPage: 10 }` to `showCategoryMenuItems`. The server compares the hex `categoryId` with the binary column and returns `[]` (MENU-D2). The screen shows `No Items Found` / `Add menu items to start building your restaurant's offerings.` Pull-to-refresh gives the same result.
3. **Add:** the sheet `Add Menu Item` has:
   - `Item Name`;
   - `Price` (numeric keypad);
   - `Description (Optional)`;
   - `Veg` / `Non-Veg` (Veg by default);
   - an image box `Tap to upload image`, which opens the camera / gallery chooser and compresses the photo (`Compressing image...`);
   - the button `Save Item` (`Saving...`).

   Empty fields show `Name is required` / `Price is required`.
4. The app sends:

   ```
   {
     name, price: Number(price), description, categoryId,
     isActive: true, isVeg, menuItemId: 0,
     image: <base64 or "">
   }
   ```

   The server answers 422 `The menu item id field must be a string.` (MENU-D1). The toast shows that message inside the sheet, and the sheet stays open with the values.
5. **Edit / switch:** unreachable, since no card is shown. Through the API both answer 500 (MENU-D2).

## Main flow (ordering menu, works)

The ordering screen calls `getMenuItems` with `{ menuItemId: "", page, perPage: 100 }` until it has every page (up to 20 pages). The server returns the restaurant's active, not-deleted items, sorted by `menuItemId`, with `pagination`.

## Alternative flows

- **No categories.** `No Categories Found`.
- **Camera access refused for good.** The dialog `Camera access is off` / `Allow Camera for this app in Settings to take photos.`, with the button `Open Settings`. For photos it reads `Photo access is off`.
- **No camera.** Toast `No camera is available on this device.`
- **Price `abc`.** Sent as `null`. The server still answers with the MENU-D1 message first.
- **An item created through the API.** It shows in the ordering menu and in Category Management's expanded list, but not in Item Management.

## Acceptance criteria

1. Given `Starters` has items, then Item Management shows `No Items Found` for it (MENU-D2).
2. Given `Paneer Tikka`, price `250`, Veg is saved, then the toast `The menu item id field must be a string.` shows, the sheet stays open, and nothing is created (MENU-D1).
3. Given empty name and price, then `Name is required` and `Price is required` show.
4. Given a photo is picked, then `Compressing image...` shows, then the preview and the toast `Image updated successfully`.
5. Given active and inactive items exist, then the ordering screen shows only the active ones.

## Platform notes

### POS API

- `ItemController::menuItemSave` → `ItemService::saveMenuItem`. An `image` is saved first, then the item.
- The create path converts `categoryId` with `hex2bin`; the update path doesn't convert `menuItemId` (MENU-D2).

### Data — MySQL

`menu_items` insert / update. The image is a file plus `imageUrl`.

### POS app

- `MenuItemManagementScreen` (the `AddMenuItemScreen.tsx` file) uses the `menuItem.lists.manage` paged list.
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

- None open. Related decisions: MENU-D1, MENU-D2, MENU-D4.
