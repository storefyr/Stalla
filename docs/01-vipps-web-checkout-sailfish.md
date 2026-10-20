# Notat 1: Vipps/MobilePay web-checkout i en Sailfish OS 5-app

Dato: 2026-02
Status: Forskning — før kodning

## Formål

Granske hvad det kræver at få **Vipps/MobilePay web-checkout** til at virke i en app til **Sailfish OS 5**, som del af en parkerings-app til Danmark.

## Baggrund: hvad "Vipps web-checkout" er i dag

- Vipps og MobilePay er fusioneret til **Vipps MobilePay**. Danmark er fuldt understøttet via **ePayment API** og **Checkout API**. Mobilappen hedder MobilePay i Danmark, Vipps i Norge — samme platform.
- **Vigtig caveat:** Vipps MobilePay har solgt deres **Checkout**-produkt til **Kustom** — Checkout bliver til "Kustom Checkout". ePayment API'et (det rene betalingsflow uden hele checkout-UI'et) påvirkes ikke direkte, men for en parkerings-app er ePayment faktisk det mest relevante.
- To relevante integrationer:
  - **Checkout API** (`POST /checkout/v3/session`) — hosted checkout-side (iFrame eller full-page redirect). Dokumentationen beskriver eksplicit en variant *"Configuration for use inside a native mobile application"*: checkout åbnes i en webview i appen, og merchant angiver en `returnUrl`.
  - **ePayment API** (`POST /epayment/v1/payments` med `userFlow: WEB_REDIRECT`) — minimal redirect-flow: API'et returnerer en `redirectUrl`, brugeren lander på Vipps/MobilePays landing page, app-switch til appen, og returneres til `returnUrl` efter betaling.

