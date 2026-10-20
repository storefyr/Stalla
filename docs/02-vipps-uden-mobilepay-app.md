# Notat 2: Vipps/MobilePay uden app på Sailfish OS

Dato: 2026-02
Status: Forskning — opdatering af notat 1
Forudsætning: **MobilePay-appen kan ikke regnes med at virke på Sailfish OS** (kræver nu Google Play Services; se [notat 1](01-vipps-web-checkout-sailfish.md)).

## Hvad ændrer sig, når MobilePay-appen sløjfes

Uden app-switch er der to betalingsveje tilbage i Vipps MobilePays ePayment API:

| Vej | Kræver MobilePay-app? | Hvordan virker det på Sailfish |
|---|---|---|
| **WALLET** (konto-til-konto) | Ja — på *en eller anden* telefon | Landing page → indtast telefonnummer → push til MobilePay på brugerens **anden** enhed. Virker kun hvis brugeren har MobilePay et sted |
| **CARD** (freestanding card) | **Nej** | Hosted card entry page → indtast kort → 3D Secure → retur til `returnUrl`. Ren webbaseret |

For en Sailfish-bruger uden MobilePay-appen bliver **CARD-flowet** derfor den primære (og i praksis eneste rene) vej. Vipps dokumenterer det eksplicit: *"The ePayment API supports freestanding card payments, where any user can pay with a card without using the app"* ([Freestanding cards](https://developer.vippsmobilepay.com/docs/APIs/epayment-api/api-guide/features/freestanding-card-payments/)).

## Den anbefalede arkitektur

```
Sailfish-app (QML/Silica)
   │  zone, tid, pris → "Betal"
   ▼
Backend (aldrig nøgler i appen)
   │  POST /epayment/v1/payments
   │  { paymentMethod: { type: "CARD" }, userFlow: "WEB_REDIRECT",
   │    returnUrl: "https://…/status?ref=…" }
   ▼
redirectUrl åbnes i Sailfish — enten indlejret Sailfish.WebView
eller ekstern Sailfish Browser
   │  hosted card entry page (Vipps) → kortindtastning → 3D Secure
   │  → redirect til returnUrl
   ▼
Backend: webhook (authorized/aborted/expired) + polling af
GET /epayment/v1/payments/{reference}
   │  ← autoritativ betalingsstatus, uanset hvad brugeren gør i browseren
   ▼
Appen viser "parkering aktiv" når backend bekræfter
```

Designvalg:

- **Ekstern browser vs. indlejret webview**: 3D Secure step-up (3DS2) kan involvere ekstra redirects, popups og bankens autentificeringsside. Det øger risikoen for, at en gammel motor fejler midt i flowet. Overvej at åbne betalingen i **ekstern Sailfish Browser** og lade appen poll'e backend i baggrunden — så er et browser-crash ikke lig et betalings-crash, og webhook'en fanger resultatet alligevel.
- **WALLET som sekundær vej**: Det koster næsten intet at tilbyde begge knapper ("Betal med kort" / "Betal med MobilePay på anden enhed"), da landing page-flowet stadig virker for brugere med MobilePay på en Android/iOS-telefon. Men lad være med at bygge UX'en omkring det.

## Nye og skærpede risici

1. **3D Secure i en gammel webview (største tekniske risiko).** Sailfish OS 5 kører Qt 5.6; Sailfish Browser/WebView er Gecko ESR91. Moderne 3DS2-flows (f.eks. bankernes app-switch eller challenge-sider) er ikke garanteret at virke. Dette skal testes tidligt — det er go/no-go for hele løsningen.
2. **Dankort.** For danske kunder er Dankort afgørende. Ifølge Vipps' PSP-dokumentation er **standalone Dankort ikke tokeniserbart** — uden `publicEncryptionKeyId` fejler betalinger med rene Dankort-kort. De fleste Dankort er co-branded Visa/Dankort, som kører Visa-flowet og virker. Afklar med Vipps MobilePay hvordan DANKORT håndteres for en almindelig dansk merchant.
3. **CARD-flow adgang for almindelige merchants.** Dokumentationen for `POST /epayment/v1/payments` siger *"Card payment (CARD) is not available in the test environment. This is only used by Payment Service Providers."* Det kan være en dokumentationsfejl, eller det kan betyde, at CARD kræver en særlig aftale. **Dette skal afklares skriftligt med Vipps MobilePay**, før der bygges noget.
4. **Værdipropositionen ændrer sig.** Uden app-switch er "Vipps-specifik integration" i praksis blevet til *kortbetaling hostet af Vipps* — funktionelt tæt på enhver anden kortindløser, bortset fra Dankort-støtte og nordisk rækkevidde. Overvej om der også skal tilbydes en almindelig kortindløser som primær vej, med Vipps som sekundær (se [notat 3](03-kortindloeser-valg.md)).

## Hvad der stadig gælder fra notat 1

- Merchant agreement med KYC/AML, dansk CVR, dansk erhvervskonto, MSN og API-nøgler — største ikke-tekniske hurdle.
- Backend med offentligt HTTPS webhook-endpoint + polling — autoritativ statuskilde.
- `returnUrl` skal være `https://`; "best effort" — brugeren kan lukke browseren uden at komme tilbage.
- Harbour: indlejret webview (`Sailfish.WebView`) er tilladt; appen skal stadig have reel native funktionalitet (kort, zoner, aktiv parkering) for at passere QA.

## Konklusion

At sløjfe MobilePay-appen gør løsningen **mere gennemførlig** på Sailfish — hele app-switch/usikkerheden omkring Alien Dalvik og Play Services falder væk. Til gengæld bliver betalingen 100% afhængig af, at Vipps' hostede card page + 3D Secure virker i Sailfish' gamle webview/browser. Det er én konkret test, der afgør projektet: **få gennemført en testbetaling med et dansk kort via ePayment CARD-flow i Sailfish Browser**.

Tre spørgsmål til Vipps MobilePay, før projektet går videre:

1. Kan en almindelig dansk merchant (ikke PSP) bruge CARD-flowet i produktion — og hvorfor står der "PSP only"?
2. Hvordan håndteres standalone Dankort for danske salgssteder?
3. Er der krav til at 3DS-challenges skal kunne køre i embedded webviews, eller anbefaler Vipps ekstern browser?

## Se også

- [Notat 1: Vipps web-checkout på Sailfish](01-vipps-web-checkout-sailfish.md)
- [Notat 3: valg af kortindløser](03-kortindloeser-valg.md)
