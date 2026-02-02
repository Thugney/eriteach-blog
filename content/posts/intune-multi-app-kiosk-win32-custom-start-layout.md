---
title: "Set Up a Multi-App Kiosk with a Custom Win32 App in Intune"
date: 2026-02-02
draft: true
tags: ["intune", "kiosk", "windows-11", "win32-app"]
categories: ["How-To"]
summary: "Configure a Windows multi-app kiosk with a custom Win32 application and custom Start menu layout - not just a browser."
---

## The Problem

We needed a kiosk for library booking. Not a browser pointing to a URL - an actual Win32 app called Kodiak that needs printer access.

Every kiosk guide I found was about browser kiosks. This needed something else.

## The Solution

Intune's multi-app kiosk mode with:
1. A Win32 app added to the kiosk configuration
2. A custom Start menu layout XML to pin the app

## Prerequisites

Before configuring the kiosk profile, you need:

1. **The Win32 app deployed to Intune** - Package Kodiak (or your app) as a Win32 app and deploy it to the target devices
2. **The app's AUMID** - Application User Model ID, needed for the Start layout XML

To find the AUMID, run this on a device where the app is installed:

```powershell
Get-StartApps | Where-Object { $_.Name -like "*Kodiak*" }
```

## Create the Kiosk Profile

1. Go to **Intune admin center** > **Devices** > [**Configuration**](https://intune.microsoft.com/#view/Microsoft_Intune_DeviceSettings/DevicesMenu/~/configuration)
2. Click **Create** > **New policy**
3. Select:
   - Platform: **Windows 10 and later**
   - Profile type: **Templates** > **Kiosk**
4. Name it something descriptive like "Kiosk - Library Booking"

## Configure Kiosk Settings

### Basic Settings

| Setting | Value |
|---------|-------|
| Select a kiosk mode | **Multi app kiosk** |
| Target devices running Windows 10/11 in S mode | **No** |
| User logon type | **Auto logon (Windows 10, version 1803 and later, or Windows 11)** |

![Intune kiosk configuration with Kodiak Win32 app](/kodiak/image1.png)

### Add the Win32 App

1. Under **Browsers and Applications**, click **Add Win32 app**
2. Enter the app details:
   - **Name**: Kodiak (or your app name)
   - **Type**: Win32 App
3. Click **Configured** under Settings to set the app path

The app appears in the Applications list with tile size options.

## Create the Start Menu Layout XML

You need an XML file to pin the app to Start.

![Start menu layout XML configuration](/kodiak/image.png)

Here's the XML I used:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LayoutModificationTemplate xmlns:defaultlayout="http://schemas.microsoft.com/Start/2014/FullDefaultLayout" xmlns="http://schemas.microsoft.com/Start/2014/LayoutModification">
  <LayoutOptions StartTileGroupCellWidth="6" />
  <DefaultLayoutOverride>
    <StartLayoutCollection>
      <defaultlayout:StartLayout GroupCellWidth="6">
        <start:Group Name="Bibliotek" xmlns:start="http://schemas.microsoft.com/Start/2014/StartLayout">
          <start:DesktopApplicationTile Size="4x4" Column="0" Row="0" DesktopApplicationID="YOUR-APP-AUMID-HERE" />
        </start:Group>
      </defaultlayout:StartLayout>
    </StartLayoutCollection>
  </DefaultLayoutOverride>
</LayoutModificationTemplate>
```

Replace `YOUR-APP-AUMID-HERE` with your app's actual AUMID.

### XML Settings Explained

| Element | Purpose |
|---------|---------|
| `StartTileGroupCellWidth="6"` | Width of the Start menu tile group |
| `Group Name="Bibliotek"` | The group name shown on Start (I used "Bibliotek" for library) |
| `Size="4x4"` | Large tile size for easy touch |
| `DesktopApplicationID` | The AUMID of your Win32 app |

### Upload the XML

1. Set **Use alternative Start layout** to **Yes**
2. Paste the XML directly or upload as a file

## Assign and Deploy

1. Click **Assignments**
2. Add the device group containing your kiosk devices
3. Review and create

Device reboots, auto-logs in, and you only see the Start menu with Kodiak.

## What to Watch Out For

- **AUMID must be exact** - One typo and the app won't appear on Start. Double-check with `Get-StartApps`
- **App must be installed first** - The Win32 app deployment must complete before the kiosk profile applies. Use assignment filters or dependencies
- **Printer access** - Multi-app kiosk mode allows printer access, unlike single-app mode. That's why we use multi-app even for one application
- **Testing** - Test on a single device first. Kiosk misconfiguration can lock you out

## Related Links

- [Windows kiosk configuration](https://learn.microsoft.com/en-us/mem/intune/configuration/kiosk-settings-windows)
- [Start layout XML reference](https://learn.microsoft.com/en-us/windows/configuration/start-layout-xml-desktop)
- [Find app AUMID](https://learn.microsoft.com/en-us/windows/configuration/find-the-application-user-model-id-of-an-installed-app)
