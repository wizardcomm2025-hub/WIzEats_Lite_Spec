# 02 — Browse restaurants — developer cases

Use case: [02-browse-restaurants.md](../02-browse-restaurants.md). Contract: [api-contract.md](../api-contract.md#restaurant-list). Fixtures: [README.md](./README.md#fixtures) (SESS, REST-MANY).

`GET` with SESS unless stated; check the HTML.

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| ADM-02-D01 | — (happy path) | `/admin` and `/admin?page=2` | 200; page 1: 10 rows ordered by `created_at` DESC; page 2: 2 rows; total 12; REST-LEG absent; each row's `ID:` = `LOWER(HEX(restaurantId))` |
| ADM-02-D02 | Authentication / Rate limiting | (a) `/admin` and `/dashboard` with no cookie (b) 100 × `/admin` in a minute with SESS | (a) 302 → `/` each (b) all 200; no 429 |
| ADM-02-D03 | Input validation / Error hygiene | `/admin?page=abc`; `?page=-1`; `?page=99` | `abc` and `-1` → page 1; `99` → 200 with no rows and the footer `Showing  to  of 12 results` (empty numbers) |
| ADM-02-D04 | Injection | Set REST-B's `restaurantName` to `<script>alert(1)</script>` and its address to `<b>x</b>` (by SQL or RST); load `/admin` | The text is shown literally (HTML-escaped: `&lt;script&gt;`); no script runs |
