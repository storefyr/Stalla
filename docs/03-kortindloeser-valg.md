# Notat 3: Bedste valg af kortindløser til en Sailfish-parkerings-app

Dato: 2026-02
Status: Forskning — før kodning
Kontekst: Se [notat 1](01-vipps-web-checkout-sailfish.md) og [notat 2](02-vipps-uden-mobilepay-app.md).

## Kriterier der betyder noget for netop denne app

| Kriterium | Hvorfor det er afgørende |
|---|---|
| **Dankort-understøttelse** | ~94% af Dankort er co-branded Visa/Dankort (kan gå via Visa), men 2–5% er rene Dankort — og kun **Nets** må indløse dem i Danmark. Dankort-gebyret er desuden det laveste (~0,31–0,36% + fast gebyr) |
| **Små beløb** | Parkering koster typisk 10–100 kr. Faste gebyrer spiser marginen — procentsatsen betyder mindre end det faste gebyr |
| **Hosted betalingsvindue (redirect)** | Betalingen skal kunne køre i Sailfish' gamle webview (Gecko ESR91) eller ekstern browser. En simpel hosted side er langt sikrere end moderne embedded JS-felter |
| **Webhook + server-side status** | Autoritative status — redirect tilbage er "best effort" |
| **Dansk juridisk enhed / support** | Indløsningsaftaler kræver dansk CVR og dansk erhvervskonto |

## Sammenligning af kandidater

Listepriser pr. 2025/2026 — **forhandlingsgrundlag, ikke facit**. Priser ændres løbende; indhent altid aktuelle tilbud.

| Udbyder | Type | Pris (ca.) | Dankort | MobilePay | Vurdering til dette projekt |
|---|---|---|---|---|---|
| **Nets Easy (Nexi Checkout)** | Alt-i-én (gateway + indløser) | Visa/MC debet: 0,99% + 0,50 kr. Visa/Dankort: ~0,31% + 0,54 kr. Månedsabonnement: ~0–199 kr. | **Ja — direkte, én aftale** | Ja (1 kr/tx + 49 kr/md) | **Stærkeste kandidat.** Hosted betalingsvindue er en ældre, simpel HTML-side → god webview-kompatibilitet. Dansk, veletableret, god API + webhooks |
| **Quickpay** | Gateway (kræver separat indløser) | 99 kr/md + 0,25 kr/tx gateway; acquirer fra 1,35% + 0,22 kr (Clearhaus: 1,45%, min 0,60 kr) | Kun via Nets-aftale (ekstra +30 kr/md for eksterne indløsere) | Ja (1 kr/tx + 49 kr/md) | Fleksibel — vælg selv indløser (Clearhaus/Nets). God API. Kræver to aftaler for fuld Dankort-dækning |
| **Clearhaus** | Ren indløser | 1,45% danske kort (min 0,60 kr), 2,75% udenlandske. Ingen måneds- eller oprettelsesgebyr. Godkendelse på 1–3 dage | **Nej** | Nej | Billig og hurtig at komme i gang med, men ingen Dankort og ingen gateway — skal kombineres med Quickpay/Pensopay |
| **Pensopay** | Gateway + Clearhaus som primær indløser | 129–149 kr/md (250 tx inkl.), derefter 0,25 kr + 1,25% indløsning. Dankort kræver separat Nets-aftale | Kun via Nets-aftale | Ja | OK, men dyrere og mere komplekst end Quickpay+Clearhaus ved lav volumen |
| **Scanpay** | Lille dansk PSP | 0,25 kr/tx + indløsningsgebyr (detaljer mindre transparente på prissiden) | Kun via Nets-aftale | Ja | Lovende pris for små beløb, ingen månedsgebyr — men mindre dokumenteret, mindre track record |
| **Stripe** | Alt-i-én (international) | ~1,5% + 1,80 kr for EU-kort | **Nej** (kun Visa-co-brand; "Stripe currently doesn't have support for the Dankort card network") | Ja (som wallet; fast 35 kr/md MobilePay-medlemskab) | Fravalg: dyrt for små beløb, ingen rene Dankort, moderne JS i Checkout øger webview-risiko |
| **Vipps MobilePay ePayment (CARD-flow)** | Nordisk wallet/cards | Ukendt for almindelige merchants — dokumentation siger CARD er "PSP only" | Delvist (Dankort ikke tokeniserbart uden særlig nøgle) | Via landing page | Usikkert — kræver skriftlig afklaring med Vipps MobilePay |

### Nøglefakta om Dankort

- Nets har **monopol** på indløsning af rene Dankort i Danmark — alle andre udbydere skal have en særskilt Nets-aftale for at tage dem.
- Nets' Dankort-onlinepris er reguleret og lav: ca. **0,31–0,36% + fast gebyr** (tidligere 0,54 kr + 0,19%, max 2,50 kr — loftet er fjernet).
- Kun ca. 2–5% af danske kort er rene Dankort; resten er co-branded og kører via Visa.

## Beregningseksempel: betaling på 25 kr.

