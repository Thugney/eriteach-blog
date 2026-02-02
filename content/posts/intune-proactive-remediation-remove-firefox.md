---
title: "Remove Firefox from All Devices with Intune Proactive Remediation"
date: 2026-02-02
draft: false
tags: ["intune", "proactive-remediation", "powershell", "defender"]
categories: ["How-To"]
summary: "Clean up unauthorized Firefox installations across your organization using Intune remediations."
---

## The Problem

Before we standardized on Edge, users could download any browser they wanted. Now we had Firefox everywhere.

I checked Defender's software inventory:

![Defender showing 800+ devices with various Firefox versions](/images/posts/defender-firefox-inventory.png)

800+ devices. 19 versions. Some with vulnerabilities. No way I'm touching each one manually.

## The Solution

Intune Proactive Remediation with two scripts:
- **Detection**: Find all Firefox installations (registry, Program Files, user profiles)
- **Remediation**: Remove everything - processes, files, shortcuts, services, scheduled tasks

## Detection Script

Firefox can hide in a few places:

1. **Registry** - Both 64-bit and 32-bit uninstall keys, plus per-user installs
2. **Program Files** - Standard install locations
3. **User Profiles** - Per-user installations in AppData

```powershell
$findings = @()

# Registry - check all uninstall locations
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

# Exit 1 if Firefox found, 0 if clean
if ($findings.Count -gt 0) { exit 1 } else { exit 0 }

# Full script: https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/firefox-removal-detection.ps1
```

## Remediation Script

The remediation nukes everything:

1. **Stop all Firefox processes** - Including helper processes like plugin-container and updater
2. **Uninstall via registry** - Uses the uninstall string from registry (handles both EXE and MSI installs)
3. **Remove directories** - Program Files, ProgramData, and user AppData folders
4. **Delete shortcuts** - Desktop and Start Menu
5. **Remove services** - MozillaMaintenance service
6. **Clean scheduled tasks** - Firefox and Mozilla update tasks

```powershell
# Stop Firefox processes first
$firefoxProcesses = @("firefox", "firefox-esr", "plugin-container", "crashreporter", "updater")
foreach ($proc in $firefoxProcesses) {
    Get-Process -Name $proc -ErrorAction SilentlyContinue | Stop-Process -Force
}
Start-Sleep -Seconds 2

# Uninstall using registry uninstall string
# Handles both helper.exe (standard) and msiexec (MSI) installs
if ($uninstallString -match 'helper\.exe') {
    Start-Process -FilePath $helperPath -ArgumentList "/S" -Wait -NoNewWindow
}

# Full script: https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/firefox-removal-remediation.ps1
```

The script outputs detailed results for Intune reporting:

```
====== FIREFOX REMOVAL RESULTS ======
Timestamp: 2026-02-02 10:30:15
Computer: PC-SCHOOL-042

REMOVED (8 items):
  [OK] Stopped process: firefox (1 instance(s))
  [OK] Uninstalled: Mozilla Firefox (x64 en-US)
  [OK] Removed directory: C:\Program Files\Mozilla Firefox
  [OK] Removed user data (student01): C:\Users\student01\AppData\Roaming\Mozilla\Firefox
  [OK] Removed shortcut: Firefox.lnk
  [OK] Removed service: MozillaMaintenance
  [OK] Removed task: Firefox Default Browser Agent

====== END OF REPORT ======
```

## Deploy in Intune

### 1. Create the Remediation

1. Go to **Intune admin center** > **Devices** > [**Scripts and remediations**](https://intune.microsoft.com/#view/Microsoft_Intune_DeviceSettings/DevicesMenu/~/intents)
2. Click **Create script package**
3. Name it "Firefox Removal"

### 2. Upload Scripts

1. Upload the detection script
2. Upload the remediation script
3. Configure:
   - Run script in 64-bit PowerShell: **Yes**
   - Run this script using the logged-on credentials: **No** (runs as SYSTEM)

### 3. Assign and Schedule

1. Assign to device groups (or All Devices if you want full cleanup)
2. Set schedule - I ran it daily until the count dropped to zero

### 4. Monitor Progress

Check results in **Devices** > **Scripts and remediations** > **Firefox Removal** > **Device status**

![Intune remediation status showing devices being cleaned](/images/posts/intune-firefox-remediation-status.png)

You'll see devices move from "With issues" (Firefox found) to "Without issues" (clean) as the remediation runs.

## What to Watch Out For

- **Firefox Developer Edition** - The scripts handle this too, but verify if you have developers who need it
- **User profile cleanup** - The script removes Firefox data from all user profiles. Warn users they'll lose bookmarks and saved passwords
- **Running Firefox** - The script force-closes Firefox. Users will lose unsaved work in open tabs

## Related Links

- [Auto-Update Firefox with Intune](/posts/intune-proactive-remediation-firefox-update/) - If you need to keep Firefox but ensure it's updated
- [Intune Remediations overview](https://learn.microsoft.com/en-us/mem/intune/fundamentals/remediations)
- [Microsoft Defender software inventory](https://learn.microsoft.com/en-us/defender-endpoint/software-inventory)
