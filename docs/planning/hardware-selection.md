# Hardver Kiválasztás

A hardver kiválasztásánál a fő szempont az energiahatékony teljesítmény (Intel i5-12400), a csendes 10" rackes elhelyezés, a hálózati kontroll (Firewalla + UniFi) és az adatintegritás (ZFS, UPS).

A 2.6-os terv szerint a nagy 19" rack helyett 10" felállás marad a home server mellett: olcsóbb, helytakarékosabb, csendesebb, és később a szerver leváltásakor sem kell külön szobába költöztetni.

## Jelenlegi hardver

- Fő szerver: Mini-ITX alapú rendszer
    - CPU: Intel i3-12100F
    - RAM: 16 GB DDR4
    - Rendszerlemez: 250 GB NVMe
- Router: MikroTik C53UiG+5HPaxD2HPaxD (hAP ax³) — a célállapotban kikerül
- Segéd szerver: Raspberry Pi 4

## Célállapot: számítási mag

| Tervezett eszköz / frissítés | Indoklás |
|------------------------------|----------|
| VGA kártya eltávolítása a fő szerverből | Felszabadul a PCIe foglalat a 2.5G NIC-eknek |
| CPU csere Intel i5-12400-ra | Erő és fogyasztás; a VGA nélküli iGPU transcode-hoz elég |
| Noctua NH-L12S CPU hűtő | Alacsony profil, rackes / asztal alatti csendes működés |
| RAM bővítés 64 GB-ra (2×32 GB Kingston FURY Beast DDR4-3200) | ZFS, Docker és KVM/QEMU együttes futtatása |
| 2× 2.5G PCIe NIC (Intel I226-V) | A 10G SFP+ út helyett; Firewalla Gold Plus 2.5G portjaihoz igazodik |
| 2× Seagate IronWolf NAS 12 TB | Redundáns ZFS pool a home serverben (később DAS/JBOD felé vihető) |

## Célállapot: hálózat és 10" rack

A korábbi MikroTik (hAP ax³ → RB5009 + CSS610/CRS310) és az Asus RT-BE88U terv helyett a 2.6-os stack:

| Tervezett eszköz | Indoklás |
|------------------|----------|
| Firewalla Gold Plus | Dedikált fizikai tűzfal / router. Az osztrák szolgáltatói modem nem hidalható, ezért a routing és a VLAN-ok ezen az eszközön történnek. |
| UniFi Lite 16 PoE Switch | Menedzselt L2 switch VLAN-okkal és PoE-val a 10" rackbe. 16× 1 GbE; a 2.5G előny a Firewalla ↔ szerver linken érvényesül. |
| UniFi U7 Pro Access Point | Wi-Fi 7 AP, PoE-ról a switchről. A hAP ax³ Wi-Fi szerepét veszi át. |
| 2× Digitus 10" 4-aljzatú 1U PDU | Rackes tápellátás 10" szélességben (a 19" Digitus DN-95441 / DN-95418 helyett). |
| Legrand Keor 800 VA UPS | Áramszünetnél szabályos leállítás; a korábbi APC Back-UPS 850VA helyett, 10" / multiplug formátum. |
| deleyCON 12 portos 10" keystone patch panel + Cat7 keystone + Cat7 patch kábelek | Tiszta kábelezés a rack elején. |
| 4× Digitus 10" 1U tálca | Firewalla, szerver, UPS és kiegészítők polcra. |
| Rackstuds R100 Series II | Szerszám nélküli rögzítés. |
| Noctua NA-SAVG1 + NA-SAV2 | Ventilátor rezgéscsillapítás a csendes 10" házhoz. |

A 3D-nyomtatott rack (PETG váz, TPU talp, egyedi előlapok) dokumentált **alternatíva**, nem a célút. Részletek: [10" rack](../infrastructure/rack.md).

## Munkaállomás (2.0, változatlan)

Ezek a 2.6-ban nem módosultak: új asztallap (180×80×2.5 cm), AV Access 4KSW41C KVM, duál gázrugós monitorkar, Blue Yeti X, Brother DCP-L2620DW, kábelmenedzsment és világítás. Részletek: [Asztal és KVM](../workstation/desk-kvm.md) és [Fázisok és BOM](phases-bom.md).

## Jövőbeli hardver (nem a mostani cél)

- 3 node mini-PC Proxmox cluster
- Home server NAS átalakítása 4–5 lemezes DAS/JBOD házzá (pl. Yottamaster FS5C3)

Részletek: [Jövőbeli bővítés](future.md).
