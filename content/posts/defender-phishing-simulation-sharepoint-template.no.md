---
title: "Sette opp en phishing-simulering med Defender Attack Simulation Training"
date: 2026-01-31
draft: false
tags: ["defender", "security", "how-to", "attack-simulation"]
categories: ["How-To"]
summary: "Hvordan opprette og distribuere en realistisk SharePoint phishing-simulering ved bruk av Microsoft Defender for Office 365 Attack Simulation Training."
---

## Scenarioet

Sikkerhetsbevissthetstrening fungerer best når brukere opplever realistiske phishing-forsøk i et trygt miljø. Microsoft Defender for Office 365 inkluderer Attack Simulation Training som lar deg sende simulerte phishing-e-poster og spore hvem som klikker.

For denne simuleringen opprettet jeg en SharePoint "massesletting av filer"-varsling - en mal som etterligner legitime Microsoft-e-poster og skaper hastverk uten å være altfor alarmerende.

## Miljø

- Microsoft 365 E5 (eller Defender for Office 365 Plan 2)
- Exchange Online
- Attack Simulation Training aktivert

## Opprette e-postmalen

De mest effektive phishing-simuleringene ligner nøye på legitime e-poster. SharePoint-slettingsvarslingen fungerer godt fordi:

- **Hastverk** - "permanent fjernet", "93 dager" nedtelling
- **Gjenkjennelighet** - Brukere mottar ekte SharePoint-varsler regelmessig
- **Flere klikkemål** - Lenker i brødtekst og CTA-knapp

### Maldesignelementer

Malen inkluderte:
- Microsoft-merkevarebygging (logo, Fluent Design-farger)
- Mørkt tema som matcher gjeldende SharePoint-varsler
- Animerte flytende filikoner for visuell appell
- Tre phishing-lenkeplasseringer:
  - "papirkurv" tekstlenke
  - "sletting og gjenoppretting av filer" hjelpelenke
  - Blå "Papirkurv" CTA-knapp

### HTML-malen

Nedenfor er den komplette HTML-malen brukt for denne simuleringen. Malen inkluderer detaljerte kommentarer som forklarer konfigurasjonsalternativer og indikatorer på kompromittering for treningsformål.

<details>
<summary>Klikk for å utvide full HTML-mal</summary>

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SharePoint - Files Deleted Notification</title>
    <!--
    ============================================================
    PHISHING SIMULATION TEMPLATE - FOR SECURITY AWARENESS TESTING
    ============================================================

    Template: SharePoint Mass File Deletion Notification
    Target: IT Department (Pilot) -> Organization-wide
    Platform: Microsoft Defender for Office 365 - Attack Simulation

    CONFIGURATION NOTES FOR DEFENDER:
    - Payload Type: Credential Harvest or Link in Attachment
    - Landing Page: Microsoft Sign-In (built-in or custom)
    - Tracking: Link click tracking via {{PhishingURL}}

    PERSONALIZATION TOKENS (Defender):
    - {{DisplayName}} - User's display name
    - {{FirstName}} - User's first name
    - {{LastName}} - User's last name
    - {{Email}} - User's email address
    - {{PhishingURL}} - Tracked phishing link

    INDICATORS OF COMPROMISE (for training):
    1. Sender address (will show simulation domain)
    2. Urgency language ("permanently removed", "93 days")
    3. Generic greeting vs personalized
    4. Hover over links to see actual URL
    5. Unexpected notification about file deletion
    ============================================================
    -->
    <!-- CSS og HTML-kode her - se engelsk versjon for full mal -->
</head>
<body>
    <!-- Se engelsk versjon for full HTML-mal -->
