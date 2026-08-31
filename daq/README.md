# DAQ fényérzékelők

> **A hardverrel kapcsolatos információkat keresed?**Maguk az érzékelők — a modellek, a rögzítés, a védőfedelek, a csatlakozók, a tápellátás és a SCANNER alkalmazás — a**[DAQ felhasználói kézikönyvben](https://mapir.gitbook.io/daq)** találhatók. Ez a fejezet az Chloros-től kezdve tárgyalja azok használatát.

Az MAPIR **DAQ** fényérzékelői a környezeti fényt radiometrikusan kalibrált spektrumként mérik. Az Chloros-ben két szerepet töltenek be:

* **Önálló spektrális műszer** — élő spektrumdiagramok, kolorimetrikus adatok és `.daq` felvételek, mindez a [Fényérzékelők fülről](gui.md), az [CLI](cli-quick-start.md) vagy az Python SDK menüpontokból.
* **Lefelé irányuló sugárzási forrás a reflexióhoz** — a feldolgozás során az Chloros interpolálja az `.daq` mérési eredményeit az egyes felvételekexpozíciós időbélyegéhez, és a mért lefelé irányuló fényt használja a kamera sugárzási intenzitásának reflektanciává történő átalakításához (`--reflectance-source daq`); a kalibrált sávokhoz nincs szükség a jelenetben elhelyezett panelre.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Három modell, egy adatformátum

| Modell | Adatátvitel | Felismerés |
| --- | --- | --- |
| **DAQ-U** | USB (soros) | soros porton keresztüli keresés |
| **DAQ-M** | Bluetooth Low Energy | név szerinti BLE-keresés |
| **DAQ-E** | Ethernet (IPv4, PoE-táplált) | mDNS `_daq-e._tcp` (gazdagépnév: `daq-e-<id>.local`) |

Mindhárom ugyanazt a vezetékes protokollt használja, és azonos adatokat szolgáltat:

* **135 pontból álló spektrum 340–1010 nm tartományban, 5 nm-es lépésekben**, valamint CIE XYZ háromstimulusz-értékek minden képkockában.
* **Radiometrikusan kalibrált spektrális sugárzási intenzitás W/m²/nm-ben** — az egyes egységek gyári kalibrációs csomagját (valamint az aktív sapkakorrekciós profilját) alkalmazzák, mielőtt az adatok eljutnának Önhöz.
* Ugyanaz az **`.daq` rögzítési formátum** (egy SQLite-fájl). A további feldolgozás azonos, függetlenül attól, hogy melyik átviteli mód hozta létre a fájlt.

A transzportrétegek (USB soros, BLE, mDNS/zeroconf) az Chloros háttérprogramon belül vannak csomagolva — nincs szükség semmilyen telepítésre ahhoz, hogy a grafikus felhasználói felületről vagy az CLI `pool-*` parancsain keresztül kommunikálj bármelyik három modellel.

***

## Kalibrált tartomány: 340–1010 nm jelentett, ~374–974 nm kalibrált

Az érzékelő a teljes 340–1010 nm-es rácsot jelenti, de a NIST-hez nyomon követhető radiometrikus erősítés körülbelül **374–974 nm** tartományra terjed ki. Az Chloros parancs elutasítja az abszolút reflexió osztását minden olyan kamerasáv esetében, amelynek spektrális súlyának kevesebb mint a fele esik a kalibrált tartományba; az elhagyott sávot az `dls-uncalibrated-band-<nm>` elhagyási okkal jelenti.

A forgalomban lévő LATTICE szűrő-típusok közül csak az **F988** érintett:

Az F988 reflektanciáját egy helyszíni reflektancia-panel segítségével kalibrálják: mivel a sáv a DAQ fényérzékelő kalibrált tartományán kívül esik, az Chloros a legutóbbi panelről rögzített értéket alkalmazza, és azt a panel-megfigyelések között megőrzi.

Ha egy F988-es rögzítést kizárólag a rendelkezésre álló DAQ-adatok alapján dolgoznak fel, az Chloros elutasítja a DAQ-alapú reflektanciát az adott sávra, az `dls-uncalibrated-band-988` kihagyási okkal — az [reflektancia-panel munkafolyamat](../calibration-targets.md) a támogatott eljárás az F988 esetében.

***

## Érzékelő-azonosítók

Minden DAQ stabil érzékelő-azonosítót jelent. Formája modelltől függően eltérő:

| Modell | Azonosító formája | Példa |
| --- | --- | --- |
| DAQ-U | 5-oktett, kötőjeles | `CB-7C-A8-2E-5F` |
| DAQ-M | 5-oktett, kötőjellel elválasztva | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Az érzékelő azonosítója:

* be van égetve minden `.daq` fájlba, amelyet rögzít,
* az a kulcs, amelyet az Chloros használ az adott egység gyári kalibrációs csomagjának letöltéséhez,
* az az érték, amelyet az CLI és `pool-*` parancsokban az `--sensor-id`-nek ad át, valamint
* a DAQ-E esetében ezenkívül annak mDNS-gazdagépnevét (`daq-e-def330.local`) — az értéket, amelyet az `--eth-host` elfogad.

***

## Gyári kalibrálás és a felhő

Minden DAQ-egységet gyárilag egyedileg kalibrálnak egy NIST-nyomon követhető radiometrikus lánc segítségével, és az Chloros betölti az egyes egységek szenzor-azonosítója alapján lekötött kalibrációs csomagját. Az egyes egységekre vonatkozó kalibrálási jelentés (PDF) letölthető az érzékelő beállításaiból a [Fényérzékelők fülön](gui.md).

{% hint style="warning" %}
**A DAQ-U és a DAQ-M kalibrálásához felhőhozzáférés szükséges.**Egyik modell sem tárol semmit a készüléken: gyári kalibrációs csomagjaik az MAPIR felhőjében találhatók, és az érzékelő azonosítója alapján kerülnek letöltésre (majd helyileg tárolásra). Az Chloros-nek internetkapcsolatra van szüksége ahhoz, hogy kalibrált W/m²/nm adatokat továbbítson egy DAQ-U-tól vagy DAQ-M-től.**A DAQ-E kivételt képez** — kalibrációját a készüléken tárolja.

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## A felvételek tárolási helye

| Felület | Alapértelmezett `.daq` célhely |
| --- | --- |
| Felhasználói felület — Fényérzékelők fül | `<project folder>/light_sensor/` (a befejezett felvételek automatikusan hozzáadódnak a projekthez) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` a háttérprogramot futtató gépen |

Minden `.daq` fájlnév tartalmazza az érzékelő azonosítóját és egy időbélyeget.

***

## Ebben a fejezetben

* [**A DAQ fül az Chloros-ben**](gui.md) — a teljes grafikus felhasználói felület bemutatása: az egyes modellek csatlakoztatása, érzékelőnkénti beállítások, spektrumdiagramok, élő kolorimetrikus adatok, kettős érzékelős reflexancia és rögzítés.
* [**CLI Gyorsindítás (pool-\*)**](cli-quick-start.md) — a DAQ-érzékelők vezérlése az `chloros-cli daq pool-*`-ből, a támogatott parancssori útvonal.
* [**Felső határprofilok és kalibrált tartomány**](caps-and-range.md) — mely felső határok léteznek modellenként, hogyan kell azokat megadni, valamint a kalibrált spektrális tartomány részletes leírása.
* [**Felvétel és a .daq formátum**](recording.md) — az `.daq` SQLite formátum és a felvételi munkafolyamatok.
* [**DAQ-E hálózatok és időszinkronizálás**](ethernet-ptp.md) — a DAQ-E átviteli módok és a PTP időszinkronizálás.
* [**Reflektancia-munkafolyamatok**](reflectance.md) — a DAQ lefelé irányuló adatainak felhasználása a reflektancia kiszámításához.
* A teljes, flag-szintű dokumentációért lásd az [CLI Referenciát](../reference/cli-reference.md) (`chloros-cli daq` szakasz) és a [SDK Referenciát](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), amelyek mindkettő úgy készült, hogy AI-asszisztensek közvetlenül felhasználhassák őket.
