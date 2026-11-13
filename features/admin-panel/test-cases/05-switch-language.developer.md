# 05 — Switch language — developer cases

Use case: [05-switch-language.md](../05-switch-language.md). Contract: [api-contract.md](../api-contract.md#language). Fixtures: [README.md](./README.md#fixtures) (SESS).

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| ADM-05-D01 | — (happy path) | SESS: `GET /set-locale/es` with `Referer: <APP_URL>/admin/analytics/<REST-A>`; then follow | 302 → that URL; the page is in Spanish (`lang/es/analytics.php` texts) |
| ADM-05-D02 | Input validation / Injection | SESS, `Referer` = the Overview URL: `/set-locale/fr`; `/set-locale/en-US`; `/set-locale/EN`; `/set-locale/de'--` | 302 back each time; the locale stays as it was (only exact `en`, `es`, `de`, `bn` are stored) |
| ADM-05-D03 | Authentication / Error hygiene (accepted ADM-D1) | (a) no cookie: `GET /set-locale/de` with no `Referer` (b) no cookie, `Referer: https://example.org/phish` (c) 100 calls a minute | (a) 302 → `/` (or the session's previous URL); a new guest session holds `de`, and the sign-in page stays English (untranslated) (b) 302 `Location: https://example.org/phish` (open redirect) (c) no 429 |
