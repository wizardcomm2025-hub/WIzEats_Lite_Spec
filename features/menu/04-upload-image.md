# 04 — Upload an image (API only)

## Summary

`POST /api/image/upload` saves a base64 image and returns its path. No current screen calls it: the item form sends the image inside `menuItemSave`. The endpoint is specified so that developers can test it. Its checks are weak (MENU-D5, *To fix*).

## Prerequisites

- **Backend:** `POST /api/image/upload` — [api-contract.md](./api-contract.md#post-apiimageupload)
- **Storage:** `public` disk, folder `image/`; served at `/storage/image/<file>` if `php artisan storage:link` has been run.

## Actors & preconditions

- **Actor:** any signed-in user today; target owner only (X-D1).

## Main flow

1. The caller sends `{ "image": "data:image/<type>;base64,<data>" }`.
2. The server checks:
   - the prefix matches `data:image/<word>;base64,`;
   - the type is `jpg`, `jpeg` or `png`, in any case;
   - the base64 decodes.
3. It writes the decoded bytes to `image/<uuid>.<type>`. Today the content is not checked to be an image; *target (MENU-D5)* it must be a real PNG or JPEG of at most 2 MB.
4. It answers 201 `{ status: true, message: "Image uploaded successfully", data: { status: true, message: "Image uploaded successfully", imagePath: "storage/image/<uuid>.<type>" } }`.

## Alternative flows

- **Wrong prefix, wrong type, or decode failure.** Today still 201 with the top level `Image uploaded successfully`, but `data` = `{ status: false, message: "Failed to upload image:", code: 500 }`. *Target (MENU-D5):* 422 `The image must be a PNG or JPEG of at most 2 MB.`.
- **`image` missing or not a string.** 422 `The image field is required.` / `The image field must be a string.`
- **Very large body.** Limited only by PHP / web server settings (`post_max_size`), which give 413 or an empty request (then the 422).

## Acceptance criteria

1. Given a valid small PNG data URI, then 201 with `data.imagePath`, and the file exists.
2. *(Target, MENU-D5)* Given `data:image/gif;base64,...`, then 422 `The image must be a PNG or JPEG of at most 2 MB.` and no file. (Today: 201 with `data.status: false`.)
3. *(Target, MENU-D5)* Given `data:image/png;base64,` + base64 of `<html>…`, or a PNG over 2 MB, then 422 and no file. (Today: 201 and the file is saved.)
4. *(Target, MENU-D5)* Given an item's image is replaced through `menuItemSave`, then the previous file is deleted.

## Platform notes

### POS API

- `ImageUploadController::store` → `ImageUploadService::store`, which wraps `saveBase64Image`.
- `menuItemSave` calls `saveBase64Image` directly. There, a bad image gives 500 `Failed to save menu item` today; target 422 with the image message.

### Data — MySQL

None.

### POS app

Not called.

### Super-admin panel

None.

### Integrations

Local disk only.

## Configuration

`FILESYSTEM_DISK` / `public` disk; `storage:link`.

## Out of scope

- Image resizing — the app compresses before sending.

## Open questions

- None open. Related decision: MENU-D5 (To fix).
