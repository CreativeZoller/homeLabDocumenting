# Célok és Követelmények: A HomeLab feladatai

A HomeLab elsődleges célja egy személyes, biztonságos, nagyteljesítményű és rugalmas otthoni IT-infrastruktúra biztosítása. Jelenleg a fókusz a média, az automatizálás és a hálózati kontroll terén van, de a terv magában foglalja a tanulást és a jövőbeni bővítést is.

## 🎯 Jelenlegi Célok

- *Adattárolás és Média:* Központi adattárolás (12 TB ZFS pool ) a szerverben. Média szerver üzemeltetése (Jellyfin, Audiobookshelf ) a tartalmak streamingjéhez.
- *Adatkontroll és Biztonság:* Saját jelszótároló (Vaultwarden ) és saját felhő (Nextcloud ) üzemeltetése a szuverenitás érdekében. Hálózati hirdetésblokkolás (Pi-hole ).
- *Otthoni Automatizálás:* Központi Home Assistant  futtatása (Docker) az okoseszközök menedzselésére.
- *Távoli Elérés:* Biztonságos távoli hozzáférés biztosítása a teljes hálózathoz a Tailscale használatával (MagicDNS-szel ).
- *Szolgáltatáskezelés:* Minden szolgáltatás Docker konténerben fut Portainer felületen keresztül a könnyű telepítés és karbantartás érdekében.

## ✨ Jövőbeli Célok és Követelmények

- *Virtualizáció és Tesztelés:* Virtuális gépek (VM-ek) futtatása tanulási és tesztelési célokra (pl. Kali Linux és TS OLINT ). Ehhez KVM/QEMU telepítése, egy dedikált ZFS dataset (tank/vm ) és bridge hálózat (br0 ) szükséges.
- *Rendszerfelügyelet:* Részletes hálózati monitorozás (LibreNMS ) és szolgáltatás állapotfigyelés (Uptime Kuma ).
- *Web Analytics:* Saját, szuverén webanalitika (Plausible ) hosztolása.
- *Bővíthetőség:* A D-Link switch SFP portja  későbbi 10 Gbps-os bővítés lehetőségét jelzi.
- *Menedzselés:* A vast collection of business apps can be held and ran locally (Odoo ).
