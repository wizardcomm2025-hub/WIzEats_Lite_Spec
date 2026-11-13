# Restaurant profile & settings — data model

Sources:

- `db_script/tables/tables.sql`
- `app/Services/Restaurant/RestaurantService.php` (`showDetails`, `updateDetails`)
- `app/Http/Controllers/Admin/AdminController.php` (`restaurantManagement`, `updateRestaurant`, `updateConfig`)
- `app/Traits/HasBinaryUuids.php`

RST adds no tables or columns.

## `restaurants` (fields RST touches)

| Column | Type | App field | API (`updateDetails`) | Panel |
|---|---|---|---|---|
| `restaurantName` | varchar(255) null | Restaurant Name | yes | yes |
| `restaurantAddress` | text null | Address | yes | yes (Address) |
| `restaurantOwnerName` | varchar(255) null | Owner Name | yes | yes |
| `restaurantContact` | varchar(255) null | Primary Contact (read-only; sent back unchanged) | yes | yes (Contact Number) |
| `alternateMobileNo` | varchar(255) null | Alternate Contact | yes | — |
| `restaurantEmail` | varchar(255) null | Restaurant Email | yes (`email` rule) | — |
| `restaurantGstId` | varchar(255) null | GSTIN Number | yes | yes (GST Number) |
| `restaurantFssaiId` | varchar(255) null | FSSAI ID | yes | yes (FSSAI Number) |
| `restaurantLogo` | varchar(255) null | — | yes (`url` rule) | — |
| `companyAddress` | varchar(255) null | — (always sent `null`, RST-D2) | yes | — |

Never changed by RST: `restaurantId`, `isOnboardingDone`, `status`, `isApproved`, `createdBy`.

## `restaurant_configs` (one row per restaurant)

| Column | Type / default | App field | API | Panel | Used by |
|---|---|---|---|---|---|
| `enableRestaurant` | tinyint NOT NULL, 1 | hidden (sent as held) | yes | Enable Restaurant | nothing today |
| `enableTakeAway` | tinyint NOT NULL, 0 | Enable Takeaway | yes | — | nothing (RST-D1) |
| `enableDelivery` | tinyint NOT NULL, 0 | hidden (sent as held) | yes | Enable Delivery | nothing (RST-D1) |
| `enableRoomService` | tinyint NOT NULL, 0 | hidden (sent as held) | yes | Enable Room Service | app menu: Floor / Room Management |
| `enableKOT` | tinyint NOT NULL, 1 | hidden (sent as `enableKot`, ignored) | **no** (RST-D2) | Enable KOT | — |
| `enableKotGrouping` | tinyint NOT NULL, 0 | — | no | — | — |
| `enableDiscount` | tinyint NOT NULL, 0 | Enable Discount | yes | — | Bill Payment discount section |
| `showKotInBill` | tinyint NOT NULL, 0 | Show KOT Number on Bill | yes | — | bill header |
| `kotDisplayName` | varchar(100), `'kot'` | KOT Name on Bill (max 20 in the app) | yes | — | bill header label |
| `taxPercent` | decimal(5,2) null | Tax Rate (%) | yes (no rule) | Tax % | copied onto new orders (ORD) |
| `serviceChargePercent` | decimal(5,2) null | — (always sent `null`, RST-D2) | yes (`decimal:2`) | Service Charge % | — |
| `invoicePrefix` | varchar(255) null | — | **no** (rule spelled `invocePrefix`, RST-D2) | Invoice Prefix | — |
| `qr` | varchar(255) null | QR ID | yes | — | Bill Payment UPI QR |
| `restaurantCode` | varchar(6) | — | no | — | public QR ordering (PUB) |

Values after onboarding: the switches as registered (the app sends Restaurant on, KOT on, Delivery and Room Service off, and Takeaway as chosen). The other columns hold the defaults: tax and service charge `NULL`, KOT name `kot`, discount and KOT-on-bill off, `qr` `NULL`.

## `order_channels`

`updateDetails` runs `UPDATE order_channels SET isActive = ? WHERE restaurantId = ? AND orderChannelId = 1 | 2 | 3`. `orderChannelId` is `binary(16)`, so no row matches (RST-D1). All four channels keep `isActive = 1` from onboarding.

## Read by `showDetails`

| Table | Filter |
|---|---|
| `restaurants` | the tenant's row |
| `restaurant_configs` | the tenant's row |
| `zones` | tenant, `isActive = 1`, `isDelete = 0` |
| `tables` | tenant, the zone, `isActive = 1`, `isDelete = 0`, ordered by `tableId` |
| `orders` | for each table, the highest `orderId` with `orderStatus` 0 or 1 → `orderId` on the table (null if none) |

## Rules

- `updateDetails` writes only fields present in the request **and** listed in the validation rules. A field present as `null` is written as `NULL`, and a NOT NULL switch then fails (500, rolled back).
- Laravel's global middleware trims strings and turns `""` into `null` for the API and the panel.
- `updateDetails` runs in one transaction. The panel's two updates are separate statements with no transaction.
- **Hex quirk (F2):** in model output, any string attribute of exactly 16 bytes that isn't all printable ASCII is turned into hex. The stored value is unchanged; only responses show hex.
