# MikroTik hAP ax³ (C53UiG+5HPaxD2HPaxD) Alapvető HomeLab Beállítás

A konfiguráció a következőket tartalmazza:

- Gyári beállítások törlése (tiszta lappal indulás).
- WAN konfiguráció (optikai/fiberglass kapcsolaton keresztül).
- LAN/Bridge konfiguráció (a portok összekötése).
- Alapvető biztonsági intézkedések (tűzfal, jelszó).
- DHCP és NAT beállítása.

*Fontos:* Mivel a hAP ax³ RouterOS 7-et használ, a parancsok ennek a verziónak megfelelőek. A konfigurációt SSH-n vagy WinBox terminálon keresztül adhatod ki.

## 🛡️ MikroTik hAP ax³ Alap Konfiguráció

### Tisztítás és Alapok (Reset Configuration)

Mindig azzal kezdj, hogy törlöd a router gyári alapkonfigurációját, hogy ne okozzon konfliktust a hálózati beállításokkal.

- Törli a router alapértelmezett beállításait. A router ÚJRAINDUL!

```bash
/system reset-configuration no-defaults=yes skip-backup=yes
```

Csatlakozz újra (az alapértelmezett IP most már nem él, ha a PC-d IP-t kapott DHCP-n, állíts be fix IP-t a PC-n, pl. 192.168.88.2/24 tartományban, és csatlakozz a hAP ax³ MAC-címén vagy az alapértelmezett 192.168.88.1-en keresztül, ha az éppen él).

### Alapvető Beállítások (Identitás és Jelszó)

Állíts be egyedi router nevet (identitást) és azonnal változtasd meg a gyári admin jelszót!

- Router nevének beállítása

```bash
/system identity set name=HomeLab_MikroTik
```

- Admin jelszó megváltoztatása (!!!EZ KÖTELEZŐ!!!)

```bash
/user set 0 password="YOUR_NEW_STRONG_PASSWORD"
```

### Hálózati Interfészek Konfigurációja (WAN és LAN)

A hAP ax³ portjai:

- sfp1 (esetünkben ez a Fiber WAN bemenet)
- ether1-ether4 (LAN portok)

#### LAN Bridge Létrehozása (Belső Hálózat)

Összekötjük a LAN portokat egyetlen belső hálózatba, és ide csatlakozik az új menedzselt Switch is.

- Bridge létrehozása a LAN hálózathoz

```bash
/interface bridge add name=bridge-lan
```

- LAN portok hozzáadása a Bridge-hez (a switch ide csatlakozik)

```bash
# ether1-4 a LAN portok a hAP ax3-on
/interface bridge port add bridge=bridge-lan interface=ether1
/interface bridge port add bridge=bridge-lan interface=ether2
/interface bridge port add bridge=bridge-lan interface=ether3
/interface bridge port add bridge=bridge-lan interface=ether4
```

#### IP-címek beállítása

WAN (sfp1) beállítása (DHCP-vel) Ha az ISP (szolgáltató) DHCP-n keresztül ad IP-címet az SFP portra, használd ezt:

```bash
/ip dhcp-client add interface=sfp1 disabled=no
```

VAGY (PPPoE, ha ezt kéri az ISP):

```bash
/interface pppoe-client add name=pppoe-out1 interface=sfp1 user="USER@ISP" password="ISP_PASSWORD" add-default-route=yes disabled=no
```

LAN (bridge-lan) statikus IP-címe A HomeLab hálózat alapértelmezett IP-címzési tartománya legyen 192.168.10.0/24 (VLAN 10 alapja).

- Statikus IP a belső bridge-re

```bash
/ip address add address=192.168.10.1/24 interface=bridge-lan
```

### NAT és Tűzfal (Alapvető Biztonság)

Ez biztosítja, hogy a belső hálózat el tudja érni az internetet (NAT/Masquerade), és alapvető védelmet ad a külső behatolások ellen.

#### NAT beállítása

A NAT (Network Address Translation) kötelező, hogy a belső (privát) IP-címek kimenjenek az internetre a publikus WAN IP-címmel.

```bash
/ip firewall nat add chain=srcnat action=masquerade out-interface=sfp1 comment="Masquerade to Internet"
```

#### Tűzfal beállítása (Alapvédelem)

Ez egy szigorú alapbeállítás, ami védi a routert és a belső hálózatot a kintről érkező kapcsolatoktól (Drop all, ha nem engedélyezett).

```bash
# 1. Bejövő forgalom (input chain) – Routerünk védelme

# Engedélyezzük a már létező és kapcsolódó forgalmat
/ip firewall filter add chain=input action=accept connection-state=established,related comment="Accept established and related"

# Engedélyezzük az SSH/Winbox elérést a belső hálózatról (bridge-lan)
/ip firewall filter add chain=input action=accept protocol=tcp src-address=192.168.10.0/24 in-interface=bridge-lan comment="Accept WinBox/SSH from LAN"

# Ejtjük (drop) a kintről a routerre érkező összes érvénytelen forgalmat
/ip firewall filter add chain=input action=drop in-interface=sfp1 comment="Drop all other from WAN"

# 2. Továbbított forgalom (forward chain) – Belső hálózat védelme

# Engedélyezzük a már létező és kapcsolódó forgalmat (szintén kötelező)
/ip firewall filter add chain=forward action=accept connection-state=established,related comment="Accept established and related"

# Ejtjük a kívülről a belső hálózat felé érkező érvénytelen forgalmat
/ip firewall filter add chain=forward action=drop connection-state=invalid comment="Drop invalid connections"

# Ejtjük a kívülről érkező, nem kért forgalmat a LAN felé
/ip firewall filter add chain=forward action=drop connection-state=new connection-nat-state=!dstnat in-interface=sfp1 comment="Drop all incoming non-requested from WAN"
```