Kilder:
- [Checkout API guide](https://developer.vippsmobilepay.com/docs/APIs/checkout-api/checkout-api-guide/)
- [ePayment API — create payment](https://developer.vippsmobilepay.com/docs/APIs/epayment-api/api-guide/operations/create/)
- [ePayment API guide](https://developer.vippsmobilepay.com/docs/APIs/epayment-api/api-guide/)
- [App flow recommendations](https://developer.vippsmobilepay.com/docs/knowledge-base/app-flow/)
- [URL validation](https://developer.vippsmobilepay.com/docs/knowledge-base/url-validation/)

## Krav fra Vipps MobilePay (uanset platform)

| Krav | Detalje |
|---|---|
| Merchant agreement | KYC/AML-check, godkendelse, Merchant Serial Number (MSN), API-nøgler (client id/secret, subscription key). Dansk CVR + dansk erhvervskonto + eID (NemID/MitID via portal) |
| Backend | **Aldrig** API-nøgler i appen. Session/betaling oprettes altid server-side |
| `returnUrl` | Skal være `https://` eller custom URL scheme; "best effort" — brugeren kan lukke appen og aldrig komme tilbage |
| Webhooks | `POST /webhooks/v1/webhooks` — callback-URL skal være offentligt tilgængelig HTTPS (port 80/443/8080) |
| Betalingsstatus | Stol aldrig på redirect alene — poll `GET /epayment/v1/payments/{reference}` og/eller brug webhooks |
| App-flow | Vipps anbefaler `WEB_REDIRECT` + universal links (https) frem for rene deep links; deep links virker ikke i embedded browsers |

## Hvad Sailfish OS 5 stiller til rådighed

**Det gode:**
- `Sailfish.WebView` / `Sailfish.WebEngine` er eksplicit **tilladt i Harbour** ([Allowed APIs](https://docs.sailfishos.org/Develop/Apps/Harbour/Allowed_APIs/)) — en indlejret webview er ikke i sig selv et Harbour-problem.
- App-switch ud til MobilePay-appen: MobilePay kører som Android-app via **Alien Dalvik/AppSupport** (Android 13 på Sailfish 5). Deep links (`vipps://`, `mobilepay://`) fra en webview *kan* potentielt åbne Android-appen, men det er ikke garanteret.

**Udfordringerne (kernen i granskningen):**

1. **WebView-motoren er gammel.** Sailfish OS 5 kører stadig på **Qt 5.6** ([Sailfish docs — Qt](https://docs.sailfishos.org/Reference/Qt/)). Den indlejrede `Sailfish.WebView` bruger Gecko **ESR91** (opdateret i SFOS 5.0); `Sailfish.WebEngine` (Chromium-baseret) svarer til en meget gammel Chromium (~v49 fra Qt 5.6-æraen). Vipps' hosted checkout er en moderne React-app — reel risiko for, at JS/CSS/TLS ikke virker. Fællesskabet har vist, at man kan bundle en moderne QtWebEngine/Chromium selv (f.eks. RooTitanium på SFOS 5.1), men det er et meget tungt bundle (hundredevis af MB) i appen.
2. **Custom URL schemes er ustabile på Sailfish.** Dokumenterede problemer med, at apps ikke kan intercept'e custom schemes fra browsere ([forum: app cannot intercept custom URI scheme](https://forum.sailfishos.org/t/app-cannot-intercept-custom-uri-scheme-from-any-web-browser/27665)). `x-scheme-handler/` i .desktop-filen virker kun delvist. Konsekvens: "tilbage til appen"-flowet bør ikke bygges på `myapp://` — brug i stedet en **https returnUrl** på egen server + **polling** af betalingsstatus i appen.
3. **MobilePay-appen på Sailfish kræver nu Google Play Services.** Nyere MobilePay-versioner nægter at starte uden det ([forumtråd](https://forum.sailfishos.org/t/mobilepay-now-needs-google-play-services/17850/)). microG kan afhjælpe delvist, og Sailfish OS 5.0 har indbyggede microG-enablers — men det er en risiko for app-switch-flowet, og bank-apps er notorisk kræsne.
4. **Harbour QA afviser rene "web-wrappers".** En parkerings-app har naturligt native funktionalitet (kort, zonevalg, aktiv parkering, historik), så det er ikke et principielt problem — men betalingsdelen alene må ikke udgøre appen.

## Foreslået arkitektur (anbefaling)

```
Sailfish-app (QML/Silica)
   │  vælger zone, ser pris, starter betaling
   ▼
Backend (server — IKKE på telefonen)
   │  POST /epayment/v1/payments
   │  (WEB_REDIRECT, returnUrl=https://ditdomæne.dk/betaling/status?ref=…)
   ▼
App åbner redirectUrl i indlejret Sailfish.WebView (eller ekstern Sailfish Browser)
   │  landing page → app-switch til MobilePay (Android via AppSupport)
   ▼
Backend modtager webhook (authorized/aborted/expired) OG appen poller
backend for status ved returnUrl / når appen kommer i forgrunden igen
```

Alternativt **Checkout API i webview** — officielt beskrevet til native apps — men givet Kustom-overtagelsen og den gamle webview-motor rangeres **ePayment + minimal egen landing page** højere for en parkerings-app (betaling for én "vare": parkering i zone X i Y minutter).

## Konklusion og risici

**Konklusion:** Det er teknisk muligt, og Vipps MobilePay understøtter Danmark og udtrykkeligt "checkout i webview i native app". Men det kræver:

1. **En backend** (merchant agreement, API-nøgler, webhook-endpoint, https returnUrl) — største ikke-tekniske hurdle: I skal være (eller gå gennem) en godkendt Vipps MobilePay-merchant med dansk CVR.
2. **En webview der kan køre Vipps' sider** — test først med `Sailfish.WebView` (Gecko ESR91); hvis checkout-siden fejler, er fallback enten ekstern Sailfish Browser eller at bundle en moderne Chromium (tungt).
3. **En strategi for "tilbage til appen"** der ikke afhænger af custom schemes: https returnUrl + polling af betalingsstatus.
4. **Test af app-switch til MobilePay på Sailfish** — især med microG/Play Services-afhængigheden — største usikkerhed.

**Top-3 risici:** (a) gammel webview-motor vs. moderne checkout-side, (b) MobilePay-appen på Sailfish/AppSupport, (c) Kustom-overtagelsen af Checkout-produktet (vælg ePayment, eller få skriftlig bekræftelse fra Vipps MobilePay).

## Se også

- [Notat 2: uden MobilePay-appen på Sailfish](02-vipps-uden-mobilepay-app.md)
- [Notat 3: valg af kortindløser](03-kortindloeser-valg.md)
