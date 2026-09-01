# 🏡 HomeLab fogalma (ebben a projektben)

A HomeLab itt egy konkrét otthoni IT-infrastruktúra: tűzfal, switch, Wi-Fi, szerver, NAS és munkaállomás egy 10" rack körül, nem általános tankönyvi áttekintés.

Röviden: privát lab, ahol a hálózat, a virtualizáció és a self-hosted szolgáltatások éleshez közeli, de otthoni kockázattal futnak.

## ⚙️ Ami ebből a labből áll

- **Számítás:** Mini-ITX home server (cél: i5-12400, 64 GB), Ubuntu Server, Docker, KVM. Raspberry Pi 4 a DNS-hez.
- **Hálózat:** Firewalla Gold Plus, UniFi Lite 16 PoE, UniFi U7 Pro. VLAN 10 Fő, 20 Vendég, 30 IoT, 40 Kliens.
- **Tárolás:** 2× 12 TB IronWolf ZFS mirror a szerverben; később DAS/JBOD opció.
- **Ház:** 10" rack UPS-sel és PDU-val a szerver mellett, nem külön szobás 19" szekrény.

A 2.6-os döntés oka: költség, hely, zaj, és hogy a későbbi szervercsere se kényszerítsen nagy rackre.

Általános előnyök és korlátok: [Előnyök és kihívások](benefits-challenges.md).
