# Felvétel és a .daq formátum

Az `.daq` fájl a Chloros

fényérzékelőjének felvételi formátuma: egy DAQ-érzékelő kalibrált spektrális képkockáiból álló **SQLite adatbázis**. Ha egy rögzítési munkamenet során ilyen fájlt készítünk, a reflexiós feldolgozó folyamat később minden képet eloszthat az adott pillanatban mért lefelé irányuló besugárzási intenzitással.

## Mit tartalmaz egy .daq fájl?

| Tulajdonság | Érték |
| --- | --- |
| Konténer | SQLite adatbázis, felvételenként és érzékelőnként egy fájl |
| Fájlnév | Tartalmazza az **érzékelő azonosítóját**és egy**időbélyeget**, pl. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektrum képkockánként | 135 pont, 340–1010 nm-es tartományban 5 nm-es lépésekben, valamint CIE XYZ háromstimulusz |
| Mértékegységek | Kalibrált spektrális sugárzási intenzitás, **W/m²/nm** (gyári kalibrációs csomag + alkalmazott sapka-profil) |
| Bejegyzett metaadatok | Érzékelő-azonosító (a kulcs az adott egység gyári kalibrációjának lekéréséhez) és az érvényes sapka-profil — lásd [Sapka-profilok és kalibrált tartomány](caps-and-range.md) |

A formátum a DAQ-U, DAQ-M, DAQ-E és DAQ-E-S eszközökön azonos, így a későbbi feldolgozás során nem számít, melyik eszköz rögzítette az adatokat. A DAQ-E-S felvételek minden spektrum mellett az eszköz dőlésszögét, forgásszögét és emelkedési szögét is tárolják.

**A felvételek áramkimaradás esetén is megmaradnak (1.3.0).**Az élő felvétel minden commit alkalmával naplóba kerül, és teljes lemezszinkronizálás történik, így egy áramkimaradás vagy egy leállított folyamat esetén csak az utolsó 3 másodperc veszik el, nem pedig a fájl. Az írás közben leállított felvétel az `.daq` fájl mellett egy `<name>.daq-journal` fájlt is hagyhat:**mindkettőt őrizze meg és másolja át együtt.** Az Chloros

az első fájlolvasáskor visszagörgeti a naplófájlt. A leterhelt lemez késlelteti a fájl írását, de soha nem az érzékelő adatait.

**Az adatátvitelt leállító DAQ-E-t újra csatlakoztatja a rendszer (1.3.0).** Amikor egy DAQ-E adatfolyama elhallgat (a kapcsolóját ki- és bekapcsolták, vagy megszakadt a kapcsolat), a háttérrendszer a korábbi beállításokkal csatlakoztatja újra, és a felvétel ugyanabban az `.daq` fájlban folytatódik, a hiányzó rész a fájlban megjelölve.

A kalibrált felvételhez szükség van az érzékelő gyári kalibrációs csomagjára. A DAQ-U és DAQ-M esetében a háttérprogram az érzékelő azonosítója alapján letölti a csomagot a MAPIR

felhőjéből (ha ez nem sikerül, a felvétel elutasításra kerül); a DAQ-E egységek kivételt képeznek, mivel a kalibrációs adatokat a készüléken tárolják.

## Felvétel a grafikus felhasználói felületről

A GUI-ban történő felvételhez **nyitott projekt** szükséges (ellenkező esetben a Felvétel gombok le vannak tiltva):

* **Minden felvétele / Minden leállítása** — a Fényérzékelők oldalsáv tetején; egyszerre indítja el vagy állítja le az `.daq` felvételt minden csatlakoztatott érzékelőn.
* **Felvétel / Felvétel leállítása** — érzékelőnként, a fogaskerék-beállítások ablakban. Felvétel közben piros „REC” jelző jelenik meg az érzékelő élő adatainak sorában.

A fájlok az `<project>/light_sensor/` mappába kerülnek, és amikor a felvétel leáll — akár a „Leállítás”, „Minden leállítása” gombokkal, akár a felvételt végző érzékelő leválasztásával —, a kész `.daq` fájl **automatikusan hozzáadódik a megnyitott projekthez**. A projekt fájllistájában manuális hozzáadási lépés nélkül jelenik meg, és máris készen áll a reflexiós feldolgozásra.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## Felvétel a CLI

