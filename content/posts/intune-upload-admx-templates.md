---
title: "Upload Custom ADMX Templates to Intune"
date: 2026-02-01
draft: false
tags: ["intune", "admx", "how-to", "firefox", "zoom"]
categories: ["How-To"]
summary: "How to import custom ADMX/ADML files into Intune for third-party apps like Firefox and Zoom."
---

## The Problem

We had Firefox and Zoom in the org. Both needed to stay updated, but Intune's Settings Catalog doesn't have policies for these apps.

For Firefox, I initially wrote a [proactive remediation script](/posts/intune-proactive-remediation-firefox-update/) that checked the local version against Mozilla's latest and auto-installed updates. It worked, but it was extra complexity to maintain.

Then I discovered ADMX imports. Upload the vendor's policy templates to Intune and you get the same settings you'd have in Group Policy. Much simpler.

## Environment

- Windows 11
- Intune

## What Are ADMX Files?

ADMX files are XML-based Group Policy templates. They define what settings are available. ADML files are the language files that provide the UI text.

You've probably used these in on-prem Group Policy. The same files work in Intune.

## Where to Get ADMX Files

Common sources:

| App | Download |
|-----|----------|
| Firefox | [Mozilla GitHub](https://github.com/AzG-IaC/ADMX-policy-templates) or included in ESR installer |
| Zoom | [Zoom Admin ADMX Templates](https://support.zoom.us/hc/en-us/articles/360039100051) |
| Chrome | [Chrome Enterprise Bundle](https://chromeenterprise.google/browser/download/) |
| Microsoft Edge | [Download from Microsoft](https://www.microsoft.com/en-us/edge/business/download) |
| Office/M365 | [Office ADMX templates](https://www.microsoft.com/en-us/download/details.aspx?id=49030) |

## Upload ADMX to Intune

### 1. Get Your ADMX and ADML Files

For Firefox:

1. Download the Firefox ADMX templates from Mozilla
2. Extract the ZIP
3. You'll find:
   - `firefox.admx` - The template file
   - `en-US\firefox.adml` - The English language file

### 2. Import into Intune

1. Go to **Intune admin center** → **Devices** → **Configuration**
2. Click the **Import ADMX** tab
3. Click **Import**
4. Upload the `.admx` file
5. Upload the matching `.adml` file (usually from `en-US` folder)
6. Click **Next** and complete the import

The import takes a few minutes. You'll see the status change from "In progress" to "Available".

### 3. Handle Dependencies

Some ADMX files depend on others. For example, many Microsoft templates need `windows.admx` as a base.

If you get a dependency error:
1. Note which file it's asking for
2. Import that file first
3. Then retry your original import

Common dependencies:
- `windows.admx` - Base Windows definitions
- `windowscomponents.admx` - Windows component definitions

### 4. Create a Policy Using Your ADMX

Once imported:

1. Go to **Devices** → **Configuration** → **Create** → **New policy**
2. Select **Windows 10 and later**
3. Select **Templates** → **Imported Administrative templates (Preview)**
4. Pick your imported template
5. Configure the settings you need
6. Assign to your groups

## Example: Firefox Auto-Update Policy

After importing Firefox ADMX:

1. Create new policy using the imported Firefox template
2. Navigate to **Mozilla** → **Firefox** → **Updates**
3. Enable **Application Update** settings:
   - `AppAutoUpdate` = Enabled
   - `BackgroundAppUpdate` = Enabled

This tells Firefox to update automatically in the background - no scripts needed.

## Example: Zoom Auto-Update Policy

After importing Zoom ADMX:

1. Create new policy using the imported Zoom template
2. Navigate to **Zoom Meetings** → **General Settings**
3. Configure update settings as needed

## Verify the Import

After importing, you can check:

1. Go to **Devices** → **Configuration** → **Import ADMX**
2. Your template should show status "Available"
3. Click on it to see all the settings it contains

## What to Watch Out For

- **One ADML per import** - You can only upload one language file. Pick `en-US` unless you have a specific need.
- **Version conflicts** - If you import an older version of an ADMX that's already built into Intune, you might get conflicts. Check if the setting exists in Settings Catalog first.
- **Preview feature** - Imported Administrative Templates is still in preview. It works, but the UI might change.
- **Processing time** - Large ADMX files can take 5-10 minutes to process.

## Why ADMX Over Scripts?

I used to run a [proactive remediation script](/posts/intune-proactive-remediation-firefox-update/) for Firefox updates. It worked, but:

- Scripts need maintenance when things change
- More moving parts = more things that can break
- ADMX policies are native and vendor-supported

If the vendor provides ADMX templates, use them. Save scripts for when there's no other option.

## Related Links

- [Import custom ADMX templates in Intune](https://learn.microsoft.com/en-us/mem/intune/configuration/administrative-templates-import-custom)
- [Firefox enterprise policies](https://support.mozilla.org/en-US/kb/customizing-firefox-using-group-policy-windows)
- [Zoom ADMX templates](https://support.zoom.us/hc/en-us/articles/360039100051)
