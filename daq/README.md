# DAQ fényérzékelők

> **A hardvert keresed?**Maguk az érzékelők — a modellek, a rögzítés, a fedelek, a csatlakozók, a tápellátás és a SCANNER alkalmazás — a**[DAQ felhasználói kézikönyvben](https://mapir.gitbook.io/daq)** vannak leírva. Ez a fejezet az Chloros alkalmazásból történő használatukat tárgyalja. A

MAPIR **DAQ** fényérzékelői a környezeti fényt radiometrikusan kalibrált spektrumként mérik. Az Chloros alkalmazásban két szerepet töltenek be:

* **Önálló spektrális műszer** — élő spektrumdiagramok, kolorimetrikus adatok és `.daq` felvételek, mindez a [Fényérzékelők fülön](gui.md), a [CLI-en](cli-quick-start.md) vagy a Python SDK-en keresztül.
* **A reflexióhoz szükséges lefelé irányuló sugárzási forrás** — a feldolgozás során a Chloros interpolálja az `.daq` mérési eredményeket az egyes felvételek expozíciós időbélyegéhez, és a mért lefelé irányuló fényt használja a kamera sugárzási intenzitásának reflektanciává (`--reflectance-source daq`) történő átalakításához; kalibrált sávok esetén nincs szükség a jelenetben lévő panelre.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Három modell, egy adatformátum

| Modell | Adatátvitel | Felismerés |
| --- | --- | --- |
| **DAQ-U** | USB (soros) | soros porton keresztüli keresés |
| **DAQ-M** | Bluetooth Low Energy | név szerinti BLE-keresés |
| **DAQ-E** | Ethernet (IPv4, PoE-táplálás) | mDNS `_daq-e._tcp` (gazdagépnév `daq-e-<id>.local`) |

**DAQ-E-S (újdonság a 1.3.0-s verzióban).** Egy DAQ-E állandó diffúzorral és dőlésérzékelővel. Az Chloros az `DAQ-E-S` néven szerepelteti, és pontosan úgy működik, mint egy DAQ-E, azzal a különbséggel, hogy nem igényel sapkakorrekciót, és minden leolvasott érték tartalmazza a készülék dőlését, forgását és emelkedését. Lásd [DAQ-E-S: rögzített diffúzor és dőlésszög](caps-and-range.md#daq-e-s-fixed-diffuser-and-tilt).

Mindhárom készülék ugyanazt a vezetékes protokollt használja, és azonos adatokat szolgáltat:

* **135 pontos spektrum 340–1010 nm tartományban, 5 nm-es lépésekben**, valamint CIE XYZ háromstimulusz-értékek minden képkockában.
* **Radiometrikusan kalibrált spektrális besugárzási erősség W/m²/nm-ben** — az adatok eljutása előtt minden egység gyári kalibrációs csomagját (valamint az aktív sapkakorrekciós profilját) alkalmazzák.
* Ugyanaz az **`.daq` felvételi formátum** (egy SQLite-fájl). A további feldolgozás azonos, függetlenül attól, hogy melyik átviteli mód hozta létre a fájlt.

A transzportrétegek (USB soros, BLE, mDNS/zeroconf) az Chloros háttérrendszerbe vannak beépítve — nincs szükség semmilyen telepítésre ahhoz, hogy a grafikus felhasználói felületről vagy a CLI `pool-*` parancsain keresztül kommunikálhasson a három modell bármelyikével.

***

## Kalibrált tartomány: jelentett 340–1010 nm, kalibrált ~374–974 nm

Az érzékelő a teljes 340–1010 nm-es rácsot jelenti, de a NIST-hez nyomon követhető radiometrikus erősítés körülbelül **374–974 nm**-re terjed ki. A Chloros nem alkalmazza az abszolút reflexió felosztását olyan kamerasávokra, amelyek spektrális súlyának kevesebb mint a fele esik a kalibrált tartományba; az kihagyott sávot az `dls-uncalibrated-band-<nm>` kihagyási okkal jelenti.

A forgalomban lévő LATTICE szűrő-SKU-k közül csak az **F988** érintett:

Az F988 reflektanciáját egy helyszíni reflektancia-panel segítségével kalibrálják: a sáv a DAQ fényérzékelő kalibrált tartományán kívül esik, ezért a Chloros a legutóbbi panelről készült felvételt alkalmazza, és azt a panel-megfigyelések között megőrzi.

Ha egy F988-es rögzítést kizárólag DAQ-adatok alapján dolgoznak fel, a Chloros elutasítja a DAQ-alapú reflektanciát az adott sávra, a kihagyás okaként az `dls-uncalibrated-band-988` kódot jelölve meg — az [reflektancia-panel munkafolyamat](../calibration-targets.md) a támogatott eljárás az F988 esetében.

***

## Érzékelő-azonosítók

Minden DAQ stabil érzékelő-azonosítót jelent. Az azonosító formája modelltől függően eltérő:

| Modell | Azonosító formája | Példa |
| --- | --- | --- |
| DAQ-U | 5-oktett, kötőjeles | `CB-7C-A8-2E-5F` |
| DAQ-M | 5-oktett, kötőjeles | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Az érzékelő azonosítója:

* be van írva minden `.daq` fájlba, amelyet rögzít,
* az a kulcs, amelyet az Chloros használ az adott egység gyári kalibrációs csomagjának letöltéséhez,
* az az érték, amelyet az `--sensor-id`-nek ad át az CLI-ben az `pool-*` parancsokkal, és
* a DAQ-E esetében emellett az mDNS-gazdagépneve (`daq-e-def330.local`) is – ez az az érték, amelyet az `--eth-host` elfogad.

***

## Gyári kalibrálás és a felhő

Minden DAQ-egységet gyárilag egyedileg kalibrálnak egy NIST-nyomon követhető radiometrikus lánc segítségével, és a Chloros betölti az egyes egységek kalibrációs csomagját az érzékelő azonosítója alapján. Az egyes egységekre vonatkozó kalibrációs jelentés (PDF) letölthető az érzékelő beállításaiból a [Fényérzékelők fülön](gui.md).

{% hint style="warning" %}
**A DAQ-U és a DAQ-M kalibrálásához felhőhozzáférés szükséges.**Egyik modell sem tárol semmit a készüléken: gyári kalibrációs csomagjaik a MAPIR felhőjében találhatók, és az érzékelő azonosítója alapján kerülnek letöltésre (majd helyileg tárolásra). A Chloros-nak internetkapcsolatra van szüksége ahhoz, hogy kalibrált W/m²/nm adatokat továbbítson egy DAQ-U-ról vagy DAQ-M-ről.**A DAQ-E kivételt képez** — kalibrációs adatait a készüléken tárolja.
{% endhint %}
<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

***

## A felvételek mentési helye

| Felület | Alapértelmezett `.daq` célhely |
| --- | --- |
| GUI — Fényérzékelők fül | `<project folder>/light_sensor/` (a befejezett felvételek automatikusan hozzáadódnak a projekthez) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` a háttérprogramot futtató gépen (a relatív `-o` a parancs futtatásának helye szerinti mappához viszonyul, ha a háttérprogram ugyanazon a számítógépen fut) |

Minden `.daq` fájlnév tartalmazza az érzékelő azonosítóját és egy időbélyeget.

***

## Ebben a fejezetben

* [**A DAQ fül az Chloros-ban**](gui.md) — a teljes grafikus felhasználói felület bemutatása: az egyes modellek csatlakoztatása, érzékelőnkénti beállítások, spektrumdiagramok, élő kolorimetrikus adatok, kettős érzékelős reflexió és felvétel.
* [**A CLI gyors útmutató (pool-\*)**](cli-quick-start.md) — a DAQ-érzékelők vezérlése az `chloros-cli daq pool-*`-ből, a támogatott parancssori útvonal.
* [**Felső határprofilok és kalibrált tartomány**](caps-and-range.md) — mely felső határok léteznek modellenként, hogyan kell azokat megadni, valamint a kalibrált spektrális tartomány részletes leírása.
* [**Felvétel és a .daq formátum**](recording.md) — az `.daq` SQLite formátum és a felvételi munkafolyamatok.
* [**DAQ-E hálózat és időszinkronizálás**](ethernet-ptp.md) — a DAQ-E átviteli módok és a PTP időszinkronizálás.
* [**Reflektancia-munkafolyamatok**](reflectance.md) — a DAQ lefelé irányuló adatainak felhasználása a reflektancia kiszámításához.
* A teljes flag-szintű dokumentációért lásd a [CLI-referenciát](../reference/cli-reference.md) (`chloros-cli daq` szakasz) és az [SDK-referenciát](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`) című dokumentumot, amelyek mindkettő úgy készült, hogy az AI-asszisztensek közvetlenül felhasználhassák őket.
