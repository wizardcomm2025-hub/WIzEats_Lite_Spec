# 01 — Import CSV files

## Summary

A caller uploads one, two or three CSV files in a single multipart request. The server reads them in order (zones, then menu, then floors) inside one transaction, and creates the missing zones, categories and floors and every table, item and room listed.

## Prerequisites

- **Backend:** `POST /api/data/import` — [api-contract.md](./api-contract.md#post-apidataimport)
- **Data:** `zones`, `tables`, `item_categories`, `menu_items`, `floors`, `rooms` — [data-model.md](./data-model.md)

## Actors & preconditions

- **Actor:** any signed-in user of the restaurant (IMPEX-D3).
- **Trigger:** a direct API call (no screen — IMPEX-D1).

## Main flow

1. The caller sends `multipart/form-data` with any of `zone_file`, `menu_file`, `floor_file`. Each must be a file of type csv / txt, up to 10 MB, and at least one must be present.
2. The server opens one transaction and reads each file, skipping its first line.
3. **Zone file**, per row `Zone Name, Table Name, Table Capacity`:
   1. Both names are trimmed. If either is empty, the row is skipped (logged).
   2. The zone with that name (any case, not deleted, active or not) is found, or created.
   3. A table is created: active, status 0, capacity = `(int)` of column 3.
4. **Menu file**, per row `Category Name, Item Name, Description, isVeg, Price, Image`:
   1. If the category or item name is empty, the row is skipped.
   2. The category is found or created.
   3. An item is created: active, `isVeg` = `(int)` of column 4, price = `(float)` of column 5, no image.
5. **Floor file**, per row `Floor Name, Room Name, Room Capacity`:
   1. If either name is empty, the row is skipped.
   2. The floor is found or created, with `createdBy` = the caller's role.
   3. A room is created: active, status 0.
6. A row whose insert fails is logged and skipped. The other rows are kept (IMPEX-D2).
7. The answer is 200 `{ status: true, message: "Import completed successfully" }`.

## Alternative flows

- **No file.** 422 `At least one CSV file is required.`
- **Wrong type or too big.** 422, e.g. `The zone file field must be a file of type: csv, txt.` / `The zone file field must not be greater than 10240 kilobytes.`
- **Header only, or every row skipped.** 200, and nothing is created.
- **Same file twice.** Tables, items and rooms are duplicated (IMPEX-D2).
- **Columns in a different order.** They are read by position, so values land in the wrong fields, for example a price of 0.
- **Unexpected failure** (file unreadable, transaction error). 400 `{ status: false, message: "<exception text>" }` (IMPEX-D3).

## Acceptance criteria

1. Given a zone file with `Main Hall,Table 5,4` and `Garden,G1,2`, then 200, `Table 5` is added to the existing Main Hall, and `Garden` is created with `G1`. Both show on Home.
2. Given a menu file with `Starters,Veg Roll,Crispy,1,120,` and `Mains,Butter Chicken,,0,350,x.png`, then the items are created: Veg Roll is Veg at 120.00, Butter Chicken is Non-Veg at 350.00 with no image. `Mains` is created, and both items are on the ordering screen.
3. Given a floor file with `Ground Floor,101,2`, then the floor and room `101` are created, and `getRoomsDetails {}` returns `101`.
4. Given rows with an empty name, then they are skipped and the others are imported, and the answer is still `Import completed successfully`.
5. Given the same zone file twice, then the tables exist twice.
6. Given no file, then 422 `At least one CSV file is required.`

## Platform notes

### POS API

- `ImportController::import` validates with `ImportRequest` and passes the files' temporary paths to `ImportService::import`.
- The restaurant is the caller's `users.restaurantId`.

### Data — MySQL

- One `DB::transaction`. A failed row's statement fails on its own; MySQL keeps the rest.
- `createdBy` is ignored for zones, tables, items and rooms (not fillable). It is set for categories and floors only where the model allows it, which is floors only.

### POS app

The hidden `ImportExcelSection` (IMPEX-D1).

### Super-admin panel

None.

### Integrations

None. Errors are also sent to the Google Sheet log service.

## Configuration

PHP `upload_max_filesize` / `post_max_size` must allow 10 MB.

## Out of scope

- Updating existing tables or items from a file — not offered.

## Open questions

- None open. Related decisions: IMPEX-D2, IMPEX-D3.
