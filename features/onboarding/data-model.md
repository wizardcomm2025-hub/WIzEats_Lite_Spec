# Onboarding & approval — data model

Source: `db_script/tables/tables.sql`, `app/Services/Restaurant/RestaurantService.php::onboard`, `app/Http/Controllers/Admin/AdminController.php::updateStatus`. ONB adds no tables or columns.

## Tables written

| Table | When | What |
|---|---|---|
| `restaurants` | Register (01), legacy setup (04), approve / reject (03) | Row created (01); profile fields and `isOnboardingDone = 1` set (01, 04); `isApproved` set (03) |
| `users` | Register (01); legacy setup (04) | Owner created (01); `name` overwritten with the owner name (01, 04 — the **caller's** row in 04) |
| `restaurant_configs` | 01, 04 | Upsert by `restaurantId` |
| `zones` | 01, 04 | Upsert by `restaurantId` only: the restaurant's first zone (or a new one) becomes `Main Hall`, `isActive = 1` |
| `tables` | 01, 04 | `Table 1`–`Table 4`, capacity 4, active — only if that zone has no tables |
| `order_channels` | 01, 04 | Upsert per `(restaurantId, channelCode)` for the four codes |
| `personal_access_tokens` | 01, 04 | One token (`pos-token`) for the owner / caller |

## Values set by register (01)

| Table.column | Value |
|---|---|
| `restaurants.restaurantName`, `restaurantAddress`, `restaurantOwnerName`, `restaurantEmail` | From the form |
| `restaurants.restaurantContact` | The phone given |
| `restaurants.createdBy` | `owner` |
| `restaurants.isOnboardingDone` | 1 |
| `restaurants.isApproved` | 0 (column default) |
| `restaurants.status` | 1 (column default) |
| `users.phone` | The phone given (must be unused — `users_phone_unique`) |
| `users.role` / `status` / `createdBy` | `owner` / 1 / `owner` |
| `users.pin` | Random 6-digit PIN, hashed |
| `users.mustChangePin` | 1 |
| `users.name` | The owner name |
| `restaurant_configs.enableRestaurant`, `enableDelivery`, `enableTakeAway`, `enableKOT`, `enableRoomService` | From the request (the app sends `true`, `false`, toggle, `true`, `false`) |
| `restaurant_configs.restaurantCode` | 6 random upper-case letters / digits, unique across restaurants; kept if one exists |
| `order_channels` | `DINE_IN` Dine In · `TAKEAWAY` Takeaway · `ROOM_SERVICE` Room Service · `DELIVERY` Delivery — all four, whatever the settings |

## Rules

- All writes of 01 / 04 run in one transaction: a failure leaves nothing behind.
- E-mails go out **after** the commit; a mail failure never undoes onboarding.
- No `restaurantEmail` uniqueness: two restaurants may share an e-mail.
- `restaurants.restaurantName` is `varchar(255)`: a longer name fails the whole transaction (500).
- Approve / reject (03) writes `isApproved` exactly as posted (ONB-D2). A non-numeric or missing value fails in the database (strict mode) and nothing changes.
- `isApproved` is never reset by onboarding, legacy setup or profile edits.

## Status values

`restaurants.isApproved`: 0 Pending · 1 Accepted · 2 Rejected. The API shows them as `"Pending"` / `"Accepted"` / `"Rejected"` in sign-in (AUTH) and onboarding responses; any other stored value shows as `"Unknown"` in the onboarding response and `"Rejected"` in the sign-in response.

## Migrations and scripts

None for ONB.

## Seed data

Starter data above is created per restaurant at onboarding, not seeded globally.
