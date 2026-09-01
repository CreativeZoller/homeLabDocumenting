# Eszközök megfelelő kötése

A hálózati terv fizikai megvalósítása: Firewalla, UniFi switch, U7 Pro, home server és a 10" rack kábelezése. Nincs 10G SFP+ gerinc; a switch 1 GbE, a 2.5G a Firewalla ↔ szerver opcionális link.

## 🔌 Fizikai bekötési terv (L1)

### 1. WAN és Firewalla

- **Kötés:** szolgáltatói modem → Firewalla Gold Plus WAN.
- **Megjegyzés:** a modem nem hidalható. WAN mód (DHCP vagy PPPoE) TBD.
- A Firewalla a rack egyik 1U tálcáján ül.

### 2. Firewalla → UniFi Lite 16 PoE

- **Kötés:** Firewalla LAN port → switch uplink (VLAN trunk: 10, 20, 30, 40).
- **Kábel:** Cat7, a patch panelen keresztül, rövid FGB Cat7 patch a rackben (0.5 m / 0.75 m).

### 3. Home server (i5-12400, 2× I226-V)

- **NIC 1 (2.5G, ajánlott):** Firewalla 2.5G LAN → szerver. Ez viszi a NAS / Docker forgalmat, ha a Firewalla portja szabad.
- **NIC 2 (1G):** UniFi Lite 16 egyik nem-PoE portja → szerver. Menedzsment / tartalék.
- **Kábel:** Cat7 S/FTP, keystone a patch panelen.

Ha a 2.5G közvetlen link még nincs bekötve, a szerver csak a switch 1 GbE portján lóg — ez működik, csak lassabb.

### 4. UniFi U7 Pro

- **Kötés:** UniFi Lite 16 PoE port → U7 Pro.
- PoE a switchről; külön táp nem kell.
- Az AP lehet a racken kívül, jobb Wi-Fi lefedettségért. Trunk a vendég (VLAN 20) és kliens (VLAN 40) SSID-khez.

### 5. Raspberry Pi 4 és IoT

- **Pi:** UniFi Lite 16 Gigabit port, VLAN 10 access. Pi-hole miatt vezetékes kapcsolat kötelező.
- **IP kamerák / IoT:** VLAN 30 access portok. Ha az eszköz PoE-s, a Lite 16 PoE portjaira mehet.

## 🛠️ Kábelezési szabályok

- **Színkód (opcionális):** kék = általános adat, piros = kritikus (szerver, Firewalla, switch), sárga = IoT.
- **Hajlítási sugár:** a Cat7 vastagabb; ne törni derékszögben.
- **Patch panel:** hosszú kábelek a keystone-ba, belül 0.5–0.75 m Cat7 patch a portra.
- **UPS:** a Legrand Keorba a Firewalla, a switch és a szerver tápja. Ha a UPS tud USB jelzést, kösd a szerverre a graceful shutdownhoz. A Legrand 800 VA multiplug USB töltőt is ad; a szerver NUT/USB leállítás TBD.

## 📦 Hardver-sorrend a 10" rackben (fentről lefelé)

Hűtés és kábelrendezés:

1. Patch panel (keystone, 1U)
2. UniFi Lite 16 PoE
3. Firewalla Gold Plus (tálcán)
4. Digitus 10" PDU
5. Home server (tálcán, alulabb: nehezebb, melegebb)
6. Legrand Keor UPS (legalul)

Az U7 Pro jellemzően a racken kívül, a szobában. A Raspberry Pi a switch egy portján, tálcán vagy a rack mellett.

Részletes rack-lista: [10" rack](../infrastructure/rack.md).
