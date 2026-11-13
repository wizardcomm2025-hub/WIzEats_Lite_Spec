# 03 — Approve or reject a restaurant

## Summary

A super admin reviews newly onboarded restaurants in the web panel and accepts or rejects each one. The decision is saved on the restaurant, the owner is e-mailed, and the app lets the restaurant in or shows it the Rejected screen. In the panel a decision is final: the buttons show only while a restaurant is Pending.

## Prerequisites

- **Backend:** `POST /admin/update-status/{restaurantId}` — [api-contract.md](./api-contract.md#post-adminupdate-statusrestaurantid-web)
- **Data:** `restaurants.isApproved`, `restaurantEmail` — [data-model.md](./data-model.md)
- **Integrations:** `RestaurantStatusMail`
- **Depends on:** ADM (panel sign-in), [01](./01-register-restaurant.md)

## Actors & preconditions

- **Actor:** super admin, signed in to the panel.
- **Trigger:** clicks `Accept` or `Reject` on a restaurant row.
- **Preconditions:** the restaurant has finished onboarding and is Pending.

## Main flow

1. The panel lists onboarded restaurants, newest first, 10 per page.
2. A Pending restaurant shows `Accept` and `Reject`.
3. The super admin clicks one; the panel posts the decision (1 or 2) with the session's CSRF token.
4. The server saves the value on the restaurant.
5. If the value changed to Accepted or Rejected and the restaurant has an e-mail, the server sends the matching e-mail.
6. The panel reloads the list (no message is shown); the row now shows `Approved` or `Rejected`.
7. The restaurant's app users see the change at their next approval check ([02](./02-wait-for-approval.md)).

## Alternative flows

- **No confirmation.** One click decides.
- **Changing a decision.** Not offered in the panel. A crafted request can set any value, including back to Pending (ONB-D2).
- **Any value.** A crafted request with `isApproved = 7` is saved; the panel then shows no badge and no buttons; the app treats it as Rejected at sign-in (ONB-D2).
- **Missing or non-numeric value.** Server error page; nothing changes.
- **Unknown restaurant.** Nothing changes; the server still answers as if it worked (redirect back).
- **E-mail fails.** The decision is saved; the failure is only logged.
- **Same decision posted twice.** Saved again; no second e-mail.
- **Session expired.** Redirect to the panel login; nothing changes.
- **Page left open too long (CSRF token expired).** `Page Expired`; nothing changes.

## Acceptance criteria

1. Given a Pending restaurant, when the super admin clicks `Accept`, then the list reloads, the row shows `Approved`, and the restaurant e-mail receives `Your restaurant has been approved`.
2. Given a Pending restaurant, when `Reject` is clicked, then the row shows `Rejected` and the e-mail `Your restaurant registration was not approved` is sent.
3. Given an Accepted or Rejected restaurant, then the row shows only its badge — no buttons.
4. Given (1), when the owner's app checks approval, then it moves past the Pending screen.
5. Given (2), when the owner's app checks approval, then it shows `Account Access\nDenied`.
6. Given a restaurant whose onboarding isn't done, then it is not listed.
7. Given the mail server is down, when `Accept` is clicked, then the decision is still saved.
8. Given no panel session, when the endpoint is called, then nothing changes.

## Platform notes

### POS API

Not an `/api` endpoint — a web route in `routes/web.php` behind the `web` group and `auth:admin`. No validation of `isApproved` (ONB-D2). See [api-contract.md](./api-contract.md#post-adminupdate-statusrestaurantid-web).

### Data — MySQL

Updates `restaurants.isApproved` and `updated_at` for the given ID.

### POS app

No screen of its own; [02](./02-wait-for-approval.md) reacts.

### Super-admin panel

`admin/index` list, `Approved` column. See [ui-ux.md §5](./ui-ux.md#5-panel--restaurant-list-approval-part).

### Integrations

`RestaurantStatusMail` to `restaurantEmail`, only when the value changes to 1 or 2.

## Configuration

Mail transport in `.env`. Page size (10) is hard-coded.

## Out of scope

- Switching a restaurant on / off, editing its data, analytics — ADM.
- Panel sign-in — ADM.

## Open questions

- None open. Related decision: ONB-D2.
