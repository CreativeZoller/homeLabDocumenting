# MikroTik CSS610-8G-2S+IN Fejlesztett HomeLab Beállítás

Haladjunk a következő, kritikus lépéssel: a VLAN-okkal szegmentált hálózat beállításával a MikroTik RB5009 router és a MikroTik CSS610-8G-2S+IN Cloud Smart Switch között.

Ez a két eszköz (RouterOS és SwOS) együttműködése biztosítja a hálózati tervben szereplő VLAN-ok (10, 20, 30) megfelelő kezelését.

A CSS610 egy SwOS (Switch OS) alapú switch, ami azt jelenti, hogy a konfiguráció webes felületen vagy SwOS CLI-n keresztül történik (nem RouterOS CLI-n, mint az RB5009). Mivel a SwOS-nak nincs "klasszikus" CLI-je, a parancsokat helyettesítem a konkrét beállítási lépésekkel és paraméterekkel, melyeket a webes felületen kell elvégezni.

*Hálózati Összekapcsolás*

| Eszköz | Port | Kapcsolat Típusa | Cél | VLAN-ok |
|---|---|---|---|---|
| RB5009 (Router) | ether1 (LAN Port) | TRUNK (Tagged) | Uplink a switch-hez | "10, 20, 30" |
| CSS610 (Switch) | SFP+1 (vagy SFP+2) | TRUNK (Tagged) | Uplink a routerhez | "10, 20, 30" |

## MikroTik RB5009 Router Konfiguráció (RouterOS)

Az RB5009-en létrehozzuk a virtuális VLAN interfészeket a már meglévő bridge-lan helyett, hogy a router tudja kezelni a VLAN-ok közötti forgalmat.

Feltételezés: A korábbi RB5009 konfigurációból az sfp-plus1 a WAN port, az ether1-től az ether7-ig a LAN portok. Az ether1-et fogjuk használni a switch Uplinkjének.

### Alap Bridge beállítása VLAN-okkal

Először törölni kell a korábbi, VLAN nélküli bridge-lan beállításokat, ha még élnének, majd létrehozzuk az új, VLAN-támogatású bridge-et.

```bash
# 1. Töröljük a korábbi (nem VLAN-os) bridge-et és portjait, ha léteztek.
# /interface bridge remove [find name=bridge-lan]
# /interface bridge port remove [find bridge=bridge-lan]

# 2. Létrehozzuk a VLAN-képes bridge-et
/interface bridge add name=vlan-bridge vlan-filtering=yes

# 3. LAN interfészek hozzáadása a VLAN Bridge-hez (ide csatlakozik a switch)
# Az RB5009 ether1 portja a TRUNK port a switch felé
/interface bridge port add bridge=vlan-bridge interface=ether1 comment="Switch Uplink TRUNK"
# A többi ether port továbbra is a belső hálózati eszközök fogadására szolgálhat (nem feltétlen szükséges, ha minden eszköz a CSS610-re megy)
/interface bridge port add bridge=vlan-bridge interface=ether2 pvid=10 comment="LAN PVID 10"
/interface bridge port add bridge=vlan-bridge interface=ether3 pvid=10 comment="LAN PVID 10"

# 4. VLAN TAG-ek (címkék) beállítása a Bridge-en
# Meghatározzuk, mely VLAN-ok futnak az egyes portokon.
/interface bridge vlan add bridge=vlan-bridge vlan-ids=10,20,30 tagged=ether1,vlan-bridge comment="TRUNK Port - Minden VLAN"
# Meghatározzuk a PVID (Primary VLAN ID) 10-et az ether2/3-ra, mint Untagged port
/interface bridge vlan add bridge=vlan-bridge vlan-ids=10 untagged=ether2,ether3 comment="Default PVID 10 (untagged) ports"

# 5. Bridge IP-címének eltávolítása (ha van) és IP-címek beállítása a VLAN Interfészekre
# Töröld a korábbi 192.168.10.1/24 beállítást, ha a bridge-lan-en volt!
# /ip address remove [find interface=bridge-lan]

/interface vlan add name=vlan10_lan interface=vlan-bridge vlan-id=10
/interface vlan add name=vlan20_iot interface=vlan-bridge vlan-id=20
/interface vlan add name=vlan30_guest interface=vlan-bridge vlan-id=30

/ip address add address=192.168.10.1/24 interface=vlan10_lan comment="Gateway for Main LAN"
/ip address add address=192.168.20.1/24 interface=vlan20_iot comment="Gateway for IoT"
/ip address add address=192.168.30.1/24 interface=vlan30_guest comment="Gateway for Guest"
```

### DHCP Szerverek beállítása

Módosítjuk a DHCP-t, hogy az új VLAN interfészeken ossza az IP-ket (Poolok és a Network részek ismétlésként, de itt már az új interface neveket használjuk!).

```bash
# 1. DHCP Poolok (korábban már lehet, hogy léteznek, ellenőrizd!)
/ip pool add name=dhcp-pool-10 ranges=192.168.10.100-192.168.10.254
/ip pool add name=dhcp-pool-20 ranges=192.168.20.100-192.168.20.254
/ip pool add name=dhcp-pool-30 ranges=192.168.30.100-192.168.30.254

# 2. DHCP Hálózatok (az új VLAN interfészekhez rendelve)
/ip dhcp-server network add address=192.168.10.0/24 gateway=192.168.10.1 dns-server=8.8.8.8,1.1.1.1 comment="VLAN 10 Network"
/ip dhcp-server network add address=192.168.20.0/24 gateway=192.168.20.1 dns-server=8.8.8.8,1.1.1.1 comment="VLAN 20 Network"
/ip dhcp-server network add address=192.168.30.0/24 gateway=192.168.30.1 dns-server=8.8.8.8,1.1.1.1 comment="VLAN 30 Network"

# 3. DHCP Szerverek létrehozása a VLAN interfészeken
/ip dhcp-server add name=dhcp-10 interface=vlan10_lan address-pool=dhcp-pool-10 disabled=no
/ip dhcp-server add name=dhcp-20 interface=vlan20_iot address-pool=dhcp-pool-20 disabled=no
/ip dhcp-server add name=dhcp-30 interface=vlan30_guest address-pool=dhcp-pool-30 disabled=no
```

