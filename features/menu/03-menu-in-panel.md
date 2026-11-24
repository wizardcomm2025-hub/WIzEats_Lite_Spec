# 03 — Menu in the panel

## Summary

A super admin manages a restaurant's categories and items on two panel pages. The Categories page works. On the Items page, the list is the only part that works today (MENU-D3, *To fix* — target: add, edit and the switch work, with the API's rules).

## Prerequisites

- **Backend (web):**
  - Categories: `GET /admin/categories-management/{restaurantId}`; `POST /admin/categories/store`, `/categories/update`, `/categories/toggle`.
  - Items: `GET /admin/items-management/{restaurantId}`; `POST /admin/items/store`, `/items/update`, `/admin/item-toggle-status`.

  See [api-contract.md](./api-contract.md#panel-routes-web).
- **Depends on:** ADM

## Actors & preconditions

- **Actor:** super admin signed in to the panel.

## Main flow — Categories page

1. The page shows:
   - the tab title `Analytics`, and the heading `Category Overview` / `Category management`;
   - a `+ Add Category` button;
   - a table with the columns `SL No`, `Category Name`, `Visible` (switch) and `Actions` (edit). It shows 10 per page and includes deleted categories.
2. **Add:** the `Add Category` dialog has `Category Name` and `Status`. On save the page reloads.
3. **Edit:** the `Edit Category` dialog has the same fields. On update the page reloads.
4. **Visible switch:** saves at once, silently. Today the category's items are **not** switched; *target (MENU-D3)* they are, as in the app.

## Main flow — Items page

1. The page shows:
   - the tab title `Analytics`, and the heading `Items Overview` / `Items management`;
   - a `+ Add Item` button;
   - a table with the columns `SL No`, `Category Name`, `Item Name`, `Price`, `Visible` and `Actions`, 10 per page.
2. **Add Item:** the dialog has a category, a name, a price and a status. Today on save the alert says `Something went wrong` and nothing is added; *target* the item is saved and the page reloads.
3. **Edit Item:** today the alert says `Something went wrong` and nothing changes; *target* saved.
4. **Visible switch:** today nothing visible happens, and a reload shows the old value; *target* saved.

## Alternative flows

- **Empty category name.** The server fails, the page doesn't reload, and nothing is shown.
- **Unknown restaurant on the Items page.** Today 500 error page; target 404.
- **Target refusals** (price ≤ 0, duplicate name, other restaurant's category, too long). Nothing saved; the page shows `Something went wrong`.

## Acceptance criteria

1. Given REST-A's Categories page, then `Starters` and `Drinks` are listed.
2. Given `Soups` is added on the Categories page, then the page reloads with `Soups`, and the app's Category Management shows it.
3. Given `Starters` is switched off on the Categories page, then after a reload it is off, and its items are still active in the database.
4. Given REST-A's Items page, then the items are listed with their category, name and price.
5. *(Target, MENU-D3)* Given an item is added or edited on the Items page, then it is saved and shown after the reload. (Today: `Something went wrong`, nothing changes.)
6. *(Target, MENU-D3)* Given an item's switch is clicked and the page reloaded, then the new value shows and the ordering screen follows. (Today: the old value.)

## Platform notes

### POS API

- Web routes behind `web` (CSRF) and `auth:admin`. No validation today (target MENU-D3).
- `storeItem` and `updateItem` use the column name `menuItemName`, which doesn't exist (MENU-D3).
- `itemsManagement` selects raw binary IDs.

### Data — MySQL

- Categories: plain writes, with no cascade to items.
- Items: no write ever succeeds from this page.

### POS app

Shows category changes at its next load.

### Super-admin panel

`resources/views/admin/categories.blade.php`, `items.blade.php`.

### Integrations

None.

## Configuration

Page size 10 (hard-coded).

## Out of scope

- Panel sign-in — ADM.

## Open questions

- None open. Related decision: MENU-D3 (To fix).
