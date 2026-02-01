---
title: "Stopp uautoriserte enhetsoverleveringer med Intune Remediations"
date: 2026-02-01
draft: false
tags: ["intune", "powershell", "compliance"]
categories: ["How-To"]
summary: "Hindre brukere fra å overta kollegers enheter ved å begrense pålogging til kun Intune-primærbrukeren."
---

## Problemet

En bruker slutter i selskapet. Eller tar langvarig permisjon. Eller får en ny bærbar PC.

Hva skjer med den gamle enheten? I teorien går den tilbake til IT for sletting og ny registrering.

I virkeligheten? En kollega bare tar den. Logger inn med sin egen konto. Begynner å jobbe.

Nå har du en enhet hvor:

- Intune fortsatt tror den gamle brukeren eier den
- Brukermålrettede policyer og apper ikke brukes riktig
- Compliance evalueres mot feil person
- Conditional Access kan feile
- Rapportene dine viser søppeldata

Dette er et problem i nesten alle organisasjoner. Brukere tenker ikke på Intune. De trenger bare en datamaskin.

## Løsningen: Hindre det fra å skje

I stedet for å rydde opp etter rotete overleveringer, blokker dem fra å skje i det hele tatt.

Idéen: begrens Windows interaktiv pålogging til **kun Intune-primærbrukeren** og lokale administratorer. Hvis noen andre prøver å logge inn, kan de ikke. De må kontakte IT. IT tilbakestiller deretter enheten ordentlig eller endrer primærbrukeren.

Dette bruker to ting:

1. **SeInteractiveLogonRight** - En Windows-sikkerhetspolicy som kontrollerer hvem som kan logge inn lokalt
2. **Intune Remediations** - Kjører deteksjons- og remediation-skript på en tidsplan

## Hvordan det fungerer

Deteksjonsskriptet:

1. Leser primærbrukerens UPN fra `HKLM:\SOFTWARE\Microsoft\Enrollments`
2. Oversetter UPN til en Windows SID
3. Sjekker om `SeInteractiveLogonRight` er satt til kun den brukeren + Administrators
4. Returnerer compliant eller non-compliant

Remediation-skriptet:

1. Samme oppslag for primærbruker
2. Sikkerhetskopierer gjeldende sikkerhetspolicy
3. Setter `SeInteractiveLogonRight` til kun å tillate primærbrukerens SID og Administrators (S-1-5-32-544)
4. Bruker med `secedit` og kjører `gpupdate /force`

```powershell
# Nøkkellogikk fra deteksjon - finn primærbruker fra Intune-registrering
$enrollmentPath = "HKLM:\SOFTWARE\Microsoft\Enrollments"
$enrollmentKeys = Get-ChildItem -Path $enrollmentPath -ErrorAction SilentlyContinue

foreach ($key in $enrollmentKeys) {
    $upn = Get-ItemProperty -Path $key.PSPath -Name "UPN" -ErrorAction SilentlyContinue
    if ($upn.UPN) {
        $primaryUserUPN = $upn.UPN
        break
    }
}

# Oversett til SID
$ntAccount = New-Object System.Security.Principal.NTAccount("AzureAD\$primaryUserUPN")
$userSID = $ntAccount.Translate([System.Security.Principal.SecurityIdentifier]).Value

# Fulle skript: https://github.com/Thugney/eriteach-scripts/tree/main/intune/remediations
```

## Sette opp Remediation

1. Gå til **Intune** → **Devices** → **Remediations**
2. Klikk **Create script package**
3. Navngi det noe som "Restrict login to primary user"
4. Last opp deteksjonsskriptet
5. Last opp remediation-skriptet
6. Sett **Run this script using the logged-on credentials** til **No** (kjører som SYSTEM)
7. Tildel til en pilotgruppe først
8. Sett tidsplan - daglig er vanligvis greit

## Ting å passe på

**Delte enheter**: Ikke distribuer dette til delte arbeidsstasjoner eller kiosk-enheter. Det er ment for personlige, tildelte enheter.

**Admins kan fortsatt logge inn**: Lokale administratorer og domeneadmins (hvis hybrid joined) kan fortsatt få tilgang til enheten. Dette er tilsiktet - IT trenger en vei inn.

**Primærbruker endres**: Hvis du endrer primærbrukeren i Intune, vil remediation oppdatere påloggingsbegrensningen ved neste kjøring. Den gamle brukeren låses ute, ny bruker får tilgang.

**Sikkerhetskopi opprettes**: Remediation sikkerhetskopierer sikkerhetspolicyen før endringer. Stien logges i output hvis du trenger å rulle tilbake.

**Test først**: Kjør deteksjonsskriptet manuelt på en testenhet. Bruk `-WhatIf`-parameteren på remediation for å se hva den ville gjøre uten å gjøre endringer.

## Resultatet

Etter utrulling av dette:

- Brukere kan fysisk ikke overta noen andres enhet
- De tvinges til å kontakte IT
- IT kan ordentlig slette, re-registrere eller tildele enheten på nytt
- Intune-dataene dine forblir rene
- Compliance og Conditional Access fungerer som forventet

Det er en liten endring som hindrer mange hodepiner.

## Relaterte lenker

- [Fulle skript på GitHub](https://github.com/Thugney/eriteach-scripts/tree/main/intune/remediations)
- [Microsoft Docs - Remediations](https://learn.microsoft.com/en-us/mem/intune/fundamentals/remediations)
- [SeInteractiveLogonRight](https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/allow-log-on-locally)
