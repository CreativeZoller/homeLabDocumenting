# 📊 Homelab Hálózati Topológia

## Induló állapot

![Hálózati Topológia](images/network-map.png)

### 🌐 induló Homelab IP-cím és Port Kiosztás

| Eszköz / Szolgáltatás | VLAN | IP-cím | Port | Protokoll | Megjegyzés |
|---|---|---|---|---|---|
| MikroTik hAP ax³ | 10 | 192.168.10.1 | - | - | Központi Router / Gateway |
| CSS610 Switch | 10 | 192.168.10.2 | - | HTTP | Menedzsment felület (SwOS) |
| Ubuntu Fő Szerver | 10 | 192.168.10.10 | 22 | SSH | Docker és KVM Host |
| Raspberry Pi 4 | 10 | 192.168.10.11 | 53 | DNS | Pi-hole (Elsődleges DNS) |
| Portainer GUI | 10 | 192.168.10.10 | 9000 | HTTP | Docker menedzsment |
| SWAG (Proxy) | 10 | 192.168.10.10 | 443 | HTTPS | SSL/Reverse Proxy kapu |
| Authentik | 10 | 192.168.10.10 | 9443 | HTTPS | SSO Azonosítás |
| Nextcloud | 10 | 192.168.10.10 | 444 | HTTPS | Adatfelhő és fájlok |
| Jellyfin | 10 | 192.168.10.10 | 8096 | HTTP | Médiaszerver |
| Home Assistant | 10 | 192.168.10.10 | 8123 | HTTP | Okosotthon központ |
| qBittorrent | 10 | 192.168.10.10 | 8080 | HTTP | Letöltő kliens |
| Sonarr / Radarr | 10 | 192.168.10.10 | 8989/7878 | HTTP | Média automatizáció |
| Immich | 10 | 192.168.10.10 | 2283 | HTTP | Fotómentés és galéria |
| Kali Linux VM | 10 | 192.168.10.50 | - | - | Biztonsági teszt VM |
| pfSense VM | 10 | 192.168.10.51 | - | - | Teszt tűzfal (WAN/LAN) |
| Tails / OSINT VM | 10 | 192.168.10.52+ | - | - | Speciális célú VM-ek |
| IP Kamerák | 20 | 192.168.20.100+ | - | RTSP | IoT szegmens (Frigate) |

## Végső állapot

![Hálózati Topológia](images/network-map-updated.png)

### 🌐 Végső Homelab IP-cím és Port Kiosztás

| Eszköz / Szolgáltatás | VLAN | IP-cím | Port | Megjegyzés |
|---|---|---|---|---|
| MikroTik hAP ax³ | 10 | 192.168.10.1 | - | Fő Router / Gateway |
| MikroTik CSS610 | 10 | 192.168.10.2 | - | 10Gb Switch Menedzsment |
| Ubuntu Server Host | 10 | 192.168.10.10 | 22 | Docker & VM Gazdagép |
| Raspberry Pi 4 | 10 | 192.168.10.11 | "53 |  80" | Pi-hole DNS & Pi.alert |
| SWAG Proxy | 10 | 192.168.10.10 | "80 |  443" | HTTPS Bejárat (Reverse Proxy) |
| Homepage | 10 | 192.168.10.10 | 3000 | Fő Dashboard |
| Authentik | 10 | 192.168.10.10 | 9443 | SSO Azonosítás |
| Nextcloud | 10 | 192.168.10.10 | 444 | Fájlfelhő |
| Odoo / Paperless | 10 | 192.168.10.10 | "8069 |  8010" | Irodai alkalmazások |
| Jellyfin | 10 | 192.168.10.10 | 8096 | Médiaszerver (iGPU Transcode) |
| qBittorrent | 10 | 192.168.10.10 | 8080 | Letöltő kliens |
| Sonarr/Radarr/Prowlarr | 10 | 192.168.10.10 | "8989 |  7878 |  9696" | Média automatizáció (*arr) |
| Immich | 10 | 192.168.10.10 | 2283 | Fotó backup & Galéria |
| Home Assistant | 10 | 192.168.10.10 | 8123 | Okosotthon központ |
| Frigate / Scrypted | 10 | 192.168.10.10 | "5000 |  10443" | Kamera kezelés & AI |
| GitLab / n8n | 10 | 192.168.10.10 | "8081 |  5678" | DevOps & Automatizáció |
| LibreNMS / Plausible | 10 | 192.168.10.10 | "8001 |  8002" | Monitorozás & Analitika |
| Kali Linux VM | 10 | 192.168.10.50 | - | KVM Virtuális Gép |
| pfSense Test VM | 10 | 192.168.10.51 | - | KVM Virtuális Gép |
| TraceLabs / CSILinux | 10 | 192.168.10.52+ | - | KVM Virtuális Gép(ek) |
| IP Kamerák | 20 | 192.168.20.100-110 | 554 | RTSP Stream (IoT VLAN) |
