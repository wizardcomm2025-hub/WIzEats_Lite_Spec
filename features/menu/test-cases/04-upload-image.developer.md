# 04 — Upload an image — developer cases

Use case: [04-upload-image.md](../04-upload-image.md). Contract: [api-contract.md](../api-contract.md#post-apiimageupload). Fixtures: [README.md](./README.md#fixtures).

`POST /api/image/upload`, JSON, OWN-A's bearer token unless stated. "Saved" = 201 with `data.status: true` and `data.imagePath`. "Quiet failure" = 201 `{ status: true, message: "Image uploaded successfully", data: { status: false, message: "Failed to upload image:", code: 500 } }` (MENU-D4).

| ID | Category | Request / manipulation | Expected | Files after |
|---|---|---|---|---|
| MENU-04-D01 | — (happy path) | `{ "image": IMG-PNG }`; then GET `<APP_URL>/<imagePath>` | Saved; `imagePath` `storage/image/<uuid>.png`; GET returns the PNG | one new file |
| MENU-04-D02 | Input validation | (a) `{}` (b) `{ "image": 123 }` | 422 (a) `The image field is required.` (b) `The image field must be a string.` | none |
| MENU-04-D03 | Input validation / Error hygiene (accepted MENU-D4) | `image` = (a) `data:image/gif;base64,…` (b) plain base64 without prefix (c) `data:image/png;base64,` + `%%%` | Quiet failure each | none |
| MENU-04-D04 | Business-rule bypass / Injection (accepted MENU-D4) | `data:image/png;base64,` + base64 of `<html><script>alert(1)</script></html>`; also `data:image/PNG;…` | Saved; file `.png` with HTML content (served as `image/png`); upper-case type → `.png` | files exist |
| MENU-04-D05 | Input validation | a 20 MB valid PNG | Saved, or 413 / 422 if over `post_max_size` — record which | — |
| MENU-04-D06 | Error hygiene | any of D03 | no path, stack trace or exception text beyond the fixed message | — |
| MENU-04-D07 | Authentication / Role / Account state / Super-admin | no token → 401; WTR-A and CHF-A → Saved (MENU-D4); OWN-P → Saved; NOREST → 403; panel cookie only → 401 | as listed | — |
| MENU-04-D08 | Rate limiting | 30 uploads in a minute | all Saved; no 429 | 30 files |
