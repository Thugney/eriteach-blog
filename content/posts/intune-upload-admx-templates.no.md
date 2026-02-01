---
title: "Last opp egendefinerte ADMX-maler til Intune"
date: 2026-02-01
draft: false
tags: ["intune", "admx", "how-to", "firefox", "zoom"]
categories: ["How-To"]
summary: "Hvordan importere egendefinerte ADMX/ADML-filer til Intune for tredjepartsapper som Firefox og Zoom."
---

## Problemet

Vi hadde Firefox og Zoom i organisasjonen. Begge trengte å holdes oppdatert, men Intunes Settings Catalog har ikke policyer for disse appene.

For Firefox skrev jeg opprinnelig et [proactive remediation-skript](/no/posts/intune-proactive-remediation-firefox-update/) som sjekket den lokale versjonen mot Mozillas nyeste og auto-installerte oppdateringer. Det fungerte, men det var ekstra kompleksitet å vedlikeholde.

Så oppdaget jeg ADMX-importer. Last opp leverandørens policy-maler til Intune, og du får de samme innstillingene du ville hatt i Group Policy. Mye enklere.

## Miljø

- Windows 11
- Intune

## Hva er ADMX-filer?

ADMX-filer er XML-baserte Group Policy-maler. De definerer hvilke innstillinger som er tilgjengelige. ADML-filer er språkfilene som gir UI-teksten.

Du har sannsynligvis brukt disse i on-prem Group Policy. De samme filene fungerer i Intune.

## Hvor får man ADMX-filer

Vanlige kilder:

| App | Nedlasting |
|-----|------------|
| Firefox | [Mozilla GitHub](https://github.com/AzG-IaC/ADMX-policy-templates) eller inkludert i ESR-installasjonsprogrammet |
| Zoom | [Zoom Admin ADMX Templates](https://support.zoom.us/hc/en-us/articles/360039100051) |
| Chrome | [Chrome Enterprise Bundle](https://chromeenterprise.google/browser/download/) |
| Microsoft Edge | [Last ned fra Microsoft](https://www.microsoft.com/en-us/edge/business/download) |
| Office/M365 | [Office ADMX templates](https://www.microsoft.com/en-us/download/details.aspx?id=49030) |

## Last opp ADMX til Intune

### 1. Skaff ADMX- og ADML-filene dine

For Firefox:

1. Last ned Firefox ADMX-malene fra Mozilla
2. Pakk ut ZIP
3. Du finner:
   - `firefox.admx` - Malfilen
   - `en-US\firefox.adml` - Den engelske språkfilen

### 2. Importer til Intune

1. Gå til **Intune admin center** → **Devices** → **Configuration**
2. Klikk **Import ADMX**-fanen
3. Klikk **Import**
4. Last opp `.admx`-filen
5. Last opp den matchende `.adml`-filen (vanligvis fra `en-US`-mappen)
6. Klikk **Next** og fullfør importen

Importen tar noen minutter. Du vil se statusen endre seg fra "In progress" til "Available".

### 3. Håndter avhengigheter

Noen ADMX-filer avhenger av andre. For eksempel trenger mange Microsoft-maler `windows.admx` som base.

Hvis du får en avhengighetsfeil:
1. Merk hvilken fil den ber om
2. Importer den filen først
3. Prøv deretter den opprinnelige importen på nytt

Vanlige avhengigheter:
- `windows.admx` - Grunnleggende Windows-definisjoner
- `windowscomponents.admx` - Windows-komponentdefinisjoner

### 4. Opprett en policy ved bruk av ADMX

Når den er importert:

1. Gå til **Devices** → **Configuration** → **Create** → **New policy**
2. Velg **Windows 10 and later**
3. Velg **Templates** → **Imported Administrative templates (Preview)**
4. Velg den importerte malen din
5. Konfigurer innstillingene du trenger
6. Tildel til gruppene dine

## Eksempel: Firefox auto-oppdateringspolicy

Etter import av Firefox ADMX:

1. Opprett ny policy ved bruk av den importerte Firefox-malen
2. Naviger til **Mozilla** → **Firefox** → **Updates**
3. Aktiver **Application Update**-innstillingene:
   - `AppAutoUpdate` = Enabled
   - `BackgroundAppUpdate` = Enabled

Dette forteller Firefox å oppdatere automatisk i bakgrunnen - ingen skript nødvendig.

## Eksempel: Zoom auto-oppdateringspolicy

Etter import av Zoom ADMX:

1. Opprett ny policy ved bruk av den importerte Zoom-malen
2. Naviger til **Zoom Meetings** → **General Settings**
3. Konfigurer oppdateringsinnstillinger etter behov

## Verifiser importen

Etter import kan du sjekke:

1. Gå til **Devices** → **Configuration** → **Import ADMX**
2. Malen din skal vise status "Available"
3. Klikk på den for å se alle innstillingene den inneholder

## Ting å passe på

- **Én ADML per import** - Du kan bare laste opp én språkfil. Velg `en-US` med mindre du har et spesifikt behov.
- **Versjonskonflikter** - Hvis du importerer en eldre versjon av en ADMX som allerede er innebygd i Intune, kan du få konflikter. Sjekk om innstillingen finnes i Settings Catalog først.
- **Preview-funksjon** - Imported Administrative Templates er fortsatt i preview. Det fungerer, men UI-et kan endre seg.
- **Behandlingstid** - Store ADMX-filer kan ta 5-10 minutter å behandle.

## Hvorfor ADMX fremfor skript?

Jeg brukte å kjøre et [proactive remediation-skript](/no/posts/intune-proactive-remediation-firefox-update/) for Firefox-oppdateringer. Det fungerte, men:

- Skript trenger vedlikehold når ting endres
- Flere bevegelige deler = flere ting som kan gå i stykker
- ADMX-policyer er innebygde og leverandørstøttet

Hvis leverandøren tilbyr ADMX-maler, bruk dem. Spar skript for når det ikke finnes noe annet alternativ.

## Relaterte lenker

- [Import custom ADMX templates in Intune](https://learn.microsoft.com/en-us/mem/intune/configuration/administrative-templates-import-custom)
- [Firefox enterprise policies](https://support.mozilla.org/en-US/kb/customizing-firefox-using-group-policy-windows)
- [Zoom ADMX templates](https://support.zoom.us/hc/en-us/articles/360039100051)
