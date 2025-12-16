# Szoftver és Szolgáltatások

Szinte minden szolgáltatás Docker konténerben fut a fő szerveren az Ubuntu Server operációs rendszeren, a tároláshoz pedig a ZFS  fájlrendszert használja. Az egyszerű CLI telepítés a teljes folyamatot fedi.

## 🚀 Fő Szerver

A legtöbb szolgáltatás konténerizáltan fut, a Portainer  a felügyeleti felület. Az adatok és konfigurációk a ZFS pool (/tank) dedikált adathalmazain (datasets) tárolódnak.

> Frissíteni a listát, miután a NAS setup véglegesedett

| Szolgáltatás | Kategória | Funkció | Forrásbeli Elhelyezés |
|---|---|---|---|
| Nextcloud | Felhő / Adatkontroll | Személyes, ön-hosztolt felhőmegoldás. Fájlszinkronizálás, naptár, kontaktok, dokumentumok kezelése. | /tank/nextcloud |
| Plausible | Monitoring / Analitika | Egyszerű, adatvédelmi fókuszú, ön-hosztolt webanalitikai eszköz. | Szerver (Docker) / Pi 4 (opcionális) |
| Frigate / Shinobi | Automatizálás / Média | Nyílt forrású videófelügyeleti rendszer. A Frigate általában a tárgyfelismerésre fókuszál. | Szerver (Docker) |
| Audiobookshelf | Média | Önhoszolt audiókönyv és podcast szerver, streameléshez. | Szerver (Docker) |
| Jellyfin / Plex | Média | Önhoszolt média szerver filmek, sorozatok, zenék streameléséhez (Jellyfin szerepel a tervben). | Szerver (Docker) |
| SWAG | Hálózati Segéd | Fordított proxy (Reverse Proxy) és HTTPS biztosítása (Let's Encrypt). | Szerver (Docker) |
| Immich | Média / Felhő | Ön-hosztolt fénykép és videó mentési megoldás (alternatívája a Google Fotóknak). | Szerver (Docker) |
| paperless-ngx | Dokumentumkezelés | Dokumentumkezelő rendszer, amely indexeli és archiválja a beszkennelt dokumentumokat (OCR-rel). | Szerver (Docker) |
| Heimdall / Dashboard | Menedzsment | Kezdőoldal/áttekintő felület (dashboard) az összes szolgáltatás könnyű eléréséhez. | Szerver (Docker) |
| Odoo | Üzleti Alkalmazás | Integrált üzleti alkalmazások gyűjteménye (CRM, raktárkezelés, számlázás, stb.) kisvállalkozások vagy személyes projektek számára. | Szerver (Docker) |
| Scrypted | Automatizálás | Okosotthon integrációs platform, gyakran IP kamerák és HomeKit integrációjára. | Szerver (Docker) |
| Calibre | Média | E-könyv menedzser és szerver. Lehetővé teszi az e-könyvek rendszerezését, konvertálását és hálózati megosztását. | Szerver (Docker) |
| GitLab | Fejlesztés | Önhoszolt Git tároló, CI/CD és projektmenedzsment eszköz. | Szerver (Docker) |
| Focalboard | Projektmenedzsment | Nyílt forrású, ön-hosztolt Trello-szerű projektmenedzsment eszköz (a Mattermost része is lehet). | Szerver (Docker) |
| Authentik | Biztonság | Hitelesítési (Authentication) és Identitáskezelési (Identity Provider) megoldás, központi belépést biztosít a HomeLab alkalmazásokhoz (SSO). | Szerver (Docker) |
| Romm | Média | Képregény-olvasó és menedzser (Comic Book Manager). | Szerver (Docker) |
| ConvertX | Média | Video konvertáló szoftver (Megjegyzés: Ez lehet egy Windows/asztali alkalmazás is, de Dockerben konvertáló konténereket is lehet futtatni). | Szerver (Docker) |
| Wallos | Éetmód | Előfizetések kezelése, követése. | Szerver (Docker) |
| Mealie | Életmód | Receptek gyűjtése és menedzselése, vásárlólista készítés receptek alapján. | Szerver (Docker) |
| BentoPDF | Média | Egy elég jó PDF szerkesztő app | Szerver (Docker) |
| N8N | Automatizálás | Egy teljesen self hostingolt automatizáló rendszer, amivel kb mindent is automatizálni lehet. | Szerver (Docker) |
| Ansible | Automatizálás | Konkrét folyamatok automatizálása és futtatása, pl. laptop teljes telepítése | Szerver (Docker) |
| Home Assistant | Automatizálás | Okos eszközök konfigurálása a házi hálózaton, automatizált megfigyeléssel és programozással (pl. lámpa kapcsolás sötétben) | Szerver (Docker) |

> Még több elérhető, self hosting appért az alábbi linket érdemes böngészni: *<https://selfh.st/apps/>*

## 🍓 Raspberry Pi 4

A Pi a könnyebb, hálózati alapú szolgáltatásokra van dedikálva, csökkentve ezzel a szerver terhelését.

- *Pi-hole + Unbound:* Hálózati szintű DNS hirdetésblokkolás, DNS szerver használattal.
- *Tailscale Node:* További Tailscale végpont a hálózatban.
- *Pi.alert:* Hálózati eszközök monitorozása.

## Kis áttekintés

- *Nextcloud:* A fájlok és adatok feletti szuverenitás központja. Ahelyett, hogy Google Drive-ot vagy Dropboxot használnál, a Nextcloud a saját szervered ZFS tárhelyén tárolja az adatokat.
- *Pi-hole + Unbound:* A Pi-hole  blokkolja a hirdetéseket és a nyomkövetőket a hálózat DNS-szintjén. Az Unbound hozzáadásával a Pi-hole nem a szolgáltató DNS szervereire támaszkodik, hanem közvetlenül a gyökér DNS szerverektől kérdezi le az IP-címeket, ezzel növelve a sebességet és a magánélet védelmét.
- *SWAG:* Mivel sok Docker konténer fut különböző portokon (pl. Portainer 9443 , Jellyfin 8096 ), a SWAG teszi lehetővé, hogy az összes szolgáltatás egyetlen kapun keresztül, biztonságosan (HTTPS) és jól megnevezve (pl. jellyfin.homelab.local) legyen elérhető.
- *Frigate / Scrypted:* Kritikus fontosságúak az okosotthon biztonsági rétegében. A Frigate képes valós idejű tárgyfelismerést (pl. ember, autó) végezni a videófolyamokban, csökkentve ezzel a fals riasztásokat a hagyományos mozgásérzékelőkkel szemben. A Scrypted a különböző gyártók (pl. IP Cam 1 , IP Cam 2 ) kameráit képes egységesen integrálni olyan rendszerekbe, mint a Home Assistant.
