# 01 — View the restaurant profile

## Summary

The app reads the restaurant's details, settings, and zones with their tables from one endpoint. Profile shows the details and settings as a form. Home and the table screens use the zones and tables. The app also keeps the last answer as a startup cache.

## Prerequisites

- **Backend:** `POST /api/restaurants/showDetails` — [api-contract.md](./api-contract.md#post-apirestaurantsshowdetails)
- **Data:** `restaurants`, `restaurant_configs`, `zones`, `tables`, `orders` (running orders) — [data-model.md](./data-model.md)
- **Depends on:** AUTH (token), ONB (the rows exist)

## Actors & preconditions

- **Actor:** a signed-in user of the restaurant. In the UI, the tablet shows Profile to the owner only; a phone shows it to every role (RST-D3).
- **Trigger:** the app starts or resumes, opens Home / Orders, or the user opens Profile.
- **Preconditions:** the restaurant is approved (the app's own gate — AUTH-D2) and the user has changed the onboarding PIN.

## Main flow

1. The app posts to `showDetails` with an empty body. Calls made at the same moment share one request.
2. The server finds the restaurant from the token's user (tenant). Anything in the body is ignored.
3. It returns the restaurant row, its settings as `restaurantConfig`, and `zone`. `zone` lists the active zones, each with its active tables; a table carries the `orderId` of its latest running order, or null.
4. The app stores the answer in Redux and in the startup cache.
5. Profile fills its form from it ([ui-ux.md §1](./ui-ux.md#1-profile-screen)):
   - Personal Information: Restaurant Name, Address, Owner Name, Role (read-only), Primary Contact (read-only), Alternate Contact, Restaurant Email.
   - Tax Details: GSTIN Number, FSSAI ID, Tax Rate (%), QR ID.
   - Restaurant Settings: Enable Takeaway, Enable Discount, Show KOT Number on Bill, and KOT Name on Bill (shown only while Show KOT Number on Bill is on).
6. `Save Changes` stays grey until a field changes.

## Alternative flows

- **Empty values.** An empty tax rate shows `0`. An empty KOT name shows the database default `kot`, in lower case. A missing discount or KOT-on-bill setting would show as on, but a new restaurant has both stored as off.
- **Changed elsewhere.** A change made in the panel shows the next time the app re-reads the details (resume, Home, Orders). If the form has unsaved edits, it is not refilled.
- **Room Service on** (set in the panel). The owner's menu gains Floor Management and Room Management. Profile has no Room Service switch.
- **16-byte non-ASCII value.** Shown as hex (finding F2).
- **No network.** Profile keeps showing what Redux already holds. Nothing new is shown.
- **No token / bad token.** 401. The app ends the session (AUTH 06).
- **User without a restaurant.** 403 `Restaurant context missing`.

## Acceptance criteria

1. Given OWN-A on a tablet, when Profile opens, then every field shows REST-A's current value, Role shows `owner`, and `Save Changes` is grey.
2. Given a newly approved restaurant, then Tax Rate shows `0`, Enable Takeaway shows what was chosen at registration, Enable Discount and Show KOT Number on Bill are off, and Primary Contact is the registration phone.
3. Given any user, then Role and Primary Contact can't be edited.
4. Given WTR-A on a phone, then the menu shows Profile and it opens with Role `waiter` (RST-D3). Given WTR-A on a tablet, then the rail has no Profile.
5. Given the panel changes the tax rate, when the app re-reads the details, then Profile shows the new value.
6. Given the panel switches Room Service on, when the app re-reads the details, then the owner's menu shows Floor Management and Room Management.
7. Given any request, then only the caller's own restaurant is returned.

## Platform notes

### POS API

`RestaurantController::showDetails` → `RestaurantService::showDetails`, behind `auth:sanctum` and `restaurant.tenant`. No validation and no role check. Rate limit: none.

### Data — MySQL

Read only. Zones: `isActive = 1`, `isDelete = 0`. Tables: the same, ordered by `tableId`. The running order is the highest `orderId` with `orderStatus` 0 or 1.

### POS app

- `fetchRestaurantDataApi` shares one in-flight request.
- `fetchRestaurantData` won't start while another is loading, and asks again if a save landed meanwhile.
- `restaurantToForm` maps the record to the form.
- The startup cache is `STORAGE_KEYS.RESTAURANT_DATA`.
- See finding F1 (IDs compared as numbers).

### Super-admin panel

Not involved (see [03](./03-edit-in-panel.md)).

### Integrations

None.

## Configuration

None.

## Out of scope

- What Home / table screens do with `zone` — TBL.
- The Import / Export section (present in code, hidden) — IMPEX.

## Open questions

- None open.
