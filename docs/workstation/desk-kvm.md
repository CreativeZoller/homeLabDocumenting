# Asztal, KVM, ergonómia

A 2.0 2–4. fázisa; a 2.6 ezt nem változtatta. A hálózat a 10" rackben van, a munkaállomás az asztalon.

## Asztal

A jelenlegi 166×66 cm lap helyett 180×80×2.5 cm tömörfa.

1. Három kábelkivezető furat (60 mm lyukfűrész).
2. Csiszolás 180-as majd 240-es.
3. Faolaj.
4. Alul: kábelcsatorna, KVM, ahol elfér, további 19" PDU.

Alkatrészek és árak: [Fázisok és BOM](../planning/phases-bom.md).

## KVM — AV Access 4KSW41C

Ugyanaz a billentyűzet, egér, monitor: fő PC, laptopok, Raspberry Pi.

- USB-C (video + 100W PD) a laptophoz.
- HDMI / DP kábelek a 1. fázis listájából (aktív DP→HDMI, HDMI 2.1, micro-HDMI a Pi-hez).
- USB 3.0 A→B a KVM host portjaira.

A KVM az asztal alatt; nem a 10" rack 1U-ja.

## Ergonómia és hang

- Duál gázrugós monitorkar + laptoptartó (MacBook / Lenovo).
- Blue Yeti X, shock mount, mikrofon gázkar — a korábbi Anua helyett, kisebb hely.

## Nyomtató

Brother DCP-L2620DW, VLAN 40 vagy 10, statikus DHCP a Firewalla-n. Paperless-ngx consume mappa a NAS-on.

## 4. fázis: kábel és fény

- Digitus kábelcsatorna + 20 m szoknya + 3M ragasztó. Az asztal magasságállításakor a kábel ne feszüljön.
- Egy furatba Icy Box USB 3.0 hub.
- Yeelight YLTD003 a monitor felett, Groove LED az asztal hátulján — később Home Assistant.

A 10" rack kábelezése külön: [Eszközök kötése](../network-setup/device-connections.md).
