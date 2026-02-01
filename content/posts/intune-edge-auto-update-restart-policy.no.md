---
title: "Hold Edge oppdatert med Intune auto-restart policy"
date: 2026-02-01
draft: false
tags: ["intune", "microsoft-edge", "how-to"]
categories: ["How-To"]
summary: "Konfigurer Intune til å auto-oppdatere og restarte Microsoft Edge med brukervennlige varsler."
---

## Problemet

Edge-oppdateringer lastes ned i bakgrunnen, men de trer ikke i kraft før nettleseren restartes. Brukere som aldri lukker nettleseren sin ender opp med å kjøre utdaterte versjoner i ukevis. Sikkerhetsoppdateringer forblir ubrukte. Vi endte opp med støy i Defender om utdaterte nettlesere og sikkerhetssårbarheter.

Nå har vi en policy som auto-oppdaterer og holder både oss og Defender fornøyde.

## Miljø

- Windows 11
- Intune
- Microsoft Edge (Chromium)

Samme prinsipp gjelder hvis du administrerer Chrome eller andre nettlesere - de har alle lignende oppdaterings/restart-policyer du kan pushe gjennom Intune.

## Løsningen

Konfigurer Edge-policyer gjennom Intune som:
1. Tvinger ventende oppdateringer til å tre i kraft
2. Varsler brukere 30 minutter før restart
3. Gir brukere et 1-times vindu til å lagre arbeidet og restarte på egne vilkår

Brukere ser en popup når en oppdatering er klar. De kan restarte umiddelbart eller vente opptil en time. Etter det restarter Edge automatisk.

## Konfigurasjonstrinn

### 1. Lag en Settings Catalog-profil

1. Gå til **Intune admin center** → **Devices** → **Configuration** → **Create** → **New policy**
2. Velg **Windows 10 and later** som plattform
3. Velg **Settings catalog** som profiltype
4. Gi den et tydelig navn som "Edge - Auto Update and Restart"

### 2. Legg til Edge Update-innstillingene

Klikk **Add settings** og søk etter "Microsoft Edge". Legg til disse innstillingene:

**Relaunch Notification Period**
- Sti: Microsoft Edge → Relaunch Notification Period
- Innstilling: `RelaunchNotificationPeriod`
- Verdi: `1800000` (30 minutter i millisekunder)

Dette kontrollerer hvor lenge varselet vises før Edge restarter.

**Relaunch Window**
- Sti: Microsoft Edge
- Innstilling: `RelaunchWindow`
- Verdi: Konfigurer start- og sluttid for restart-vinduet

Dette gir brukere et forutsigbart vindu når restarter kan skje.

**Force Browser Restart After Update**
- Sti: Microsoft Edge
- Innstilling: `RelaunchNotification`
- Verdi: `Required`

Dette sikrer at brukere ikke kan ignorere oppdateringen for alltid.

### 3. Alternativ: Bruk Administrative Templates

Hvis du foretrekker ADMX-baserte policyer, må du importere Edge ADMX-malene først. Se [Last opp egendefinerte ADMX-maler til Intune](/no/posts/intune-upload-admx-templates/) for hvordan du gjør det.

Når de er importert:

1. Gå til **Devices** → **Configuration** → **Create** → **New policy**
2. Velg **Windows 10 and later** → **Templates** → **Imported Administrative templates (Preview)**
3. Velg Edge-malen din og naviger til **Update**
4. Konfigurer disse policyene:

| Policy | Verdi |
|--------|-------|
| Notify a user that a browser restart is recommended or required | Enabled - Required |
| Set the time period for update notifications | 1800000 |
| Set the time period before required update | 3600000 |

### 4. Tildel profilen

1. Klikk **Assignments**
2. Legg til enhets- eller brukergruppene dine
3. Gjennomgå og opprett

## Hva brukere ser

Når en oppdatering venter:

1. Et varsel vises i Edge: "En oppdatering er tilgjengelig. Restart innen 30 minutter."
2. Brukere kan klikke **Restart nå** for å bruke umiddelbart
3. Hvis de venter, restarter Edge automatisk etter at tidtakeren utløper
4. Pågående arbeid får sesjongjenoppretting - faner åpnes på nytt etter restart

Varselet er ikke aggressivt. Det er et lite banner som minner brukere uten å avbryte arbeidet deres.

## Verifiser at policyer er brukt

Brukere (eller du under testing) kan sjekke anvendte policyer:

1. Åpne Edge
2. Gå til `edge://policy/`
3. Se etter `RelaunchNotification` og `RelaunchNotificationPeriod`

Hvis policyene vises her, er de aktive.

## Ting å passe på

- **Millisekunder, ikke sekunder** - Tidsverdiene er i millisekunder. 1800000 = 30 minutter. Ikke sett 1800 ved et uhell (1.8 sekunder).
- **Brukerklager** - Noen brukere vil protestere mot tvungne restarter. 1-times vinduet hjelper. Forklar at det er for sikkerhet.
- **Sesjongjenoppretting** - Edge skal gjenopprette faner etter restart, men minn brukere på å lagre arbeid i webapper som ikke auto-lagrer.

## Relaterte lenker

- [Microsoft Edge policies - Browser restart](https://learn.microsoft.com/en-us/deployedge/microsoft-edge-policies#relaunch-policies)
- [Configure Microsoft Edge using Intune](https://learn.microsoft.com/en-us/mem/intune/configuration/administrative-templates-configure-edge)
