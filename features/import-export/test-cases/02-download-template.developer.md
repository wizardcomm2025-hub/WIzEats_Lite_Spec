# 02 — Download a template — developer cases

Use case: [02-download-template.md](../02-download-template.md). Contract: [api-contract.md](../api-contract.md#post-apiexportzones--menu--floors). Fixtures: [README.md](./README.md#fixtures).

| ID | Category | Request / manipulation | Expected |
|---|---|---|---|
| IMPEX-02-D01 | — (happy path) | OWN-A: `POST /api/export/zones`, `/menu`, `/floors` | 200; `Content-Type: text/csv`; `Content-Disposition: attachment; filename=zone_file.csv` / `menu_file.csv` / `floor_file.csv`; body exactly the header line from the contract |
| IMPEX-02-D02 | Error hygiene (accepted IMPEX-D3) | same, with REST-A full of zones, items and rooms | bodies still only the header — no restaurant data |
| IMPEX-02-D03 | Authentication / Account state / Super-admin | no token → 401; `GET` → 405; OWN-P → 200; NOREST → 403; panel cookie only → 401 | as listed |
| IMPEX-02-D04 | Role | WTR-A, CHF-A | 200 each |
| IMPEX-02-D05 | Rate limiting / round trip | 30 calls in a minute; then fill the downloaded zone template with one row and import it | no 429; the import is Done and creates the row |
