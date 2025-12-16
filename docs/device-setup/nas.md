# 🖥️ NAS - Appok telepítése és beállítása

Ahhoz, hogy a NAS-t kényelmesen használd is, van két apró, de kritikus lépés, amit érdemes megtenni, mielőtt fejest ugrasz a Portainerbe:

- *Samba (SMB) megosztás:* Hogy a Windowsos gépedről vagy laptopodról egyszerűen rá tudj másolni fájlokat a 12 TB-os lemezre.
- *Docker jogosultságok:* Biztosítani kell, hogy a Docker konténereknek legyen joguk írni/olvasni a ZFS mappákba.

## NAS Elérése (Samba megosztás)

Ha szeretnéd látni a hálózaton a media mappát, futtasd le ezeket:

```bash
# Samba telepítése
sudo apt install samba -y

# Mappa megosztása (szerkesztés)
sudo nano /etc/samba/smb.conf
```

Görgess a fájl végére, és add hozzá ezt:

```
[Media]
   path = /tank/media
   read only = no
   browsable = yes
   guest ok = no
```

Állíts be egy jelszót a hálózati eléréshez a homelab usernek:

```bash
sudo smbpasswd -a homelab
sudo systemctl restart smbd
```

## Felkészülés a Portainer-re (Jogosultságok)

A Docker konténerek gyakran egy speciális felhasználó (pl. PUID 1000) nevében futnak. Ahhoz, hogy a Jellyfin vagy a Nextcloud ne kapjon "Permission Denied" hibát:

```bash
# Biztosítjuk, hogy a homelab useré legyen minden a tank-en
sudo chown -R homelab:homelab /tank
sudo chmod -R 775 /tank
```

## Portainer és a "Stack-ek"

Ahhoz, hogy a rendszer ne omoljon össze a függőségek miatt, a telepítést logikai rétegekre kell osztani. Nem érdemes mindent egyetlen óriási fájlba tenni; a Portainer "Stacks" funkcióját használva csoportonként fogjuk őket telepíteni.

### 🏗️ Telepítési Sorrend és Csoportosítás

*1. Réteg: Alapvető Infrastruktúra és Biztonság*

Ezek nélkül a többi szolgáltatás nem érhető el kívülről, vagy nem tudsz belépni hozzájuk.

- *SWAG:* A kapu. Ez kezeli a HTTPS-t és irányítja a forgalmat a konténerekhez.
- *Authentik:* A központi beléptető (SSO). Ezt kötjük rá a SWAG-re, hogy egy jelszóval mindenbe beléphess.
- *Heimdall / Dashboard / Homepage:* Az "indítópult", ahol minden ikon ott lesz.

*2. Réteg: Adatkezelés és Iroda*

Ezek az alapvető tárolási és munkavégzési eszközeid.

- *NextCloud:* A fájljaid otthona.
- *Paperless-ngx:* A szkennelt dokumentumok digitális archívuma.
- *Odoo:* Üzleti folyamatok és adminisztráció.
- *BentoPDF:* PDF kezelő segédlet.

*3. Réteg: Média és Szórakozás (A "Servarr" Stack)*

Ezek szorosan együttműködnek, egy Stack-ben érdemes kezelni őket.

- *Letöltők:* qBittorrent, Prowlarr, Jackett.
- *Kezelők:* Sonarr (sorozatok), Radarr (filmek), Readarr/OpenBook (könyvek), Calibre (e-book).
- *Szerverek:* Jellyfin (vagy Plex), Audiobookshelf (hangoskönyv), Romm (játék ROM-ok).

*4. Réteg: Fotók és Személyes Emlékek*

- *Immich:* A Google Fotók alternatívája (erőforrásigényes, ezért külön érdemes kezelni).

*5. Réteg: Okosotthon és Biztonság*

- *Home Assistant:* A központ.
- *Frigate / Scrypted:* Kamerafigyelés és mesterséges intelligencia alapú felismerés.
- *Wallos:* Előfizetések kezelése.
- *Mealie:* Receptkönyv és étkezés tervező.

*6. Réteg: Fejlesztés, DevOps és Monitorozás*

