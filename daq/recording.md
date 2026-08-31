# Felvétel és a .daq formátum

Az `.daq` fájl az Chloros fényérzékelőjének felvételi formátuma: egy DAQ-érzékelő kalibrált spektrális képkockáiból álló **SQLite adatbázis**. Ha egy rögzítési munkamenet során ilyen fájlt készítünk, a reflexiós feldolgozó folyamat később minden képet eloszthat az adott pillanatban mért lefelé irányuló sugárzási intenzitással.

## Mit tartalmaz egy .daq fájl?

| Tulajdonság | Érték |
| --- | --- |
| Konténer | SQLite adatbázis, érzékelőnként és felvételenként egy fájl |
| Fájlnév | Tartalmazza az **érzékelő azonosítóját**és egy**időbélyeget**, pl. `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Spektrum képkockánként | 135 pont, 340–1010 nm-es tartományban 5 nm-es lépésekben, valamint CIE XYZ háromstimulusz |
| Mértékegységek | Kalibrált spektrális besugárzási intenzitás, **W/m²/nm** (gyári kalibrációs csomag + alkalmazott sapka-profil) |
| Bejegyzett metaadatok | Érzékelő-azonosító (a kulcs az adott egység gyári kalibrációjának lekéréséhez) és az érvényes sapka-profil — lásd [Sapka-profilok és kalibrált tartomány](caps-and-range.md) |

A formátum a DAQ-U, DAQ-M és DAQ-E esetében azonos, így a későbbi feldolgozás során nem számít, melyik eszköz rögzítette az adatokat.

A kalibrált rögzítéshez az érzékelő gyári kalibrációs csomagjára van szükség. A DAQ-U és DAQ-M esetében a háttérrendszer az érzékelő-azonosító alapján letölti a csomagot az MAPIR felhőjéből (ha ez nem sikerül, a felvétel elutasításra kerül); a DAQ-E egységek kivételt képeznek, mivel a kalibrációt maguk az eszközök tárolják.

## Felvétel a grafikus felhasználói felületről

A GUI-ban történő rögzítéshez **nyitott projekt** szükséges (ellenkező esetben a Rögzítés gombok le vannak tiltva):

* **Minden rögzítése / Minden leállítása** — a Fényérzékelők oldalsáv tetején; egyszerre indítja el vagy állítja le az `.daq` rögzítést minden csatlakoztatott érzékelőn.
* **Felvétel / Felvétel leállítása** — érzékelőnként, a fogaskerék-beállítási ablakban. Felvétel közben piros „REC” jelző jelenik meg az érzékelő élő adatainak sorában.

A fájlok az `<project>/light_sensor/` mappába kerülnek, és amikor a felvétel leáll — akár a „Leállítás”, „Minden leállítása” gombokkal, akár a felvételt végző érzékelő leválasztásával —, a kész `.daq` fájl **automatikusan hozzáadódik a megnyitott projekthez**. A projekt fájllistájában megjelenik anélkül, hogy kézzel hozzá kellene adni, így máris készen áll a reflexiós feldolgozásra.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Felvétel az CLI-ről

Az CLI a háttérprogram érzékelőpoolján keresztül rögzít (a háttérprogramnak futnia kell – ezek a parancsok vékony HTTP kliensek):

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

Az `--sensor-id` érték lekérése az `chloros-cli daq pool-list`-ból. Két érdemes tudni alapértelmezés:

| Opció | Alapértelmezés |
| --- | --- |
| `--duration` | `0` — rögzítés `pool-record --stop`-ig |
| `--output` / `-o` | `~/Documents/DAQ Live View/` a **háttérrendszer** fájlrendszerén, nem az CLI-én |

A kimeneti könyvtár megkülönböztetése akkor fontos, ha az CLI egy másik gépen futó háttérrendszert céloz meg: a fájl oda kerül, ahol a háttérrendszer fut.

## Felvétel az Python-ből

Az `DAQSensorSession` (amelyet az `chloros_sdk.connect_daq_sensor()` ad vissza) ugyanazt a közös felvételt teszi elérhetővé: Az `record_start(output_dir=None, device_name=None)` a fájl elérési útját adja vissza, az `record_stop()` pedig az `{path, rows}`-et. A teljes munkamenet API leírását lásd az [SDK referencia](../reference/sdk-reference.md) alatt. Az SDK közvetlen hardverosztályai (csak asztali telepítések esetén) alapértelmezés szerint az `~/Documents/DAQ/`-ba írják a felvételeket; a kiadott verziók esetében a fenti összevont elérési út a támogatott módszer.

## .daq fájl használata a feldolgozás során

A képekből származó reflexió kiszámításához az Chloros-nek minden expozícióhoz illeszkedő lefelé irányuló besugárzási értékre van szüksége:

* **Tartsa meg az `.daq` fájlt a képekkel együtt.**A feldolgozás során a feldolgozási folyamat automatikusan kiszámítja az**időbélyeggel egyező lefelé irányuló sugárzást** egy rögzített `.daq` fájlból (bármely DAQ-modell) – vagy egy DAQ-M natív `.csv` fájlból, amely a képek mellett található. A GUI-felvételek automatikusan megfelelnek ennek a követelménynek, mivel a projektbe a leállás pillanatában kerülnek hozzáadásra.
* **A kalibrációt igény szerint tölti le a rendszer.** Ha egy kamera- vagy DAQ-specifikus gyári kalibrációs csomag még nincs helyileg tárolva, az Chloros az első használatkor automatikusan letölti azt az MAPIR felhőjéből (egyszeri internetkapcsolatra van szükség; a `~/.chloros/` alatt kerül tárolásra).
* **Az élő felvételek saját sidecar-fájlt írnak.** Bármely élőben rögzített reflektancia-képkocka esetében a ténylegesen felhasznált DAQ-leolvasási érték `.daq` sidecar-fájlként kerül mentésre a kép mellé, így a felvétel később az eredeti felvétel nélkül is újra feldolgozható.

## A besugárzás visszakeresése

A projekt feldolgozása során a rendszer minden benne található fényérzékelő-felvételt exportál egy
`Light Sensor/` nevű mappába a képtermékek mellé. Ehhez **nincs** szükség képanyagra: egy
önállóan repült fényérzékelő is teljes felvételt jelent, és egy kizárólag `.daq`
fájlokat tartalmazó mappa is érvényes bemeneti adat. A futtatás jelentése tartalmazza, hogy hány fényérzékelő-terméket írt ki.

| Termék | Mi ez? |
| --- | --- |
| `<name>_calibrated.daq` | Újrafeldolgozható archívum, amely ugyanazt a sémát követi, mint az élő felvétel, és most már tartalmazza a létrehozásához használt kalibrációs csomagot. Újbóli importálása **nem** eredményezi a második kalibrálást. |
| `<name>_calibrated.csv` | Spektrális besugárzás W/m²/nm-ben az érzékelő saját hullámhossz-rácsán, egy sor egy-egy leolvasáshoz, valamint fotometrikus oszlopok: teljes teljesítmény, fotopikus és skotopikus lux, PPFD kék/zöld/vörös bontással, valamint csúcshullámhossz. |

Az olyan DAQ-U vagy DAQ-M, amelynek kalibrációs csomagja nem tölthető le — mert nincs internetkapcsolat, vagy
az adott érzékelőhöz nincs kalibráció a rendszerben —, **ok megjelölésével kihagyásra kerül**, és soha nem kerül kiírásra
„kalibrált” fájlként, amely nyers számadatokat tartalmaz. Csatlakozzon az internethez, és futtassa újra a műveletet. A DAQ-E
saját kalibrációt hordoz, így erre csak akkor van szüksége, ha a készülék nincs csatlakoztatva, és
helyileg semmi sincs tárolva a gyorsítótárban.

### DAQ-A: nyers számlálási adatok, és miért ez a helyes megoldás

A **DAQ-A** a soronkénti kalibrációs csomagrendszer bevezetése előtt készült, ezért nincs letöltendő
csomagja. Ez nem figyelmetlenség: a DAQ-A-t a helyszínen kalibrálják egy
reflektancia-célpont alapján, és a célpont-alapú kalibráláshoz csak az érzékelő *relatív*
válaszára van szükség — ami pontosan a nyers számlálási eredményei. Az Chloros ma is ezekkel kalibrál.

Tehát a DAQ-A felvételei exportálhatók, de más néven:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, nem pedig `_calibrated` – egy másik fájlnév, nem pedig egy jelző a fájl belsejében,
mert az adatnak meg kell maradnia akkor is, ha a fájlt puszta névként továbbítják e-mailben. Az `.csv`
fejlécben az `raw spectral sensor counts (NOT irradiance)` szerepel, és figyelmeztet arra, hogy az értékek
**a** fájlon belül összehasonlíthatók, érzékelők között azonban nem. Azok az oszlopok, amelyek csak
a valós besugárzási intenzitás esetében jelentenek valamit — teljes teljesítmény, lux, PPFD — üresen maradnak, ahelyett, hogy
a számlálásokból számítanák ki őket.

A régebbi DAQ-A-SD-felvételek (v1.01 / v1.02 sémák) csak a fájl írási idejét rögzítik, nem pedig
olvasásonkénti időbélyeget. Az Chloros nem fogja összehasonlítani a képeket ezekkel – egy képkockát egy
írási időhöz rendelni helytelen lenne, anélkül hogy valaha is hibásnak tűnne –, de az exportálás problémamentesen olvassa be őket, és
az CSV megadja, melyik órára vonatkozik.

A teljes reflektancia-leíráshoz – egyérzékelős kamerával, valamint kettős érzékelős környezeti/objektum-érzékeléssel – lásd a [Reflektancia-munkafolyamatok](reflectance.md) című részt.
