# {{FEATURE_NAME}} — test cases

Rules: [conventions.md → Test cases](../../../conventions.md#test-cases-test-cases). Results go in `laravel-pos-api/api/specs/<feature>/test-results.md`, never here.

## Fixtures

Describe test data by name and state, never by ID.

| Fixture | State |
|---|---|
| REST-A | Restaurant, approved, onboarding done, active |
| REST-B | A second approved restaurant (for isolation tests) |
| REST-P | Restaurant pending approval |
| OWN-A / WTR-A / CHF-A | Active owner, waiter, chef of REST-A, each with a known PIN |
| WTR-A-OFF | Waiter of REST-A switched off (`status = 0`) |
| OWN-B | Active owner of REST-B |
| <!-- feature fixtures, e.g. ZN-A1 active zone of REST-A --> | |

## Coverage matrix

Every acceptance criterion and alternative flow is covered by at least one test.

| Use case | Acceptance criterion / alternative flow | Tester IDs | Developer IDs |
|---|---|---|---|
| 01 | AC1 | <!-- {{FEATURE_CODE}}-01-T01 --> | <!-- --> |

## Developer endpoint checklist

Each endpoint × category has at least one developer case, or `n/a` with the reason.

| Endpoint | Auth | Account state | Restaurant isolation | Role | Super-admin boundary | Input validation | Business-rule bypass | Over-posting | Concurrency | Rate limit | Injection | Error hygiene |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `POST /api/<!-- -->` | | | | | | | | | | | | |

## How to run the developer cases

PHPUnit feature tests in `laravel-pos-api/api/tests/Feature/<FeatureFolder>/`: one class per use case, one test method per case ID (e.g. `test_{{FEATURE_CODE}}_01_D01`). Tokens via `POST /api/auth/loginPin` per fixture user (or `Sanctum::actingAs`). Check the database with assertions or a SQL client (`HEX()` for IDs).
