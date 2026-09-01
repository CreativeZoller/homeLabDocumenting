# UniFi U7 Pro — access point

A U7 Pro a 2.6-os Wi-Fi. A hAP ax³ AP-módú szerepét veszi át. PoE-ról az UniFi Lite 16-ról él; tipikusan a racken kívül, a szobában.

Adatlap: [UniFi U7 Pro](https://techspecs.ui.com/unifi/wifi/u7-pro?subcategory=wifi-flagship). Menedzsment: ugyanaz az UniFi Network, ami a switché.

## Bekötés

- Cat7 a Lite 16 PoE portjára (javasolt port 4).
- Port profil: native VLAN 10, tagged 20 és 40 (és 30, ha van IoT SSID).
- PoE költségvetés: a Lite 16 össz PoE-ja szűkös, ha sok kamera is PoE-n van. Az AP maradjon a switchen; kamerák mehetnek injektorra.

Statikus IP: `192.168.10.3`, gateway `192.168.10.1`.

## SSID-k

| SSID (példa) | VLAN | Sáv | Megjegyzés |
|--------------|------|-----|------------|
| Lab-Fo | 10 | 5/6 GHz | Admin, munkaállomás, ha kell vezeték nélküli |
| Lab-Vendeg | 20 | 2.4/5 | Vendég, kliens elszigetelés, nincs LAN |
| Lab-IoT | 30 | 2.4 | Csak ami nem mehet Etherneten |
| Lab-Kliens | 40 | 5/6 | Telefon, laptop a napi használathoz |

WPA2/WPA3, külön jelszó VLAN-onként. A vendég hálózat a Firewalla-n nem lát RFC1918-at.

## Adoptálás

1. Switch port PoE + AP profil.
2. UniFi Network: Devices → adopt U7 Pro.
3. Firmware frissítés, majd a fenti WLAN-ok hozzárendelése.
4. Ellenőrzés: vendég DHCP `192.168.20.x`, fő háló `192.168.10.x` nem pingelhető a vendégről.

A switch oldali profilok: [UniFi Lite 16 PoE](switch.md).
