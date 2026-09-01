# Topológia

A célállapot egy Firewalla-központú, VLAN-okkal szegmentált hálózat: a routingot és a tűzfalat a Firewalla Gold Plus végzi, a L2 kiosztást az UniFi Lite 16 PoE, a Wi-Fi-t az UniFi U7 Pro.

## 🌐 A logikai felépítés pillérei

### 1. A Firewalla mint agy (L3)

A Gold Plus a WAN mögött ül (az osztrák modem nem hidalható). Ő a tűzfal, a DHCP, a VLAN-ok és az inter-VLAN routing.

- **VLAN-ok:** 10 Fő, 20 Vendég, 30 IoT, 40 Kliens. A Firewalla válogatja szét a zónákat.
- **Inter-VLAN:** alapesetben a VLAN-ok nem látják egymást. A Firewalla szabályai határozzák meg, hogy a Home Assistant (VLAN 10) beláthasson a kamerákhoz (VLAN 30), de a kamerák ne érjék el a szerver konfigurációját.
- **2.5G:** a Gold Plus 2.5G portjai. A home server I226-V kártyái ide (vagy a switch 1G portjára) csatlakozhatnak.

### 2. Az UniFi switch mint gerinc (L2)

A Lite 16 PoE feladata a portok kiosztása, a VLAN tagging és a PoE.

- **VLAN tagging:** a port tudja, melyik eszköz hova tartozik. IoT a VLAN 30 access portra, szerver a VLAN 10-re.
- **Helyi forgalom:** PC → szerver másolás VLAN 10-en belül a switchen marad. A switch 1 GbE, nem 10G.
- **PoE:** az U7 Pro a switchről kap áramot.

### 3. Az Ubuntu szerver: hibrid végpont

A szerver egyszerre Docker host és KVM host.

- **Docker Bridge:** a konténerek belső hálózaton beszélgetnek. A SWAG a kifelé nyíló kapu.
- **KVM Bridge (br0):** a VM-ek (Kali, teszt tűzfal) saját MAC/IP-t kapnak a VLAN 10-en, mintha fizikailag a switchben lennének.

### 🛡️ Biztonsági zónák

- **Trusted (VLAN 10):** ZFS, GitLab, Docker menedzsment, admin.
- **Guest (VLAN 20):** vendég Wi-Fi / vendég eszközök; csak internet.
- **Isolated IoT (VLAN 30):** lámpák, kamerák, szenzorok. Internet alapból tiltva. Home Assistant és Frigate éri el őket.
- **Client (VLAN 40):** mindennapi kliensek, elválasztva az admin zónától.
- **Public face (SWAG):** az egyetlen pont, ahol a 443-as port az internet felé nyílik (Firewalla port forwardon keresztül, ha kell).

## 🚀 Kábelezési segédlet

- Szolgáltatói modem → Firewalla WAN.
- Firewalla LAN → UniFi Lite 16 PoE (fő uplink).
- Firewalla 2.5G (opcionális) → Home Server NIC 1: a 2.5G adatút a NAS/Docker forgalomnak.
- UniFi Lite 16 PoE → Home Server NIC 2 (menedzsment / tartalék, 1 GbE).
- UniFi Lite 16 PoE PoE port → UniFi U7 Pro.
- UniFi Lite 16 PoE → Raspberry Pi.
- Patch panel: a rack elején a Cat7 keystone-ok fogadják a hosszú kábeleket; belül rövid Cat7 patch.

Részletek: [Eszközök kötése](../network-setup/device-connections.md).
