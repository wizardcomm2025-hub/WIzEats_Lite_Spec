# Menu — categories & items

The owner builds the menu: **categories** (such as "Starters" or "Drinks") and the **menu items** in them, each with a name, price, description, Veg / Non-Veg flag and an optional photo. Categories and items can be switched on or off. The ordering screens (ORD) and the public QR menu (PUB) show the active items. The super-admin panel has a Categories page and an Items page.

The spec records today's behaviour plus targets (re-synced 2026-11-24 against API commit d54d6c8 and `POS_APP-kot`). The GUID handling (MENU-D2) and the app's add-item payload (MENU-D1) are fixed, and the server checks most item rules since 2026-11-18, so categories and items can now be added, edited and switched from the app. MENU-D3 to MENU-D6 are *To fix*.

**Feature code:** `MENU` (test IDs `MENU-01-T01`, `MENU-02-D04`, …)

## Use cases

| # | Use case | Actor | Tester file | Developer file |
|---|---|---|---|---|
| 01 | [Manage categories](./01-manage-categories.md) | Owner (today any signed-in user through the API — MENU-D4) | [tester](./test-cases/01-manage-categories.tester.md) | [developer](./test-cases/01-manage-categories.developer.md) |
| 02 | [Manage menu items](./02-manage-menu-items.md) | Owner (today any signed-in user through the API — MENU-D4) | [tester](./test-cases/02-manage-menu-items.tester.md) | [developer](./test-cases/02-manage-menu-items.developer.md) |
| 03 | [Menu in the panel](./03-menu-in-panel.md) | Super admin | [tester](./test-cases/03-menu-in-panel.tester.md) | [developer](./test-cases/03-menu-in-panel.developer.md) |
| 04 | [Upload an image](./04-upload-image.md) | Today any signed-in user; target owner (API only) | Tester: n/a — no screen of its own | [developer](./test-cases/04-upload-image.developer.md) |

## Key concepts

- **Category** — `item_categories`: name, description, active, deleted.
- **Menu item** — `menu_items`: category, name, description, price (2 decimals), Veg flag, image URL, active, deleted.
- **Ordering menu** — `getMenuItems`: active, not-deleted items, 10 per page by default (max 100), ordered by `created_at` (since 2026-11-18). The app loads it in full for the ordering screen and hides items of inactive categories (from `getItemsCategory`).
- **Item image** — a base64 `data:image/png|jpg|jpeg` string, saved to `storage/app/public/image/<uuid>.<type>` and stored as a full URL.

## Documents

- [data-model.md](./data-model.md) · [api-contract.md](./api-contract.md) · [ui-ux.md](./ui-ux.md) · [test-cases/README.md](./test-cases/README.md) · [developer-guide.html](./developer-guide.html)
- Implementation: `laravel-pos-api/api/specs/menu/` (created with the first plan)

## Depends on

- **AUTH**, **ONB** (the restaurant).
- **IMPEX** — importing categories and items.
- **ORD / PUB** — read the active items. Each order line copies the item's name and price.
- **ADM** — panel sign-in.

## Decisions