</body>
</html>
```

</details>

### Viktige malkomponenter

**Personaliseringstokens** - Malen bruker Defenders innebygde tokens:
- `{{DisplayName}}` i hilsenen gjør den personlig
- `{{PhishingURL}}` på alle klikkbare elementer for sporing

**Flere klikkemål** - Tre separate lenker tester brukeratferd:
1. Inline "papirkurv"-lenke
2. "sletting og gjenoppretting av filer" hjelpelenke
3. Primær CTA-knapp

**Visuell autentisitet** - Microsoft-merkevarebygging, mørkt tema og den kjente SharePoint-varslingsstilen gjør dette overbevisende.

**Bunnteksttilpasning** - Oppdater "Modum kommune" til organisasjonens tenant-navn.

## Distribusjonstrinn

### 1. Gå til Attack Simulation Training

Naviger til [Microsoft Defender-portalen](https://security.microsoft.com) → **Email & collaboration** → **Attack simulation training**

### 2. Opprett ny simulering

1. Klikk **Simulations** → **Launch a simulation**
2. Velg teknikk: **Credential Harvest**
3. Navngi simuleringen din (f.eks. "SharePoint Deletion - IT Pilot")

### 3. Konfigurer payload

Hvis du bruker en egendefinert mal:

1. Gå til **Content library** → **Payloads** → **Create a payload**
2. Velg **Email** som leveringsmetode
3. Velg **Credential Harvest**-teknikk
4. Last opp HTML-malen din

### 4. Bruk personaliseringstokens

Defender støtter dynamiske tokens som gjør e-poster mer overbevisende:

```html
Hi {{DisplayName}},

A large number of files were deleted from your SharePoint...

<a href="{{PhishingURL}}">Go to recycle bin</a>
```

Tilgjengelige tokens:
- `{{DisplayName}}` - Brukerens visningsnavn
- `{{FirstName}}` - Fornavn
- `{{LastName}}` - Etternavn
- `{{PhishingURL}}` - Auto-generert sporet lenke
- `{{Email}}` - Brukerens e-postadresse

### 5. Velg landingsside

For credential harvest-simuleringer, bruk Microsofts innebygde påloggingsside-replika. Dette tester om brukere legger inn legitimasjon på falske påloggingssider.

### 6. Målrett brukere

**Start med en pilotgruppe:**
- Målrett IT-avdelingen først (15-30 brukere)
- Dette tester verktøyet og validerer malen
- Selv IT-ansatte kan klikke - det er verdifull data

### 7. Konfigurer treningstildeling

Når brukere klikker eller sender inn legitimasjon, tildel trening:
- Microsofts innebygde bevissthetskurs
- Egendefinert treningsinnhold
- Anbefalt: "How to recognize phishing"

### 8. Planlegg og start

- Sett umiddelbar start eller planlegg for spesifikt tidspunkt
- Unngå mandager (innboks-overbelastning) og fredager (redusert oppmerksomhet)
- Midt i uken, midt på formiddagen fungerer godt

## Resultater fra IT-pilot

Å kjøre piloten med IT-avdelingen ga nyttig data:

- Noen IT-ansatte klikket på lenkene (beviser at selv teknisk kyndige brukere kan bli lurt)
- Validerer at simuleringsverktøyet fungerer korrekt
- Identifiserer eventuelle leveringsproblemer før masseutrulling
- Skaper interne forkjempere som forstår programmet

## Rulle ut organisasjonsbredt

Etter vellykket pilot:

1. **Gjennomgå pilotmålinger** - Klikkrate, legitimasjonsinnsendingsrate
2. **Juster om nødvendig** - Mal, timing, landingsside
3. **Utvid gradvis** - Avdeling for avdeling eller prosentbasert
4. **Spor trender** - Sammenlign klikkrater over flere kampanjer
5. **Varier maler** - Ikke gjenbruk samme mal gjentatte ganger

## Beste praksis

- **Varsle ledelsen** før du kjører simuleringer
- **Ikke skam klikkere** - Fokuser på opplæring
- **Kjør regelmessig** - Kvartalsvise simuleringer opprettholder bevissthet
- **Varier vanskelighetsgrad** - Bland åpenbare og sofistikerte maler
- **Mål forbedring** - Spor klikkrater over tid

## Målinger å spore

| Måling | Beskrivelse |
|--------|-------------|
| Leveringsrate | E-poster vellykket levert |
| Åpningsrate | Brukere som åpnet e-posten |
| Klikkrate | Brukere som klikket på en lenke |
| Legitimasjonsrate | Brukere som la inn legitimasjon |
| Rapporteringsrate | Brukere som rapporterte som phishing |

## Relaterte lenker

- [Microsoft Docs: Attack simulation training](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-get-started)
- [Simulation techniques explained](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-simulations)
- [Payload automations](https://learn.microsoft.com/en-us/defender-office-365/attack-simulation-training-payload-automations)
