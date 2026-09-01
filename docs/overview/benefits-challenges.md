# 🚀 Előnyök és kihívások

A lab tipikus használata: tanulás, média és backup, Home Assistant, saját tűzfal, self-hosted felhő.

## ✅ Előnyök

- Gyakorlati hálózat, ZFS, Docker, VLAN.
- Adatok otthon maradnak (Nextcloud, Immich, Vaultwarden).
- A stack a saját igényre szabható.
- Hosszú távon olcsóbb lehet, mint több felhős előfizetés — a 2.0/2.6 BOM és a [pályázatok](../planning/funding.md) ezt számolhatóvá teszik.

## ❌ Hátrányok

### Hardver és energia

- Kezdeti költség: Firewalla, UniFi, szerver upgrade, lemezek, rack, asztal — lásd [BOM](../planning/phases-bom.md).
- 24/7 fogyasztás. A 10" / Mini-ITX / i5-12400 választás pont ezt és a zajt célozza, nem egy full-size 19" szervert.
- Hő és rezgés: Noctua hűtő és NA-SAV rezgéscsillapítás a rackben.

### Üzemeltetés

- Te vagy a rendszergazda: frissítés, backup, Firewalla szabályok.
- Internet felé nyitás kockázat; Tailscale a preferált távoli út.
- Redundancia: ZFS mirror a két IronWolfon, UPS a Legrand Keorral, offsite backup továbbra is kell.
