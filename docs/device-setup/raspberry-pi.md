# Raspberry Pi 4 Alapvető HomeLab Beállítás

*Előfeltételek:*

- A Raspberry Pi 4 rendelkezzen telepített operációs rendszerrel (pl. Raspberry Pi OS vagy Ubuntu Server for Pi).
- A felhasználó rendelkezzen sudo jogosultsággal.
- A Pi a hálózatban van (Switch 3-as port) és statikus IP-címmel rendelkezik.
- Docker/Docker Compose telepítve van a Pi-n, mivel a Pi-hole és a többi szolgáltatás is konténerben fut.

## Rendszerfrissítés és Alapcsomagok Telepítése

- Frissítjük a csomaglistákat és a telepített csomagokat

```bash
sudo apt update && sudo apt upgrade -y
```

- Telepítjük a szükséges segédeszközöket

```bash
sudo apt install -y curl git htop net-tools
```

## Tailscale Telepítése a Pi-n (Biztonságos Távoli Elérés)

A Tailscale biztosítja a biztonságos távoli elérést a MagicDNS-szel együtt, ami megkönnyíti a szolgáltatások elérését a belső hálózaton kívülről.

- Telepítjük a Tailscale szkripttel

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

- Bejelentkezés a Tailscale hálózatba. - Ez kiír egy linket, amit be kell illesztened a böngésződbe a bejelentkezéshez.

```bash
sudo tailscale up --authkey tskey-yourkey # Használj Auth Key-t, ha lehetséges
```

_Ha nincs authkey-d, futtasd a sudo tailscale up-ot, és kövesd a konzolon megjelenő bejelentkezési linket._

## Pi-hole Telepítése Docker Konténerben

A Pi-hole a hálózati szintű DNS hirdetésblokkolásért felelős. Mivel a Pi a hálózati DNS-szolgáltató lesz, itt futtatjuk.

- Létrehozzuk a Pi-hole mappa struktúrát és bemegyünk a mappába

```bash
mkdir -p ~/pihole && cd ~/pihole
```

- Létrehozzuk a docker-compose.yml fájlt (ajánlott egy szerkesztővel, pl. nano)

```bash
nano docker-compose.yml
```

Példa tartalom YAML:

```yaml
version: "3"
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    # Frissítsd az IP-címet a Pi statikus IP-címére!
    # A DNS port (53) a legfontosabb.
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80/tcp" # Webes felület
    environment:
      TZ: 'Europe/Budapest' # Állítsd be a megfelelő időzónát
      WEBPASSWORD: 'YourStrongAdminPassword' # Állítsd be a saját jelszavad
      # Opcionális: a Pi-hole upstream DNS szervereinek beállítása
      # Pl. Cloudflare: DNS1=1.1.1.1; DNS2=1.0.0.1
      # Ha Unboundot használsz, ez a beállítás más lesz (lásd alább!)
    # DHCP hálózaton nem feltétlenül szükséges, de jó, ha van
    cap_add:
      - NET_ADMIN
    restart: unless-stopped
    volumes:
      - './etc-pihole/:/etc/pihole/'
      - './etc-dnsmasq.d/:/etc/dnsmasq.d/'
```

- A Pi-hole indítása

```bash
sudo docker compose up -d
```

## Unbound Telepítése

Ahogy korábban említettük, az Unbound hozzáadásával a Pi-hole nem a szolgáltató DNS-ére támaszkodik, hanem közvetlenül a gyökérszerverekhez fordul, növelve az adatvédelmet. Ha az Unboundot is a Pi-n konténerben futtatod, a Pi-hole beállítást módosítanod kell, hogy az Unbound felé mutasson.

- Készítsünk egy külön mappát az Unboundnak

```bash
mkdir -p ~/unbound && cd ~/unbound
```

- Itt is létrehozunk egy docker-compose.yml fájlt

```bash
nano docker-compose.yml
```

Példa tartalom YAML:

```yaml
version: "3.5"
services:
  unbound:
    container_name: unbound
    image: mvance/unbound:latest
    ports:
      - "5335:5335/udp" # Ezen a porton fogja elérni a Pi-hole
    volumes:
      - ./config:/opt/unbound/etc/unbound/
    restart: unless-stopped
```

- Az Unbound indítása

```bash
sudo docker compose up -d
```

## Pi-hole konfiguráció módosítása az Unbound használathoz

Módosítsd a Pi-hole webes felületén az Upstream DNS szervert a Pi-hole konténerből elérhető Unbound IP-jére és portjára (pl. 127.0.0.1#5335 vagy a Pi belső IP-címe 5335-ös porton, ha külön hálózaton futnak).

## További Pi-szolgáltatások Telepítése

### Pi.alert telepítése (hálózati eszközök monitorozása)

A Pi.alert a Git-klónozás után egy telepítő szkripttel futtatható

- Menjünk a felhasználói könyvtárunkba

```bash
cd ~
```

- Klónozzuk a Pi.Alert repository-t

```bash
git clone https://github.com/jokob-sk/Pi.Alert.git
cd Pi.Alert
```

- Indítjuk a telepítést (ez végigvezet a konfiguráción)

```bash
sudo ./install.sh
```

Miután a Pi-hole fut, győződj meg róla, hogy a MikroTik routeren a DHCP-beállításokban a DNS-szerver a Raspberry Pi statikus IP-címére mutat. Ezzel válik a Pi-hole a teljes hálózat hirdetésblokkolójává.
