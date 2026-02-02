---
title: "Fjern Firefox fra alle enheter med Intune Proactive Remediation"
date: 2026-02-02
draft: false
tags: ["intune", "proactive-remediation", "powershell", "defender"]
categories: ["How-To"]
summary: "Rydd opp i uautoriserte Firefox-installasjoner på tvers av organisasjonen med Intune remediations."
---

## Problemet

Før vi standardiserte på Edge kunne brukerne laste ned hvilken nettleser de ville. Nå hadde vi Firefox overalt.

Jeg sjekket Defender sin programvareoversikt:

![Defender viser 800+ enheter med ulike Firefox-versjoner](/images/posts/defender-firefox-inventory.png)

800+ enheter. 19 versjoner. Noen med sårbarheter. Ikke aktuelt å ta hver enhet manuelt.

## Løsningen

Intune Proactive Remediation med to skript:
- **Deteksjon**: Finn alle Firefox-installasjoner (registry, Program Files, brukerprofiler)
- **Remediation**: Fjern alt - prosesser, filer, snarveier, tjenester, planlagte oppgaver

## Deteksjonsskript

Deteksjonsskriptet sjekker tre steder hvor Firefox kan gjemme seg:

1. **Registry** - Både 64-bit og 32-bit avinstalleringsnøkler, pluss per-bruker installasjoner
2. **Program Files** - Standard installasjonsplasseringer
3. **Brukerprofiler** - Per-bruker installasjoner i AppData

```powershell
$findings = @()

# Registry - sjekk alle avinstalleringsplasseringer
$uninstallPaths = @(
    "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall",
    "HKLM:\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall",
    "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall"
)

foreach ($path in $uninstallPaths) {
    if (Test-Path $path) {
        $apps = Get-ItemProperty "$path\*" -ErrorAction SilentlyContinue |
            Where-Object { $_.DisplayName -like "*Firefox*" }
        foreach ($app in $apps) {
            $findings += "Registry: $($app.DisplayName)"
        }
    }
}

# Exit 1 hvis Firefox funnet, 0 hvis rent
if ($findings.Count -gt 0) { exit 1 } else { exit 0 }

# Fullt skript: https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/firefox-removal-detection.ps1
```

## Remedieringsskript

Remedieringen gjør en grundig opprydding:

1. **Stopp alle Firefox-prosesser** - Inkludert hjelpeprosesser som plugin-container og updater
2. **Avinstaller via registry** - Bruker avinstalleringsstrengen fra registry (håndterer både EXE og MSI)
3. **Fjern mapper** - Program Files, ProgramData og brukerens AppData-mapper
4. **Slett snarveier** - Skrivebord og Startmeny
5. **Fjern tjenester** - MozillaMaintenance-tjenesten
6. **Rydd planlagte oppgaver** - Firefox og Mozilla oppdateringsoppgaver

```powershell
# Stopp Firefox-prosesser først
$firefoxProcesses = @("firefox", "firefox-esr", "plugin-container", "crashreporter", "updater")
foreach ($proc in $firefoxProcesses) {
    Get-Process -Name $proc -ErrorAction SilentlyContinue | Stop-Process -Force
}
Start-Sleep -Seconds 2

# Avinstaller med registry uninstall string
# Håndterer både helper.exe (standard) og msiexec (MSI) installasjoner
if ($uninstallString -match 'helper\.exe') {
    Start-Process -FilePath $helperPath -ArgumentList "/S" -Wait -NoNewWindow
}

# Fullt skript: https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/firefox-removal-remediation.ps1
```

Skriptet gir detaljerte resultater for Intune-rapportering:

```
====== FIREFOX REMOVAL RESULTS ======
Timestamp: 2026-02-02 10:30:15
Computer: PC-SKOLE-042

REMOVED (8 items):
  [OK] Stopped process: firefox (1 instance(s))
  [OK] Uninstalled: Mozilla Firefox (x64 en-US)
  [OK] Removed directory: C:\Program Files\Mozilla Firefox
  [OK] Removed user data (elev01): C:\Users\elev01\AppData\Roaming\Mozilla\Firefox
  [OK] Removed shortcut: Firefox.lnk
  [OK] Removed service: MozillaMaintenance
  [OK] Removed task: Firefox Default Browser Agent

====== END OF REPORT ======
```

## Deploy i Intune

### 1. Opprett Remediation

1. Gå til **Intune admin center** > **Devices** > [**Scripts and remediations**](https://intune.microsoft.com/#view/Microsoft_Intune_DeviceSettings/DevicesMenu/~/intents)
2. Klikk **Create script package**
3. Gi den navnet "Firefox Removal"

### 2. Last opp skript

1. Last opp deteksjonsskriptet
2. Last opp remedieringsskriptet
3. Konfigurer:
   - Run script in 64-bit PowerShell: **Yes**
   - Run this script using the logged-on credentials: **No** (kjører som SYSTEM)

### 3. Tilordne og planlegg

1. Tilordne til enhetsgrupper (eller All Devices for full opprydding)
2. Sett tidsplan - jeg kjørte daglig til antallet var null

### 4. Overvåk fremdrift

Sjekk resultater i **Devices** > **Scripts and remediations** > **Firefox Removal** > **Device status**

![Intune remediation-status som viser enheter som ryddes opp](/images/posts/intune-firefox-remediation-status.png)

Du vil se enheter flytte fra "With issues" (Firefox funnet) til "Without issues" (rent) etter hvert som remedieringen kjører.

## Ting å være obs på

- **Firefox Developer Edition** - Skriptene håndterer dette også, men verifiser om du har utviklere som trenger det
- **Brukerprofil-opprydding** - Skriptet fjerner Firefox-data fra alle brukerprofiler. Advar brukerne om at de mister bokmerker og lagrede passord
- **Firefox som kjører** - Skriptet tvangslukker Firefox. Brukere vil miste ulagret arbeid i åpne faner

## Relaterte lenker

- [Auto-oppdater Firefox med Intune](/posts/intune-proactive-remediation-firefox-update/) - Hvis du trenger å beholde Firefox men sikre at den er oppdatert
- [Intune Remediations oversikt](https://learn.microsoft.com/en-us/mem/intune/fundamentals/remediations)
- [Microsoft Defender programvareoversikt](https://learn.microsoft.com/en-us/defender-endpoint/software-inventory)