Statuses per [conventions.md → Decision status](../../conventions.md#decision-status), set 2026-11-24. Cross-cutting items: [project-overview.md → Known gaps and targets](../../project-overview.md#known-gaps-and-targets).

| ID | Status | Short | Tests |
|---|---|---|---|
| MENU-D1 | **Fixed in the app** (found 2026-11-24) | Add Menu Item sent `menuItemId: 0` | 02-D05, 02-T03 |
| MENU-D2 | **Fixed in code 2026-11-05 / 2026-11-18** (and in the app) | Hex IDs not converted; Item Management empty | 01-D03, 01-D04, 02-D03, 02-D08; 01-T05, 01-T06, 02-T01, 02-T09, 02-T10 |
| MENU-D3 | **To fix** | Panel Items page broken | 03-D03–D07, 03-D11; 03-T03, 03-T05, 03-T06 |
| MENU-D4 | **To fix** (partly fixed 2026-11-18; with X-D1, X-D4) | Server rules: roles, limits, 404s, duplicates, filters | 01-D06, 01-D08, 01-D09, 01-D12, 01-D15; 02-D04, 02-D11, 02-D16, 02-D21, 02-D22, 04-D07; 01-T01 |
| MENU-D5 | **To fix** (new 2026-11-24, split from MENU-D4) | Image upload: no real checks, false success, files never removed | 02-D20, 04-D03–D05, 04-D09 |
| MENU-D6 | **To fix** (new 2026-11-24) | Editing a category in the app wipes its description | 01-T09 (API context: 01-D16) |
| AUTH-D2 (inherited) | **To fix** (X-D2) | Switched-off / unapproved callers keep access | 01-D11, 02-D17 |

- **MENU-D1 — Fixed in the app** (seen in `POS_APP-kot` on 2026-11-24). Add Menu Item sends `menuItemId: ""`, so the create rules apply. The server still answers 422 `The menu item id field must be a string.` to a numeric `menuItemId`.
- **MENU-D2 — Fixed in code** (2026-11-05: list filters; 2026-11-18: category and item saves) **and in the app** (string ID comparisons). Category edit and switch (cascading to its items), item add, edit and switch, and Item Management's paged list all work. Leftover: the `getItemsCategory` filter — now part of MENU-D4.
- **MENU-D3 — To fix (2026-11-24, was accepted 2026-11-10).** *Target:* the Items page writes `name` (not `menuItemName`), puts hex IDs in its HTML so its switch works, and applies the API's item rules (price > 0 and ≤ 99 999 999.99, own non-deleted category, unique name per category, max 255) → JSON 422 `{ message, errors }`; unknown item, category or restaurant IDs → 404 (also on the Items page `GET`). The Categories page switch also switches the category's items, like the API. The pages' scripts keep showing their existing alerts.
- **MENU-D4 — To fix (2026-11-24, was accepted 2026-11-10; partly fixed in code 2026-11-18).** *Fixed 2026-11-18:* price must be greater than 0; the item's category must be a non-deleted category of the restaurant (`Invalid category ID` / `Invalid category. Category not found`); a blank item name is refused; item names are unique per category among non-deleted items (`Menu item name already exists in this category`); category names are unique per restaurant (`Category name already exists`); a malformed `menuItemId` → `Invalid menu item ID`. These messages stay as they are. *Target for the rest:* all writes owner only, reads `getItemsCategory` / `getMenuItems` owner and waiter, `showCategoryMenuItems` owner (X-D1); `price` `max:99999999.99`; `name` / `itemCategoryName` `max:255` → 422 instead of 500; an unknown, deleted or other restaurant's `menuItemId` (valid hex) → 404 `Menu item not found`, and an unknown / other restaurant's / malformed `itemCategoryId` on `categorySave` → 404 `Category not found` (X-D4); deleted categories no longer count in the category-name check; the `getItemsCategory` `itemCategoryId` filter takes hex (dashes allowed) and works; each category's `items` exclude deleted items; malformed `menuItemId` / `categoryId` filters on the two item lists → 422 `Validation error` instead of 500.
- **MENU-D5 — To fix (new 2026-11-24, split from MENU-D4).** *Target:* `/api/image/upload` and the `image` field of `menuItemSave` accept only bytes that really are PNG or JPEG, at most 2 MB decoded → else 422 `The image must be a PNG or JPEG of at most 2 MB.` (no 201 for a failure); a disk write failure → 500 `{ "status": false, "message": "Failed to upload image", "data": null }`; when an item gets a new image, its previous file is deleted.
- **MENU-D6 — To fix (new 2026-11-24).** *Today:* the app's Edit Category sheet starts with an empty description and sends it, so every edit clears the stored description. *Target:* the sheet is filled with the stored description and sends what the user leaves there.
- **Inherited:** AUTH-D2 (To fix, X-D2). RST finding F2: a 16-byte non-ASCII name comes back as hex.

Original decision texts (history):

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
- **Inherited (2026-11-10):** AUTH-D2. RST finding F2: a 16-byte non-ASCII name comes back as hex.

## Facts worth knowing

- Switching a category through the API (and the app, since 2026-11-18) also switches all its items. The panel's category switch doesn't (target: it does — MENU-D3).
- The category list includes inactive categories. Each category's `items` include inactive **and deleted** items, so `Items: <n>` counts them all (target: deleted excluded — MENU-D4).
- Both item lists are sorted by `created_at` (since 2026-11-18; before, by the random `menuItemId`).
- In the app, a failed category add or edit keeps the sheet open and shows the server message as a toast (e.g. `Category name already exists`). A failed switch shows a toast and the switch flips back.

## Cross-cutting open questions

- None open.

## Feature-wide out of scope

- Taking orders from the menu — ORD. The public QR menu — PUB.
- Import / export of the menu — IMPEX.
- Variants (`menu_item_variants`) — not used by any current endpoint (the table is missing from the dump — X-D7).
