---
title: "Stop Unauthorized Device Handoffs with Intune Remediations"
date: 2026-02-01
draft: false
tags: ["intune", "powershell", "compliance"]
categories: ["How-To"]
summary: "Prevent users from taking over colleagues' devices by restricting login to the Intune primary user only."
---

## The Problem

A user leaves the company. Or goes on extended leave. Or gets a new laptop.

What happens to their old device? In theory, it goes back to IT for a wipe and re-enrollment.

In reality? A colleague just takes it. Logs in with their own account. Starts working.

Now you have a device where:

- Intune still thinks the old user owns it
- User-targeted policies and apps don't apply correctly
- Compliance evaluates against the wrong person
- Conditional Access might fail
- Your reports show garbage data

This is a problem in almost every organization. Users don't think about Intune. They just need a computer.

## The Fix: Prevent It From Happening

Instead of cleaning up after messy handoffs, block them from happening at all.

The idea: restrict Windows interactive logon to **only the Intune primary user** and local administrators. If someone else tries to log in, they can't. They have to contact IT. IT then properly resets the device or changes the primary user.

This uses two things:

1. **SeInteractiveLogonRight** - A Windows security policy that controls who can log in locally
2. **Intune Remediations** - Runs detection and remediation scripts on a schedule

## How It Works

The detection script:

1. Reads the primary user UPN from `HKLM:\SOFTWARE\Microsoft\Enrollments`
2. Translates the UPN to a Windows SID
3. Checks if `SeInteractiveLogonRight` is set to only that user + Administrators
4. Returns compliant or non-compliant

The remediation script:

1. Same lookup for primary user
2. Backs up the current security policy
3. Sets `SeInteractiveLogonRight` to only allow the primary user SID and Administrators (S-1-5-32-544)
4. Applies with `secedit` and runs `gpupdate /force`

```powershell
# Key logic from detection - find primary user from Intune enrollment
$enrollmentPath = "HKLM:\SOFTWARE\Microsoft\Enrollments"
$enrollmentKeys = Get-ChildItem -Path $enrollmentPath -ErrorAction SilentlyContinue

foreach ($key in $enrollmentKeys) {
    $upn = Get-ItemProperty -Path $key.PSPath -Name "UPN" -ErrorAction SilentlyContinue
    if ($upn.UPN) {
        $primaryUserUPN = $upn.UPN
        break
    }
}

# Translate to SID
$ntAccount = New-Object System.Security.Principal.NTAccount("AzureAD\$primaryUserUPN")
$userSID = $ntAccount.Translate([System.Security.Principal.SecurityIdentifier]).Value

# Full scripts: https://github.com/Thugney/eriteach-scripts/tree/main/intune/remediations
```

## Setting Up the Remediation

1. Go to **Intune** → **Devices** → **Remediations**
2. Click **Create script package**
3. Name it something like "Restrict login to primary user"
4. Upload the detection script
5. Upload the remediation script
6. Set **Run this script using the logged-on credentials** to **No** (runs as SYSTEM)
7. Assign to a pilot group first
8. Set schedule - daily is usually fine

## What to Watch Out For

**Shared devices**: Don't deploy this to shared workstations or kiosk devices. It's meant for personal, assigned devices.

**Admins can still log in**: Local administrators and domain admins (if hybrid joined) can still access the device. This is intentional - IT needs a way in.

**Primary user changes**: If you change the primary user in Intune, the remediation will update the login restriction on the next run. The old user gets locked out, new user gets access.

**Backup is created**: The remediation backs up the security policy before changes. Path is logged in the output if you need to roll back.

**Test first**: Run the detection script manually on a test device. Use the `-WhatIf` parameter on the remediation to see what it would do without making changes.

## The Result

After rolling this out:

- Users physically cannot take over someone else's device
- They're forced to contact IT
- IT can properly wipe, re-enroll, or reassign the device
- Your Intune data stays clean
- Compliance and Conditional Access work as expected

It's a small change that prevents a lot of headaches.

## Related Links

- [Full scripts on GitHub](https://github.com/Thugney/eriteach-scripts/tree/main/intune/remediations)
- [Microsoft Docs - Remediations](https://learn.microsoft.com/en-us/mem/intune/fundamentals/remediations)
- [SeInteractiveLogonRight](https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/allow-log-on-locally)
