# Teknisk Granskning: OPARKO og Sailfish OS-app til parkering i Danmark

**Projekt:** Stalla – Sailfish OS 5 parkeringsapp
**Dato:** 2026-02-05
**Status:** Forskningsrapport

---

## 1. Executive Summary

Denne rapport gransker [OPARKO](https://oparko.com) (tidligere Parkeringskompagniet) som teknisk grundlag for en Sailfish OS-app til betaling af parkering i Danmark. Rapporten fokuserer på:

1. Hvad OPARKO tilbyder teknisk (platform, app, API)
2. Hvad der kræves for at bygge en tilsvarende Sailfish OS-app
3. Tekniske udfordringer og begrænsninger på Sailfish OS 5
4. Alternative tilgange (åbne data, andre operatører)

**Konklusion:** En Sailfish OS-app til parkering er teknisk mulig, men står over for betydelige udfordringer: OPARKOs API er lukket/B2B, Sailfish OS 5 bruger Qt 5.6 (fra 2016), og betalingsintegration (MobilePay) kræver særlig opmærksomhed på en sandboxed platform.

---

## 2. OPARKO – Teknisk Analyse

### 2.1 Virksomhed og Platform

| Aspekt | Detaljer |
|--------|----------|
| **Firma** | OPARKO A/S (tidligere Parkeringskompagniet ApS) |
| **CVR** | DK-37127485 |
| **Adresse** | Trindsøvej 4, 8000 Aarhus C, Danmark |
| **Grundlagt** | 2015 |
| **Platform** | Cloud-baseret SaaS (Software as a Service) |
| **Webportal** | [park.oparko.com](https://park.oparko.com) |

### 2.2 App-økosystem

OPARKO har flere apps på Google Play og Apple App Store:

| App | Package ID | Formål |
|-----|-----------|--------|
| **OPARKO** | `com.parkeringskompagniet.parkeringskompagniet` | Administration: gæstetilladelser, kontrol, p-vagter |
| **KK p-kontrol** | `com.parkeringskompagniet.parkeringskompagniet.cph` | Københavns Kommune specifik kontrol-app |
| **Parkeringskompagniet** | `dk.saleslock.beboerparkering` | Beboerparkering (white-label) |
| **OPARKO** (iOS) | `id6446219531` | iOS-version af admin-appen |

**Vigtig observation:** OPARKO-appen er primært et **B2B-værktøj** for parkeringsadministratorer og p-vagter – ikke en forbruger-app til at betale for parkering på kommunale gader.

> "You must have an agreement with OPARKO and a login to use the app, and you cannot create an account directly in the app." – [Google Play](https://play.google.com/store/apps/details?id=com.parkeringskompagniet.parkeringskompagniet)

### 2.3 Tekniske Funktioner

Fra offentlig dokumentation og app-beskrivelser:

- **ANPR (Automatic Number Plate Recognition):** Kameraer læser nummerplader ved ind/udkørsel
- **P-tilladelser:** Digitale gæste- og beboertilladelser
- **Betaling:** Integration med betalingsudbydere (ikke specificeret hvilke)
- **Håndhævelse:** P-vagt app til at udstede kontrolafgifter (geo-tagged, fotodokumentation)
- **Rapportering:** Dashboard med statistik og eksport
- **API:** "Open REST API" nævnes, men **ingen offentlig dokumentation fundet**

### 2.4 API-status – Kritisk Fund

| Aspekt | Status |
|--------|--------|
| Offentlig API-dokumentation | ❌ Ingen fundet |
| Developer portal | ❌ Ingen fundet |
| OpenAPI/Swagger spec | ❌ Ingen fundet |
| GitHub organisation | ❌ Ingen fundet for OPARKO specifikt |
| Kontakt for API-adgang | ✅ `service@oparko.com` / `+45 88 77 31 50` |

OPARKO markedsfører en "open REST API", men denne ser ud til at være **lukket og kun tilgængelig for betalende B2B-kunder** med en kommerciel aftale. Der er ingen offentlig dokumentation, sandbox eller self-service registrering.

### 2.5 Web-platform

- **park.oparko.com** – Hovedportal for parkeringsadministration
- Sproglige versioner: dansk, engelsk, svensk, norsk, tysk
- Understøtter light/dark theme
- White-label kapacitet (kan brandes til forskellige kunder)

---

## 3. Det Danske Parkeringsapp-økosystem

### 3.1 Godkendte apps i København (officiel liste)

Fra [Københavns Kommune](https://www.kk.dk/borger/parkering-trafik-og-veje/parkering/parkeringsapps):

| App | Udbyder | Gebyr | Betalingsmetoder |
|-----|---------|-------|------------------|
| **EasyPark** | EasyPark Group | 15% (min. 4 kr.) | MobilePay, Dankort, Visa/MC, Apple Pay |
| **APCOA FLOW** | APCOA Parking Danmark | ~10% | MobilePay, kort, Apple Pay, Google Pay |
| **ParkMan** | ParkMan (nu ParkMobile?) | Procent + fast gebyr | Kort, MobilePay |
| **OK-appen** | OK a.m.b.a. | 15% (min. 4 kr.) | OK Kort, MobilePay, Apple Pay, Dankort, Visa/MC |
| **PARKPARK** | PARKPARK | Fast gebyr | MobilePay, Dankort, kreditkort |
| **ParkOne** | ParkOne | 10% (min. 4 kr.) | MobilePay, kort |
| **Legal Parking** | Innova Apps | 15% (min. 4 kr.) | Kort, MobilePay |

### 3.2 Kommunale forskelle

| Kommune | Godkendte apps |
|---------|---------------|
| København | EasyPark, APCOA FLOW, ParkMan, OK, PARKPARK, ParkOne, Legal Parking |
| Frederiksberg | ParkMan, OK, EasyPark, PARKPARK, ParkOne, APCOA FLOW |
| Aarhus | ParkMan, OK, EasyPark, PARKPARK, ParkOne |
| Odense | ParkMan, OK, EasyPark, ParkOne |
| Aalborg | ParkMan, OK, EasyPark, PARKPARK, ParkOne, Park 4 U, FreeSpot |

### 3.3 Åbne Data – Alternativ vej

[Open Data DK](https://www.opendata.dk) tilbyder åbne parkeringsdata:

- **Københavns Kommune:** Parkeringszoner, parkeringspladser, parkeringshuse (WFS/GeoJSON)
- **Aarhus Kommune:** Parkeringsdata fra sensorer (1000+ pladser på Frederiksbjerg)
- **Vejle Kommune:** Live parkeringsdata API

**Bemærk:** Disse data er **kun information** (placering, priser, ledighed) – ikke betaling.

### 3.4 EasyPark Developer Portal

EasyPark har en [offentlig developer portal](https://developer.easyparkgroup.com/), men:

> "Currently, all EasyPark APIs are only available with commercial agreement."

---

## 4. Sailfish OS 5 – Teknisk Platform

### 4.1 Grundlæggende Specifikationer

| Aspekt | Version/Detalje |
|--------|-----------------|
| **Qt version** | **5.6** (frigivet 2016, EOL marts 2019) |
| **UI framework** | Sailfish Silica (proprietær QML modul) |
| **Programmeringssprog** | QML + JavaScript (UI), C++ (backend), Python (via PyOtherSide) |
| **Wayland** | Ja (erstatter X11) |
| **Sandboxing** | Sailjail (Firejail-baseret) siden 4.4.0 |
| **Pakkeformat** | RPM |
| **App store** | Jolla Harbour (officiel), OpenRepos, Chum (community) |

### 4.2 Udviklingsværktøjer

- **Sailfish SDK:** Qt Creator-baseret IDE med VirtualBox/Docker build engine
- **Cross-compilation:** ARM (armv7hl, aarch64) og x86 (i486, emulator)
- **Emulator:** VirtualBox-baseret Sailfish OS VM
- **Deployment:** RPM-pakker via `sfdk` tool

### 4.3 Netværk og HTTP

Sailfish OS bruger Qt 5.6's netværksstack:

| Funktion | Implementation |
|----------|---------------|
| HTTP/HTTPS client (QML) | `XMLHttpRequest` JavaScript objekt |
| HTTP/HTTPS client (C++) | `QNetworkAccessManager` |
| JSON parsing | Indbygget `JSON.parse()` / `JSON.stringify()` |
| WebSocket | Qt 5.6 WebSocket modul (begrænset) |

**Kendte begrænsninger med XMLHttpRequest på Sailfish OS:**
- POST requests med JSON body kan fejle (Content-Type header problemer)
- Ingen `fetch` API (ældre JavaScript engine)
- Ingen native Promise/async-await support
- Begrænsede HTTP methods (PROPFIND understøttes ikke)

### 4.4 SSL/TLS og Certifikater

| Aspekt | Detalje |
|--------|----------|
| OpenSSL version | 1.1.x (siden Sailfish OS 4.0.1) |
| TLS version | TLS 1.2 understøttet |
| Certifikathåndtering | System CA store (`/etc/pki/tls/certs/`) |
| Kendte problemer | Qt 5.6 + OpenSSL kompatibilitet kan give "TLS initialization failed" |

**Kritisk for betalingsapps:** Korrekt SSL/TLS er afgørende for sikker kommunikation med betalingsgateways.

### 4.5 Sailjail Sandboxing og Permissions

Siden Sailfish OS 4.4.0 kører alle apps i en sandbox. Permissions deklareres i `.desktop` filen:

```ini
[Desktop Entry]
Type=Application
Name=Stalla
Icon=harbour-stalla
Exec=harbour-stalla

[X-Sailjail]
Permissions=Internet;Location
OrganizationName=org.stalla
ApplicationName=Stalla
```

**Tilgængelige permissions inkluderer:**
- `Internet` – Netværksadgang
- `Location` – GPS/positionering
- `Camera` – Kameraadgang
- `Documents`, `Pictures`, `Videos`, `Music` – Filsystemadgang
- `Bluetooth`, `NFC`, etc.

**Automatisk whitelisted mapper:**
- `$HOME/.local/share/<org>/<app>`
- `$HOME/.cache/<org>/<app>`
- `$HOME/.config/<org>/<app>`

### 4.6 Positionering (GPS)

Sailfish OS bruger **GeoClue** framework:

```qml
import QtPositioning 5.0

PositionSource {
    id: positionSource
    active: true
    updateInterval: 1000
    onPositionChanged: {
        console.log("Latitude:", position.latitude)
        console.log("Longitude:", position.longitude)
    }
}
```

**Kræver pakker:** `qt5-qtpositioning`, `qt5-qtdeclarative-import-positioning`

### 4.7 Lokal Lagring

| Metode | Brug |
|--------|------|
| `QtQuick.LocalStorage` | SQLite database via QML |
| `Nemo.Configuration` | DConf-baserede indstillinger |
| `Sailfish.Secrets` | Sikker lagring af tokens/credentials |

**Vigtigt:** Sailjail begrænser skriveadgang til app-specifikke mapper.

### 4.8 Notifikationer

- **Nemo Notifications** plugin til lokal notifikationer
- **Ingen centraliseret push-notifikationssystem** (i modsætning til Firebase/APNs)
- Hver app skal vedligeholde egen baggrundsforbindelse (WebSocket, MQTT, polling)
- D-Bus aktivering mulig for baggrundsservices (siden 4.5.0)

### 4.9 Harbour (Jolla Store) Krav

| Krav | Detalje |
|------|----------|
| Kun tilladte APIs | [Harbour Allowed APIs](https://docs.sailfishos.org/Develop/Apps/Harbour/Allowed_APIs/) |
| Kun tilladte permissions | [Allowed Permissions](https://docs.sailfishos.org/Develop/Apps/Harbour/Allowed_Permissions/) |
| RPM validator | Kører automatisk ved deploy |
| Ingen binære blobs | Kun open source biblioteker tilladt |
| Open source licens | GPL-3.0 er OK (repo bruger denne) |
| Filrettigheder | Filer må ikke være executable (755 → 644) |
| Ikoner | 86×86, 108×108, 128×128, 172×172 px |

---

## 5. Betalingsintegration – MobilePay og Kort

### 5.1 MobilePay API

MobilePay (nu Vipps MobilePay) tilbyder udvikler-API:

- **Developer portal:** [developer.vippsmobilepay.com](https://developer.vippsmobilepay.com/)
- **ePayment API:** Online betalinger
- **App Switch flow:** Redirect til MobilePay app for godkendelse
- **Krav:** Kommerciel aftale som PSP (Payment Service Provider) eller merchant

**For en uafhængig app:** Direkte MobilePay integration kræver kommerciel aftale. Alternativt kan app'en deep-linke til MobilePay app.

### 5.2 Kortbetaling

For Dankort/Visa/Mastercard kræves typisk:
- PCI-DSS compliance
- Integration med betalingsgateway (Nets, Bambora, Stripe, Adyen)
- **Høj kompleksitet** for en lille open source app

### 5.3 Web-baseret alternativ

En Sailfish OS app kunne åbne en **WebView** til eksisterende betalingsportal – men Sailfish OS har begrænset WebView support (Qt WebEngine er ikke tilladt i Harbour).

---

## 6. Strategiske Veje for en Sailfish OS Parkeringsapp

### 6.1 Vej A: OPARKO API-integration (kræver samarbejde)

**Forudsætninger:**
1. Kommerciel aftale med OPARKO om API-adgang
2. API nøgler og dokumentation
3. Godkendelse som betalingsapp (hvis relevant)

**Teknisk implementation:**
```
Sailfish OS App (QML/C++)
    ↓ HTTPS/REST
OPARKO Backend API
    ↓
Betalingsgateway (MobilePay/kort)
```

**Udfordringer:**
- Lukket API – ingen offentlig dokumentation
- Ukendt om OPARKO ønsker tredjepartsapps
- B2B fokus – måske ikke interesseret i forbrugerapps

### 6.2 Vej B: Multi-operatør app (EasyPark, APCOA, etc.)

**Forudsætninger:**
1. Kommercielle aftaler med flere operatører
2. Hver operatørs API skal integreres separat
3. Kompleks vedligeholdelse

**Udfordringer:**
- EasyPark API kræver kommerciel aftale
- Forskellige API'er for hver operatør
- Meget høj kompleksitet

### 6.3 Vej C: Åbne data + vejledning (realistisk MVP)

**Funktioner:**
- Find parkeringszoner via Open Data DK
- Vis priser og regler
- GPS-baseret kort over nærliggende parkering
- Link til eksisterende betalingsapps (EasyPark, APCOA FLOW, etc.)

**Fordele:**
- Ingen kommercielle aftaler nødvendige
- Kan bygges med åbne data med det samme
- Nyttig for Sailfish OS brugere

**Kodeeksempel – Open Data DK API:**
```javascript
// Hent parkeringszoner fra Københavns Kommune
function fetchParkingZones() {
    var xhr = new XMLHttpRequest()
    xhr.onreadystatechange = function() {
        if (xhr.readyState === XMLHttpRequest.DONE && xhr.status === 200) {
            var data = JSON.parse(xhr.responseText)
            // Opdater UI med zoner
        }
    }
    xhr.open("GET", "https://api.opendata.dk/...", true)
    xhr.send()
}
```

### 6.4 Vej D: Reverse-engineered API (ikke anbefalet)

- Dekompilering af Android APK for at finde API endpoints
- Juridisk gråzone
- Ulovligt efter dansk/EU lovgivning uden samtykke
- **Anbefales ikke**

---

## 7. Teknisk Arkitektur foreslået (Vej C MVP)

```
┌─────────────────────────────────────┐
│         Sailfish OS App             │
│         (harbour-stalla)            │
├─────────────────────────────────────┤
│  QML UI (Sailfish Silica)           │
│  ├── Kort (QtLocation/OSM)          │
│  ├── Zone-liste                     │
│  ├── Prisberegner                   │
│  └── Indstillinger                  │
├─────────────────────────────────────┤
│  C++ Backend (valgfrit)             │
│  ├── NetworkManager (API calls)     │
│  ├── Positioning (GPS)              │
│  └── Database (SQLite cache)        │
├─────────────────────────────────────┤
│  Sailjail Sandbox                   │
│  Permissions: Internet, Location    │
└─────────────────────────────────────┘
         ↓ HTTPS
┌─────────────────────────────────────┐
│  Open Data DK (kommunale data)      │
│  - Parkeringszoner                  │
│  - Priser                           │
│  - Live ledighed (hvor tilgængeligt)│
└─────────────────────────────────────┘
         ↓ Deep links
┌─────────────────────────────────────┐
│  Eksisterende betalingsapps         │
│  - EasyPark                         │
│  - APCOA FLOW                       │
│  - ParkMan                          │
│  - OK-appen                        │
└─────────────────────────────────────┘
```

---

## 8. Kritiske Tekniske Udfordringer på Sailfish OS

### 8.1 Qt 5.6 Begrænsninger

| Problem | Konsekvens | Løsning |
|---------|-----------|---------|
| Gammel JavaScript engine | Ingen `fetch`, `async/await`, `Promise` | Brug `XMLHttpRequest` callbacks |
| XMLHttpRequest POST problemer | JSON body kan fejle | Brug C++ `QNetworkAccessManager` eller form-encoded data |
| Ingen Qt WebEngine i Harbour | Ingen indlejret browser | Deep links til eksterne apps |
| Qt 5.6 SSL quirks | TLS forbindelsesfejl | Test grundigt, overvej C++ backend |

### 8.2 Sandboxing (Sailjail)

| Problem | Konsekvens | Løsning |
|---------|-----------|---------|
| Internet permission nødvendig | App blokeres uden | Deklarer `Permissions=Internet` |
| Location permission | GPS ikke tilgængelig | Deklarer `Permissions=Location` |
| Filesystem begrænset | Kun app-mapper skrivbare | Brug `$HOME/.local/share/org.stalla/stalla` |
| Ingen baggrundsarbejde | Polling stopper når app lukkes | D-Bus aktivering eller accepter begrænsning |

### 8.3 Betalingssikkerhed

- **PCI-DSS:** Direkte korthåndtering kræver compliance – undgås ved at bruge MobilePay/app-switch
- **Token storage:** Brug `Sailfish.Secrets` API til sikker lagring af credentials
- **HTTPS:** Altid – Qt 5.6 understøtter TLS 1.2

---

## 9. Anbefalinger

### 9.1 Kortsigtet (MVP – 1-3 måneder)

1. **Byg en informationsapp** med åbne data fra Open Data DK
2. **Funktioner:** Kort over parkeringszoner, priser, regler, GPS-position
3. **Deep links** til eksisterende betalingsapps (EasyPark, APCOA FLOW, etc.)
4. **Teknologi:** 100% QML + JavaScript, ingen C++ backend nødvendig
5. **Distribution:** OpenRepos eller Chum (community) først

### 9.2 Mellemlangt sigt (3-12 måneder)

1. **Kontakt OPARKO** om API-samarbejde – spørg specifikt om:
   - Offentlig eller partner-API til tredjepartsapps
   - Mulighed for white-label Sailfish OS app
   - Betalingsintegration (MobilePay)

2. **Kontakt EasyPark** via deres developer portal om partner-API

3. **Overvej Harbour** submission når app er stabil

### 9.3 Langsigtet (1-2 år)

1. **Multi-operatør integration** hvis kommercielle aftaler falder på plads
2. **Direkte betaling** i app (kræver PCI-compliance og betalingsaftaler)
3. **Offline support** med cached zone-data

---

## 10. Næste Skridt

| Prioritet | Opgave | Estimat |
|-----------|--------|---------|
| 🔴 Høj | Sæt Sailfish SDK op, byg "Hello World" app | 1 dag |
| 🔴 Høj | Undersøg Open Data DK API'er for parkering | 2-3 dage |
| 🟡 Medium | Kontakt OPARKO (service@oparko.com) om API-adgang | 1 dag + svar |
| 🟡 Medium | Kontakt EasyPark om partner API | 1 dag + svar |
| 🟢 Lav | Design UI mockups i Sailfish Silica stil | 3-5 dage |
| 🟢 Lav | Implementer kort med QtLocation + OSM | 1 uge |
| 🟢 Lav | Implementer zone-liste og prisvisning | 1 uge |

---

## 11. Kilder og Referencer

### OPARKO
- [oparko.com](https://oparko.com) – Officiel hjemmeside
- [park.oparko.com](https://park.oparko.com) – Webportal
- [Google Play – OPARKO](https://play.google.com/store/apps/details?id=com.parkeringskompagniet.parkeringskompagniet)
- [App Store – OPARKO](https://apps.apple.com/dk/app/oparko/id6446219531)

### Danske parkeringsapps
- [Københavns Kommune – Parkeringsapps](https://www.kk.dk/borger/parkering-trafik-og-veje/parkering/parkeringsapps)
- [EasyPark Danmark](https://www.easypark.com/da-dk)
- [APCOA FLOW](https://www.apcoa.dk)
- [OK Parkering](https://www.ok.dk/privat/produkter/appen/parkering)
- [ParkeringsInfo.dk – Sammenligning](https://www.parkeringsinfo.dk/guide/bedste-parkeringsapps/)

### Åbne Data
- [Open Data DK](https://www.opendata.dk)
- [KK – Parkeringszoner](https://www.opendata.dk/city-of-copenhagen/parkeringszoner-information)
- [Aarhus – Parkeringsdata](https://www.opendata.dk/city-of-aarhus/parkeringsdata-fra-sensorer)

### Sailfish OS
- [Sailfish OS Documentation](https://docs.sailfishos.org)
- [Sailfish OS Developer](https://sailfishos.org/develop/)
- [Harbour FAQ](https://harbour.jolla.com/faq)
- [Sailjail Permissions](https://github.com/sailfishos/sailjail-permissions)
- [Qt 5.6 Documentation](https://doc.qt.io/qt-5.6/)

### Betaling
- [Vipps MobilePay Developer](https://developer.vippsmobilepay.com/)
- [EasyPark Developer Portal](https://developer.easyparkgroup.com/)

---

## 12. Bilag: Desktop-fil eksempel

```ini
[Desktop Entry]
Type=Application
Name=Stalla
Comment=Parkering i Danmark
Icon=harbour-stalla
Exec=harbour-stalla
X-Nemo-Single-Instance=yes

[X-Sailjail]
Permissions=Internet;Location
OrganizationName=org.stalla
ApplicationName=Stalla
```

## 13. Bilag: QML Netværkseksempel

```qml
import QtQuick 2.0
import Sailfish.Silica 1.0

Page {
    id: page

    function fetchParkingZones() {
        var xhr = new XMLHttpRequest()
        xhr.onreadystatechange = function() {
            if (xhr.readyState === XMLHttpRequest.DONE) {
                if (xhr.status === 200) {
                    var data = JSON.parse(xhr.responseText)
                    console.log("Zones:", data.length)
                    // Opdater model
                } else {
                    console.error("Error:", xhr.status)
                }
            }
        }
        xhr.open("GET", "https://api.opendata.dk/parking-zones", true)
        xhr.setRequestHeader("Accept", "application/json")
        xhr.send()
    }
}
```

---

*Rapport udarbejdet som del af Stalla forskningsprojektet – Sailfish OS 5 parkeringsapp.*
