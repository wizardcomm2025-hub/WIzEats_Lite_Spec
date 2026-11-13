# Staff management — data model

Sources:

- `db_script/tables/tables.sql`
- `app/Services/Staff/StaffService.php` (`create`, `update`, `fetch`)
- `app/Http/Controllers/Admin/AdminController.php` (`userManagement`, `updateUser`)
- `app/Models/User.php`

STAFF adds no tables or columns.

## `users`

| Column | Type | Set by STAFF |
|---|---|---|
| `userId` | binary(16) PK | new GUID on create |
| `restaurantId` | binary(16) null | the caller's restaurant on create |
| `name` | varchar(255) null | create; app/API update; panel update |
| `phone` | varchar(255) NOT NULL, **unique across all restaurants** (`users_phone_unique`) | create only |
| `pin` | varchar(255) null, hashed | `123456` (hashed) on create (STAFF-D4) |
| `mustChangePin` | tinyint NOT NULL | 1 on create |
| `role` | varchar(255) null | create (as sent — the app sends `Chef` / `Waiter`); panel update (free text) |
| `status` | tinyint NOT NULL, 1 | 1 on create; API update; panel update |
| `createdBy` | varchar(255) null | the caller's role on create |
| `countryCode`, `email`, `phone_verified_at` | — | not touched |
| `created_at`, `updated_at` | timestamps | set by Eloquent on create and API update; **not** by the panel update |

## Rules

- **Uniqueness:** a phone can belong to one user only, across every restaurant. A duplicate insert fails in the database, which gives a 500.
- **Roles:** the database stores any text. The app recognises `owner`, `waiter` and `chef` in any case. The staff list query matches `waiter` / `chef` regardless of case (`utf8mb4_unicode_ci`).
- **Responses:** `pin`, `password` and `remember_token` are hidden from JSON. IDs are returned as 32-character hex.
- **Limit:** at most one active chef and one active waiter — app only.
- **Isolation:**
  - the list is filtered by the caller's restaurant;
  - create always uses the caller's restaurant;
  - update is **not** filtered (STAFF-D2).
