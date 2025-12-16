# Topológia

Jelen esetben egy "Router-on-a-Stick" alapú, szegmentált hálózati modellt építettünk fel, ahol a forgalom irányítását a MikroTik végzi, a végrehajtást pedig a Switch és az Ubuntu szerver.

## 🌐 A Logikai Felépítés Pillérei

*1. A MikroTik mint "Agy" (L3 réteg)*

A MikroTik hAP ax³ nemcsak egy Wi-Fi pont, hanem a tűzfalad és a hálózati rendőröd is.

- *VLAN Trunking:* A router és a switch között egyetlen kábelen (SFP+) fut az összes VLAN forgalma. Ez a "Trunk". Itt a MikroTik válogatja szét, hogy mi tartozik az Admin, az IoT vagy a Vendég hálózatba.
- *Inter-VLAN Routing:* Alapesetben a VLAN-ok nem látják egymást. A MikroTik tűzfal szabályai (Firewall Rules) határozzák meg, hogy például a Home Assistant (VLAN 10) beláthasson az IP kamerákhoz (VLAN 20), de a kamerák ne érhessék el a szerver konfigurációs fájljait.

*2. A Switch mint "Gerinc" (L2 réteg)*

A CSS610 switch feladata a nagy sebességű adatátvitel.

- *VLAN Tagging:* A switch portjai tudják, hogy melyik eszköz hova tartozik. Ha bedugsz egy új kamerát a 4-es portba, a switch automatikusan ráaggatja a "VLAN 20" címkét, így az eszköz azonnal elszigetelődik a fő hálózattól.
- *Helyi forgalom:* Ha a PC-dről (VLAN 10) másolsz a szerverre (VLAN 10), az nem terheli a routert, a switch elintézi "házon belül" 10Gbps sebességgel.

*3. Az Ubuntu Szerver: A "Hibrid" Végpont*

Ez a legérdekesebb pontja a topológiádnak. A szervered egyszerre viselkedik Hostként (Docker) és Switchként (KVM Bridge).

- *Docker Bridge:* A konténerek egy belső, virtuális hálózaton (proxy-tier) beszélgetnek. Csak a SWAG lát ki a fizikai hálózatra, ő a "portás".
- *KVM Bridge (br0):* A virtuális gépeid (Kali, pfSense) nem a szerver IP-je mögött rejtőznek. A hálózati híd (bridge) segítségével úgy jelennek meg a MikroTik felé, mintha fizikailag is be lennének dugva a switchbe. Saját MAC címet és IP-t kapnak.

*🛡️ Biztonsági Zónák (A Topológia lelke)*

- *Trusted Zone (VLAN 10):* Itt van a "tudás". Itt lakik a ZFS pool, a GitLab és a Docker menedzsment. Ide csak te lépsz be.
- *Isolated Zone (VLAN 20):* Ide kerül minden "okos" eszköz (lámpák, kamerák, kínai szenzorok). Ezek hírhedten gyenge biztonságúak, ezért ebben a topológiában ki van vágva az internet elérésük. Csak a Home Assistant és a Frigate éri el őket lokálisan.
- *Public Face (SWAG):* Ez az egyetlen pont, ahol a hálózatod "érintkezik" az internettel a 443-as porton. A topológia lényege, hogy ha egy támadó feltöri a webszerveredet, akkor is csak egy elszigetelt Docker hálózatban találja magát, nem fér hozzá a routerhez vagy a ZFS fájlokhoz.

*🚀 Miért jó ez a végső felállás?*

- *Skálázhatóság:* Ha veszel még egy szervert, csak bedugod a switchbe, megadod neki a VLAN 10-et, és máris a rendszer része.
- *Hibakeresés:* Ha a Pi-hole leáll, csak a DNS esik ki, a belső forgalom (IP alapon) és a virtuális gépek zavartalanul futnak tovább.
- *Adatbiztonság:* A 12 TB-os ZFS pool fizikailag egy gépben van, de a hálózaton keresztül bárhonnan (jogosultsággal) elérhető, mintha egy profi Enterprise tárolód lenne.

Ez a topológia professzionális szintű; nem egy átlagos otthoni Wi-Fi-ről beszélünk, hanem egy mini-adatközpontról.
