# 05 — Switch language

## Summary

The language dropdown in a restaurant page's header switches the panel's texts between English, Español, Deutsch and বাংলা for the rest of the session. Only the texts in `lang/<locale>/analytics.php` change: the Overview page, the sidebar and the header controls.

## Prerequisites

- **Backend:** `GET /set-locale/{locale}`; see [api-contract.md](./api-contract.md#language)

## Actors & preconditions

- **Actor:** a signed-in super admin (the route itself needs no sign-in, ADM-D1).

## Main flow

1. The admin picks a language. The browser opens `/set-locale/<en|es|de|bn>`.
2. The server stores the locale in the session and redirects back to the previous page (the `Referer`).
3. Every later panel request in this session uses that locale (the `SetLocale` middleware).

## Alternative flows

- **Unsupported locale** (e.g. `/set-locale/fr`): nothing changes; redirect back.
- **Untranslated texts:** the sign-in page, the restaurant list, `/dashboard`, the Order / Item History tables, and any page body text that isn't in `analytics.php`, stay in English (ADM-D3).
- **Logout:** the session is invalidated, so the next sign-in starts in English.
- **`Referer` from another site:** the redirect goes to that site (ADM-D1).

## Acceptance criteria

1. Given the Overview of REST-A, when `Deutsch` is picked, then the page title reads `Dashboard-Übersicht`, the first figure `Gesamtumsatz`, the period `Heute`, and the sidebar link `Bestellverlauf`.
2. Given Deutsch chosen, when the admin opens Item History, then the sidebar is German but the table headings are English.
3. Given Deutsch chosen, when the admin logs out and signs in again, then the Overview is in English.

## Platform notes

### Super-admin panel

Closure route in `routes/web.php` (allows `en`, `es`, `de`, `bn`); `App\Http\Middleware\SetLocale` (appended to the `web` group); `lang/{en,es,de,bn}/analytics.php`.

### POS app, POS API, Data — MySQL

None (the session row only).

## Configuration

None.

## Out of scope

- Translating the remaining pages.

## Open questions

- None open.
