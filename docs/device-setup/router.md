# 🛡️ Firewalla Gold Plus — tűzfal és router

A Gold Plus a 2.6-os célállapot szerinti fizikai tűzfal és router. A MikroTik hAP ax³ / RB5009 és az Asus RT-BE88U terv helyére lép. Az osztrák szolgáltatói modem nem hidalható, ezért a Firewalla a modem mögött, router módban ül.

A konfiguráció az hivatalos Firewalla appból (iOS / Android) és a Firewalla Box webes / asztali felületéről történik, nem CLI-ből.

## Feladat a topológiában

- WAN a szolgáltatói modem felé
- LAN / VLAN: 10 Fő, 20 Vendég, 30 IoT, 40 Kliens
- DHCP a VLAN-okon
- Inter-VLAN tűzfal
- Opcionális 2.5G link a home server felé

Hardver: [Hardver kiválasztás](../planning/hardware-selection.md). Bekötés: [Eszközök kötése](../network-setup/device-connections.md).

## Fizikai portok

A Gold Plus 2.5GBase-T portokkal rendelkezik. Tipikus kiosztás (a végső portszám a doboz feliratát kövesse):

| Port | Szerep | Megjegyzés |
|------|--------|------------|
| WAN | Modem | DHCP vagy PPPoE — TBD, ISP szerint |
| LAN | UniFi Lite 16 uplink | Trunk: VLAN 10, 20, 30, 40 |
| LAN 2.5G | Home server NIC 1 | Opcionális, NAS/Docker forgalom |
| További LAN | Tartalék | Pl. ideiglenes notebook a beállításhoz |

## Alapbeállítás

1. Firewalla táp a Digitus 10" PDU-ról / Legrand UPS-ről.
2. WAN kábel a modemről, LAN a switchre. Az első beállításnál a telefon ugyanarra a LAN-ra csatlakozzon (vagy a Firewalla Wi-Fi pairing folyamatára).
3. App: eszköz párosítása, firmware frissítés.
4. Mód: **Router** (ne Bridge). Simple mód nem ad VLAN-okat.
5. WAN: DHCP client, vagy PPPoE ha az ISP azt kéri. TBD.
6. LAN alaphálózat: `192.168.10.0/24`, gateway `192.168.10.1` (VLAN 10).

## VLAN-ok

A Firewalla Network / VLAN felületén:

| VLAN | Név | Hálózat | DHCP | DNS |
|------|-----|---------|------|-----|
| 10 | Fo | 192.168.10.0/24 | 192.168.10.100–254 | Raspberry Pi 192.168.10.11 (Pi-hole), tartalék a Firewalla |
| 20 | Vendeg | 192.168.20.0/24 | 192.168.20.100–254 | Firewalla vagy Pi-hole, szigorú szűrés |
| 30 | IoT | 192.168.30.0/24 | 192.168.30.100–254 | Pi-hole |
| 40 | Kliens | 192.168.40.0/24 | 192.168.40.100–254 | Pi-hole |

A switch felé menő LAN port legyen tagged trunk ezekkel a VLAN ID-kkel. A VLAN 10 mehet native/untagged az uplinken, ha az UniFi port profil így van felvéve — a két oldalnak egyeznie kell.

## Tűzfalszabályok (irány)

A Gold Plus Rules / Group szabályaival:

1. VLAN 10 → VLAN 30: engedélyezve (Home Assistant, Frigate).
2. VLAN 30 → VLAN 10: tiltva, kivéve a HA/Frigate által kezdeményezett, established forgalom.
3. VLAN 20 → RFC1918: tiltva; VLAN 20 → WAN: engedélyezve.
4. VLAN 30 → WAN: alapból tiltva; kivételek eszközönként (OTA, felhős híd).
5. VLAN 40 → VLAN 10: tiltva (kliens ne érje el az admin/szerver zónát), kivéve a reverse proxy / szükséges szolgáltatások.
6. WAN → LAN: csak explicit port forward (pl. 443 → SWAG), ha egyáltalán kell. Tailscale a preferált távoli elérés.

## Statikus címek

A Firewalla DHCP reservation:

- `192.168.10.2` UniFi switch
- `192.168.10.3` U7 Pro
- `192.168.10.10` home server
- `192.168.10.11` Raspberry Pi

## DNS

Ha a Pi-hole fut, a VLAN DHCP DNS mezője a Pi címe. A Firewalla saját DNS-e tartalék. A Pi-hole unbounddal a [Raspberry Pi](raspberry-pi.md) oldalon van.

## Ellenőrzés

- Appban a WAN zöld, a LAN eszközök megjelennek.
- VLAN 10-es kliens kap `192.168.10.x` címet.
- Vendég SSID (U7 Pro, VLAN 20) nem pingeli a `192.168.10.10`-et.
- IoT VLAN 30-as kamera streamje eléri a Frigate-et VLAN 10-ről, fordítva az SSH a szerverre nem megy a kameráról.
