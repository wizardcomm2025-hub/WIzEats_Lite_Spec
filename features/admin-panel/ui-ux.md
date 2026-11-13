# Super-admin panel — UI / UX

Texts exactly as shown in English (2026-11-12). The Overview, sidebar and header also exist in `es`, `de`, `bn` (`lang/<locale>/analytics.php`).

## 1. Sign-in page (`/`)

| Part | Text |
|---|---|
| Browser title | `Admin Login` |
| Heading | `Admin Panel` / `Please sign in to continue` |
| Fields | `Username` (placeholder `Enter your username`), `Password` (placeholder `Enter your password`, eye toggle), `Forgot password?` (dead link) |
| Checkbox | `Remember me for 30 days` |
| Button | `Sign In` → `Signing in...` (spinner) → `Success!` (green) |
| Error box (red) | `Invalid credentials`; `Too many attempts. Please try again in a minute.`; `Session expired. Please refresh the page.`; or the server's `message` (e.g. a validation text) |
| Empty field | red border and a shake, no request |
| Footer | `© 2026 Wizard Communication Pvt Ltd.`, `Privacy Policy` • `Terms of Service` (dead links) |

## 2. Restaurant list (`/admin`)

| Part | Text |
|---|---|
| Header | `Restaurant Management` / `Manage all registered restaurants and their compliance details`; counters `Total Restaurants`, `Active`, `Inactive`; `Logout` |
| Card | `Restaurant Directory`, search `Search restaurants...` |
| Columns | `#`, `Restaurant Details` (name, `ID: <hex>`), `Owner`, `Contact`, `Location`, `Status` (`Active` / `Inactive`), `Approved` (`Accept` / `Reject` / `Approved` / `Rejected`) |
| Details row | `Restaurant ID`, `GST ID` / `No gst id Given`, `FSSAI License` / `No fssai id Given`, `View Analytics` |
| Empty | `No restaurants found` / `Get started by adding your first restaurant to the system.` / `Add Restaurant` (does nothing) |
| Footer | `Showing <a> to <b> of <n> results`, Bootstrap pagination |

## 3. Restaurant pages (shared layout)

| Part | Text |
|---|---|
| Sidebar | `WizardComm` / `Analytics Dashboard`; `Overview`, `Order History`, `Item History`, `Zone Management`, `Table Management`, `Category Management`, `Item Management`, `Restaurant Settings`, `User Management` |
| Header | `Dashboard Overview` / `Welcome back! Here's what's happening today.` (all three ADM pages); period dropdown `Today`, `Yesterday`, `Last 7 Days`, `Last 15 Days`, `Last 30 Days`; language `English`, `Español`, `Deutsch`, `বাংলা`; bell (dead); `Export Report` (dead); `Logout` |

## 4. Overview

| Block | Text |
|---|---|
| Figures | `Total Revenue` (`525.00`), `Total Orders`, `Total Items Sold`, `Avg Order Value` (`₹262.50`) |
| Revenue chart | `Revenue Overview` / `Daily revenue for the current month`; dataset `Revenue` |
| Channels | `Order Channel` / `Distribution of order channels`; centre `<n>` `Total Orders`; rows `<channel>` `<p>%` |
| Popular | `Popular Items` / `Top performing menu items this week`; rows `0<rank>`, name, `<qty> orders`, `₹<amount>` |
| Recent | `Recent Orders` / `Latest incoming orders`, `Live`; columns `Order ID`, `Items` (`<n> items`), `Total`, `Payment Status` (`Paid` / `Unpaid`) |

## 5. Order History / Item History (English only)

| Page | Columns |
|---|---|
| Order History | `Order ID`, `Channel`, `Total Amount` (`₹ 525.00`), `Payment Status` (`Paid` green / `Unpaid` red), `Date & Time` (`12 Nov 2026, 08:30 AM`, UTC) |
| Item History | `SL No`, `Item Name`, `Quantity` + sort arrow, `Amount` + sort arrow (`↕️` unsorted, `⬆️` asc, `⬇️` desc) |

## 6. German samples (for the tester cases)

`Dashboard-Übersicht`, `Gesamtumsatz`, `Heute`, `Bestellverlauf`.