-ről

A CLI

a háttérrendszer érzékelőpoolján keresztül rögzít (a háttérrendszernek futnia kell — ezek a parancsok vékony HTTP

-kliensek):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Az `--sensor-id` érték lekérése az `chloros-cli daq pool-list`-ből. Két érdemes tudni alapértelmezés:

| Opció | Alapértelmezés |
| --- | --- |
| `--duration` | `0` — rögzítés `pool-record --stop`-ig |
| `--output` / `-o` | `~/Documents/DAQ Live View/` a **háttérrendszer** fájlrendszerén, nem az CLI

-én |

A kimeneti könyvtár megkülönböztetése akkor fontos, ha a CLI

-nek egy másik gépen futó háttérrendszere van: a fájl oda kerül, ahol a háttérrendszer fut. Ha a háttérrendszer ugyanazon a számítógépen van, akkor egy relatív `-o` (például a fenti `./out`) a parancs futtatásának helye szerinti mappához van rendelve.

## Felvétel Python

ból Az `DAQSensorSession` (amelyet az `chloros_sdk.connect_daq_sensor()` ad vissza) ugyanazt a csoportosított felvételt teszi elérhetővé: Az `record_start(output_dir=None, device_name=None)` a fájl elérési útját adja vissza, az `record_stop()` pedig az `{path, rows}`-et. A teljes munkamenet-API

-t lásd az [SDK

-referenciában](../reference/sdk-reference.md). Az SDK

közvetlen hardverosztályai (csak asztali telepítések esetén) alapértelmezés szerint az `~/Documents/DAQ/`-ba írják a felvételeket; a kiadott verziók esetében a fenti összevont elérési út a támogatott módszer.

## .daq fájl használata a feldolgozás során

A képekből származó reflexió kiszámításához a Chloros

-nak minden expozícióhoz illeszkedő lefelé irányuló besugárzási értékre van szüksége:

* **Tartsa meg az `.daq` fájlt a képekkel együtt.**A feldolgozás során a feldolgozási folyamat automatikusan meghatározza az**időbélyeggel egyező lefelé irányuló sugárzást** egy rögzített `.daq` fájlból (bármely DAQ-modell) — vagy egy DAQ-M natív `.csv` fájlból, amelyet a képek mellett talál. A GUI-felvételek automatikusan megfelelnek ennek a követelménynek, mivel a leállás pillanatában hozzáadódnak a projekthez.
* **A kalibrációt igény szerint tölti be a rendszer.** Ha egy kamera- vagy DAQ-specifikus gyári kalibrációs csomag még nincs helyileg tárolva, a Chloros

az első használatkor automatikusan letölti azt a MAPIR

felhőjéből (egyszeri internetkapcsolat szükséges; az `~/.chloros/` fájlban tárolódik).
* **Az élő felvételek saját kiegészítő fájlt írnak.** Minden élőben rögzített reflektancia-képkocka esetében a ténylegesen használt DAQ-érték `.daq` fájlként kerül mentésre a rögzítések `daq/` mappájába, így a rögzítés később az eredeti felvétel nélkül is újra feldolgozható.
* **Felvételek az `chloros_scripts` felvevőkről (1.3.0).**A nyilvános [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) felvevők által írt `.daq` fájl szerepel a létrehozó DAQ-modellként, és a felső határa a**Projektbeállítások → DAQ fényérzékelő** menüpontban állítható be, akárcsak bármely más felvétel esetében. Ezek a felvevők nyers érzékelőértékeket írnak; az Chloros

soros kapcsolaton keresztül letölti az érzékelő gyári kalibrációját, és azt alkalmazza.
* **Kinek a mérési értéke ez (1.3.0).**Minden `.daq` felvétel mellett egy `.daq.cameras.json` megnevezi azt a kamerát (vagy kamerákat), amelynek (amelyeknek) képkockáira az adott mérési érték vonatkozik, és a feldolgozás**kizárólag az azokból származó képekhez** rendeli hozzá az értéket. Egy önálló kamera, amelynek nincs saját fényérzékelője, ezért már nem veszi át egy másik kamera mérési eredményét – csatlakoztasson hozzá egy érzékelőt, vagy rögzítsen egyet. A DAQ-val közvetlenül készített felvétel (GUI Record, `daq pool-record`), valamint minden olyan `.daq` fájl, amely nem tartalmazza ezt a fájlt (beleértve a régebbi rögzítéseket is), továbbra is minden kamerát kiszolgál.