| Løsning | Estimeret samlet gebyr |
|---|---|
| Nets Easy, Visa/Dankort | ~0,31% + 0,54 kr ≈ **0,62 kr** |
| Nets Easy, MC/Visa debet | 0,99% + 0,50 kr ≈ **0,75 kr** |
| Quickpay + Clearhaus | 0,25 + 1,45% (min 0,60) ≈ **0,85 kr** (+ 99 kr/md) |
| Scanpay | 0,25 + ~1,25% ≈ **0,56 kr** (uden månedsgebyr) |
| Pensopay | 0,25 + 1,25% ≈ **0,56 kr** (+ 129–149 kr/md) |
| Stripe | 1,5% + 1,80 kr ≈ **2,18 kr** |

Ved små parkeringsbeløb er forskellen markant: Stripe koster 3–4× så meget som Nets Easy på Dankort/Visa.

## Anbefaling

**Primær: Nets Easy (Nexi Checkout)**
- Én aftale giver Dankort, Visa, Mastercard, MobilePay og Vipps — alt hvad en dansk parkeringskunde forventer.
- Hosted betalingsvindue via redirect passer arkitekturen (webview/ekstern browser + backend-polling).
- Nets' checkout-side er teknisk konservativ (ældre HTML/JS) — langt bedre odds i Gecko ESR91 end moderne React-checkouts.
- Dansk support, dansk juridisk enhed.

**Sekundær/alternativ: Quickpay + Clearhaus (+ Nets Dankort-aftale)**
- Hvis der forventes høj volumen og vil optimeres på acquirer-gebyret, eller ønskes mulighed for at skifte indløser uden at skifte gateway.
- Quickpay understøtter også Vipps MobilePay direkte (1 kr/tx) — relevant hvis nordiske brugere skal kunne betale med Vipps uden app-switch.

**Budget-alternativ: Scanpay** — hvis lav volumen og 0 kr. i månedsgebyr vejer tungere end Nets' fulde metode-dækning. Bemærk at ren Dankort stadig kræver separat Nets-aftale hos alle andre end Nets selv.

**Fravalg: Stripe** (ingen rene Dankort, dyr ved små beløb, webview-risiko) og **Vipps ePayment CARD** (utilgængeligt for almindelige merchants ifølge dokumentationen — se notat 2).

## Sailfish-specifikke bemærkninger

1. **Brug hosted redirect-flow, ikke embedded card fields.** PCI-kravene ved embedded fields er tunge, og gamle webviews slås med moderne JS. Hosted side = ingen kortdata på jeres server (SAQ A).
2. **Åbn betalingen i ekstern Sailfish Browser som fallback** hvis den indlejrede webview fejler — Nets' side bør virke i Sailfish Browser (Gecko ESR91), som bør testes først.
3. **Status via webhook + polling fra backend** — redirect tilbage til appen er ikke pålidelig (og custom URL schemes er ustabile på Sailfish).
4. **Overvej kommunale krav**: mange danske kommuner/parkeringsoperatører (EasyPark, ParkOne, Q-Park m.fl.) har egne betalingsaftaler og API'er. Hvis appen skal integreres med en specifik parkeringszone-udbyder, kan deres krav til betalingsløsning afgøre valget.

## Næste skridt

1. Kontakt Nets og bed om en erhvervsaftale på Nets Easy med Dankort aktiveret — spørg specifikt om hosted-vinduets browserkompatibilitet.
2. Få tilbud fra Quickpay+Clearhaus og Scanpay til sammenligning.
3. Test Nets' hosted betalingsvindue i Sailfish Browser (ESR91) med et dansk testkort — det er go/no-go for hele betalingsdelen.

## Kilder

- [Nets Easy developer docs (Nexi)](https://developer.nexigroup.com/nets-easy/en-EU/api/payment-v1/)
- [Nets Easy — MobilePay](https://developer.nexigroup.com/nets-easy/en-EU/docs/mobilepay/)
- [Nets Easy — Dankort](https://developer.nexigroup.com/nets-easy/en-EU/docs/dankort/)
- [Quickpay priser](https://quickpay.net/prices-dk/)
- [Quickpay — Nets som indløser](https://quickpay.net/helpdesk/acquirers/nets/)
- [Quickpay — indløsningsaftale](https://quickpay.net/dk/helpdesk/acquirer-agreement/)
- [Clearhaus priser](https://www.clearhaus.com/pricing/)
- [Pensopay priser](https://pensopay.com/en/pricing/) og [indløsningsaftale-priser](https://help.pensopay.com/hc/da/articles/11693462727581-Hvad-koster-en-indl%C3%B8sningsaftale)
- [Scanpay priser](https://scanpay.dk/priser) og [API-docs](https://docs.scanpay.dev/)
- [Stripe priser Danmark](https://stripe.com/en-dk/pricing) og [MobilePay via Stripe](https://docs.stripe.com/payments/mobilepay)
- [Dankort priser](https://dankort.dk/priser)
- [Frisbii — Dankort kræver Nets-aftale](https://help.frisbii.com/docs/danish-payment-card-dankort)
- Sammenligningssider: [erhvervskontopris.dk](https://erhvervskontopris.dk/indloesningsaftale/), [ai0.dk](https://ai0.dk/den-komplette-guide-til-betalingsloesninger-og-indloesningsaftaler-i-danmark/), [advisoa.dk](https://www.advisoa.dk/blog/billigste-betalingslosning-til-din-webshop-find-den-mest-omkostningseffektive-losning)

## Se også

- [Notat 1: Vipps web-checkout på Sailfish](01-vipps-web-checkout-sailfish.md)
- [Notat 2: uden MobilePay-appen på Sailfish](02-vipps-uden-mobilepay-app.md)
