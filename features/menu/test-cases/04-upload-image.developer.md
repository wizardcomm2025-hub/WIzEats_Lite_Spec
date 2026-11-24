# 04 — Upload an image — developer cases

Use case: [04-upload-image.md](../04-upload-image.md). Contract: [api-contract.md](../api-contract.md#post-apiimageupload). Fixtures: [README.md](./README.md#fixtures).

`POST /api/image/upload`, JSON, OWN-A's bearer token unless stated. "Saved" = 201 with `data.status: true` and `data.imagePath`. "Refused" = 422 `{ status: false, message: "The image must be a PNG or JPEG of at most 2 MB.", data: null }`. "Quiet failure" (today) = 201 `{ status: true, message: "Image uploaded successfully", data: { status: false, message: "Failed to upload image:", code: 500 } }`. Cases tagged *Fails until … is fixed* assert the target; today's result is in brackets.

| ID | Category | Request / manipulation | Expected | Files after |
|---|---|---|---|---|
| MENU-04-D01 | — (happy path) | `{ "image": IMG-PNG }`; then GET `<APP_URL>/<imagePath>` | Saved; `imagePath` `storage/image/<uuid>.png`; GET returns the PNG | one new file |
| MENU-04-D02 | Input validation | (a) `{}` (b) `{ "image": 123 }` | 422 (a) `The image field is required.` (b) `The image field must be a string.` | none |
| MENU-04-D03 | Input validation / Error hygiene — *Fails until MENU-D5 is fixed* | `image` = (a) `data:image/gif;base64,…` (b) plain base64 without prefix (c) `data:image/png;base64,` + `%%%` | Refused each (today: quiet failure) | none |
| MENU-04-D04 | Business-rule bypass / Injection — *Fails until MENU-D5 is fixed* | `data:image/png;base64,` + base64 of `<html><script>alert(1)</script></html>`; also a real PNG labelled `data:image/PNG;…` | HTML: Refused (today: Saved as a `.png` file with HTML content); upper-case label with a real PNG: Saved as `.png` | only the real PNG's file |
| MENU-04-D05 | Input validation — *Fails until MENU-D5 is fixed* | `{ "image": IMG-BIG }` (≈ 3 MB) | Refused (today: Saved, or 413 if over `post_max_size` — record which) | none |
| MENU-04-D06 | Error hygiene | the responses of D03 | no path, stack trace or exception text beyond the fixed message | — |
| MENU-04-D07 | Authentication / Role / Account state / Super-admin — role and OWN-P parts *Fails until MENU-D4 is fixed* / *Fails until AUTH-D2 is fixed* | no token → 401; WTR-A and CHF-A → 403 `{ status: false, message: "You do not have permission to perform this action", data: null }` (today Saved); OWN-P → 403 `Your restaurant is not approved.` (today Saved); NOREST → 403; panel cookie only → 401 | as listed | — |
| MENU-04-D08 | Rate limiting | 30 uploads in a minute | all Saved; no 429 | 30 files |
| MENU-04-D09 | Business rule — *Fails until MENU-D5 is fixed* | `menuItemSave` create with IMG-PNG; note the file; update the item with another PNG; then update it with no `image` | Updated each time; after the second image, the first file is gone; the update without `image` keeps the second | one file for the item (today: both files remain) |
