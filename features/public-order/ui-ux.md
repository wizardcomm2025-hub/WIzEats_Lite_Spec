# Public QR ordering — UI / UX

There is no screen. Neither the POS app (`POS_APP-master`) nor the Laravel project has a customer-facing menu or order page, and nothing generates a QR code that links to one:
- the app's QR codes (`useUpiQr`, `QrZoomModal`, the bill receipt) are UPI payment codes built from `restaurant_configs.qr`;
- the super-admin panel has no public page either.

So this feature has **no tester cases**. Everything is checked through the API ([test-cases/README.md](./test-cases/README.md)).

If a customer page is built later, its screens, texts and tester cases are specified then. A new decision would replace PUB-D1 to D3.
