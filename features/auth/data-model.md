# Sign-in & PIN — data model

Source: `laravel-pos-api-dev_guid/db_script/tables/tables.sql`, `database/migrations/2026_10_06_000000_add_pin_login_columns.php`, `app/Models/User.php`, `app/Models/PersonalAccessToken.php`. AUTH adds no tables or columns.

## Tables

| Table | Primary key | Purpose | Role in AUTH |
|---|---|---|---|
| `users` | `userId` binary(16) | Restaurant users (owner, waiter, chef) | Read at sign-in; `pin`, `mustChangePin`, `phone_verified_at` written |
| `personal_access_tokens` | `id` bigint | Sanctum sessions | Row created at sign-in, deleted at logout |
| `restaurants` | `restaurantId` binary(16) | The user's restaurant | Read: `isOnboardingDone`, `isApproved`, `restaurantEmail`, `restaurantName` |

AUTH also **creates** a `restaurants` row and an owner `users` row through the legacy register endpoint ([07](./07-legacy-otp-and-register.md)).

## Columns used

### `users`

| Column | Type | Rule in AUTH |
|---|---|---|
| `phone` | varchar(255), **unique** (`users_phone_unique`) | The sign-in name. One account per phone across all restaurants. API accepts 10 digits starting 6–9 (legacy OTP: any 10 digits). Stored without country code. |
| `pin` | varchar(255) | Bcrypt hash (model cast `hashed`). Never returned (`$hidden`). Null = cannot sign in with PIN. |
| `mustChangePin` | tinyint(1), default 0 | 1 after a reset or a legacy default PIN; 0 after a successful change. |
| `status` | tinyint(1), default 1 | 0 = inactive. Checked only by PIN sign-in (AUTH-D2). |
| `role` | varchar(255) | `owner` / `waiter` / `chef`. Returned at sign-in; not checked by AUTH. |
| `restaurantId` | binary(16) | Tenant. A token whose user has none is refused by the tenant middleware (403). |
| `phone_verified_at` | timestamp | Set by legacy `verifyOtp`. Decides the `type` of legacy register. |
| `name`, `email`, `createdBy` | | Returned at sign-in. |

### `personal_access_tokens`

| Column | Rule in AUTH |
|---|---|
| `tokenable_id` | The user's `userId` (raw binary). `tokenable_type` = `App\Models\User`. |
| `token` | SHA-256 of the secret part. The client holds `<id>|<secret>`. |
| `name` | `pos-token` for every sign-in. |
| `abilities` | `["*"]`. |
| `expires_at` | Always null (AUTH-D3). |
| `last_used_at` | Updated by Sanctum on each authenticated call. |

### `restaurants` (read)

| Column | Use |
|---|---|
| `isOnboardingDone` | Returned at sign-in; the app routes to onboarding when false. |
| `isApproved` | 0 → `"Pending"`, 1 → `"Accepted"`, 2 → `"Rejected"` in the sign-in response. A missing restaurant row also gives `"Pending"`. |
| `restaurantEmail` | Where Forgot PIN sends the new PIN — for every user of the restaurant, including waiters and chefs. |
| `restaurantName` | Used in the PIN e-mail. |
| `status` | **Not** read by AUTH (AUTH-D2). |

## Rules

- A user can hold any number of tokens at once (one per sign-in, per device). No limit, no expiry (AUTH-D3).
- Logout deletes only the token used for that call.
- PIN change and PIN reset leave all tokens in place (AUTH-D4).
- Reset order: the e-mail is sent first; the PIN is replaced only after the send succeeds, so a mail failure leaves the old PIN working.
- New PINs from a reset are 6 random digits (`random_int`, zero-padded).
- Switching a user off, or rejecting / switching off a restaurant, does not delete tokens (AUTH-D2).
- Legacy register writes the restaurant and its owner together or not at all; no restaurant may be left without a user (AUTH-D11 — change from today).
- A token whose user row no longer exists still authenticates as an empty user (the token model's relation uses `withDefault()`); the tenant middleware then refuses it with 403 `Restaurant context missing`.

## Row lifecycle

Users are never deleted by any API endpoint; they are switched off (`status = 0`). Tokens are deleted only by logout.

## Migrations and scripts

None for AUTH. Note the schema drift listed in [project-overview.md → Known gaps](../../project-overview.md#known-gaps-as-of-2026-11-05).

## Indexes that matter

- `users_phone_unique` — sign-in lookup and one account per phone.
- `personal_access_tokens_token_unique` — token lookup.

## Seed data

None. Test fixtures are listed in [test-cases/README.md](./test-cases/README.md#fixtures).
