---
title: "Automatisk diskrensing med Intune Proactive Remediation"
date: 2026-02-01
draft: false
tags: ["intune", "powershell", "proactive-remediation"]
categories: ["How-To"]
summary: "Stille diskrensing for enheter med begrenset SSD-plass - ingen brukerinteraksjon nødvendig."
---

## Problemet

Vi har student-PC-er med 200GB SSD-er. OneDrive tar seg av dokumentsikkerhetskopiering, men studenter laster ned spill, videoer og andre store filer. Disken fylles opp. Windows begynner å klage. Enhetene blir trege.

Jeg prøvde først Diskopprydding (`cleanmgr.exe`), men den krever GUI-interaksjon. Å kjøre den via Intune som SYSTEM viser ingenting på skjermen - den bare henger og venter på brukerinndata som aldri kommer.

## Løsningen

Jeg bygde en Proactive Remediation med to skript:
- **Detection**: Sjekker om diskplass er under terskel
- **Remediation**: Rydder opp temp-filer, cacher og søppel - helt stille

## Deteksjonslogikk

Deteksjonsskriptet bruker doble terskler. En enhet er ikke-compliant hvis:
- Ledig plass er under **15 GB**, ELLER
- Ledig plass er under **10%**

Dette fungerer godt på tvers av forskjellige diskstørrelser. En 128GB SSD med 10GB ledig (7.8%) utløser remediation. En 500GB disk med 40GB ledig (8%) utløser også. Prosenten fanger store disker, den absolutte verdien fanger små.

```powershell
$MinFreeSpaceGB = 15
$MinFreeSpacePercent = 10

$disk = Get-CimInstance -ClassName Win32_LogicalDisk -Filter "DeviceID='C:'"
$freeSpaceGB = [math]::Round($disk.FreeSpace / 1GB, 2)
$freeSpacePercent = [math]::Round(($disk.FreeSpace / $disk.Size) * 100, 1)

$isCompliant = ($freeSpaceGB -ge $MinFreeSpaceGB) -and ($freeSpacePercent -ge $MinFreeSpacePercent)

# Fullt skript: https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/diskspace-detection.ps1
```

## Hva som renses

Remediation-skriptet mitt renser disse plasseringene stille:

| Plassering | Hva | Aldersfilter |
|------------|-----|--------------|
| `C:\Windows\Temp` | System temp-filer | Alle |
| `C:\Windows\Prefetch` | Prefetch cache | Eldre enn 7 dager |
| `C:\Windows\SoftwareDistribution\Download` | Windows Update cache | Alle |
| `C:\Windows\Logs` | Windows-logger | Eldre enn 14 dager |
| `C:\Users\*\AppData\Local\Temp` | Bruker temp-filer | Alle |
| Papirkurv | Slettede filer | Alle |
| `C:\ProgramData\Microsoft\Windows\WER` | Feilrapporter | Alle |
| Delivery Optimization cache | Oppdateringsdelings-cache | Alle |
| Miniatyrbildebuffer | Explorer-miniatyrbilder | Alle |

Jeg beholder nylige prefetch-filer (siste 7 dager) fordi Windows bruker dem til å fremskynde app-oppstart. Logger eldre enn 14 dager er trygge å fjerne.

For Windows Update cache stopper jeg `wuauserv`-tjenesten først, renser mappen, og starter tjenesten på nytt.

```powershell
Stop-Service -Name wuauserv -Force -ErrorAction SilentlyContinue
Start-Sleep -Seconds 2
# Rens SoftwareDistribution\Download
Start-Service -Name wuauserv -ErrorAction SilentlyContinue

# Fullt skript: https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/diskspace-remediation.ps1
```

## Intune-oppsett

1. Gå til **Intune** > **Devices** > **Remediations**
2. Klikk **Create script package**
3. Navngi det "Disk Space Cleanup"
4. Last opp deteksjonsskriptet
5. Last opp remediation-skriptet
6. Sett **Run this script using the logged-on credentials**: No
7. Sett **Run script in 64-bit PowerShell**: Yes
8. Tildel til en enhetsgruppe (studentenheter)
9. Sett tidsplan - daglig eller hver 8. time avhengig av hvor aggressiv du vil være

## Logging

Remediation-skriptet logger til `C:\ProgramData\Intune\Logs\DiskSpace-Remediation.log`. Du vil se oppføringer som:

```
2026-02-01 10:30:15 - === Starting disk cleanup ===
2026-02-01 10:30:15 - Free space before: 8.45 GB
2026-02-01 10:30:18 - Windows Temp : Freed 245.32 MB
2026-02-01 10:30:22 - User Temp (student01) : Freed 1024.50 MB
2026-02-01 10:30:25 - Recycle Bin: Freed 3500.00 MB
2026-02-01 10:30:26 - === Cleanup completed ===
2026-02-01 10:30:26 - Actual freed: 4850.23 MB
2026-02-01 10:30:26 - Free space after: 13.19 GB
```

## Resultater

I vårt miljø med ~150 studentenheter frigjør typisk opprydding 2-8 GB per enhet. De største gevinstene kommer fra:
- Papirkurv (studenter sletter men tømmer ikke)
- Bruker temp-filer (nettleserbuffer, installasjonsprogrammer)
- Windows Update cache etter funksjonsoppdateringer

## Skript

- [diskspace-detection.ps1](https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/diskspace-detection.ps1)
- [diskspace-remediation.ps1](https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/diskspace-remediation.ps1)
