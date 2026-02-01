---
title: "Keep Edge Updated with Intune Auto-Restart Policy"
date: 2026-02-01
draft: false
tags: ["intune", "microsoft-edge", "how-to"]
categories: ["How-To"]
summary: "Configure Intune to auto-update and restart Microsoft Edge with user-friendly notifications."
---

## The Problem

Edge updates download in the background, but they don't apply until the browser restarts. Users who never close their browser end up running outdated versions for weeks. Security patches sit unused. We ended up with noise in Defender about outdated browsers and security vulnerabilities.

Now we have a policy that auto-patches and keeps both us and Defender happy.

## Environment

- Windows 11
- Intune
- Microsoft Edge (Chromium)

Same principle applies if you're managing Chrome or other browsers - they all have similar update/restart policies you can push through Intune.

## The Solution

Configure Edge policies through Intune that:
1. Force pending updates to apply
2. Notify users 30 minutes before restart
3. Give users a 1-hour window to save their work and restart on their terms

Users see a popup when an update is ready. They can restart immediately or wait up to an hour. After that, Edge restarts automatically.

## Configuration Steps

### 1. Create a Settings Catalog Profile

1. Go to **Intune admin center** → **Devices** → **Configuration** → **Create** → **New policy**
2. Select **Windows 10 and later** as platform
3. Select **Settings catalog** as profile type
4. Name it something clear like "Edge - Auto Update and Restart"

### 2. Add the Edge Update Settings

Click **Add settings** and search for "Microsoft Edge". Add these settings:

**Relaunch Notification Period**
- Path: Microsoft Edge → Relaunch Notification Period
- Setting: `RelaunchNotificationPeriod`
- Value: `1800000` (30 minutes in milliseconds)

This controls how long the notification shows before Edge restarts.

**Relaunch Window**
- Path: Microsoft Edge
- Setting: `RelaunchWindow`
- Value: Configure start and end time for the relaunch window

This gives users a predictable window when restarts can happen.

**Force Browser Restart After Update**
- Path: Microsoft Edge
- Setting: `RelaunchNotification`
- Value: `Required`

This ensures users can't ignore the update forever.

### 3. Alternative: Use Administrative Templates

If you prefer ADMX-based policies, you'll need to import the Edge ADMX templates first. See [Upload Custom ADMX Templates to Intune](/posts/intune-upload-admx-templates/) for how to do that.

Once imported:

1. Go to **Devices** → **Configuration** → **Create** → **New policy**
2. Select **Windows 10 and later** → **Templates** → **Imported Administrative templates (Preview)**
3. Select your Edge template and navigate to **Update**
4. Configure these policies:

| Policy | Value |
|--------|-------|
| Notify a user that a browser restart is recommended or required | Enabled - Required |
| Set the time period for update notifications | 1800000 |
| Set the time period before required update | 3600000 |

### 4. Assign the Profile

1. Click **Assignments**
2. Add your device or user groups
3. Review and create

## What Users See

When an update is pending:

1. A notification appears in Edge: "An update is available. Restart within 30 minutes."
2. Users can click **Restart now** to apply immediately
3. If they wait, Edge restarts automatically after the timer expires
4. Work in progress gets session restore - tabs reopen after restart

The notification isn't aggressive. It's a small banner that reminds users without interrupting their work.

## Verify Policies Are Applied

Users (or you during testing) can check applied policies:

1. Open Edge
2. Go to `edge://policy/`
3. Look for `RelaunchNotification` and `RelaunchNotificationPeriod`

If the policies show up here, they're active.

## What to Watch Out For

- **Milliseconds, not seconds** - The time values are in milliseconds. 1800000 = 30 minutes. Don't accidentally set 1800 (1.8 seconds).
- **User complaints** - Some users will push back on forced restarts. The 1-hour window helps. Explain it's for security.
- **Session restore** - Edge should restore tabs after restart, but remind users to save work in web apps that don't auto-save.

## Related Links

- [Microsoft Edge policies - Browser restart](https://learn.microsoft.com/en-us/deployedge/microsoft-edge-policies#relaunch-policies)
- [Configure Microsoft Edge using Intune](https://learn.microsoft.com/en-us/mem/intune/configuration/administrative-templates-configure-edge)
