# Restaurant profile & settings — data model

Sources:

- `db_script/pos_stage.sql` (re-checked 2026-11-24: no RST table changed)
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
| `companyAddress` | varchar(255) null | — (today always sent `null`, RST-D2; target: not sent) | yes | — |

Never changed by RST: `restaurantId`, `isOnboardingDone`, `status`, `isApproved`, `createdBy`.

## `restaurant_configs` (one row per restaurant)

| Column | Type / default | App field | API | Panel | Used by |
|---|---|---|---|---|---|
| `enableRestaurant` | tinyint NOT NULL, 1 | hidden (sent as held) | yes | Enable Restaurant | nothing today |
| `enableTakeAway` | tinyint NOT NULL, 0 | Enable Takeaway | yes (+ `TAKEAWAY` channel) | — | Takeaway on Home, through the channel |
| `enableDelivery` | tinyint NOT NULL, 0 | hidden (sent as held) | yes (+ `DELIVERY` channel) | Enable Delivery (channel only in the target, RST-D5) | Delivery on Home, through the channel |
| `enableRoomService` | tinyint NOT NULL, 0 | hidden (sent as held) | yes (+ `ROOM_SERVICE` channel) | Enable Room Service (channel only in the target, RST-D5) | app menu: Floor / Room Management; Room Service on Home, through the channel |
| `enableKOT` | tinyint NOT NULL, 1 | hidden (sent as `enableKot`) | today **no**; target yes (RST-D2) | Enable KOT | — |
| `enableKotGrouping` | tinyint NOT NULL, 0 | — | no | — | — |
| `enableDiscount` | tinyint NOT NULL, 0 | Enable Discount | yes | — | Bill Payment discount section |
| `showKotInBill` | tinyint NOT NULL, 0 | Show KOT Number on Bill | yes | — | bill header |
| `kotDisplayName` | varchar(100), `'kot'` | KOT Name on Bill (max 20 in the app) | yes | — | bill header label |
| `taxPercent` | decimal(5,2) null | Tax Rate (%) | yes (no rule) | Tax % | copied onto new orders (ORD) |
| `serviceChargePercent` | decimal(5,2) null | — (today always sent `null`, RST-D2; target: not sent) | yes (`decimal:2`; target numeric 0–100) | Service Charge % | — |
| `invoicePrefix` | varchar(255) null | — | today **no** (rule spelled `invocePrefix`); target yes (RST-D2) | Invoice Prefix | — |
| `qr` | varchar(255) null | QR ID | yes | — | Bill Payment UPI QR |
| `restaurantCode` | varchar(6) | — | no | — | public QR ordering (PUB) |

Values after onboarding: the switches as registered (the app sends Restaurant on, KOT on, Delivery and Room Service off, and Takeaway as chosen). The other columns hold the defaults: tax and service charge `NULL`, KOT name `kot`, discount and KOT-on-bill off, `qr` `NULL`.

## `order_channels`

Since 2026-11-16 (RST-D1 fixed) `updateDetails` runs `UPDATE order_channels SET isActive = ? WHERE restaurantId = ? AND channelCode = 'TAKEAWAY' | 'DELIVERY' | 'ROOM_SERVICE'` for each matching flag sent, inside its transaction. `DINE_IN` is never changed by RST.

- The panel's `updateConfig` does **not** touch `order_channels` today; target: it sets `DELIVERY` and `ROOM_SERVICE` the same way (RST-D5).
- Restaurants created before 2026-11-16 still have all four channels `isActive = 1` until their next app save; target: one-off script `db_script/sync_order_channels.sql` (RST-D6):

```sql
UPDATE order_channels oc JOIN restaurant_configs rc ON rc.restaurantId = oc.restaurantId
SET oc.isActive = CASE oc.channelCode
  WHEN 'TAKEAWAY' THEN rc.enableTakeAway
  WHEN 'DELIVERY' THEN rc.enableDelivery
  WHEN 'ROOM_SERVICE' THEN rc.enableRoomService
  ELSE 1 END;
```

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
- `updateDetails` runs in one transaction. The panel's two updates are separate statements with no transaction (target RST-D5: the config write and its channel updates in one transaction).
- **Hex quirk (F2):** in model output, any string attribute of exactly 16 bytes that isn't all printable ASCII is turned into hex. The stored value is unchanged; only responses show hex.
