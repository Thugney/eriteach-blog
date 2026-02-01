---
title: "Auto-oppdater Firefox med Intune Proactive Remediation"
date: 2026-02-01
draft: false
tags: ["intune", "proactive-remediation", "powershell", "firefox"]
categories: ["How-To"]
summary: "Bruk Intune proactive remediation til å holde Firefox oppdatert ved å sjekke versjoner og auto-installere oppdateringer."
---

## Problemet

Firefox var rullet ut i organisasjonen, men oppdateringer skjedde ikke konsekvent. Noen maskiner var uker bak på oppdateringer. Vi trengte en måte å tvinge oppdateringer uten brukerintervensjon.

Jeg bygde en proactive remediation som sjekker den installerte versjonen mot Mozillas nyeste og installerer oppdateringer automatisk. Det fungerte, men jeg har siden byttet til [ADMX-maler](/no/posts/intune-upload-admx-templates/) - Firefox leverer policy-maler som håndterer oppdateringer innebygd. Færre bevegelige deler, mindre vedlikehold.

Dette innlegget dekker proactive remediation-tilnærmingen i tilfelle du trenger det for apper som ikke har ADMX-støtte.

## Miljø

- Windows 11
- Intune
- Firefox (standard release)

## Hva er Proactive Remediation?

Proactive remediation (nå kalt Remediations i Intune) kjører to skript:

1. **Deteksjonsskript** - Sjekker om det er et problem
2. **Remediation-skript** - Fikser problemet hvis det oppdages

Intune kjører deteksjonsskriptet på en tidsplan. Hvis det returnerer en ikke-null exit-kode, kjøres remediation-skriptet.

## Deteksjonsskriptet

Sjekker den installerte Firefox-versjonen mot Mozillas nyeste. Returnerer exit-kode 1 hvis oppdatering trengs.

```powershell
<#
.SYNOPSIS
Oppdager om Firefox trenger oppdatering.

.DESCRIPTION
Sammenligner installert Firefox-versjon mot Mozillas produkt-API.
Returnerer exit 1 hvis oppdatering trengs, exit 0 hvis oppdatert.

.NOTES
Author: Eriteach
Version: 1.0
Intune Run Context: System
#>

# Hent installert versjon fra registeret
$firefox = Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" |
    Where-Object { $_.DisplayName -like "Mozilla Firefox*" }

# Hent nyeste versjon fra Mozilla API
$response = Invoke-RestMethod -Uri "https://product-details.mozilla.org/1.0/firefox_versions.json"
$latestVersion = [version]$response.LATEST_FIREFOX_VERSION

# Sammenlign og avslutt deretter
if ($installedVersion -lt $latestVersion) { exit 1 }
exit 0

# Fullt skript: https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/firefox-update-detection.ps1
```

## Remediation-skriptet

Laster ned og installerer nyeste Firefox stille.

```powershell
<#
.SYNOPSIS
Installerer nyeste Firefox-versjon.

.DESCRIPTION
Laster ned Firefox-installasjonsprogram fra Mozilla og kjører stille installasjon.
Rydder opp installasjonsprogrammet etter fullføring.

.NOTES
Author: Eriteach
Version: 1.0
Intune Run Context: System
#>

# Last ned fra Mozillas alltid-nyeste URL
$downloadUrl = "https://download.mozilla.org/?product=firefox-latest&os=win64&lang=en-US"
Invoke-WebRequest -Uri $downloadUrl -OutFile "$env:TEMP\FirefoxSetup.exe" -UseBasicParsing

# Stille installasjon
Start-Process -FilePath "$env:TEMP\FirefoxSetup.exe" -ArgumentList "/S" -Wait

# Fullt skript: https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/firefox-update-remediation.ps1
```

## Distribuer i Intune

### 1. Opprett Remediation

1. Gå til **Intune admin center** → **Devices** → **Remediations**
2. Klikk **Create script package**
3. Navngi det "Firefox Auto-Update"

### 2. Last opp skript

1. Last opp deteksjonsskriptet
2. Last opp remediation-skriptet
3. Konfigurer:
   - Run script in 64-bit PowerShell: **Yes**
   - Run this script using the logged-on credentials: **No** (kjører som SYSTEM)

### 3. Sett tidsplanen

1. Klikk **Assignments**
2. Legg til enhetsgruppene dine
3. Sett tidsplan (jeg brukte daglig)

### 4. Overvåk resultater

Etter distribusjon, sjekk resultater i:

**Devices** → **Remediations** → **Firefox Auto-Update** → **Device status**

Du vil se:
- Enheter hvor Firefox allerede var oppdatert
- Enheter hvor remediation kjørte og oppdaterte Firefox
- Eventuelle feil

## Ting å passe på

- **Språk** - Nedlastings-URL-en bruker `lang=en-US`. Endre dette hvis du trenger et annet språk.
- **Arkitektur** - Skriptet laster ned 64-bit Firefox. Juster `os=win64` til `os=win` for 32-bit.
- **Firefox ESR** - Hvis du bruker ESR, endre produktparameteren til `firefox-esr-latest`.
- **Nettverk** - Enheter trenger internettilgang for å sjekke versjoner og laste ned oppdateringer.
- **Kjørende Firefox** - Installasjonsprogrammet håndterer kjørende instanser, men brukere kan se Firefox restarte.

## Et enklere alternativ

Dette skriptet fungerte bra, men det er vedlikeholdsoverhead. Hver gang Mozilla endrer noe, kan du måtte oppdatere skriptet.

Nå bruker jeg [ADMX-maler](/no/posts/intune-upload-admx-templates/) i stedet. Firefox leverer policy-maler som lar deg konfigurere auto-oppdateringer innebygd gjennom Intune. Ingen skript å vedlikeholde.

Bruk proactive remediation når:
- Appen ikke har ADMX/policy-støtte
- Du trenger tilpasset logikk leverandøren ikke tilbyr
- Du gjør noe utover standardinnstillinger

## Relaterte lenker

- [Intune Remediations overview](https://learn.microsoft.com/en-us/mem/intune/fundamentals/remediations)
- [Firefox enterprise deployment](https://support.mozilla.org/en-US/kb/deploy-firefox-msi-installers)
- [Last opp ADMX-maler til Intune](/no/posts/intune-upload-admx-templates/) - Den enklere tilnærmingen
