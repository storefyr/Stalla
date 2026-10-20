# Stalla

Forskningsprojekt: Hvad vil det kræve at lave en app til **Sailfish OS 5**, der gør det muligt at **betale for parkering i Danmark**?

Projektet dokumenterer den tekniske og forretningsmæssige granskning af betalingsdelen, før der skrives kode.

## Notater

1. [Vipps/MobilePay web-checkout i en Sailfish OS-app](docs/01-vipps-web-checkout-sailfish.md) — hvad Vipps MobilePays API'er kræver, og hvad Sailfish OS 5 stiller til rådighed (webview, Harbour-regler, app-switch).
2. [Samme, men uden MobilePay-appen på Sailfish](docs/02-vipps-uden-mobilepay-app.md) — konsekvensen af at sløjfe app-switch; CARD-flowet som primær vej.
3. [Bedste valg af kortindløser](docs/03-kortindloeser-valg.md) — sammenligning af Nets Easy, Quickpay+Clearhaus, Scanpay, Pensopay, Stripe og Vipps ePayment med fokus på Dankort, små beløb og webview-kompatibilitet.

## Hovedkonklusion (kort)

- Betaling skal gå gennem en **backend** (merchant-aftale, API-nøgler, webhook, polling) — aldrig nøgler i appen.
- **Nets Easy (Nexi Checkout)** er den stærkeste kortindløser-kandidat: én aftale dækker Dankort, Visa, Mastercard og MobilePay, og det hostede betalingsvindue er simpelt nok til Sailfish' gamle webview.
- Største tekniske risiko: **3D Secure / hosted betalingsvindue i Sailfish' webview (Gecko ESR91)** — skal testes først.
- MobilePay-appen på Sailfish (Alien Dalvik + Play Services-krav) bør **ikke** regnes med; app-switch sløjfes.
