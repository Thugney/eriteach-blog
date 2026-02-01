---
title: "Fjern uautoriserte apper med Intune Proactive Remediation"
date: 2026-02-01
draft: false
tags: ["intune", "proactive-remediation", "powershell", "compliance"]
categories: ["Veiledning"]
summary: "Bruk Intune Proactive Remediation til å oppdage og fjerne apper brukere installerte da de hadde administratorrettigheter."
---

## Problemet

Brukere hadde tidligere lokale administratorrettigheter. De installerte hva de ville - Zoom, personlige verktøy, tilfeldig programvare.

Nå har vi låst det ned. Apper kommer kun gjennom Company Portal. Men det gamle ligger fortsatt på enhetene.

På tide å rydde opp.

## Løsningen

Intune Proactive Remediation kjører to skript:
1. **Deteksjon** - Sjekker om appen finnes (exit 1 = funnet, exit 0 = ikke funnet)
2. **Utbedring** - Fjerner den hvis funnet

## Deteksjonsskriptet

Skriptet sjekker registry, WMI og programlisten. Du konfigurerer det ved å endre disse variablene øverst:

```powershell
$AppDisplayName = "Zoom"
$AppPublisher = ""
$AppProductCode = "{86B70A45-00A6-4CBD-97A8-464A1254D179}"
$UsePartialMatch = $true
```

For å finne produktkoden til en app:

```powershell
Get-WmiObject Win32_Product | Format-Table Name, IdentifyingNumber
```

Skriptet logger alt til `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\` for feilsøking.

Komplett skript: [Detect-UnwantedApp.ps1](https://github.com/Thugney/eriteach-scripts/blob/main/proactive-remediation/Detect-UnwantedApp.ps1)

## Utbedringsskriptet

Når appen er oppdaget, avinstallerer utbedringsskriptet appen ved hjelp av avinstalleringsstrengen fra registry eller MSI-produktkoden.

Komplett skript: [Remove-UnwantedApp.ps1](https://github.com/Thugney/eriteach-scripts/blob/main/proactive-remediation/Remove-UnwantedApp.ps1)

## Oppsett i Intune

1. Gå til **Intune** → **Enheter** → **Utbedringer**
2. Klikk **Opprett skriptpakke**
3. Gi den navn: "Fjern Zoom" (eller den aktuelle appen)
4. Last opp:
   - Deteksjonsskript: `Detect-UnwantedApp.ps1`
   - Utbedringsskript: `Remove-UnwantedApp.ps1`
5. Innstillinger:
   - Kjør skript i 64-bit PowerShell: **Ja**
   - Kjør med påloggede legitimasjoner: **Nei** (kjører som SYSTEM)
6. Tilordne til en enhetsgruppe
7. Sett tidsplan (daglig eller hver time avhengig av hvor hastig det er)

## Ting å passe på

- **Test først** - Kjør deteksjon på en pilotgruppe før du aktiverer utbedring
- **Produktkoder endres** - Ulike versjoner av en app kan ha forskjellige koder
- **Risiko med delvis treff** - `$UsePartialMatch = $true` kan treffe apper du ikke hadde tenkt (f.eks. "Zoom" treffer "Zoom Plugin for Outlook")
- **Brukerdata** - Noen apper lagrer brukerdata. Varsle brukere før massefjerning

## Skalering til flere apper

Opprett separate utbedringspakker for hver app, eller modifiser skriptet til å sjekke en liste:

```powershell
$UnwantedApps = @(
    @{Name = "Zoom"; ProductCode = "{86B70A45-00A6-4CBD-97A8-464A1254D179}"},
    @{Name = "TeamViewer"; ProductCode = ""},
    @{Name = "AnyDesk"; ProductCode = ""}
)
```

## Relaterte lenker

- [Microsoft: Proactive Remediations](https://learn.microsoft.com/en-us/mem/intune/fundamentals/remediations)
- [Intune skriptkrav](https://learn.microsoft.com/en-us/mem/intune/apps/intune-management-extension)
