# Legrand Keor UPS

A 2.6 a Legrand Keor 800 VA USB-s multiplug UPS-t választotta a 10" / asztal melletti formátumhoz. A korábbi APC Back-UPS 850VA (BE850G2-GR) kikerül.

## Szerep

Áramszünetnél a Firewalla, a switch és a home server maradjon fent annyi ideig, hogy a szerver szabályosan leálljon. 800 VA szűkös három eszközre — terhelést mérni kell, és ha kell, a szervert NUT-tal leállítani előbb.

## Bekötés

1. UPS a falra.
2. PDU a UPS kimenetére (vagy a kritikus eszközök közvetlenül a Keor aljzataiba, ha a 4 aljzat elég).
3. Firewalla, UniFi Lite 16, home server a PDU-n / UPS-en.
4. Nem UPS-re: monitor, nyomtató, töltők, LED — ezek a második PDU-ra vagy sima aljzatra.

USB: a Keor multiplug USB töltőt is ad. A szerver graceful shutdown (NUT / `usbhid-ups`) **TBD** — ellenőrizni, hogy a modell ad-e HID UPS jelet, vagy csak töltő USB-t.

## Ellenőrzés

- Teszt: UPS-t húzd ki a falból, a lab maradjon fent.
- Runtime mérés valós terhelésen; 800 VA-nál ne számíts hosszú futásra.
- Home server: ha van hid UPS, `nut-monitor` leállítás. Ha nincs, a Firewalla/switch túlélése a rövidebb, a szerver filesystemje ZFS — akkor is jobb a tiszta shutdown.

Táp a rackben: [10" rack](rack.md).
