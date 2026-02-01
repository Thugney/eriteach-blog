---
title: "Injiser WiFi-drivere i Windows ISO for smidigere tilbakestillinger"
date: 2026-02-01
draft: false
tags: ["intune", "powershell", "autopilot", "deployment"]
categories: ["How-To"]
summary: "Ingen WiFi under Windows-oppsett etter tilbakestilling? Injiser drivere i ISO-en så skolepersonalet kan håndtere tilbakestillinger uten IT."
---

## Problemet

Vi opplevde dette gjentatte ganger: en student laster ned noe skummelt, Defender XDR flagger enheten, den blir isolert. Nå trenger den en tilbakestilling.

Jeg jobber i en kommune med flere skoler. Hver skole har en konsulent som håndterer lokale IT-oppgaver. De starter fra USB, tilbakestiller Windows, og gir den bærbare PC-en tilbake til studenten.

Bortsett fra... etter tilbakestilling er det ingen WiFi. Den generiske Windows 11 ISO-en inkluderer ikke drivere for HP-PC-ene vi bruker. Enheten kan ikke koble til nettverket. Autopilot-registrering feiler ved OOBE.

Skolekonsulenten måtte fysisk levere den bærbare PC-en til IT-kontoret vårt. For noe som burde ta 30 minutter, så vi på flere dagers forsinkelse.

## Løsningen

Så slo det meg - hva om jeg injiserer WiFi-drivere direkte i Windows `install.wim`? Gi den modifiserte USB-en til hver skolekonsulent. Nå når de tilbakestiller en enhet, fungerer WiFi rett ut av boksen.

## Hva jeg bygde

Jeg skrev et skript som:

1. Monterer Windows 11 ISO-en din
2. Laster ned WiFi-drivere fra HPs FTP-server:
   - Realtek RTL8852/8822/8821
   - Intel Wi-Fi 6E AX211
   - HP ProBook G10 driverpakke
   - HP WinPE driverpakke
3. Injiserer drivere i `install.wim` for valgte utgaver (Education, Pro, Enterprise)
4. Produserer en modifisert WIM-fil

HP CMSL er ikke nødvendig. Skriptet laster ned direkte fra HPs offentlige FTP.

## Kjøre skriptet

Last ned Windows 11 ISO fra Microsoft. Kjør deretter:

```powershell
.\inject-wifi-drivers.ps1 -ISOPath "C:\Win11_24H2.iso" -OutputPath "C:\ModifiedWIM"

# Fullt skript: https://github.com/Thugney/eriteach-scripts/blob/main/deployment/inject-wifi-drivers.ps1
```

Skriptet tar ca. 15-30 minutter avhengig av diskhastigheten din. Det meste av tiden går til å laste ned driverpakkene (~2GB totalt).

## Resultat

Du får en modifisert `install.wim` i output-mappen din. Skriptet forteller deg hva du skal gjøre videre:

```
NESTE STEG:
1. Kopier til USB:
   copy "C:\ModifiedWIM\install.wim" E:\sources\install.wim

2. Eller lag ny USB med Rufus og bytt ut install.wim
```

## Lage USB-en

Alternativ 1 - Erstatt WIM på eksisterende USB:
1. Lag en standard Windows 11 USB med Rufus eller Media Creation Tool
2. Kopier den modifiserte `install.wim` til `E:\sources\` (erstatt eksisterende)

Alternativ 2 - Ny USB med Rufus:
1. Åpne Rufus, velg ISO-en din
2. Lag USB-en
3. Erstatt `sources\install.wim` med den modifiserte versjonen

## Hvilke utgaver som modifiseres

Som standard injiserer skriptet drivere kun i:
- Windows 11 Education
- Windows 11 Pro
- Windows 11 Enterprise

Home-utgaven hoppes over siden vi ikke bruker den. Hvis du trenger alle utgaver:

```powershell
.\inject-wifi-drivers.ps1 -ISOPath "C:\Win11.iso" -OutputPath "C:\ModifiedWIM" -AllEditions
```

Eller spesifiser nøyaktig hvilke utgaver:

```powershell
.\inject-wifi-drivers.ps1 -ISOPath "C:\Win11.iso" -OutputPath "C:\ModifiedWIM" -EditionsToInject @("Education", "Enterprise")
```

## HP-driverkilder

Skriptet laster ned disse Softpaq-ene fra HPs FTP:

| Driver | Softpaq | Dekker |
|--------|---------|--------|
| Realtek WiFi | sp155482 | RTL8852, RTL8822, RTL8821 |
| Intel AX211 | sp138607 | Wi-Fi 6E AX211 |
| ProBook G10 Pack | sp145027 | Full driverpakke inkludert WiFi |
| WinPE Drivers | sp155634 | Nettverksdrivere for WinPE/deployment |

Hvis du har andre HP-modeller, kan du finne de riktige Softpaq-numrene på HPs drivernedlastingsside og modifisere skriptet.

## Resultatet

Våre skolekonsulenter kan nå tilbakestille enheter selvstendig:
1. Start fra den modifiserte USB-en
2. Tilbakestill Windows
3. WiFi kobler til ved OOBE
4. Autopilot-registrering starter
5. Studenten får den bærbare PC-en tilbake samme dag

Ikke mer levering av bærbare PC-er på tvers av byen. Ikke mer flere dagers forsinkelser for en enkel tilbakestilling. Dette var et "aha-øyeblikk" for oss - enkel idé, stor effekt.

## Skript

- [inject-wifi-drivers.ps1](https://github.com/Thugney/eriteach-scripts/blob/main/deployment/inject-wifi-drivers.ps1)
