# Menu — UI / UX

Screen text exactly as in the code on 2026-11-10; re-checked in `POS_APP-kot` 2026-11-24.

## 1. Category Management (app)

| Part | Content |
|---|---|
| Header | `CATEGORY MANAGEMENT` · restaurant name · `Manage food categories & items` · green `+` |
| Card | Category name · `Items: <n>` · edit button · switch; faded when inactive; tap to expand |
| Expanded | Each item: name, description (if any), `₹<price>`; or `No menu items` |
| Empty | `No Categories Found` / `Create menu categories to organize your food and beverage items.` |
| Add sheet | `Add Category` · `Category name` · `Description (optional)` · `Add Category`; empty name → `Category name is required` |
| Edit sheet | `Edit Category` · same fields · `Update Category`. Today the description starts empty (and is cleared on save — MENU-D6); *target* it shows the stored description |
| Saved | Sheet closes; toast `Category added` / `Category updated` |
| Save failure | Sheet stays open; toast with the server message (e.g. `Category name already exists`) or `Failed to add category` / `Failed to update category` |
| Switch dialog | `Deactivate category?` / `Are you sure you want to deactivate "<name>"?`; or `Activate category?` |
| Switch result | Toast `Category "<name>" deactivated` / `Category "<name>" activated`; its items follow. On failure the card flips back and the toast shows the server message |

## 2. Item Management (app)

| Part | Content |
|---|---|
| Header | Category picker and add button |
| List | The selected category's items (active and inactive), 10 at a time, more on scrolling down (MENU-D2 fixed); none → `No Items Found` / `Add menu items to start building your restaurant's offerings.`; no categories → `No Categories Found`; loading → `Loading menu items...`; pull to refresh |
| Card | Image (if any) · name · `₹<price>` · edit button · switch; faded when inactive |
| Add sheet | `Add Menu Item` · `Item Name` · `Price` · `Description (Optional)` · `Veg` / `Non-Veg` · image box `Tap to upload image` · `Save Item` (`Saving...`) |
| Edit sheet | `Edit Menu Item` · same fields, filled · `Update Item` (`Updating...`); the image is sent only if a new one was picked |
| Field errors | `Name is required` · `Price is required` |
| Image | Chooser sheet (camera / gallery) → `Compressing image...` → preview + toast `Image updated successfully`; errors: `Camera permission is needed to take a photo.`, `No camera is available on this device.`, `Could not read the selected image.`; blocked: `Camera access is off` / `Photo access is off` with `Open Settings` |
| Saved | Sheet closes; toast `Menu item added` / `Menu item updated` |
| Save failure | Sheet stays open with the values; toast with the server message, e.g. `The price field must be greater than 0.`, `Menu item name already exists in this category` (target also `The image must be a PNG or JPEG of at most 2 MB.`) |
| Switch | Dialog `Deactivate menu item?` / `Activate menu item?`; toast `Menu item "<name>" deactivated` / `… activated` |

## 3. Panel pages

| Page | Titles | Columns | Actions | Feedback |
|---|---|---|---|---|
| Categories | Tab `Analytics`; `Category Overview` / `Category management` | `SL No` · `Category Name` · `Visible` · `Actions` | `+ Add Category` → `Add Category` (`Category Name`, `Status`); edit → `Edit Category`; switch saves | Reload on success; errors silent (also the target 422s) |
| Items | Tab `Analytics`; `Items Overview` / `Items management` | `SL No` · `Category Name` · `Item Name` · `Price` · `Visible` · `Actions` | `+ Add Item` → `Add Item`; edit → `Edit Item`; switch | Today add / edit: alert `Something went wrong` (MENU-D3); switch: nothing. *Target:* saves work (success as the page shows today on reload); refusals still end in `Something went wrong` |