- *GitLab + Focalboard:* Kód és projektmenedzsment.
- *N8N:* Automatizáció (pl. ha jön egy levél, mentse a Paperlessbe).
- *LibreNMS + Plausible + Uptime Kuma:* Hálózat és web analitika.
- *Ansible + Rancher:* Ha több szervered lesz, ezekkel vezérled őket.
- *GestióIP:* IP címek rendszerezése.

#### 🏗️ 1. Réteg: Infrastruktúra és Hálózat (Stack: proxy-tier)

Ezt a stack-et hozzuk létre először. Ez kezeli a HTTPS-t (SWAG) és a kezdőoldaladat (Homepage vagy egyéb alternatíva).

*Portainer -> Stacks -> Add stack ->* Név: infrastructure

```yaml
version: "3.8"
services:
  swag:
    image: linuxserver/swag
    container_name: swag
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Budapest
      - URL=sajatdomened.hu # Cseréld ki!
      - SUBDOMAINS=wildcard
      - VALIDATION=dns
      - DNSPLUGIN=cloudflare
    volumes:
      - /tank/config/swag:/config
    ports:
      - 443:443
      - 80:80
    restart: unless-stopped
    networks:
      - proxy-tier

  authentik-server:
    image: ghcr.io/goauthentik/server
    container_name: authentik
    command: server
    environment:
      - AUTHENTIK_REDIS__HOST=redis
      - AUTHENTIK_POSTGRESQL__HOST=postgresql
    volumes:
      - /tank/config/authentik/media:/media
      - /tank/config/authentik/templates:/templates
    networks:
      - proxy-tier
    restart: unless-stopped

  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    volumes:
      - /tank/config/homepage:/config
      - /var/run/docker.sock:/var/run/docker.sock:ro
    ports:
      - 3000:3000
    networks:
      - proxy-tier
    restart: unless-stopped

networks:
  proxy-tier:
    external: true
```

##### 🛠️ SWAG Beállítások (Reverse Proxy konfigurálása):

Miután a SWAG elindult, SSH-n keresztül engedélyezd az aldomaineket:

- Menj ide: `cd /tank/config/swag/nginx/proxy-confs/`
- Másold le a mintákat (minden apphoz, amit ki akarsz tenni): `cp nextcloud.subdomain.conf.sample nextcloud.subdomain.conf`
- Indítsd újra a SWAG-et Portainerben.

#### 🎬 2. Réteg: Média és Letöltés (Stack: media-center)

Ez a "Servarr" stack. Itt mindent a /tank/media alá irányítunk.

*Portainer -> Stacks -> Add stack ->* Név: media

```yaml
version: "3"
services:
  jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfin
    devices:
      - /dev/dri:/dev/dri # i5-12600 GPU gyorsítás
    volumes:
      - /tank/config/jellyfin:/config
      - /tank/media:/media
    networks:
      - proxy-tier

  qbittorrent:
    image: linuxserver/qbittorrent
    container_name: qbittorrent
    volumes:
      - /tank/config/qbittorrent:/config
      - /tank/media/downloads:/downloads
    networks:
      - proxy-tier

  prowlarr:
    image: linuxserver/prowlarr
    volumes:
      - /tank/config/prowlarr:/config
    networks:
      - proxy-tier

  sonarr:
    image: linuxserver/sonarr
    volumes:
      - /tank/config/sonarr:/config
      - /tank/media/tv:/tv
    networks:
      - proxy-tier

  radarr:
    image: linuxserver/radarr
    volumes:
      - /tank/config/radarr:/config
      - /tank/media/movies:/movies
    networks:
      - proxy-tier

  audiobookshelf:
    image: advplyr/audiobookshelf:latest
    volumes:
      - /tank/config/audiobookshelf:/config
      - /tank/media/audiobooks:/audiobooks
    networks:
      - proxy-tier

  calibre:
    image: linuxserver/calibre
    volumes:
      - /tank/config/calibre:/config
    networks:
      - proxy-tier

  romm:
    image: romm/romm:latest
    volumes:
      - /tank/config/romm:/config
      - /tank/media/roms:/roms
    networks:
      - proxy-tier
```

#### ☁️ 3. Réteg: Személyes Felhő és Dokumentumok (Stack: cloud-office)

NextCloud és a Paperless-ngx a dokumentumaid digitális kezeléséhez.