### DHCP Server Beállítása (IP-címek Automatikus Osztása)

A DHCP szerver osztja ki automatikusan az IP-címeket a hálózati eszközöknek.

```bash
# 1. Létrehozzuk a DHCP poolt (192.168.10.100-254 tartomány)
/ip pool add name=dhcp-pool-lan ranges=192.168.10.100-192.168.10.254

# 2. Létrehozzuk a DHCP hálózatot
/ip dhcp-server network add address=192.168.10.0/24 gateway=192.168.10.1 netmask=24 dns-server=8.8.8.8,1.1.1.1 comment="LAN DHCP Network"

# 3. Létrehozzuk a DHCP szervert a bridge-lan interfészen
/ip dhcp-server add name=dhcp-lan interface=bridge-lan address-pool=dhcp-pool-lan disabled=no
```

*Későbbi lépés (HomeLab):* Amikor a Raspberry Pi-hole DNS-szerver beállítása megtörténik, a DHCP szerver beállításainál a dns-server mezőt módosítani kell a Pi statikus IP-címére (192.168.10.X).

### DNS Beállítás

Állítsd be, hogy a router mely külső DNS szervereket használja.

```bash
/ip dns set allow-remote-requests=yes servers=1.1.1.1,8.8.8.8
```

Ezzel az alap konfigurációval a MikroTik routered biztonságosan felépül:

- Van internetkapcsolat (SFP WAN).
- A belső hálózat működik (bridge-lan, DHCP).
- Az alapvető tűzfal védelem be van állítva a behatolások ellen.

## 🔄 MikroTik RB5009UG+S+IN Konfiguráció - Fejlesztett HomeLab

A MikroTik RB5009UG+S+IN egy kiváló, professzionális szintű eszköz, ami jelentős hardveres teljesítményugrást jelent a hAP ax³-hoz képest s tökéletesen beillik mind a 10" mind a 19" szekrényekbe.

A konfigurációs logika és a parancsok 90%-a megmarad, mivel mindkét eszköz RouterOS v7-et használ.

Azonban van néhány hardverrel kapcsolatos különbség a portok elnevezésében és fizikai felépítésében, amit figyelembe kell venni a CLI parancsoknál.

A fő eltérés a fizikai interfészek (portok) elnevezésében van. Az RB5009-en nincsenek dedikált ether1 vagy sfp1 portok, hanem számozott portokat használ:

- *WAN port (Fiber/SFP+):* Az RB5009-en ez a sfp-plus1 interfész.
- *LAN portok:* ether1-től ether7-ig mennek (7 darab 2.5G port).

A MikroTik RB5009 használatával a korábbi konfigurációhoz képest a parancsok csak minimálisan változnak (interfész nevek), de a HomeLab teljesítménye és a jövőbeni bővíthetőség (különösen a 10G SFP+ port) jelentősen javul.

### Változó parancsok az RB5009-hez

A korábbi hAP ax³ parancsokban csak az interfész neveket kell lecserélni.

| Konfigurációs Elem | hAP ax³ Parancsban Használt Interfész Neve | RB5009-re Átírt Interfész Neve |
|---|---|---|
| WAN Bemenet | sfp1 | sfp-plus1 |
| LAN Bridge Portok | "ether1, ether2, ether3, ether4" | ether1-től ether7-ig |
| NAT és Tűzfal | sfp1 | sfp-plus1 |

### Konfigurációs Parancsok RB5009-re Átírva

#### LAN Bridge Létrehozása (Belső Hálózat)

Az összes LAN portot (ether1-ether7) hozzáadjuk a belső hálózathoz.

```bash
# 1. Bridge létrehozása a LAN hálózathoz
/interface bridge add name=bridge-lan

# 2. LAN portok hozzáadása a Bridge-hez (ether1-ether7)
/interface bridge port add bridge=bridge-lan interface=ether1
/interface bridge port add bridge=bridge-lan interface=ether2
/interface bridge port add bridge=bridge-lan interface=ether3
/interface bridge port add bridge=bridge-lan interface=ether4
/interface bridge port add bridge=bridge-lan interface=ether5
/interface bridge port add bridge=bridge-lan interface=ether6
/interface bridge port add bridge=bridge-lan interface=ether7
```

#### WAN (sfp-plus1) beállítása (DHCP-vel)

Az ISP-től kapott DHCP IP-cím igénylése az SFP+ porton:

```bash
/ip dhcp-client add interface=sfp-plus1 disabled=no
```

#### NAT beállítása

A NAT beállítása a külső SFP+ portra:

```bash
/ip firewall nat add chain=srcnat action=masquerade out-interface=sfp-plus1 comment="Masquerade to Internet"
```

#### Tűzfal módosítása

A tűzfal szabályokat is át kell írni az új WAN interfész nevére:

```bash
# Ejtjük (drop) a kintről a routerre érkező összes érvénytelen forgalmat
/ip firewall filter add chain=input action=drop in-interface=sfp-plus1 comment="Drop all other from WAN"

# Ejtjük a kívülről érkező, nem kért forgalmat a LAN felé
/ip firewall filter add chain=forward action=drop connection-state=new connection-nat-state=!dstnat in-interface=sfp-plus1 comment="Drop all incoming non-requested from WAN"
```
