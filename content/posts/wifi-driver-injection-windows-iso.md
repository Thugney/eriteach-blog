---
title: "Inject WiFi Drivers into Windows ISO for Smoother Resets"
date: 2026-02-01
draft: false
tags: ["intune", "powershell", "autopilot", "deployment"]
categories: ["How-To"]
summary: "No WiFi during Windows setup after reset? Inject drivers into the ISO so school staff can handle resets without IT."
---

## The Problem

We kept running into this: a student downloads something sketchy, Defender XDR flags the device, it gets isolated. Now it needs a reset.

I work in a municipality with multiple schools. Each school has a consultant who handles local IT tasks. They boot from USB, reset Windows, and hand the laptop back to the student.

Except... after reset, there's no WiFi. The generic Windows 11 ISO doesn't include drivers for the HP laptops we use. The device can't connect to the network. Autopilot enrollment fails at OOBE.

The school consultant had to physically deliver the laptop to our IT office. For something that should take 30 minutes, we were looking at days of delay.

## The Solution

Then it hit me - what if I inject WiFi drivers directly into the Windows `install.wim`? Give the modified USB to each school consultant. Now when they reset a device, WiFi works out of the box.

## What I Built

I wrote a script that:

1. Mounts your Windows 11 ISO
2. Downloads WiFi drivers from HP's FTP server:
   - Realtek RTL8852/8822/8821
   - Intel Wi-Fi 6E AX211
   - HP ProBook G10 driver pack
   - HP WinPE driver pack
3. Injects drivers into `install.wim` for selected editions (Education, Pro, Enterprise)
4. Outputs a modified WIM file

No HP CMSL required. The script downloads directly from HP's public FTP.

## Running the Script

Download the Windows 11 ISO from Microsoft. Then run:

```powershell
.\inject-wifi-drivers.ps1 -ISOPath "C:\Win11_24H2.iso" -OutputPath "C:\ModifiedWIM"

# Full script: https://github.com/Thugney/eriteach-scripts/blob/main/deployment/inject-wifi-drivers.ps1
```

The script takes about 15-30 minutes depending on your disk speed. Most of that is downloading the driver packs (~2GB total).

## Output

You get a modified `install.wim` in your output folder. The script tells you what to do next:

```
NEXT STEPS:
1. Copy to USB:
   copy "C:\ModifiedWIM\install.wim" E:\sources\install.wim

2. Or create new USB with Rufus and replace install.wim
```

## Creating the USB

Option 1 - Replace WIM on existing USB:
1. Create a standard Windows 11 USB with Rufus or Media Creation Tool
2. Copy your modified `install.wim` to `E:\sources\` (replace existing)

Option 2 - Fresh USB with Rufus:
1. Open Rufus, select your ISO
2. Create the USB
3. Replace `sources\install.wim` with your modified version

## What Editions Get Modified

By default, the script only injects drivers into:
- Windows 11 Education
- Windows 11 Pro
- Windows 11 Enterprise

Home edition is skipped since we don't use it. If you need all editions:

```powershell
.\inject-wifi-drivers.ps1 -ISOPath "C:\Win11.iso" -OutputPath "C:\ModifiedWIM" -AllEditions
```

Or specify exactly which editions:

```powershell
.\inject-wifi-drivers.ps1 -ISOPath "C:\Win11.iso" -OutputPath "C:\ModifiedWIM" -EditionsToInject @("Education", "Enterprise")
```

## HP Driver Sources

The script downloads these Softpaqs from HP's FTP:

| Driver | Softpaq | Covers |
|--------|---------|--------|
| Realtek WiFi | sp155482 | RTL8852, RTL8822, RTL8821 |
| Intel AX211 | sp138607 | Wi-Fi 6E AX211 |
| ProBook G10 Pack | sp145027 | Full driver pack including WiFi |
| WinPE Drivers | sp155634 | Network drivers for WinPE/deployment |

If you have different HP models, you can find the right Softpaq numbers on HP's driver download page and modify the script.

## The Result

Our school consultants can now reset devices independently:
1. Boot from the modified USB
2. Reset Windows
3. WiFi connects at OOBE
4. Autopilot enrollment starts
5. Student gets their laptop back the same day

No more delivering laptops across town. No more multi-day delays for a simple reset. This was an "aha moment" for us - simple idea, big impact.

## Scripts

- [inject-wifi-drivers.ps1](https://github.com/Thugney/eriteach-scripts/blob/main/deployment/inject-wifi-drivers.ps1)