### Tűzfal (Inter-VLAN forgalom szabályozása)

A legfontosabb biztonsági lépés, ami megakadályozza, hogy az IoT (VLAN 20) eszközök kommunikáljanak a Fő hálózattal (VLAN 10).

```bash
# 1. Alapvető szabály: Engedélyezzük a forgalmat a LAN-on belüli VLAN-ok között.
# Ezt kell finomítani! Alapértelmezésben engedélyezzük a Fő LAN (10) számára a kimenő forgalmat (pl. IoT felé)
/ip firewall filter add chain=forward action=accept src-address=192.168.10.0/24 dst-address=192.168.20.0/24 comment="LAN 10 can talk to IoT 20 (monitoring, HA)"

# 2. Blokkolási szabály: A szigorúbb (IoT) VLAN NE férjen hozzá a Fő LAN-hoz!
/ip firewall filter add chain=forward action=drop src-address=192.168.20.0/24 dst-address=192.168.10.0/24 comment="DROP: IoT 20 to LAN 10"

# 3. Blokkolási szabály: A vendéghálózat NE férjen hozzá sehova, csak az internetre
/ip firewall filter add chain=forward action=drop src-address=192.168.30.0/24 dst-address=!192.168.30.0/24 comment="DROP: Guest 30 to other LANs"

# Minden más tűzfal szabály (WAN input/forward) marad a korábban beállított módon.
```

## MikroTik CSS610-8G-2S+IN Switch Konfiguráció (SwOS)

A CSS610 webes felületén (alapértelmezett IP: 192.168.88.1, ha SwOS módban van) kell beállítani a portokat.

*Feltételezés:* Az RB5009 router ether1 portja a CSS610 SFP+1 portjához csatlakozik.

### Alapkonfiguráció (System fül)

- *Identity:* Változtasd meg a switch nevét: CSS610_Homelab.
- *Jelszó:* Változtasd meg az alapértelmezett admin jelszót (admin).
-*IP beállítás:* A switch statikus IP-t kap a Fő hálózati (VLAN 10) tartományból. Pl: 192.168.10.2/24 és a Gateway: 192.168.10.1.

### VLAN Mód beállítása (VLANs fül)

A CSS610 alapértelmezetten Disabled módban van. Át kell kapcsolni a Port Based VLAN módra.

- *VLAN Mode:* Állítsd be: Enabled
- *Default VLAN ID (PVID):* 1 (Ezt nem fogjuk használni, de legyen beállítva)
- *VLAN Receive:* any

### Portok Hozzárendelése (VLANs fül)

Minden portot beállítunk a hálózati terv szerint (Router Uplink, Szerver, Pi, IoT).

| Port | Cél | Default VLAN ID (PVID) | VLAN Tagging (Mode) |
|---|---|---|---|
| SFP+1 | Router Uplink | 10 | Enabled (Trunk port: továbbítja az összes VLAN ID-t) |
| Ether1 | Szerver | 10 | Disabled (Access port: csak a VLAN 10-et fogadja Untagged) |
| Ether2 | Raspberry Pi | 10 | Disabled |
| Ether3 | AP/Kliens | 10 | Disabled |
| Ether4-8 | Kliensek/Tartalék | 10 | Disabled |
| SFP+2 | Tartalék Uplink | 10 | Disabled |

Mentsd el a beállításokat (Save Configuration).

### VLAN Mappings beállítása (VLANs fül)

Itt állítjuk be, hogy mely VLAN ID-k engedélyezettek az egyes portokon, és melyek legyenek Tagged (címkézett) vagy Untagged (címkézetlen).

| VLAN ID | SFP+1 (Uplink) | Ether1 (Szerver) | Ether2 (Pi) | Ether4 (IoT) |
|---|---|---|---|---|
| 10 | always tag | untag if default | untag if default | untag if default |
| 20 | always tag | forbidden | forbidden | untag if default |
| 30 | always tag | forbidden | forbidden | forbidden |

Magyarázat a fenti táblázathoz:

- *SFP+1 (Uplink):* Az összes VLAN forgalom ezen a porton megy a router felé, ezért mindig címkézett (always tag).
- *Ether1 & 2 (Szerver és Pi):* Ezek az eszközök a Fő hálózaton (VLAN 10) vannak. A PVID 10 (lásd feljebb), a forgalom pedig címkézetlen (untag if default) – a készülékek nem tudnak a VLAN-okról.
- *Ether4 (IoT):* Az IoT eszközök a VLAN 20-ra kerülnek. A port Default VLAN ID-je 20-ra állítandó a 3. lépésben, és a 20-as sorban untag if default beállítással kerülnek be.

Ezzel a konfigurációval a CSS610 megfelelően továbbítja a VLAN-okkal címkézett forgalmat a router felé, miközben a klienseszközöknek nem kell tudniuk a VLAN-ok létezéséről (Access Portok).
