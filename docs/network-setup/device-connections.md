# Eszközök megfelelő kötése

A hálózati terv fizikai megvalósítása, azaz a hardvereszközök helyes összekötése kulcsfontosságú a 10 Gbps sebesség és a stabil működés eléréséhez.

## 🔌 Fizikai Bekötési Terv (L1 - Layer 1)

### 1. A MikroTik "Gerinc" (10 Gbps Backbone)

A legfontosabb kapcsolat a MikroTik hAP ax³ router és a MikroTik CSS610 switch között van.

- A kötés: Használd a router SFP+ portját és a switch egyik SFP+ portját.
- Kábel: Ide egy DAC (Direct Attach Copper) kábel a legjobb választás. Ez egy készre szerelt rézkábel, ami natívan tudja a 10 Gbps-ot, minimális késleltetéssel és hőtermeléssel.

### 2. A Fő Szerver Bekötése (i5-12600)

Mivel a szervered a hálózat kiszolgálója (ZFS, Docker, VM-ek), itt van a legnagyobb forgalom.

- A kötés: A szerver hálózati kártyáját (lehetőleg a 2.5 GbE vagy 10 GbE portot) a CSS610 Switch egyik szabad portjába kösd.
- Kábel: Használj minimum Cat6a (vagy Cat7) S/FTP árnyékolt patch kábelt. Ez biztosítja, hogy a 12 TB-os ZFS pool adatátvitele ne akadjon meg az elektromos zajok miatt.

### 3. Raspberry Pi 4 és egyéb kiegészítők

- Raspberry Pi: A Pi-t közvetlenül a CSS610 Switch egyik Gigabit portjába dugd. Mivel a Pi-hole DNS-ként funkcionál, a stabil vezetékes kapcsolat kötelező.
- IP Kamerák: Ha PoE-képesek, és a switch nem ad áramot, szükséged lesz PoE injektorokra. Ezeket a switch és a kamera közé kell kötni.

## 🛠️ Kábelezési Aranyszabályok a Homelabhoz

- *Színkódolás (Opcionális, de profi):*
    - *Kék:* Általános adatforgalom (PC, Laptop).
    - *Piros:* Kritikus eszközök (Szerver, Router, Switch).
    - *Sárga:* IoT eszközök és kamerák.
    - Ez segít, ha fél év múlva a rack mögött kell keresgélned egy hibát.
- *Hajlítási sugár:* A Cat6a kábelek vastagabbak és merevebbek. Ne törd meg őket derékszögben a falnál vagy a switchnél, mert az rontja a jelminőséget és adatcsomag-vesztéshez vezethet.
- *UPS (Szünetmentes) bekötés:* * A szünetmentes tápegységbe (APC) a Router, a Switch és a Szerver tápkábeleit dugd be.
    - Ne felejtsd el összekötni az APC USB kábelét a szerverrel, hogy az Ubuntu automatikusan le tudjon állni (Graceful Shutdown), ha elmegy az áram.

## 📦 Hardver-sorrend a szekrényben (fentről lefelé)

A hűtés és a kábelrendezés miatt ezt a sorrendet javaslom a polcon vagy rackben:

- MikroTik hAP ax³: Legfelül, hogy a Wi-Fi antennák szabadon sugározhassanak.
- MikroTik CSS610 Switch: Közvetlenül a router alatt (rövid DAC kábel miatt).
- Kábelrendező tálca: Itt futnak el a patch kábelek oldalra.
- Ubuntu Szerver: Ez a legnehezebb és legmelegebb eszköz, kerüljön alulra.
- APC UPS: Legalulra, mert ez a legnehezebb, és itt nem melegíti az érzékenyebb eszközöket.

Ezzel a fizikai felépítéssel a hálózatod nemcsak logikailag, hanem fizikailag is "bulletproof" lesz.
