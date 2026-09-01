# 10" rack

A 2.6-os döntés: 10" felállás a home server mellett, nem külön szobás 19" szekrény. Ok: költség, hely, zaj, későbbi szervercsere.

A **célút a kereskedelmi BOM** (tálcák, PDU, patch panel). A 3D-nyomtatott váz csak alternatíva.

## Beltartalom

| Szerep | Eszköz |
|--------|--------|
| Tűzfal | Firewalla Gold Plus |
| Switch | UniFi Lite 16 PoE |
| AP | U7 Pro — általában a racken kívül |
| Compute | Mini-ITX home server tálcán |
| Táp | 2× Digitus 10" 4-aljzat 1U PDU |
| UPS | Legrand Keor 800 VA |
| Kábel | deleyCON 12 port keystone, Cat7 keystone + rövid patch |
| Rögzítés | Rackstuds R100 |
| Rezgés | Noctua NA-SAVG1, NA-SAV2 |

A 10" kabinet pontos SKU a 2.6 PDF első sorából hiányzik — pótolandó a beszerzéskor. Négy Digitus 1U tálca (44×254×150 mm) van a listán.

## Javasolt sorrend fentről

1. Patch panel 1U
2. UniFi Lite 16 PoE
3. Firewalla (tálca)
4. PDU
5. Home server (tálca, melegebb, lejjebb)
6. UPS legalul

Részletes kábelezés: [Eszközök kötése](../network-setup/device-connections.md).

## Kábelezés a rackben

- Kívülről jövő Cat7 a keystone-ba.
- Belül 0.5 m / 0.75 m FGB Cat7 patch a portra.
- Táp: eszköz → 10" PDU → Legrand UPS → fal.

## Alternatíva: 3D-nyomtatott rack

Nem a célút. Ha mégis:

- Anyag: legalább PETG; lábakra TPU rezgéselnyelő.
- Magasság: most 8U, később 12–14U a clusterig.
- Egyedi előlap eszközönként.
- Borítás: akril vagy más, hőt nem nyelő, akár sötétített.
- Felső/alsó ventilátorok tápja és fordulatszám-vezérlés külön feladat.
- A Yottamaster DAS-hoz is TPU talp és egyedi előlap kellene.

Amíg a kereskedelmi BOM a terv, ez a bekezdés csak tartalék.
