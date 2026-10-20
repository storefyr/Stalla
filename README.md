# Stalla

Et forskningsprojekt, der undersøger, hvad det vil kræve at lave en app til Sailfish OS 5, der vil gøre det muligt at betale for parkering i Danmark.

## Projektbeskrivelse

Stalla er et forskningsprojekt med fokus på at kortlægge de tekniske, juridiske og forretningsmæssige krav for at udvikle en parkeringsapp til Sailfish OS 5.

### Mål

- Undersøge eksisterende parkeringsoperatører i Danmark (OPARKO, EasyPark, APCOA, etc.)
- Analysere API'er og integrationsmuligheder
- Kortlægge Sailfish OS 5's tekniske krav og begrænsninger
- Udarbejde en teknisk arkitektur for en prototype

## Dokumentation

- [Teknisk Granskning: OPARKO og Sailfish OS](docs/teknisk-granskning-oparko-sailfish.md) – Grundig teknisk analyse af OPARKO platformen og hvad der skal til for en Sailfish OS-app

## Teknisk Stack (planlagt)

- **Platform:** Sailfish OS 5
- **Sprog:** QML, JavaScript, C++
- **UI Framework:** Sailfish Silica
- **Netværk:** Qt 5.6 QNetworkAccessManager / XMLHttpRequest
- **Kort:** QtLocation + OpenStreetMap
- **Data:** Open Data DK, kommunale API'er
- **Betaling:** MobilePay API (eller deep links til eksisterende apps)

## Status

🔬 **Fase 1: Research** – I gang

- [x] Teknisk granskning af OPARKO
- [ ] Kontakt til parkeringsoperatører
- [ ] Sailfish SDK setup
- [ ] Prototype udvikling

## Licens

Dette projekt er licenseret under [GNU GPL v3](LICENSE).

## Kontakt

Dette er et forskningsprojekt. For spørgsmål, kontakt projektets maintainers.
