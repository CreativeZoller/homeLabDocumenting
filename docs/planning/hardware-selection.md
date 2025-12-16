# Hardver Kiválasztás

A hardver kiválasztásánál a fő szempont az energiahatékony teljesítmény (i5-12600) és az adatintegritás (ZFS, UPS).

## 🖥️ Jelenlegi Hardver Eszközök

- Fő Szerver: Mini-ITX alapú rendszer
    - CPU: Intel i3-12100F.
    - RAM: 16 GB DDR4 (ajánlott 32 GB  a ZFS és a VM-ek miatt).
    - Rendszerlemez: 250 GB NVMe.
- Router: MikroTik C53UiG+5HPaxD2HPaxD (hAP ax³).
- Segéd Szerver: Raspberry Pi 4.

## 🔄 Tervezett Eszközök és Indoklás

| Tervezett Eszköz / Frissítés | Indoklás |
|------------------------------|----------|
| VGA eltávolítása fő szerverből | Felszabadul PCI-E foglalat |
| CPU csere i5-12600-ra fő szerverben | Erő és energiatakarékosság, plusz VGA nem szükséges már |
| SFP modul (10G) beépítése | Jövőbeni sebesség növelés a szerver és a switch között egy már birtokolt PCIe kártyával |
| RAM Bővítés 32 GB-ra | A ZFS fájlrendszer és a jövőbeli VM-ek (KVM/QEMU) stabil futtatásához |
| Switch hozzáadása a topográfiához: MikroTik CSS610-8G-2S+IN  | Menedzselt switch külön vlanokkal, SFP kapcsolattal, 10" méretben |
| Router cserélye: MikroTik RB5009UG+S+IN  | Erősebb router, külön menedzselt vlan opcióval, SFP kapcsolattal, 10" méretben  |
| Tápellátás DIGITUS 4-Way Power Strip - DN-95418  | Hosszú távon minden eszköz a rackben ellátható árammal |
| APC Back-Ups 850VA - BE850G2-GR | Egy szünetmentes táp elegendő árammal képes ellátni áramszünet esetén az eszközöket annyi időre, amíg azok normálisan lekapcsolhatóak |
