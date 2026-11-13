# Menu — UI / UX

Screen text exactly as in the code on 2026-11-10.

## 1. Category Management (app)

| Part | Content |
|---|---|
| Header | `CATEGORY MANAGEMENT` · restaurant name · `Manage food categories & items` · green `+` |
| Card | Category name · `Items: <n>` · edit button · switch; faded when inactive; tap to expand |
| Expanded | Each item: name, description (if any), `₹<price>`; or `No menu items` |
| Empty | `No Categories Found` / `Create menu categories to organize your food and beverage items.` |
| Add sheet | `Add Category` · `Category name` · `Description (optional)` · `Add Category` (empty name: nothing happens) |
| Edit sheet | `Edit Category` · same fields (description starts empty) · `Update Category` |
| Switch dialog | `Deactivate category?` / `Are you sure you want to deactivate "<name>"?`; or `Activate category?` |
| Switch result today | Card flips, then back; toast `Failed to save item category` (MENU-D2) |
| Add / edit failure | Nothing shown |

## 2. Item Management (app)

| Part | Content |
|---|---|
| Header | Category picker and add button |
| List | Today always `No Items Found` / `Add menu items to start building your restaurant's offerings.` (MENU-D2); no categories → `No Categories Found`; loading → `Loading menu items...`; pull to refresh |
| Add sheet | `Add Menu Item` · `Item Name` · `Price` · `Description (Optional)` · `Veg` / `Non-Veg` · image box `Tap to upload image` · `Save Item` (`Saving...`) |
| Field errors | `Name is required` · `Price is required` |
| Image | Chooser sheet (camera / gallery) → `Compressing image...` → preview + toast `Image updated successfully`; errors: `Camera permission is needed to take a photo.`, `No camera is available on this device.`, `Could not read the selected image.`; blocked: `Camera access is off` / `Photo access is off` with `Open Settings` |
| Save result today | Toast `The menu item id field must be a string.` inside the sheet; sheet stays open (MENU-D1) |
| Edit sheet (unreachable) | `Edit Menu Item` · `Update Item` (`Updating...`); success toast would be `Menu item updated` |

## 3. Panel pages

| Page | Titles | Columns | Actions | Feedback |
|---|---|---|---|---|
| Categories | Tab `Analytics`; `Category Overview` / `Category management` | `SL No` · `Category Name` · `Visible` · `Actions` | `+ Add Category` → `Add Category` (`Category Name`, `Status`); edit → `Edit Category`; switch saves | Reload on success; errors silent |
| Items | Tab `Analytics`; `Items Overview` / `Items management` | `SL No` · `Category Name` · `Item Name` · `Price` · `Visible` · `Actions` | `+ Add Item` → `Add Item`; edit → `Edit Item`; switch | Add / edit: alert `Something went wrong` (MENU-D3); switch: nothing |
