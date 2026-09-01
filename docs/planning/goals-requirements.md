# 🎯 Célok és követelmények

A HomeLab elsődleges célja egy személyes, biztonságos, rugalmas otthoni IT-infrastruktúra. A fókusz: média, automatizálás, hálózati kontroll, tanulás, későbbi bővítés. A 2.6-os fizikai keret: 10" rack a home server mellett (költség, hely, zaj).

## Jelenlegi célok

- **Adattárolás és média:** 2× 12 TB IronWolf ZFS mirror a szerverben. Jellyfin, Audiobookshelf.
- **Adatkontroll és biztonság:** Vaultwarden, Nextcloud, Pi-hole. Tűzfal: Firewalla Gold Plus, VLAN 10/20/30/40.
- **Otthoni automatizálás:** Home Assistant Dockerben (később Yeelight / Groove LED).
- **Távoli elérés:** Tailscale (MagicDNS). Publikus port forward csak ha muszáj.
- **Szolgáltatáskezelés:** Docker + Portainer. Compute: Ubuntu Server + KVM a home serveren.

## Jövőbeli célok

- **Virtualizáció a home serveren:** Kali, OSINT, teszt VM-ek. ZFS `tank/vm`, bridge `br0`.
- **Proxmox cluster:** 3 mini-PC, a home server NAS szerepe DAS/JBOD felé vihető. Nem a mostani Ubuntu host cseréje.
- **Felügyelet:** LibreNMS, Uptime Kuma, Plausible.
- **Üzlet:** Odoo helyben.
- **Munkaállomás:** asztal, KVM, ergonómia — 2.0 2–4. fázis, a 2.6 nem változtatta.

A D-Link / 10 Gbps SFP bővítés **nem** cél. A gerinc 1 GbE (UniFi Lite 16), 2.5G a Firewalla ↔ szerver linken.
