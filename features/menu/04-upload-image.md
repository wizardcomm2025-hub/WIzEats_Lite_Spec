# 04 — Upload an image (API only)

## Summary

`POST /api/image/upload` saves a base64 image and returns its path. No current screen calls it: the item form sends the image inside `menuItemSave`. The endpoint is specified so that developers can test it. Its checks are weak (MENU-D4).

## Prerequisites

- **Backend:** `POST /api/image/upload` — [api-contract.md](./api-contract.md#post-apiimageupload)
- **Storage:** `public` disk, folder `image/`; served at `/storage/image/<file>` if `php artisan storage:link` has been run.

## Actors & preconditions

- **Actor:** any signed-in user (MENU-D4).

## Main flow

1. The caller sends `{ "image": "data:image/<type>;base64,<data>" }`.
2. The server checks:
   - the prefix matches `data:image/<word>;base64,`;
   - the type is `jpg`, `jpeg` or `png`, in any case;
   - the base64 decodes.
3. It writes the decoded bytes to `image/<uuid>.<type>`. The content is not checked to be an image (MENU-D4).
4. It answers 201 `{ status: true, message: "Image uploaded successfully", data: { status: true, message: "Image uploaded successfully", imagePath: "storage/image/<uuid>.<type>" } }`.

## Alternative flows

- **Wrong prefix, wrong type, or decode failure.** Still 201 with the top level `Image uploaded successfully`, but `data` = `{ status: false, message: "Failed to upload image:", code: 500 }` (MENU-D4).
- **`image` missing or not a string.** 422 `The image field is required.` / `The image field must be a string.`
- **Very large body.** Limited only by PHP / web server settings (`post_max_size`), which give 413 or an empty request (then the 422).

## Acceptance criteria

1. Given a valid small PNG data URI, then 201 with `data.imagePath`, and the file exists.
2. Given `data:image/gif;base64,...`, then 201 with `data.status: false` and no file.
3. Given `data:image/png;base64,` + base64 of `<html>…`, then 201 and an HTML file is saved with a `.png` name (MENU-D4).

## Platform notes

### POS API

- `ImageUploadController::store` → `ImageUploadService::store`, which wraps `saveBase64Image`.
- `menuItemSave` calls `saveBase64Image` directly. There, a bad image gives 500 `Failed to save menu item`.

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

- None open. Related decision: MENU-D4.
