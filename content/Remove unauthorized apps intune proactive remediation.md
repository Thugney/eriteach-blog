---
title: "Remove Unauthorized Apps with Intune Proactive Remediation"
date: 2026-02-01
draft: false
tags: ["intune", "proactive-remediation", "powershell", "compliance"]
categories: ["How-To"]
summary: "Use Intune Proactive Remediation to detect and remove apps users installed when they had admin rights."
---

## The Problem

Users used to have local admin rights. They installed whatever they wanted - Zoom, personal tools, random software.

Now we've locked that down. Apps come through Company Portal only. But the old stuff is still sitting on devices.

Time to clean up.

## The Solution

Intune Proactive Remediation runs two scripts:
1. **Detection** - Checks if the app exists (exit 1 = found, exit 0 = not found)
2. **Remediation** - Removes it if found

## The Detection Script

The script checks registry, WMI, and Programs list. You configure it by changing these variables at the top:

```powershell
$AppDisplayName = "Zoom"
$AppPublisher = ""
$AppProductCode = "{86B70A45-00A6-4CBD-97A8-464A1254D179}"
$UsePartialMatch = $true
```

To find the product code for an app:

```powershell
Get-WmiObject Win32_Product | Format-Table Name, IdentifyingNumber
```

The script logs everything to `C:\ProgramData\Microsoft\IntuneManagementExtension\Logs\` for troubleshooting.

Full script: [Detect-UnwantedApp.ps1](https://github.com/Thugney/eriteach-scripts/blob/main/proactive-remediation/Detect-UnwantedApp.ps1)

## The Remediation Script

Once detected, the remediation script uninstalls the app using its uninstall string from registry or MSI product code.

Full script: [Remove-UnwantedApp.ps1](https://github.com/Thugney/eriteach-scripts/blob/main/proactive-remediation/Remove-UnwantedApp.ps1)

## Setting Up in Intune

1. Go to **Intune** → **Devices** → **Remediations**
2. Click **Create script package**
3. Name it: "Remove Zoom" (or whatever app)
4. Upload:
   - Detection script: `Detect-UnwantedApp.ps1`
   - Remediation script: `Remove-UnwantedApp.ps1`
5. Settings:
   - Run script in 64-bit PowerShell: **Yes**
   - Run with logged-on credentials: **No** (runs as SYSTEM)
6. Assign to a device group
7. Set schedule (daily or hourly depending on urgency)

## What to Watch Out For

- **Test first** - Run detection on a pilot group before enabling remediation
- **Product codes change** - Different versions of an app might have different codes
- **Partial match risk** - `$UsePartialMatch = $true` might catch apps you didn't intend (e.g., "Zoom" matches "Zoom Plugin for Outlook")
- **User data** - Some apps store user data. Warn users before mass removal

## Scaling to Multiple Apps

Create separate remediation packages for each app, or modify the script to check a list:

```powershell
$UnwantedApps = @(
    @{Name = "Zoom"; ProductCode = "{86B70A45-00A6-4CBD-97A8-464A1254D179}"},
    @{Name = "TeamViewer"; ProductCode = ""},
    @{Name = "AnyDesk"; ProductCode = ""}
)
```

## Related Links

- [Microsoft: Proactive Remediations](https://learn.microsoft.com/en-us/mem/intune/fundamentals/remediations)
- [Intune script requirements](https://learn.microsoft.com/en-us/mem/intune/apps/intune-management-extension)