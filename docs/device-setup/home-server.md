# 🖥️ HomeLab Fő Szerver Tervezés: OS, ZFS és Virtualizáció

Feltételezések:

- Ubuntu Server LTS telepítve az NVMe lemezre.
- A 12 TB-os HDD elérhető a rendszer számára (pl. /dev/sdX).
- A fizikai hálózati interfész neve: eno1.

### Dedikált Felhasználó Létrehozása (homelab)

Ezt a felhasználót fogjuk használni a további beállításokhoz.

```bash
# 1. Hozzuk létre a 'homelab' felhasználót
sudo adduser homelab

# 2. Adjuk hozzá a 'homelab' felhasználót a 'sudo' csoporthoz (admin jogosultságok)
sudo usermod -aG sudo homelab

# 3. Váltás az új felhasználóra
su - homelab

# (Mostantól minden parancsot a 'homelab' felhasználóval és szükség esetén 'sudo'-val futtatunk!)
```

### Rendszerfrissítés és Alapvető Eszközök Telepítése

```bash
# Rendszerfrissítés
sudo apt update && sudo apt upgrade -y

# Alapvető segédprogramok és tűzfal telepítése
sudo apt install -y curl git htop net-tools openssh-server ufw

# UFW (Tűzfal) beállítása: Engedélyezzük az SSH-t és a kritikus portokat
# (A többi Docker szolgáltatás portját a SWAG Reverse Proxy fogja kezelni, és csak a 443-as portot nyitjuk ki az internet felé!)
sudo ufw allow 22/tcp  # SSH hozzáférés engedélyezése
sudo ufw enable
```

### ZFS Pool Létrehozása és Datasetek Szegmentálása (NAS)

```bash
# 1. ZFS csomagok telepítése
sudo apt install zfsutils-linux -y

# 2. Lemezazonosító megkeresése (EZT ELLENŐRIZD!)
# PÉLDA: a 12 TB-os HDD legyen /dev/sdX (a valós nevet írd be helyette)
sudo fdisk -l
ls -l /dev/disk/by-id/ 

# 3. Pool létrehozása a lemezen (tank)
# CSERÉLD KI a /dev/sdX-et a 12 TB-os lemez VALÓS eszközazonosítójára!
sudo zpool create -f tank /dev/sdX

# 4. Alapértelmezett ZFS tulajdonságok beállítása (tömörítés, hozzáférési idők)
sudo zfs set compression=lz4 tank
sudo zfs set atime=off tank

# 5. Datasetek létrehozása a struktúrának megfelelően
sudo zfs create tank/media
sudo zfs create tank/config
sudo zfs create tank/containers
sudo zfs create tank/vm
sudo zfs create tank/backups

# 6. VM dataset optimalizálása és korlátozása
sudo zfs set recordsize=128k tank/vm
sudo zfs set quota=500G tank/vm # Példa: 500 GB kvóta

# 7. Jogosultságok beállítása a 'homelab' felhasználónak
sudo chown -R homelab:homelab /tank/
```

### ZFS Pool Létrehozása és Datasetek Szegmentálása (NAS)

```bash
# 1. ZFS csomagok telepítése
sudo apt install zfsutils-linux -y

# 2. Lemezazonosító megkeresése (EZT ELLENŐRIZD!)
# PÉLDA: a 12 TB-os HDD legyen /dev/sdX (a valós nevet írd be helyette)
sudo fdisk -l
ls -l /dev/disk/by-id/ 

# 3. Pool létrehozása a lemezen (tank)
# CSERÉLD KI a /dev/sdX-et a 12 TB-os lemez VALÓS eszközazonosítójára!
sudo zpool create -f tank /dev/sdX

# 4. Alapértelmezett ZFS tulajdonságok beállítása (tömörítés, hozzáférési idők)
sudo zfs set compression=lz4 tank
sudo zfs set atime=off tank

# 5. Datasetek létrehozása a struktúrának megfelelően
sudo zfs create tank/media
sudo zfs create tank/config
sudo zfs create tank/containers
sudo zfs create tank/vm
sudo zfs create tank/backups

# 6. VM dataset optimalizálása és korlátozása
sudo zfs set recordsize=128k tank/vm
sudo zfs set quota=500G tank/vm # Példa: 500 GB kvóta

# 7. Jogosultságok beállítása a 'homelab' felhasználónak
sudo chown -R homelab:homelab /tank/
```

### Docker és Portainer Telepítése

```bash
# 1. Docker telepítése (a hivatalos Docker repository-ból)
# (A curl, gnupg, ca-certificates már telepítve van)
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt remove $pkg; done

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=\"$(dpkg --print-architecture)\" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# 2. Hozzáadás a docker csoporthoz (hogy ne kelljen sudo-t használni)
sudo usermod -aG docker homelab

# 3. Portainer (A Docker GUI menedzsment) telepítése
# Hozzunk létre egy kötetet a Portainer adatainak
docker volume create portainer_data

# Futtassuk a Portainer-t
docker run -d -p 9000:9000 -p 9443:9443 --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    portainer/portainer-ce:latest
```

Mielőtt a következő lépésre lépnél, KI KELL LÉPNED ÉS VISSZA KELL LÉPNED az SSH/konzol munkamenetbe, hogy az új docker és libvirt csoport tagságok érvénybe lépjenek!

```bash
logout
# (újra SSH-zz be 'homelab' felhasználóval)
```

