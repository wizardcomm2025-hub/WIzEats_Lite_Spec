# 01 — Load the public menu

## Summary

A customer's page (not built yet) asks for a restaurant's menu by its ID and gets the restaurant name, tax rate, active channels and items. **Today a normal (hex) ID returns empty data, and the raw ID returns every item including inactive and deleted ones (PUB-D1, PUB-D2 — To fix).**

## Prerequisites

- **Backend:** `POST /api/public/menu` — [api-contract.md](./api-contract.md#post-apipublicmenu)
- **Data:** `restaurants`, `restaurant_configs`, `order_channels`, `menu_items`, `item_categories`

## Actors & preconditions

- **Actor:** anyone; no sign-in.
- **Preconditions:** the caller knows the restaurant's ID (e.g. from a printed QR link — not built yet).

## Main flow

1. The caller posts `{ "restaurantId": "<hex>" }`.
2. *Target:* the server converts the ID, checks the restaurant exists, is approved and active, and returns its name, tax %, active channel codes and its active, not-deleted items in active, not-deleted categories (IDs hex). *Today:* the hex ID matches nothing; 200 with empty data.

## Alternative flows

- **Unknown, malformed, pending, rejected or switched-off restaurant.** *Target:* 404 `Restaurant not found`. Today: empty data (hex), or the full menu (raw ID) whatever its state.
- **More than 60 calls a minute from one IP.** *Target:* 429.
- **Database failure.** 500 `Failed to retrieve public menu items`.

## Acceptance criteria

1. *(Target, PUB-D1)* Given REST-A's hex ID, then 200 with `restaurantName` `Rest A`, `taxPercent` `5.00`, its active channel codes and its active items. (Today: empty data.)
2. *(Target, PUB-D2)* Given an inactive item, a deleted item and an item in an inactive category, then none of them is listed.
3. *(Target, PUB-D2)* Given a pending, rejected or switched-off restaurant, then 404 `Restaurant not found`.
4. Given `{}`, then 422 `The restaurant id field is required.`

## Platform notes

### POS API

`PublicController::menu` → `PublicService::getPublicMenu`.

### Data — MySQL

Read only. See [data-model.md](./data-model.md#menu-reads-use-case-01).

### POS app

None.

### Super-admin panel

None.

### Integrations

Failures go to the Google Sheet log.

## Configuration

None (target: the rate limit in the route).

## Out of scope

- The customer page — not built.

## Open questions

- None open. Related decisions: PUB-D1, PUB-D2, PUB-D3.
