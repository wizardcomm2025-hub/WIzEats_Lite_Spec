# 02 — Edit profile and settings in the app

## Summary

The user changes fields on Profile and taps `Save Changes`. The app checks a few things, then sends the whole form to the server. The server saves the fields it knows and returns the new record, and the app shows `Profile updated successfully`. Some settings take effect elsewhere:

- **Tax rate:** new orders.
- **Discount:** the Bill Payment discount section.
- **KOT number and its name:** the bill header.
- **UPI ID:** the payment QR.

The Takeaway switch is only stored (RST-D1).

## Prerequisites

- **Backend:** `POST /api/restaurants/updateDetails` — [api-contract.md](./api-contract.md#post-apirestaurantsupdatedetails)
- **Data:** `restaurants`, `restaurant_configs`, `order_channels` (not effectively changed — RST-D1) — [data-model.md](./data-model.md)
- **Depends on:** [01](./01-view-restaurant-profile.md); ORD / BILL for the effects

## Actors & preconditions

- **Actor:** the owner (tablet or phone). A waiter or chef on a phone, or any user through the API (RST-D3).
- **Trigger:** taps `Save Changes`.
- **Preconditions:** Profile is filled (01) and at least one field has changed.

## Main flow

1. The user edits one or more fields; `Save Changes` turns active.
2. On tap, the app checks:
   - GSTIN: empty, or the 15-character pattern. Otherwise the field shows `Enter a valid 15-character GSTIN, e.g. 22AAAAA0000A1Z5` and a toast says `Please fix the highlighted fields`.
   - Restaurant name not blank, else toast `Restaurant name is required`.
   - Owner name not blank, else toast `Owner name is required`.
3. The app sends the whole form, trimmed. Empty optional texts go as `""` (the server stores them as null). The payload also includes:
   - the hidden settings as they were;
   - `serviceChargePercent: null` and `companyAddress: null` (RST-D2);
   - `enableKot` (ignored by the server — RST-D2);
   - `taxPercent`, which is `"0"` when empty;
   - `kotDisplayName`, which is `KOT` when blank.
4. The server validates (a few rules — RST-D4). In one transaction it updates the restaurant fields that were sent, then the settings that were sent, then tries to set the order channels (matches nothing — RST-D1).
5. It answers 200 with the restaurant and settings. The app re-reads the full record if the reply isn't complete, stores it, refreshes the order channel list, and shows the toast `Profile updated successfully`. The form refills and the button greys out.

## Alternative flows

- **Change and change back.** The form is no longer different from the saved values, so the button greys again.
- **E-mail invalid** (no format check in the app). Toast with the server message `The restaurant email field must be a valid email address.`. The typed values stay.
- **Tax rate:**
  - **Negative** (for example `-5`): saved as `-5.00`; new orders get a negative tax (RST-D4).
  - **Text, or 1000 and above:** toast `Failed to update restaurant details`; nothing is saved (RST-D4).
  - **More than two decimals:** rounded to two by the database.
- **Very long text** (over 255 characters in a text field, over 100 in the KOT name — the KOT name field allows 20). Toast `Failed to update restaurant details`; nothing is saved.
- **E-mail cleared.** Saved as empty. PIN e-mails (Forgot PIN) then have no address (see AUTH-D6 / ONB-D3).
- **Takeaway switched off.** Saved; Home still offers Takeaway (RST-D1).
- **Service charge set in the panel.** Cleared by this save (RST-D2).
- **Name shown as hex** (F2). Any save writes the hex back as the name.
- **No network.** Toast `Failed to update restaurant profile`; the typed values stay; saving again later works.
- **Double tap.** The button is disabled while saving, so there is one request.
- **Background refresh while editing.** Unsaved edits are not overwritten.
- **Token gone** (401). The session ends (AUTH 06).

## Acceptance criteria

1. Given OWN-A changes the restaurant name and saves, then the toast `Profile updated successfully` shows, the header shows the new name, the button greys, and the panel shows the new name.
2. Given a blank restaurant name or owner name, when Save is tapped, then the matching toast shows and nothing is sent.
3. Given an invalid GSTIN, when Save is tapped, then the field shows the GSTIN message in red, the toast `Please fix the highlighted fields` shows, and nothing is sent.
4. Given an invalid e-mail, then the server message shows as a toast and nothing is saved.
5. Given a tax rate of `5`, when saved, then Profile shows `5.00`, and a new order is taxed at 5 %; existing orders keep their tax.
6. Given Enable Discount off, then Bill Payment shows no discount section; on, it shows.
7. Given Show KOT Number on Bill on and KOT name `Token`, then a bill header shows `Token : <n>`. Blank shows `KOT : <n>`. Off: the name field hides and the bill header shows no KOT numbers.
8. Given a QR ID (UPI ID), then the Bill Payment QR encodes that UPI ID.
9. Given Enable Takeaway off, then it is saved and Home still shows Takeaway (RST-D1).
10. Given the panel set a service charge, when the owner saves anything in the app, then the service charge is empty afterwards (RST-D2).
11. Given WTR-A on a phone, then Profile saves succeed (RST-D3).
12. Given no network, then the error toast shows and the typed values stay.

## Platform notes

### POS API

- `RestaurantController::updateDetails` validates with `RestaurantProfileRequest`, then calls `RestaurantService::updateDetails`.
- Only the fields that pass validation are used. A field sent as `null` is written as `null`.
- Errors inside the service give 500 `Failed to update restaurant details`, with no exception text.

### Data — MySQL

- One transaction. A database error (bad tax value, null in a NOT NULL switch, too long) rolls back both rows.
- `restaurant_configs` is upserted by `restaurantId`.
- The `order_channels` updates match no row (RST-D1).

### POS app

- `ProfileScreen.onSubmit` → `updateRestaurantProfile` thunk → `mergeRestaurant`; it may re-read the details, then calls `fetchOrderChannels`.
- The Enable Restaurant, KOT, Delivery and Room Service switches exist in code but are commented out.
- The Import section is hidden.

### Super-admin panel

Shows the change on the next page load ([03](./03-edit-in-panel.md)).

### Integrations

None.

## Configuration

None.

## Out of scope

- How the bill prints the KOT name, discount and QR — BILL / PRN. Here only "shows / doesn't show" is checked.
- The logo (`restaurantLogo`): accepted by the API, but there is no field in the app.

## Open questions

- None open. Related decisions: RST-D1 to RST-D4.
