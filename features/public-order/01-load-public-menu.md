# 01 — Load the public menu

## Summary

A customer's page sends the restaurant ID (and optionally a table or room ID) to `POST /api/public/menu`. It is meant to get back the restaurant's name, tax rate, active order channels and menu. **Today a hex restaurant ID returns empty data. A raw 16-byte ID returns the restaurant's whole item list, deleted and switched-off items included (PUB-D2).**

## Prerequisites

- **Backend:** `POST /api/public/menu`; see [api-contract.md](./api-contract.md#post-apipublicmenu)
- **Data:** `restaurants`, `restaurant_configs`, `order_channels`, `menu_items`, `item_categories`

## Actors & preconditions

- **Actor:** anyone; no token is needed.
- **Preconditions:** none. The restaurant need not exist, be approved or be switched on.

## Main flow

1. The caller posts `{ "restaurantId": "<id>", "tableId": <optional>, "roomId": <optional> }`.
2. The server validates that `restaurantId` is a string. If `tableId` and `roomId` are present, they must be strings.
3. It reads, each by `restaurantId` as sent:
   - the restaurant's name;
   - `restaurant_configs.taxPercent`;
   - the `channelCode` of every `order_channels` row with `isActive` 1, joined with commas;
   - every `menu_items` row, left-joined to its category's name and description.
4. It answers 200 `{ status: true, message: "Menu items retrieved successfully", data: { restaurantName, restaurantConfig, orderChannel, items } }`.

## Alternative flows

- **Hex `restaurantId` (32 characters, as every other endpoint uses).** No row matches. The reply is 200 with `restaurantName: null`, `restaurantConfig: null`, `orderChannel: ""`, `items: []`.
- **Raw 16-byte `restaurantId` (form-encoded body).** The rows match:
  - `items` holds every item of that restaurant, with hex IDs, in no fixed order. Switched-off and deleted items are included, and so are items in switched-off categories (PUB-D2);
  - `orderChannel` lists the active channel codes, in no fixed order;
  - pending, rejected and switched-off restaurants are served the same way.
- **Unknown restaurant.** The same empty 200 as a hex ID.
- **Validation failure.** 422 `{ status: false, message: <first error>, errors }`.
- **Database failure.** 500 `{ status: false, message: "Failed to retrieve public menu items", data: null }`.

## Acceptance criteria

1. Given REST-A's hex ID in JSON, when the menu is requested, then 200 with `restaurantName` null, `restaurantConfig` null, `orderChannel` `""` and `items` `[]`.
2. Given REST-A's raw ID in a form body, then 200 with `restaurantName` `Rest A`, `restaurantConfig` `{ taxPercent: "5.00" }`, the four channel codes, and every REST-A item including ITEM-3 (inactive) and ITEM-DEL (deleted).
3. Given REST-P's raw ID, then its menu is returned the same way (no approval check).
4. Given `{}`, then 422 `The restaurant id field is required.`

## Platform notes

### POS API

`PublicController::menu`: `PublicMenuRequest` → `PublicService::getPublicMenuItems`. Items are Eloquent models, so `HasBinaryUuids::toArray` turns their binary IDs into hex. Any exception is caught, sent to the Google Sheet log, and answered with the generic 500.

### Data — MySQL

Read only. No `isActive` / `isDelete` filter on items or categories.

### POS app

None. The app never calls the public endpoints.

### Super-admin panel

None.

### Integrations

Google Sheet error log (only on a 500, when configured).

## Configuration

`GOOGLE_SHEET_LOG_URL`, `GOOGLE_SHEET_LOG_SECRET`, `GOOGLE_SHEET_LOG_THROTTLE_MINUTES`.

## Out of scope

- Prices with tax, images, availability. The reply is the raw item rows.

## Open questions

- None open.
