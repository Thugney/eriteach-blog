---
title: "Automatic Disk Cleanup with Intune Proactive Remediation"
date: 2026-02-01
draft: false
tags: ["intune", "powershell", "proactive-remediation"]
categories: ["How-To"]
summary: "Silent disk cleanup for devices with limited SSD space - no user interaction required."
---

## The Problem

We have student laptops with 200GB SSDs. OneDrive handles document backup, but students download games, videos, and other large files. Disk fills up. Windows starts complaining. Devices slow down.

I first tried Disk Cleanup (`cleanmgr.exe`), but it requires GUI interaction. Running it via Intune as SYSTEM shows nothing on screen - it just hangs waiting for user input that never comes.

## The Solution

I built a Proactive Remediation with two scripts:
- **Detection**: Check if disk space is below threshold
- **Remediation**: Clean up temp files, caches, and junk - completely silent

## Detection Logic

The detection script uses dual thresholds. A device is non-compliant if:
- Free space is below **15 GB**, OR
- Free space is below **10%**

This works well across different disk sizes. A 128GB SSD with 10GB free (7.8%) triggers remediation. A 500GB disk with 40GB free (8%) also triggers. The percentage catches large disks, the absolute value catches small ones.

```powershell
$MinFreeSpaceGB = 15
$MinFreeSpacePercent = 10

$disk = Get-CimInstance -ClassName Win32_LogicalDisk -Filter "DeviceID='C:'"
$freeSpaceGB = [math]::Round($disk.FreeSpace / 1GB, 2)
$freeSpacePercent = [math]::Round(($disk.FreeSpace / $disk.Size) * 100, 1)

$isCompliant = ($freeSpaceGB -ge $MinFreeSpaceGB) -and ($freeSpacePercent -ge $MinFreeSpacePercent)

# Full script: https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/diskspace-detection.ps1
```

## What Gets Cleaned

My remediation script cleans these locations silently:

| Location | What | Age Filter |
|----------|------|------------|
| `C:\Windows\Temp` | System temp files | All |
| `C:\Windows\Prefetch` | Prefetch cache | Older than 7 days |
| `C:\Windows\SoftwareDistribution\Download` | Windows Update cache | All |
| `C:\Windows\Logs` | Windows logs | Older than 14 days |
| `C:\Users\*\AppData\Local\Temp` | User temp files | All |
| Recycle Bin | Deleted files | All |
| `C:\ProgramData\Microsoft\Windows\WER` | Error reports | All |
| Delivery Optimization cache | Update sharing cache | All |
| Thumbnail cache | Explorer thumbnails | All |

I keep recent prefetch files (last 7 days) because Windows uses them to speed up app launches. Logs older than 14 days are safe to remove.

For Windows Update cache, I stop the `wuauserv` service first, clean the folder, then restart the service.

```powershell
Stop-Service -Name wuauserv -Force -ErrorAction SilentlyContinue
Start-Sleep -Seconds 2
# Clean SoftwareDistribution\Download
Start-Service -Name wuauserv -ErrorAction SilentlyContinue

# Full script: https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/diskspace-remediation.ps1
```

## Intune Setup

1. Go to **Intune** > **Devices** > **Remediations**
2. Click **Create script package**
3. Name it "Disk Space Cleanup"
4. Upload the detection script
5. Upload the remediation script
6. Set **Run this script using the logged-on credentials**: No
7. Set **Run script in 64-bit PowerShell**: Yes
8. Assign to a device group (student devices)
9. Set schedule - daily or every 8 hours depending on how aggressive you want to be

## Logging

The remediation script logs to `C:\ProgramData\Intune\Logs\DiskSpace-Remediation.log`. You'll see entries like:

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

## Results

In our environment with ~150 student devices, typical cleanup frees 2-8 GB per device. The biggest wins come from:
- Recycle Bin (students delete but don't empty)
- User temp files (browser caches, installers)
- Windows Update cache after feature updates

## Scripts

- [diskspace-detection.ps1](https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/diskspace-detection.ps1)
- [diskspace-remediation.ps1](https://github.com/Thugney/eriteach-scripts/blob/main/intune/remediations/diskspace-remediation.ps1)
