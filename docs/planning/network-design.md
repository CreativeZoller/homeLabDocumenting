# Hálózati Tervezés

A hálózat központi eleme egy MikroTik Router és egy menedzselt Mikrotik Switch, ami lehetővé teszi a hálózati szegmentációt (VLAN-ok) a biztonság és a rend kedvéért.

## 🕸️ Jelenlegi Hálózati Topológia

A hálózat csillag topológiában épül fel, switch még nincsen. A bejövő kapcsolat közveltenüla routerbe kapcsolódik WAN porton át, majd onnét megy tovább a végkészülékekbe.

## 🛡️ Hálózati Szegmentáció (VLAN Terv)

A terv szerint a hálózat a biztonság növelése érdekében szegmentálva van:

| VLAN ID | Szerep | IP Tartomány |
|---------|--------|--------------|
| 10 | Fő Hálózat | 192.168.10.0/24 (Szerverek, admin, munkaállomások) |
| 20 | IoT | 192.168.20.0/24 (Okoseszközök, Kamerák) |
| 30 | Vendéghálózat | 192.168.30.0/24 |

A MikroTik router végzi a VLAN tagelést és a forgalomirányítást a különböző tartományok között.
