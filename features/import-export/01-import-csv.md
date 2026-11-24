# 01 — Import CSV files

## Summary

A caller uploads one, two or three CSV files in a single multipart request. The server reads them in order (zones, then menu, then floors) inside one transaction, and creates the missing zones, categories and floors and the tables, items and rooms listed. Today every row with names is created whatever its values, nothing is reported, and a second import duplicates everything (IMPEX-D2, IMPEX-D3 — *To fix*).

## Prerequisites

- **Backend:** `POST /api/data/import` — [api-contract.md](./api-contract.md#post-apidataimport)
- **Data:** `zones`, `tables`, `item_categories`, `menu_items`, `floors`, `rooms` — [data-model.md](./data-model.md)

## Actors & preconditions

- **Actor:** today any signed-in user of the restaurant; target the owner (IMPEX-D3 / X-D1).
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
6. A row whose insert fails is logged and skipped. The other rows are kept.
7. The answer is 200 `{ status: true, message: "Import completed successfully" }`.

**Target (IMPEX-D2 / IMPEX-D3).** Before step 3 the server checks the caller is the owner. In steps 3–5 each row is checked against the rules in [api-contract.md](./api-contract.md#post-apidataimport) (names, length, capacity ≥ 1, price > 0 and ≤ 99 999 999.99, `isVeg` values, no duplicate child name in the parent or earlier in the file); a failing row is skipped with its reason. A parent is created only together with a valid row. Step 7 adds `data` with the per-file counts and the skipped rows.

## Alternative flows

- **No file.** 422 `At least one CSV file is required.`
- **Wrong type or too big.** 422, e.g. `The zone file field must be a file of type: csv, txt.` / `The zone file field must not be greater than 10240 kilobytes.`
- **Header only, or every row skipped.** 200, and nothing is created (target: counts 0, and the skipped rows listed).
- **Same file twice.** Today tables, items and rooms are duplicated. *Target:* the second import creates nothing; every row is skipped as already existing.
- **Bad values** (capacity `four` or `-3`, price `abc` or `-50`, isVeg `yes`). Today cast and saved (0, -3, 0.00, -50.00, Non-Veg). *Target:* skipped with a reason.
- **Columns in a different order.** They are read by position, so values land in the wrong fields (target: most such rows are then skipped by the checks).
- **Row in an inactive zone, category or floor.** Created there, and so hidden from Home / ordering (accepted).
- **Unexpected failure** (file unreadable, transaction error). Today 400 `{ status: false, message: "<exception text>" }`. *Target:* 500 `Import failed`, nothing written.
- **Waiter or chef.** Today allowed. *Target:* 403.

## Acceptance criteria

1. Given a zone file with `Main Hall,Table 5,4` and `Garden,G1,2`, then 200, `Table 5` is added to the existing Main Hall, and `Garden` is created with `G1`. Both show on Home.
2. Given a menu file with `Starters,Veg Roll,Crispy,1,120,` and `Mains,Butter Chicken,,0,350,x.png`, then the items are created: Veg Roll is Veg at 120.00, Butter Chicken is Non-Veg at 350.00 with no image. `Mains` is created, and both items are on the ordering screen.
3. Given a floor file with `Ground Floor,101,2`, then the floor and room `101` are created, and `getRoomsDetails {}` returns `101`.
4. Given rows with an empty name, then they are skipped and the others are imported; *(target)* the answer lists them with their line numbers.
5. *(Target, IMPEX-D2)* Given the same zone file twice, then the second answer has `created` 0 and every row skipped as `Table name already exists.`; no table exists twice. (Today the tables exist twice.)
6. Given no file, then 422 `At least one CSV file is required.`
7. *(Target, IMPEX-D3)* Given rows with capacity `four`, price `-50` or isVeg `yes`, then they are skipped with the matching reason and nothing of theirs is created.
8. *(Target, IMPEX-D3)* Given a waiter's token, then 403 and nothing is created.

## Platform notes

### POS API

- `ImportController::import` validates with `ImportRequest` and passes the files' temporary paths to `ImportService::import`.
- The restaurant is the caller's `users.restaurantId`.

### Data — MySQL

- One `DB::transaction`. Today a failed row's statement fails on its own; MySQL keeps the rest. Target: row checks run before any insert, so a skipped row writes nothing; an unexpected failure rolls back the whole import.
- `createdBy` is ignored for zones, tables, items and rooms (not fillable). It is set for floors only.

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