## A besugárzás visszakeresése

A projekt feldolgozása során a rendszer minden benne található fényérzékelő-felvételt exportál egy
`Light Sensor/` mappába a képtermékek mellé. Ehhez **nincs** szükség képanyagra: egy
önmagában rögzített fényérzékelő-felvétel is teljes rögzítésnek számít, és egy kizárólag `.daq`
fájlokat tartalmazó mappa is érvényes bemenetnek minősül. Egy kizárólag fényérzékelő-felvételeket tartalmazó projekt felsorolja azokat, és
az alkalmazásból és a CLI

-ből egyaránt feldolgozza őket, és a futtatás után is a listán maradnak. A futtatás
jelenti, hogy hány fényérzékelő-terméket írt ki.

| Termék | Mi ez? |
| --- | --- |
| `<name>_calibrated.daq` | Újrafeldolgozható archívum, amely ugyanazt a sémát követi, mint az élő felvétel, és most már megadja azt a kalibrációs csomagot is, amely létrehozta. Újbóli importálása **nem** eredményezi a második kalibrálást. |
| `<name>_calibrated.csv` | Spektrális besugárzás W/m²/nm-ben az érzékelő saját hullámhossz-rácsán, egy sor minden egyes mérési értékre, valamint fotometrikus oszlopok: teljes teljesítmény, fotopikus és skotopikus lux, PPFD kék/zöld/vörös bontással, valamint csúcshullámhossz. |

Egy olyan DAQ-U vagy DAQ-M, amelynek kalibrációs csomagja nem tölthető le — mert nincs internetkapcsolat, vagy
az érzékelőhöz nincs kalibráció a rendszerben —, **ok megjelölésével kihagyásra kerül**, soha nem kerül kiírásra
„kalibrált” fájlként, amely nyers számlálási adatokat tartalmaz. Csatlakozzon az internethez, és futtassa újra a műveletet. A DAQ-E
saját kalibrációval rendelkezik, így erre csak akkor van szüksége, ha a készülék nincs csatlakoztatva, és
helyileg semmi sincs tárolva.

### DAQ-A: nyers számlálási adatok, és miért ez a helyes megoldás

A **DAQ-A** a soronkénti kalibrációs csomagrendszer bevezetése előtt készült, ezért nincs letöltendő
csomagja. Ez nem figyelmetlenség: a DAQ-A-t a helyszínen kalibrálják egy
reflektancia-célponttal szemben, és a célpont-alapú kalibráláshoz csak az érzékelő *relatív*
válaszára van szükség — ami pontosan a nyers számlálási értékei. A Chloros

ma is ezekkel kalibrál.

Tehát a DAQ-A felvétele exportálható, de más név alatt:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, nem pedig `_calibrated` — egy másik fájlnév, nem pedig egy jelző a fájl belsejében,
mert az azonosítónak meg kell maradnia akkor is, ha a fájlt puszta névként továbbítják e-mailben. Az `.csv`
fejlécben az `raw spectral sensor counts (NOT irradiance)` szerepel, és figyelmeztet arra, hogy az értékek
**a** fájlon belül összehasonlíthatók, érzékelők között viszont nem. Azok az oszlopok, amelyek csak
a valós besugárzási értékek esetében jelentenek valamit — teljes teljesítmény, lux, PPFD — üresen maradnak, ahelyett, hogy
a számlálásokból lennének kiszámítva.

A régebbi DAQ-A-SD-felvételek (v1.01 / v1.02 sémák) csak a fájl írási idejét rögzítik, nem pedig
olvasatonkénti időbélyeget. A Chloros

nem fogja ezekhez a képeket illeszteni — egy képkockát egy
írási időhöz párosítani helytelen lenne, anélkül hogy valaha is hibásnak tűnne —, de az exportálás problémamentesen olvassa be őket, és
a CSV

-fájl megadja, hogy melyik órához tartoznak.

A teljes reflektancia-leíráshoz – egyérzékelős kamerával, valamint kétérzékelős környezeti/objektum-méréssel – lásd [Reflektancia-munkafolyamatok](reflectance.md).
