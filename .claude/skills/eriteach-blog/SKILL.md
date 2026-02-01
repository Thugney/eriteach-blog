---
name: eriteach-blog
description: Write technical blog posts for eriteach.com about Microsoft cloud technologies (Intune, Autopilot, Entra ID, Defender, Purview). Use when the user describes a scenario, problem, or solution they encountered and wants a blog post created. Triggers on requests like "write a blog post about...", "create a post for...", "document this scenario...", or when user shares a technical problem they solved.
---

# Eriteach Blog Post Skill

Write practical, scenario-based blog posts for blog.eriteach.com.

## Voice & Style

- **Simple language** - No jargon. If you must use acronyms, explain them in plain words
- **Short sentences** - Get to the point. No fluff
- **Real-world framing** - "You're enrolling a new laptop and it gets stuck..." not "When implementing device enrollment strategies..."
- **Not verbose** - Say it once, say it clearly, move on
- **No guru tone** - No "Let's dive in", "In this blog post we will", "As you may know"
- **Norwegian context** - Author works in Norway, Kommune environment

## Blog Focus

Microsoft cloud tech:
- **Intune** - Device management, app deployment, compliance
- **Autopilot** - Device enrollment, ESP, deployment profiles
- **Entra ID** - Identity, conditional access, authentication
- **Defender** - Security, threat protection
- **Purview** - Data governance, sensitivity labels

## Post Structure

```markdown
---
title: "Clear Title - What This Fixes or Does"
date: YYYY-MM-DD
draft: true
tags: ["primary-tech", "secondary-tech"]
categories: ["Troubleshooting" | "How-To" | "Quick-Tip"]
summary: "One sentence - what you'll learn"
---

## The Problem

What went wrong. Be specific. Use real-world scenario.

Example: "A user gets a new laptop. Autopilot starts, then sits at 'Identifying' for 45 minutes before timing out."

## Environment

- Windows 11 24H2
- Intune standalone
- Hybrid Entra joined

Keep it short. Only what's relevant.

## What I Checked

What you looked at to find the cause. Be specific about where you clicked, what logs you pulled.

Example:
1. Opened Intune portal → Devices → Windows → Windows enrollment
2. Checked the device in Autopilot devices - status showed "Assigned"
3. Ran `mdmdiagnosticstool.exe -area Autopilot -cab c:\temp\diag.cab`
4. Found error in logs: "App XYZ stuck in installing state"

## The Fix

Step-by-step. Number each step. Include exact paths in Intune/Entra.

Example:
1. Go to **Intune** → **Apps** → **Windows apps**
2. Find the app "Company Portal"
3. Click **Properties** → **Detection rules**
4. Change from file path to registry key:
   - Path: `HKLM\SOFTWARE\Microsoft\CompanyPortal`
   - Key: `Installed`
   - Value: `1`
5. Save and sync

## Script (if applicable)

For scripts, show only a snippet that explains the logic. Full script lives in GitHub.

```powershell
# Check if device is Autopilot registered
$serial = (Get-WmiObject win32_bios).SerialNumber
# Full script: https://github.com/Thugney/eriteach-scripts/blob/main/autopilot/check-registration.ps1
```

Always link to full script in GitHub repo: `github.com/Thugney/eriteach-scripts`

## What to Watch Out For

Optional section. Include if there are common mistakes or things that broke along the way.

## Related Links

- Link to Microsoft docs if helpful
- Link to related posts
```

## Code Block Rules

ALWAYS use proper markdown code fences with language specified:

```powershell
Get-MgDevice -Filter "displayName eq 'PC001'"
```

```json
{
  "setting": "value"
}
```

Never paste raw code without fences. Never use inline code for multi-line scripts.

## Length Guide

- **Quick-Tip**: 200-400 words. One problem, one fix.
- **How-To**: 400-700 words. Step-by-step guide.
- **Troubleshooting**: 500-900 words. Problem → investigation → fix.

Shorter is better. If it can be said in 300 words, don't write 600.

## Tags

`intune`, `autopilot`, `entra-id`, `defender`, `purview`, `powershell`, `graph-api`, `conditional-access`, `compliance`, `windows-11`, `hybrid-join`

## Categories

- **Troubleshooting** - Something broke, here's why and how to fix
- **How-To** - Step-by-step to set something up
- **Quick-Tip** - Fast, focused, under 400 words

## File Output

Save to: `J:\Projects\eriteach-blog\content\drafts\`

Filename: `kebab-case-describing-topic.md`

Example: `autopilot-esp-timeout-win32-detection.md`

## Workflow

1. User describes scenario
2. Ask clarifying questions if needed:
   - What error did you see?
   - What Windows/Intune version?
   - What did you try first?
3. Write post in user's voice (simple, direct, no fluff)
4. Save to drafts folder
5. Mention if a script should be added to GitHub repo

## Example - What NOT to Write

❌ "In this comprehensive guide, we will explore the intricacies of Windows Autopilot Enrollment Status Page timeout issues and dive deep into the resolution methodology."

## Example - What TO Write

✅ "Your laptop is stuck at 'Identifying' during Autopilot. It's been 30 minutes. Here's what's probably wrong and how to fix it."
