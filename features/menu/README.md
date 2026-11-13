# Menu — categories & items

The owner builds the menu: **categories** (such as "Starters" or "Drinks") and the **menu items** in them, each with a name, price, description, Veg / Non-Veg flag and an optional photo. Categories and items can be switched on or off. The ordering screens (ORD) and the public QR menu (PUB) show the active items. The super-admin panel has a Categories page and an Items page.

The spec records today's behaviour. Decisions MENU-D1 to MENU-D4 keep the current rules as accepted risks. **Today the app can add and list categories, but can't add, edit or switch menu items, and can't edit or switch categories (MENU-D1, MENU-D2).**

**Feature code:** `MENU` (test IDs `MENU-01-T01`, `MENU-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Manage categories](./01-manage-categories.md) | Owner (any signed-in user through the API — MENU-D4) | [tester](./test-cases/01-manage-categories.tester.md) | [developer](./test-cases/01-manage-categories.developer.md) |
| 02 | [Manage menu items](./02-manage-menu-items.md) | Owner (any signed-in user through the API — MENU-D4) | [tester](./test-cases/02-manage-menu-items.tester.md) | [developer](./test-cases/02-manage-menu-items.developer.md) |
| 03 | [Menu in the panel](./03-menu-in-panel.md) | Super admin | [tester](./test-cases/03-menu-in-panel.tester.md) | [developer](./test-cases/03-menu-in-panel.developer.md) |
| 04 | [Upload an image](./04-upload-image.md) | Any signed-in user (API only) | Tester: n/a — no screen of its own | [developer](./test-cases/04-upload-image.developer.md) |

## Key concepts

- **Category** — `item_categories`: name, description, active, deleted.
- **Menu item** — `menu_items`: category, name, description, price (2 decimals), Veg flag, image URL, active, deleted.
- **Ordering menu** — `getMenuItems`: active, not-deleted items, 10 per page by default (max 100). The app loads it in full for the ordering screen.
- **Item image** — a base64 `data:image/png|jpg|jpeg` string, saved to `storage/app/public/image/<uuid>.<type>` and stored as a full URL.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api-dev_guid/specs/menu/` (created with the first plan)

## Depends on

- **AUTH**, **ONB** (the restaurant).
- **IMPEX** — today the only working way, besides a direct API call, to create menu items.
- **ORD / PUB** — read the active items. Each order line copies the item's name and price.
- **ADM** — panel sign-in.

## Decisions

- **MENU-D1 (2026-11-10):** The app's Add Menu Item keeps sending `menuItemId: 0`. The server treats `0` as filled, so it applies the update rules. The `string` rule then fails, and every add answers 422 `The menu item id field must be a string.`, shown as a toast. Accepted.
- **MENU-D2 (2026-11-10):** The GUID handling stays broken:
  - `saveCategory` (update) and `saveMenuItem` (update) look up the hex ID as sent, so every edit and on/off switch of a category or an item answers 500 (`Failed to save item category` / `Failed to save menu item`);
  - the `itemCategoryId`, `categoryId` and `menuItemId` filters on the list endpoints never match;
  - the app's Item Management filters with `Number(categoryId)` as well;
  - so Item Management always shows `No Items Found`.

  Listing categories with their items, adding a category, and the ordering menu work. Accepted.
- **MENU-D3 (2026-11-10):** The panel's Items page stays broken:
  - Add and Edit write a column `menuItemName`, which doesn't exist (the column is `name`), so both always end in the alert `Something went wrong`;
  - the page puts raw binary IDs in its HTML, so its Visible switch changes nothing;
  - an unknown restaurant ID gives a 500 page.

  The Categories page works, but its switch doesn't change the items. Accepted.
- **MENU-D4 (2026-11-10):** No server rules:
  - any role can change the menu;
  - price can be negative, and a value of 100 000 000 or more gives a 500;
  - `categoryId` isn't checked on item create, so an item can be put in another restaurant's category or a made-up one;
  - names over 255 characters give a 500;
  - the image endpoint checks only the `data:image/…` prefix: any bytes labelled png/jpg are saved, with no size limit, and it answers 201 `Image uploaded successfully` even when saving fails;
  - old image files are never removed.

  Accepted risk.
- **Inherited:** AUTH-D2. RST finding F2: a 16-byte non-ASCII name comes back as hex.

## Facts worth knowing

- Switching a category through the API (unreachable today) also switches all its items. The panel's category switch doesn't.
- The category list includes inactive categories. Each category's `items` include inactive **and deleted** items, so `Items: <n>` counts them all.
- The ordering menu is sorted by `menuItemId`, a random GUID, so the order is stable but arbitrary.
- In the app, failures to add or edit a category show no message. A failed category switch shows a toast and the switch flips back.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Taking orders from the menu — ORD. The public QR menu — PUB.
- Import / export of the menu — IMPEX.
- Variants (`menu_item_variants`) — not used by any current endpoint.