*Portainer -> Stacks -> Add stack ->* Név: media

```yaml
version: "3"
services:
  nextcloud:
    image: linuxserver/nextcloud
    container_name: nextcloud
    volumes:
      - /tank/config/nextcloud:/config
      - /tank/media/nextcloud_data:/data
    networks:
      - proxy-tier
    restart: unless-stopped

  paperless-ngx:
    image: ghcr.io/paperless-ngx/paperless-ngx:latest
    container_name: paperless
    volumes:
      - /tank/config/paperless/data:/data
      - /tank/media/documents/consume:/consume
    networks:
      - proxy-tier

  odoo:
    image: odoo:latest
    container_name: odoo
    volumes:
      - /tank/config/odoo:/var/lib/odoo
    networks:
      - proxy-tier

  bentopdf:
    image: bentopdf/bentopdf:latest
    container_name: bentopdf
    networks:
      - proxy-tier
```

#### 📸 4. Réteg: Speciális és Fotó (Stack: immich)

Az Immich az egyik legjobb Google Fotók alternatíva, de sok mikroszolgáltatásból áll, ezért érdemes neki saját Stacket adni.

*Portainer -> Stacks -> Add stack ->* Név: immich

- Az Immich hivatalos docker-compose.yml fájlját töltsd be a Portainerbe.
- A `UPLOAD_LOCATION` változót állítsd ide: `/tank/media/photos`.

```yaml
version: "3"
services:
  immich-server:
    image: ghcr.io/immich-app/immich-server:latest
    volumes:
      - /tank/media/photos:/usr/src/app/upload
    networks:
      - proxy-tier
```

> (Az Immich-nek szüksége van Redis-re és Postgres-re is, ezeket érdemes az Immich hivatalos mintája alapján ugyanide tenni.)

#### 🏠 5. Réteg: Okosotthon (Stack: smarthome)

*Portainer -> Stacks -> Add stack ->* Név: smarthome

```yaml
version: "3"
services:
  homeassistant:
    image: ghcr.io/home-assistant/home-assistant:stable
    network_mode: host # Javasolt az okosotthonhoz
    privileged: true
    volumes:
      - /tank/config/homeassistant:/config

  frigate:
    image: ghcr.io/blakeblackshear/frigate:stable
    shm_size: "128mb"
    devices:
      - /dev/dri:/dev/dri
    volumes:
      - /tank/config/frigate:/config
      - /tank/media/recordings:/media/frigate
    networks:
      - proxy-tier

  scrypted:
    image: koush/scrypted
    container_name: scrypted
    networks:
      - proxy-tier

  mealie:
    image: ghcr.io/mealie-recipes/mealie:latest
    volumes:
      - /tank/config/mealie:/app/data
    networks:
      - proxy-tier

  wallos:
    image: bellamy/wallos:latest
    volumes:
      - /tank/config/wallos:/var/www/html/db
    networks:
      - proxy-tier
```

#### 🛠️ 6. Réteg: Fejlesztés, DevOps és Monitorozás

Ez felel a rendszer állapotáért és az automatizációért.

*Portainer -> Stacks -> Add stack ->* Név: devops

```yaml
version: "3"
services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    container_name: gitlab
    volumes:
      - /tank/config/gitlab/config:/etc/gitlab
      - /tank/config/gitlab/logs:/var/log/gitlab
      - /tank/config/gitlab/data:/var/opt/gitlab
    networks:
      - proxy-tier

  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    volumes:
      - /tank/config/n8n:/home/node/.n8n
    networks:
      - proxy-tier

  librenms:
    image: librenms/librenms:latest
    container_name: librenms
    volumes:
      - /tank/config/librenms:/data
    networks:
      - proxy-tier

  plausible:
    image: plausible/analytics:latest
    container_name: plausible
    networks:
      - proxy-tier

  gestioip:
    image: gestioip/gestioip:latest
    container_name: gestioip
    networks:
      - proxy-tier

  focalboard:
    image: mattermost/focalboard:latest
    container_name: focalboard
    networks:
      - proxy-tier

  ansible-semaphore:
    image: semaphoreui/semaphore:latest
    container_name: semaphore
    networks:
      - proxy-tier
```
