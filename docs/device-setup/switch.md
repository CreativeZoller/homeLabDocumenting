# UniFi Lite 16 PoE — switch

A Lite 16 PoE a 2.6-os L2 gerinc. A MikroTik CSS610 / CRS310 helyére lép. 16× 1 GbE, 8 PoE port (802.3af/at, összes PoE költségvetés a modell adatlapja szerint, tipikusan 45 W). Nincs SFP+; 10G gerinc nincs.

Kezelés: UniFi Network Application (önállóan a home serveren Dockerben, vagy külön UniFi Console). A switch önmagában nem RouterOS.

## Hálózati összekapcsolás

| Eszköz | Port (javaslat) | Típus | Cél | VLAN |
|--------|-----------------|-------|-----|------|
| Firewalla | LAN | Trunk | Uplink a switch 1. portjára | 10, 20, 30, 40 tagged |
| Lite 16 | Port 1 | Trunk | Firewalla | ugyanaz |
| Lite 16 | Port 2 | Access | Home server NIC 2 | 10 |
| Lite 16 | Port 3 | Access | Raspberry Pi | 10 |
| Lite 16 | Port 4 | PoE + Access vagy Trunk | UniFi U7 Pro | 10 native, 20/40 tagged az SSID-khez |
| Lite 16 | Port 5–8 | PoE Access | IP kamerák / IoT | 30 |
| Lite 16 | Port 9–16 | Access | Kliensek / tartalék | 10 vagy 40 |

A pontos portszám a rack kábelezésénél rögzítendő. Az U7 Pro-nak PoE kell, és trunk, ha több SSID/VLAN megy rá.

## UniFi Network: hálózatok

Settings → Networks, a Firewalla VLAN-jaival egyezően. A switch **L2 only**: ne legyen a UniFi a DHCP/router, azt a Firewalla adja.

| Név | VLAN | Alhálózat (csak dokumentáció) | DHCP |
|-----|------|-------------------------------|------|
| Fo | 10 | 192.168.10.0/24 | Firewalla |
| Vendeg | 20 | 192.168.20.0/24 | Firewalla |
| IoT | 30 | 192.168.30.0/24 | Firewalla |
| Kliens | 40 | 192.168.40.0/24 | Firewalla |

Switch menedzsment VLAN: 10, statikus `192.168.10.2`, gateway `192.168.10.1`.

## Port profilok

Settings → Profiles → Switch Ports:

- **Trunk-Firewalla:** native VLAN 10, tagged 20, 30, 40.
- **AP-U7:** native VLAN 10, tagged 20 és 40 (vendég és kliens SSID). IoT SSID ha kell: tagged 30.
- **Access-10:** VLAN 10 untagged, többi tiltva.
- **Access-30-IoT:** VLAN 30 untagged, többi tiltva.
- **Access-40-Kliens:** VLAN 40 untagged.

Port 1 ← Trunk-Firewalla, Port 2–3 ← Access-10, Port 4 ← AP-U7, PoE kamera portok ← Access-30-IoT.

## PoE

- U7 Pro: PoE+ (ellenőrizd az AP adatlapját; a Lite 16 45 W-os büdzséje mellett az AP + néhány kamera fér el).
- Ha a büdzsé kevés: kamerák injektorra, AP marad a switch PoE-ján.

## Wi-Fi (röviden)

Az SSID-k az U7 Pro-n készülnek, a VLAN a switch/AP profilból jön. Részletek: [UniFi U7 Pro](unifi-ap.md).

## Ellenőrzés

- UniFi Networkben a switch Adopted, firmware aktuális.
- Firewalla látja a switch MAC-jét a VLAN 10-en.
- Access portos PC csak a saját VLAN DHCP-jét kapja.
- Trunk porton tagged forgalom megy az AP-re (vendég SSID = 192.168.20.x).
