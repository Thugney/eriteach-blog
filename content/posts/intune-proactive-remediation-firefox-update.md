---
title: "Auto-Update Firefox with Intune Proactive Remediation"
date: 2026-02-01
draft: false
tags: ["intune", "proactive-remediation", "powershell", "firefox"]
categories: ["How-To"]
summary: "Use Intune proactive remediation to keep Firefox updated by checking versions and auto-installing updates."
---

## The Problem

Firefox was deployed across the org, but updates weren't happening consistently. Some machines were weeks behind on patches. We needed a way to force updates without user intervention.

I built a proactive remediation that checks the installed version against Mozilla's latest and installs updates automatically. It worked, but I've since switched to [ADMX templates](/posts/intune-upload-admx-templates/) - Firefox provides policy templates that handle updates natively. Less moving parts, less maintenance.

This post covers the proactive remediation approach in case you need it for apps that don't have ADMX support.

## Environment

- Windows 11
- Intune
- Firefox (standard release)

## What Is Proactive Remediation?

Proactive remediation (now called Remediations in Intune) runs two scripts:

1. **Detection script** - Checks if there's a problem
2. **Remediation script** - Fixes the problem if detected

Intune runs the detection script on a schedule. If it returns a non-zero exit code, the remediation script runs.

## The Detection Script

Checks the installed Firefox version against Mozilla's latest. Returns exit code 1 if update needed.

```powershell
<#
.SYNOPSIS
Detects if Firefox needs updating.

.DESCRIPTION
Compares installed Firefox version against Mozilla's product API.
Returns exit 1 if update needed, exit 0 if up to date.

.NOTES
Author: Eriteach
Version: 1.0
Intune Run Context: System
#>

# Get installed version from registry
$firefox = Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" |
    Where-Object { $_.DisplayName -like "Mozilla Firefox*" }

# Get latest version from Mozilla API
$response = Invoke-RestMethod -Uri "https://product-details.mozilla.org/1.0/firefox_versions.json"
$latestVersion = [version]$response.LATEST_FIREFOX_VERSION

# Compare and exit accordingly
if ($installedVersion -lt $latestVersion) { exit 1 }
exit 0

# Full script: https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/firefox-update-detection.ps1
```

## The Remediation Script

Downloads and installs the latest Firefox silently.

```powershell
<#
.SYNOPSIS
Installs latest Firefox version.

.DESCRIPTION
Downloads Firefox installer from Mozilla and runs silent install.
Cleans up installer after completion.

.NOTES
Author: Eriteach
Version: 1.0
Intune Run Context: System
#>

# Download from Mozilla's always-latest URL
$downloadUrl = "https://download.mozilla.org/?product=firefox-latest&os=win64&lang=en-US"
Invoke-WebRequest -Uri $downloadUrl -OutFile "$env:TEMP\FirefoxSetup.exe" -UseBasicParsing

# Silent install
Start-Process -FilePath "$env:TEMP\FirefoxSetup.exe" -ArgumentList "/S" -Wait

# Full script: https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/firefox-update-remediation.ps1
```

## Deploy in Intune

### 1. Create the Remediation

1. Go to **Intune admin center** > **Devices** > [**Scripts and remediations**](https://intune.microsoft.com/#view/Microsoft_Intune_DeviceSettings/DevicesMenu/~/intents)
2. Click **Create script package**
3. Name it "Firefox Auto-Update"

### 2. Upload Scripts

1. Upload the detection script
2. Upload the remediation script
3. Configure:
   - Run script in 64-bit PowerShell: **Yes**
   - Run this script using the logged-on credentials: **No** (runs as SYSTEM)

### 3. Set the Schedule

1. Click **Assignments**
2. Add your device groups
3. Set schedule (I used daily)

### 4. Monitor Results

After deployment, check results in:

**Devices** > **Scripts and remediations** > **Firefox Auto-Update** > **Device status**

You'll see:
- Devices where Firefox was already up to date
- Devices where remediation ran and updated Firefox
- Any failures

## What to Watch Out For

- **Language** - The download URL uses `lang=en-US`. Change this if you need a different language.
- **Architecture** - The script downloads 64-bit Firefox. Adjust `os=win64` to `os=win` for 32-bit.
- **Firefox ESR** - If you use ESR, change the product parameter to `firefox-esr-latest`.
- **Network** - Devices need internet access to check versions and download updates.
- **Running Firefox** - The installer handles running instances, but users may see Firefox restart.

## A Simpler Alternative

This script worked well, but it's maintenance overhead. Every time Mozilla changes something, you might need to update the script.

Now I use [ADMX templates](/posts/intune-upload-admx-templates/) instead. Firefox provides policy templates that let you configure auto-updates natively through Intune. No scripts to maintain.

Use proactive remediation when:
- The app doesn't have ADMX/policy support
- You need custom logic the vendor doesn't provide
- You're doing something beyond standard settings

## Related Links

- [Remove Firefox from All Devices](/posts/intune-proactive-remediation-remove-firefox/) - If you've standardized on Edge and need to clean up Firefox
- [Intune Remediations overview](https://learn.microsoft.com/en-us/mem/intune/fundamentals/remediations)
- [Firefox enterprise deployment](https://support.mozilla.org/en-US/kb/deploy-firefox-msi-installers)
- [Upload ADMX templates to Intune](/posts/intune-upload-admx-templates/) - The simpler approach
