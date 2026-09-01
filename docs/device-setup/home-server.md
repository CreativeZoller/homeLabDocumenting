# 🖥️ HomeLab fő szerver: OS, ZFS és virtualizáció

Cél hardver: Mini-ITX Szerver (Intel i5-12400, Noctua NH-L12S, 64 GB DDR4-3200, VGA kártya nélkül, 2× Intel I226-V 2.5G NIC, 2× 12 TB IronWolf ZFS-hez), Ubuntu Server LTS OS, a Proxmox cluster külön, jövőbeli 3 node képében valósul meg. Lásd [Jövőbeli bővítés](../planning/future.md).

Feltételezések:

- Ubuntu Server LTS az NVMe-n.
- Két 12 TB HDD a rendszernek (ZFS mirror).
- NIC nevek: `enp1s0` (2.5G → Firewalla, opcionális) és `enp2s0` (1G → UniFi). A valós neveket `ip -br link` adja.

## Dedikált felhasználó (homelab)

```bash
sudo adduser homelab
sudo usermod -aG sudo homelab
su - homelab
```

## Rendszerfrissítés és alapok

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git htop net-tools openssh-server ufw
sudo timedatectl set-timezone Europe/Vienna
sudo ufw allow 22/tcp
sudo ufw enable
```

A Docker szolgáltatások portját a SWAG kezeli. Az internet felé a Firewalla engedélyez portot, nem az UFW a WAN-on.

## ZFS pool (két IronWolf, mirror)

A 2.0 két 12 TB lemezt rendel. Mirror: egy lemez kieshet adatvesztés nélkül. A későbbi DAS/JBOD átalakítás: [Jövőbeli bővítés](../planning/future.md).

```bash
sudo apt install zfsutils-linux -y
sudo fdisk -l
ls -l /dev/disk/by-id/
```

A `/dev/sdX` helyett mindig a `/dev/disk/by-id/` azonosítót használd.

```bash
sudo zpool create -f tank mirror /dev/disk/by-id/ata-...lemez1 /dev/disk/by-id/ata-...lemez2
sudo zfs set compression=lz4 tank
sudo zfs set atime=off tank
sudo zfs create tank/media
sudo zfs create tank/config
sudo zfs create tank/containers
sudo zfs create tank/vm
sudo zfs create tank/backups
sudo zfs set recordsize=128k tank/vm
sudo zfs set quota=500G tank/vm
sudo chown -R homelab:homelab /tank/
```

Egyetlen lemezes pool csak ideiglenes; a második IronWolf érkezésekor `zpool attach`.

## Docker és Portainer

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt remove $pkg; done

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=\"$(dpkg --print-architecture)\" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo usermod -aG docker homelab
docker volume create portainer_data
docker network create proxy-tier
docker run -d -p 9000:9000 -p 9443:9443 --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    portainer/portainer-ce:latest
```

Logout, majd új SSH a `homelab` userrel, hogy a `docker` csoport éljen.

## Hálózat: dual NIC és bridge a VM-eknek

A 2.5G haszna a Firewalla közvetlen linken van. A switch felé 1 GbE.

Példa Netplan (`/etc/netplan/01-br.yaml`) — a interfészneveket cseréld:

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      dhcp4: no
    enp2s0:
      dhcp4: no
  bridges:
    br0:
      interfaces: [enp2s0]
      dhcp4: no
      addresses: [192.168.10.10/24]
      routes:
        - to: default
          via: 192.168.10.1
      nameservers:
        addresses: [192.168.10.11, 192.168.10.1]
      parameters:
        stp: false
        forward-delay: 0
```

Ha a 2.5G Firewalla-link be van kötve, `enp1s0` kaphat külön címet vagy bond/metric TBD. Első körben elég a `br0` a switches NIC-en, hogy a KVM vendégek a VLAN 10-en jelenjenek meg.

```bash
sudo netplan apply
ip a show br0
```

## KVM/QEMU

```bash
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients virtinst bridge-utils
sudo usermod -aG libvirt homelab
kvm-ok
sudo systemctl enable --now libvirtd
mkdir -p /tank/vm/iso
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

### VM sablonok (virt-install)

ISO-k: `/tank/vm/iso/`. `--graphics none` = konzolos telepítés.

Kali:

```bash
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

Teszt tűzfal VM (tanulás, nem a produkciós Firewalla helyettesítője):

```bash
sudo virt-install \
    --name Test_Firewall \
    --os-variant freebsd12 \
    --ram 2048 \
    --vcpus 1 \
    --disk path=/tank/vm/testfw.qcow2,size=20,bus=virtio \
    --network bridge=br0,model=virtio \
    --network bridge=br0,model=virtio \
    --graphics none \
    --console pty,target_type=serial \
    --location /tank/vm/iso/pfSense-CE-latest.iso \
    --extra-args 'console=tty0 console=ttyS0,115200n8'
```

Tails, Trace Labs OSINT, CSILinux ugyanazzal a mintával (`/tank/vm/…`, 2–4 GB RAM, `bridge=br0`). A produkciós routing a Firewalla, ezek labor VM-ek.
