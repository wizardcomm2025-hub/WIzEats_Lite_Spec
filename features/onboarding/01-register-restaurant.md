# 01 — Register a new restaurant

## Summary

A new owner whose mobile number has no account fills in the restaurant profile. The server creates the restaurant, the owner account and the starter data in one step, e-mails the owner a PIN and tells the platform admin. The app confirms the submission and explains that the owner can log in once approved. This is how every new restaurant joins.

## Prerequisites

- **Backend:** `POST /api/restaurants/setup` without a token — [api-contract.md](./api-contract.md#post-apirestaurantssetup)
- **Data:** `restaurants`, `users`, `restaurant_configs`, `zones`, `tables`, `order_channels` — [data-model.md](./data-model.md)
- **Integrations:** outgoing mail (`PinMail`, admin notice)
- **Depends on:** [AUTH 01](../auth/01-check-phone.md) (routes a new number here)

## Actors & preconditions

- **Actor:** new owner, signed out.
- **Trigger:** taps `Save & Continue` on Restaurant Profile.
- **Preconditions:** the phone entered on Login has no account.

## Main flow

1. The app shows Restaurant Profile for the phone entered on Login.
2. The owner enters restaurant name, owner name, e-mail and business address, and optionally switches on Takeaway.
3. The app checks the fields and sends them with the phone and fixed settings (active, KOT on, delivery off, room service off).
4. The server validates the request, including that the phone is still unused.
5. In one transaction the server creates the restaurant (Pending), the owner (active, random PIN, must change PIN), the settings with a unique restaurant code, zone Main Hall with Table 1–4, and the four order channels (Dine In on; Takeaway on only if switched on; Room Service and Delivery off).
6. After the commit the server e-mails the PIN to the restaurant e-mail and sends the admin notice.
7. The server answers with the restaurant, a token and `mustChangePin: true`; the app does not keep the token.
8. The app shows `Success!` / `Restaurant submitted for approval`, then Registration Submitted with the e-mail address.
9. `Go to Login` returns to an empty Login screen. The owner signs in after approval ([02](./02-wait-for-approval.md)).

## Alternative flows

- **Missing field.** `Save & Continue` stays disabled; clearing a field shows `<Label> is required`.
- **E-mail outside .com / .net / .org / .in.** `Email must end with .com, .net, .org, .in`; button disabled. (The server alone accepts any valid e-mail.)
- **Spaces only.** Counts as empty; `Save & Continue` stays disabled. Values are trimmed before sending.
- **Phone registered meanwhile.** Alert `This phone number is already registered. Please log in.`; nothing created.
- **Name over 255 characters.** Alert `Failed to onboard restaurant`; nothing created.
- **PIN e-mail fails or e-mail mistyped.** Registration still succeeds; the owner has no PIN and no way to recover it (ONB-D3).
- **Admin notice fails** (or `ADMIN_EMAIL` unset). Registration still succeeds.
- **Too many registrations.** 11th call within a minute from one IP: `Too Many Attempts.`
- **No network.** Alert `Error` / `Failed to setup restaurant`; nothing created; the form keeps its values.
- **Double tap.** The button is disabled while saving; one restaurant is created.
- **Back.** Returns to Login without creating anything.

## Acceptance criteria

1. Given a new phone and a complete form, when `Save & Continue` is tapped, then `Restaurant submitted for approval` shows and Registration Submitted names the e-mail entered.
2. Given (1), then the restaurant e-mail receives `Your login PIN` with a 6-digit PIN, and the admin address receives `Restaurant Onboarding Completed`.
3. Given (1), then the restaurant is Pending, has zone Main Hall with Table 1–4, and has the four order channels.
4. Given (1), when the owner signs in with the e-mailed PIN before approval, then the Pending screen shows ([02](./02-wait-for-approval.md)).
5. Given an empty field, then `Save & Continue` is disabled.
6. Given `owner@cafe.co`, then `Email must end with .com, .net, .org, .in` shows and the button is disabled.
7. Given the phone was registered in the meantime, then `This phone number is already registered. Please log in.` shows and nothing is created.
8. Given the PIN e-mail can't be sent, then registration still completes.
9. Two quick taps create one restaurant.
10. Takeaway on → the restaurant's takeaway setting and its Takeaway channel are on, and after approval Home offers Takeaway; off → both off, and Home does not offer Takeaway.

## Platform notes

### POS API

`POST /api/restaurants/setup`, public, throttle `onboarding` (10 / min per IP), FormRequest `RestaurantOnboardingRequest`. Envelope `status`. Messages and response in [api-contract.md](./api-contract.md#post-apirestaurantssetup).

### Data — MySQL

One transaction across six tables (channel `isActive` set from the flags since 2026-11-16); values in [data-model.md](./data-model.md#values-set-by-register-01).

### POS app

`RestaurantProfileScreen` → thunk `setupRestaurant` → `RegistrationSubmittedScreen`. Nothing is stored on the device. See [ui-ux.md §2–3](./ui-ux.md#2-restaurant-profile).

### Super-admin panel

The new restaurant appears at the top of the list as Pending with `Accept` / `Reject` ([03](./03-approve-or-reject.md)).

### Integrations

`PinMail` (reason `onboarding`) to `restaurantEmail`; plain-text admin notice to `ADMIN_EMAIL`. Both after commit; failures go to the Google Sheets error log and the server log.

## Configuration

`ADMIN_EMAIL` in `.env` (not in `.env.example` today; unset → the notice silently fails). Allowed e-mail endings are fixed in the app. Starter zone / tables / channels are hard-coded.

## Out of scope

- Onboarding with a token — [04](./04-legacy-setup-with-token.md).
- Editing the profile later — RST.

## Open questions

- None open. Related decision: ONB-D3.
