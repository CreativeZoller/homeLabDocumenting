# Fázisok és BOM

Single source of truth: IT központ terv **v2.0**, a **v2.6** felülírásaival. Az árak a PDF-ekből, euróban, tájékoztató jellegűek. Az elvetett tételek áthúzva, nem számítanak az összegbe.

A 2.6 új rack-fázisa a hálózati stacket (Firewalla, UniFi) és a 10" házat hozza be. A 2.0 1. fázisából kiesik az Asus RT-BE88U és a 10G DAC; a 3. fázisból a 19" PDU és a gurulós szervertároló.

## Fázisok áttekintése

| Fázis | Forrás | Tartalom |
|-------|--------|----------|
| 1. Alapvető hardver | 2.0, NIC: 2.6 | CPU, RAM, hűtő, 2.5G NIC, munkaállomás kábelek |
| Rack és beltartalom | 2.6 (új) | Firewalla, UniFi switch/AP, 10" PDU, UPS, patch, tálcák |
| 2. Ergonómia és adatbiztonság | 2.0 | 2× 12 TB NAS, monitorkar, mikrofon, nyomtató |
| 3. Fizikai építés | 2.0, PDU a rack-fázisban | Asztallap, KVM, szerszámok |
| 4. Kábelmenedzsment, esztétika | 2.0 | Csatornák, LED, USB hub |
| 5. Funkcionális konfiguráció | 2.0 | VLAN, Docker, Home Assistant — lásd az eszköz-oldalakat |

## 1. fázis — hardver és kábelek

| Db | Megnevezés | Ár (€) | Megjegyzés |
|----|------------|--------|------------|
| 1 | Intel i5-12400 | 212 | Home server CPU |
| 1 | Noctua NH-L12S | 71 | Alacsony profilú hűtő |
| 1 | Kingston FURY Beast 64 GB (2×32) DDR4-3200 | 524 | RAM |
| 2 | 2.5G PCIe NIC, I226-V | 24 | 2.6: a 10G kártya / DAC helyett |
| 1 | USB-C to USB-C (video + 100W PD) | 22 | KVM / laptop |
| 2 | 3 m DisplayPort → HDMI 2.0 aktív | 62 | |
| 1 | Micro-HDMI → HDMI | 9 | Pi / mini eszköz |
| 1 | 2 m DisplayPort → HDMI 2.0 aktív | 27 | |
| 1 | 2 m HDMI 2.1 | 9 | |
| 1 | USB 3.0 Type-A hosszabbító | 6 | |
| 3 | 3 m USB 3.0 Type-A → Type-B | 33 | |
| ~~1~~ | ~~Asus RT-BE88U~~ | ~~260~~ | Elvetve: Firewalla + UniFi |
| ~~1~~ | ~~10Gtek SFP+ DAC 0.9 m~~ | ~~17~~ | Elvetve: nincs 10G gerinc |

**Részösszeg (élő tételek):** kb. 999 €.

## Rack-fázis (2.6)

| Db | Megnevezés | Ár (€) | Megjegyzés |
|----|------------|--------|------------|
| 1 | 10" rack szekrény | TBD | A 2.6 PDF első sora hiányos; tálcák/PDU megvannak, a kabinet SKU pótolandó |
| 1 | Firewalla Gold Plus | 535 | Tűzfal / router |
| 1 | UniFi Lite 16 PoE | 232 | L2 switch |
| 1 | UniFi U7 Pro | 189 | AP |
| 2 | Digitus 10" 4-aljzat 1U PDU | 19 | 19" DN-95441 helyett |
| 1 | Legrand Keor 800 VA UPS | 83 | |
| 1 | deleyCON 12 port keystone patch, 10" 1U | 15 | |
| 1 | Rackstuds R100 Series II (100 db) | 54 | |
| 2 | FGB Cat7 keystone (10 db/csomag) | 25 | |
| 4 | Digitus 10" 1U tálca | 17 | |
| 1 | FGB Cat7 patch 0.75 m ×10 | 23 | |
| 1 | FGB Cat7 patch 0.5 m ×10 | 13 | |
| 2 | Noctua NA-SAVG1 | 15 | Rezgéscsillapító perem |
| 1 | Noctua NA-SAV2 chromax.red | 10 | Ventilátor szilentblokk |

**Részösszeg:** kb. 1300 € (PDF), a kabinet SKU nélkül.

A 3D-nyomtatott rack (PETG, TPU talp) **alternatíva**, nem ez a BOM. Lásd [10" rack](../infrastructure/rack.md).

## 2. fázis — ergonómia és adatbiztonság

| Db | Megnevezés | Ár (€) |
|----|------------|--------|
| 2 | Seagate IronWolf NAS 12 TB | 774 |
| 1 | Duál gázrugós monitorkar | 81 |
| 1 | Laptoptartó a karra | 35 |
| 1 | Blue Yeti X | 146 |
| 1 | Mikrofon shock mount | 75 |
| 1 | Mikrofon gázkar | 112 |
| 1 | Brother DCP-L2620DW | 172 |

**Részösszeg:** 1395 €.

## 3. fázis — asztal és KVM

| Db | Megnevezés | Ár (€) | Megjegyzés |
|----|------------|--------|------------|
| 1 | Munkalap 180×80×2.5 cm | 85 | Bauhaus |
| 1 | Faolaj | 16 | |
| 2 | Kábelvezető | 16 | |
| 1 | 60 mm fafúró / lyukfűrész | 14 | Három kábelkivezetés |
| 1 | 180-as csiszolópapír | 1 | |
| 3 | 240-es csiszolópapír | 2 | |
| 1 | Bosch excentercsiszoló | 60 | |
| 3 | 240-es csiszolólap | 3 | |
| 1 | AV Access 4KSW41C-KVM | 240 | PC, laptopok, Pi |
| ~~1~~ | ~~Gurulós tároló~~ | ~~17~~ | 10" rack helyettesíti |
| ~~1~~ | ~~Gurulós tároló szervernek~~ | ~~17~~ | ugyanaz |
| ~~2~~ | ~~Digitus DN-95441 19" PDU~~ | ~~82~~ | 10" PDU a rack-fázisban |

**Részösszeg (élő tételek):** kb. 437 €.

## 4. fázis — kábelmenedzsment és esztétika

| Db | Megnevezés | Ár (€) |
|----|------------|--------|
| 2 | Digitus kábelrendező csatorna | 38 |
| 1 | 20 m kábelvezető szoknya | 39 |
| 1 | 3M duplaoldalú ragasztó | 14 |
| 1 | Groove LED csík | 27 |
| 1 | Yeelight monitorlámpa YLTD003 | 91 |
| 1 | Icy Box USB hub | 38 |

**Részösszeg:** 247 €.

## 5. fázis — szoftver

Nincs hardver-BOM. VLAN a Firewalla + UniFi oldalon, szolgáltatások a Docker stackekben, világítás később Home Assistantból.

## Összegzés (élő tételek)

| Blokk | Kb. € |
|-------|-------|
| 1. fázis | 999 |
| Rack 2.6 | 1300 |
| 2. fázis | 1395 |
| 3. fázis | 437 |
| 4. fázis | 247 |
| **Összesen** | **~4378** |

A 10" kabinet SKU és a pályázati önrész: [Pályázatok](funding.md). Linkek a v2.0 / v2.6 PDF-ben (repo: `it_20.pdf`, `it_26.pdf`).