### KVM/QEMU Virtualizáció Telepítése és Hálózat Beállítása

```bash
# 1. KVM/QEMU és kiegészítők telepítése
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients virtinst bridge-utils

# 2. Felhasználó hozzáadása a libvirt csoporthoz (!!! EZT CSAK A BIZTONSÁG KEDVÉÉRT ISMÉTLJÜK !!!)
sudo usermod -aG libvirt homelab

# 3. KVM és libvirt szolgáltatások ellenőrzése
kvm-ok
sudo systemctl enable --now libvirtd
```

Bridge Hálózat Beállítása (br0) a VM-eknek

```bash
# 1. Netplan konfigurációs fájl megnyitása
sudo nano /etc/netplan/01-br.yaml
```

Tartalom (A hálózati kártyád neve (eno1) cserélendő!):

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eno1:
      dhcp4: no       # DHCP kikapcsolása a fizikai interfészen
  bridges:
    br0:
      interfaces: [eno1] # A fizikai interfész hozzáadása a bridge-hez
      dhcp4: yes       # DHCP kérés a bridge interfészen
      parameters:
        stp: true
        forward-delay: 0
```

```bash
# 2. Konfiguráció alkalmazása (a kapcsolat rövid időre megszakadhat!)
sudo netplan apply

# 3. Ellenőrzés
ip a show br0
```

### Speciális VM-ek Előkészítése (virt-install)

Készítsd elő az ISO fájlok tárolására szolgáló mappát, és másold be ide a letöltött ISO fájlokat (Kali, pfSense, Tails, Trace Labs, CSILinux).

```bash
# Mappa az ISO-knak
mkdir -p /tank/vm/iso

# 1. Libvirt hálózati bridge definiálása
# Ez csak megerősíti a Netplan beállítást.
sudo virsh net-define /dev/stdin <<EOF
<network>
  <name>br0_network</name>
  <forward mode='bridge'/>
  <bridge name='br0'/>
</network>
EOF
sudo virsh net-start br0_network
sudo virsh net-autostart br0_network
```

#### VM Telepítési Sablonok (virt-install)

Ezek a parancsok elindítják a VM telepítését. A --graphics none beállítás miatt a telepítést a konzolon keresztül tudod követni.

##### Kali Linux (IT Biztonság)

```bash
# --disk: egy 50 GB-os fájl jön létre a ZFS /tank/vm/ dataseten
sudo virt-install \
    --name Kali_Linux \
    --os-variant debian12 \
    --ram 4096 \
    --vcpus 2 \
    --disk path=/tank/vm/kali.qcow2,size=50,bus=virtio \
    --network bridge=br0 \
    --graphics none \
    --console pty,target_type=serial \
    --location /tank/vm/iso/kali-latest.iso \
    --extra-args 'console=tty0 console=ttyS0,115200n8'
```

##### pfSense (Teszt Tűzfal/Router)

```bash
# Két hálózati interfész beállítása a WAN/LAN szimulációhoz
sudo virt-install \
    --name pfSense_Router \
    --os-variant freebsd12 \
    --ram 2048 \
    --vcpus 1 \
    --disk path=/tank/vm/pfsense.qcow2,size=20,bus=virtio \
    --network bridge=br0,model=virtio \
    --network bridge=br0,model=virtio \
    --graphics none \
    --console pty,target_type=serial \
    --location /tank/vm/iso/pfSense-CE-latest.iso \
    --extra-args 'console=tty0 console=ttyS0,115200n8'
```

##### Tails VM (Anonimitás / Adatvédelem)

A Tails általában Live módban fut. Itt a telepítési folyamatot indítjuk, de meg kell győződni arról, hogy az adott Tails verzió támogatja-e a tartós telepítést.

```bash
sudo virt-install \
    --name Tails_Anon \
    --os-variant debian12 \
    --ram 2048 \
    --vcpus 2 \
    --disk path=/tank/vm/tails.qcow2,size=30,bus=virtio \
    --network bridge=br0 \
    --graphics none \
    --console pty,target_type=serial \
    --location /tank/vm/iso/tails-latest.iso \
    --extra-args 'console=tty0 console=ttyS0,115200n8'
```

##### Trace Labs OSINT VM (Nyílt Forrású Hírszerzés)

```bash
sudo virt-install \
    --name TraceLabs_OSINT \
    --os-variant debian12 \
    --ram 4096 \
    --vcpus 2 \
    --disk path=/tank/vm/tracelabs.qcow2,size=60,bus=virtio \
    --network bridge=br0 \
    --graphics none \
    --console pty,target_type=serial \
    --location /tank/vm/iso/tracelabs-osint-latest.iso \
    --extra-args 'console=tty0 console=ttyS0,115200n8'
```

##### CSILinux VM (Kibernetikai Nyomozás)

```bash
sudo virt-install \
    --name CSILinux \
    --os-variant debian12 \
    --ram 4096 \
    --vcpus 2 \
    --disk path=/tank/vm/csilinux.qcow2,size=60,bus=virtio \
    --network bridge=br0 \
    --graphics none \
    --console pty,target_type=serial \
    --location /tank/vm/iso/csilinux-latest.iso \
    --extra-args 'console=tty0 console=ttyS0,115200n8'
```

Ezzel a listával a fő szerver teljesen be van állítva a ZFS tárolásra, a Docker konténerek futtatására és a speciális biztonsági/vizsgálati virtuális gépek kezelésére.
