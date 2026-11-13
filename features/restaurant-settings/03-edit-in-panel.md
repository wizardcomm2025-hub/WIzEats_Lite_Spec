# 03 — Edit restaurant and configuration in the panel

## Summary

A super admin opens a restaurant's management page in the web panel. The page has two independent forms:

- **Restaurant Details:** name, contact, owner, GST, FSSAI, address.
- **Configuration:** Enable Restaurant, Enable Delivery, Enable KOT, Enable Room Service, Tax %, Service Charge %, Invoice Prefix.

Each form saves with its own button and confirms with a browser alert. The panel is the only place that can change the KOT switch, the service charge and the invoice prefix (RST-D2).

## Prerequisites

- **Backend (web):**
  - `GET /admin/restaurant-management/{restaurantId}`
  - `POST /admin/restaurant/update`
  - `POST /admin/restaurant/config/update`

  See [api-contract.md](./api-contract.md#panel-routes-web).
- **Data:** `restaurants`, `restaurant_configs` — [data-model.md](./data-model.md)
- **Depends on:** ADM (panel sign-in, navigation to the page)

## Actors & preconditions

- **Actor:** super admin signed in to the panel.
- **Trigger:** opens the restaurant management page for a restaurant, edits, and clicks `Save Restaurant Details` or `Save Configuration`.
- **Preconditions:** the restaurant exists and has a settings row (every onboarded restaurant has one).

## Main flow

1. The page loads the restaurant and its settings into the two forms. The browser tab title is `Analytics`, and the page heading and subtitle are `Category Overview` / `Category management` (copy kept as-is).
2. **Restaurant Details:** the admin edits and clicks `Save Restaurant Details`. The page posts all six fields as JSON with the CSRF token. The server writes them as sent (empty → null). The browser shows the alert `Restaurant Updated`.
3. **Configuration:** the admin edits and clicks `Save Configuration`. The page posts the four switches as 1 / 0, plus tax, service charge and prefix. The server writes them as sent. The alert `Config Updated` shows.
4. The app sees the change the next time it re-reads the details ([01](./01-view-restaurant-profile.md)).

## Alternative flows

- **Empty field.** Saved as empty, including name and owner name (RST-D4).
- **Contact Number changed.** Only `restaurants.restaurantContact` changes; the owner still signs in with the old number (login uses `users.phone`).
- **Tax or service charge not a number, or out of range.** The server fails (500). No alert appears and nothing is saved.
- **Negative tax or service charge.** Saved (RST-D4).
- **Service charge without two decimals** (`7`). Saved as `7.00`; the panel has no decimal rule.
- **Delivery / Room Service switch.** Stored only; Home still offers all four channels (RST-D1). Room Service on shows the Floor / Room menus in the app.
- **Enable Restaurant switch.** Stored; nothing in the app reads it today.
- **Next app save.** Clears the service charge again (RST-D2).
- **Session expired or CSRF token stale.** No alert; nothing is saved (the reply isn't JSON).
- **Unknown restaurant in the URL.** Server error page.

## Acceptance criteria

1. Given ADMIN-1 opens REST-A's page, then both forms show REST-A's current values.
2. Given the name is changed and `Save Restaurant Details` is clicked, then the alert `Restaurant Updated` shows, a reload shows the new name, and the app shows it after its next refresh.
3. Given tax `5`, service charge `10`, prefix `INV` and Enable KOT off, when `Save Configuration` is clicked, then the alert `Config Updated` shows and a reload shows `5.00`, `10.00`, `INV`, unticked.
4. Given tax `abc`, when saved, then no alert shows and a reload shows the old tax.
5. Given Enable Room Service ticked, then the app owner's menu shows Floor Management and Room Management after a refresh.
6. Given Enable Delivery unticked, then Home still offers Delivery (RST-D1).
7. Given an empty Owner Name is saved, then the alert shows and the app shows an empty owner name (RST-D4).
8. Given no panel session, then nothing changes.

## Platform notes

### POS API

Web routes, not `/api`. They run behind the `web` group (session, CSRF) and `auth:admin`. `AdminController::restaurantManagement`, `updateRestaurant` and `updateConfig` don't validate anything. Both update routes always answer `{ "success": true }` when no exception occurs.

### Data — MySQL

- `updateRestaurant` updates by ID. An unknown ID changes 0 rows and still reports success.
- `updateConfig` runs `updateOrInsert` by `restaurantId`. With no config row it tries an insert without `configId`, which fails (500).

### POS app

Not involved; it reflects the result ([01](./01-view-restaurant-profile.md)).

### Super-admin panel

`resources/views/admin/restaurant.blade.php`. Values are HTML-escaped in the inputs. The page uses `fetch`, then `alert`, with no error handling.

### Integrations

None.

## Configuration

None.

## Out of scope

- Panel sign-in, the restaurant list, and links to this page — ADM.
- Users, items, tables, categories and zones pages — ADM / STAFF / MENU / TBL.
- Approve / reject — ONB 03.

## Open questions

- None open. Related decisions: RST-D1, RST-D2, RST-D4.
