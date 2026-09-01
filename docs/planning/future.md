# Jövőbeli bővítés

A 2.6 záróbekezdése és a 2.0 bevezető „fő asztali PC frissítése”. Nem a mostani célállapot.

## 3 node Proxmox cluster

Három mini-PC helyi Proxmox clusternek. A home server Ubuntu + Docker + ZFS marad, amíg a cluster nincs kész. A 2.0 „Proxmox/Unraid” említése ide tartozik, nem az i5-12400 host azonnali cseréjéhez.

A 10" rack 8U-ról 12–14U-ra nőhet (kereskedelmi tálca vagy a 3D alternatíva).

## NAS → DAS / JBOD

A 2.6 szerint a home serveres NAS átalakítható 4–5 lemezes DAS/JBOD házzá, és hálózatra köthető, ahelyett hogy külön mini-PC lenne tárolónak.

Példa a PDF-ből: Yottamaster 5 bay, USB-C 10 Gbps, daisy chain, max. 5×24 TB (FS5C3). USB általában, de a lab szempontjából a lényeg: a lemezek kikerülnek a Mini-ITX-ből, TPU talp és egyedi előlap a rackbe.

Addig: 2× 12 TB IronWolf ZFS mirror a szerverben. [Fő szerver](../device-setup/home-server.md).

## Asztali PC

A 2.0 fázislistában szerepel, részletes BOM nélkül. TBD, ha a munkaállomás CPU/GPU cseréje aktuális. A KVM már számol több hosttal.

## Pályázat

A cluster / DAS a Wachstums!Schritt felé mutat: [Pályázatok](funding.md).
