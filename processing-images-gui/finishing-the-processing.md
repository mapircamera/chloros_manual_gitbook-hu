# A feldolgozás befejezése

Miután az Chloros befejezte a feldolgozást, itt az ideje, hogy áttekintse az eredményeket, ellenőrizze a kimeneti minőséget, és előkészítse a feldolgozott képeket a munkafolyamatban való felhasználásra. Ez az oldal végigvezeti Önt a záró lépéseken és a következő teendőkön.

## A feldolgozás befejezésének jelzése

Amikor a feldolgozás sikeresen befejeződik, több jelzés is megjelenik:

* ✅ **Haladási sáv**: Eléri a 100%-os befejezést
* ✅ **Hibakeresési napló**: Megjeleníti az `[RUN-SUMMARY]` végső sorát a számadatokkal együtt (képek, kameracsoportok, célpontok, kalibrált képek, írt fájlok)
* ✅ **Indítás gomb**: Újra aktiválódik (kész a következő feldolgozási futtatásra)
* ✅ **Kimeneti fájlok**: Az összes feldolgozott kép elmentésre került a projekt kimeneti mappájába (lent)

{% hint style="warning" %}
**Az a futtatás, amely nem ír ki képeket, sikertelennek minősül.** Ha képtermékeket kért, és a futtatás nem írt ki egyet sem, az Chloros hibaüzenetet jelez — az `[RUN-SUMMARY]` a napló nevében utal a valószínűsíthető okra (nem importáltak semmit, nem észleltek célt, vagy minden kért terméket kihagytak, mivel nem volt alkalmazható). Az CLI megfelelője nem nulla kóddal lép ki. A szándékosan kizárólag metaadatokra irányuló futtatás (minden exportált termék kikapcsolva, nincs index) továbbra is sikeresnek minősül. Lásd [az CLI referencia](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## A feldolgozott képek megkeresése

### A kimeneti mappa megnyitása

1. Kattintson a **Főmenü** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> ikonjára (bal felső sarokban)
2. Válassza a **„Projektmappa megnyitása”** lehetőséget
3. A fájlkezelő a projektkönyvtárban nyílik meg
4. Keresse meg a projektet a neve alapján

### A kimeneti fa

A termékek **a projektmappa alatt, kameránként, majd fájlformátum szerint csoportosítva** kerülnek mentésre:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Kamera mappa**: `LATT-<sensor>-<lens>-F<filter>` a LATTICE esetében (megegyezik a felvétel EXIF-adatával: `Model`), `<model>_<filter>` az Survey3 esetében (pl. `Survey3N_RGN`). Két olyan kamera, amely ugyanazt az érzékelőt és szűrőt használja, de objektívük eltérő, külön fa struktúrákat kap — a vignettálás, a látómező és a torzítás eltérő.
* **Formátummappa**: az exportformátum-beállításnak megfelelően — `tiff16`, `tiff8`, `png8`, `jpg8`, vagy `tiff32` az TIFF esetében (32 bites, százalékos). A Radiance mindig float32 típusú, és mindig az `tiff32` alatt található.
* **Termékmappák**:
  * `Reflectance_Calibrated_Images/` — kalibrált reflexió
  * `Debayered_Images/` — lineáris debayering (LATTICE)
  * `Preview_Images/` — képernyőn megjelenő előnézet (LATTICE)
  * `Radiance_Images/` — float32 spektrális sugárzási érték, W/m²/sr/nm (LATTICE multispektrális)
  * `Vignette_Corrected_Images/` **vagy** `Sensor_Response_Images/` — a nem kalibrált tartalék megoldás olyan képkockákhoz, amelyeknél nincs reflektancia-referencia; futásonként pontosan az egyik létezik, amelyet a Vignette korrekciós beállítás határoz meg
  * `<INDEX>_Index_Images/` — egy mappa a kiválasztott indexenként (pl. `NDVI_Index_Images`)

{% hint style="info" %}
**Minden exportált termék megtartja a FORRÁS fájl nevét.**Az `capture_..._raw.tif` sugárzási exportja továbbra is `capture_..._raw.tif` néven szerepel — csupán az `tiff32/Radiance_Images/` mappában található.**A terméket a mappa azonosítja, nem a fájlnév**, ezért az `*radiance*.tif` keresése nem ad eredményt; helyette a könyvtárat keresse.
{% endhint %}



<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-… camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images) and tiff32 (Radiance_Images) subfolders visible -->### Hány fájlnak kell lennie?

Ne számoljon képlet alapján — a kimeneti fájlok száma attól függ, hogy mely termékek voltak engedélyezve, és melyek vonatkoznak az egyes kamerákra (pl. az RGB kamerák nem kapnak sugárzási/visszaverődési adatokat). A hiteles szám a naplóban található: az utolsó `[RUN-SUMMARY]` sor pontosan jelzi, hány fájl került írásra, a magyarázó sorok pedig elmagyarázzák, mi maradt ki.

***

## A feldolgozott képek áttekintése

### Gyors előnézet a Fájlkezelőben

**Windows beépített előnézet:**

1. Keresse meg a termékmappát (pl. `tiff16/Reflectance_Calibrated_Images/`)
2. Válasszon ki egy képfájlt
3. Az előnézet megjelenik az Windows Explorer előnézeti ablakában
4. A nyílgombokkal lapozhat a képek között

### Előnézet külső képnézegetőkben

**Ajánlott képnézegetők:*** **QGIS** – Ingyenes GIS-szoftver (legalkalmasabb georeferált multispektrális elemzéshez)
* **IrfanView** – Gyors, könnyű képnézegető (támogatja az TIFF formátumot)
* **Adobe Photoshop** – Professzionális képszerkesztés (TIFF támogatás)
* **GIMP** – Ingyenes alternatíva a Photoshophoz
* **Windows Photos** – Alapvető képnézegetés (lehet, hogy nem támogatja a 16 bites TIFF formátumot)

### Előnézet az Chloros képnézegetőben

Használja az Chloros beépített Képnézegetőjét a fejlett megjelenítéshez:

1. Kattintson egy kép miniatűrjére a Fájlkezelőben
2. A kép megnyílik a fő előnézeti területen
3. Kattintson a bal oldali oldalsávban az **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> fülre
4. Az interaktív elemzéshez használja az [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) funkciót

Részletes útmutatást az [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md) oldalon talál.

***

## A reflexiós pixelértékek kiolvasása (GIS / Pix4D / szkriptek)

A reflexió egész számú DN-ként van tárolva, és **az a DN-érték, amely ρ = 1,0-t jelent, a forráskamerától függ**:

| Forrás          | ρ = 1,0 értéke | Hogyan lehet megállapítani                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (tartalék ρ 2,0-ig) | Az `Chloros:PixelScale=32768` XMP-címke szerepel a fájlban |
| Survey3         | **65535** (ρ 1,0-nál levágva)     | Nincs `Chloros:*` XMP-címke — ez a hiány jelzi |

**Olvassa be az `Chloros:PixelScale` címkét, és ossza el vele**, ahelyett, hogy általánosan 65535-öt feltételezne – a LATTICE-reflektancia 65535-tel történő osztása minden értéket észrevétlenül felére csökkenti. Egy szélsőséges eset tervezés szerint nem tartalmaz skálázási értéket: egy 8 bites forrásból származó, 8 bites kimenetként írt felvétel levágásra kerül, nem pedig újraskálázásra, és szándékosan nem kap skálázási címkét — az osztás helyett inkább 16 bites vagy 32 bites formátumban exportálja újra. A teljes leírásért lásd [Kimeneti képformátumok](../output-image-formats.md).***

## Az exportált fájlokba átvitt metaadatok

Minden termék megőrzi a forrásfelvétel **GPS-blokkját**és**EXIF al-IFD-jét**, így az
export tartalmazza az `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` és
`CameraSerialNumber` kódokat, valamint a georeferenciálást.

{% hint style="warning" %}
**Ha egy ortomosaik abszurd méretarányú eredményt ad, először ellenőrizze az `FocalLength` értéket.**
A Pix4D a gyújtótávolság és a magasság alapján számítja ki a földi mintavételi távolságot. A címke hiányában
a rendszer egy teljesen helytelen méretarányra tér vissza — egy 49 felvételből álló repülés során egy 411 m × 160 m-es
narancsligetet 47,8 km × 13 km-esnek rekonstruált, ami egy 455 megapixeles ortofotót eredményezett, amelyen többnyire
üres tér látható. A lassú mozaikozás és a váratlanul hatalmas fájlméret ennek a tünetei, nem pedig különálló
problémák.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Nem *minden* címke kerül másolásra. Az IFD0 szerkezeti címkéit szándékosan kihagyjuk (másolásuk
megrontja a LATTICE kimenetét), az `ExifImageWidth` / `ExifImageHeight` címkék pedig kizárásra kerülnek,
mert az eredeti felvételt írják le – egy átméretezett export egyébként olyan
méreteket jelölne meg, amelyek ellentmondanak a saját raszterének.

***

## A hibakeresési napló áttekintése

### Figyelmeztetések vagy hibák ellenőrzése

1. Nyissa meg a **Hibakeresési napló** „<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">” fület
2. Görgessen végig az üzeneteken
3. Keresse meg a sárga figyelmeztetéseket vagy a piros hibákat
4. Olvassa el az `[RUN-SUMMARY]` sorokat és az esetleges tippeket
5. Forduljon az MAPIR ügyfélszolgálathoz segítségért

### A napló mentése

A feldolgozás nyilvántartása vagy az MAPIR ügyfélszolgálatnak való elküldés érdekében:

1. Kattintson a **„Másolás”**vagy a**„Letöltés”** gombra
2. Mentse szöveges fájlként a projektmappába
3. Csatolja a projekt dokumentációjához
4. Problémák esetén küldje el az MAPIR ügyfélszolgálatnak

***

## Gyakori kimeneti problémák és megoldások

### Probléma: Hiányzó kimeneti fájlok

**Lehetséges okok:**

* A termék nem alkalmazható az adott kamerára (pl. sugárzás/visszaverődés RGB kamerák esetén — ezt a napló is jelzi)
* Hiányzott egy szükséges referencia (pl. reflektancia célpont és `.daq` lefelé irányuló sugárzás nélkül)
* A termék exportálására szolgáló jelölőnégyzet le volt tiltva a Projektbeállításokban
* Az exportálás közben elfogyott a lemezterület

**Megoldások:**

1. Ellenőrizze a hibakeresési naplóban az `[RUN-SUMMARY]` tippeket és az `[EXPORT-CHECK]` sorokat — ezek magyarázzák a kameránkénti kihagyásokat
2. Ellenőrizze az exportált termék jelölőnégyzeteit a [Projektbeállítások](adjusting-project-settings.md) menüpontban
3. Ellenőrizze, hogy elegendő-e a lemezterület
4. A hiba okának kijavítása után futtassa újra a feldolgozást

### Probléma: Sötét vagy világos szélek (a vignettálás továbbra is látható)

**Lehetséges okok:**

* A vignettakorrekció ki van kapcsolva
* A kamera/objektív nem szerepel az Chloros profiladatbázisban
* A vignettálás olyan mértékű, hogy a korrekció nem képes kezelni

**Megoldások:**

1. Ellenőrizze, hogy a projektbeállításokban engedélyezve van-e a vignettálás-korrekció
2. Ellenőrizze, hogy a kamera modelljét helyesen ismerte-e fel a rendszer
3. Ha a vignettálás továbbra is fennáll, vegye fel a kapcsolatot az MAPIR ügyfélszolgálattal

### Probléma: Helytelen színek vagy értékek

**Lehetséges okok:**

* Nem észlelt kalibrációs célpontokat
* Helytelen kalibrációs célpontmodell választása
* A fényvisszaverődési kalibráció le van tiltva
* Rossz minőségű célpontképek

**Megoldások:**

1. Ellenőrizze, hogy a fényvisszaverődési kalibráció engedélyezve van-e
2. Ellenőrizze a „Célpont megtalálva” üzeneteket a hibakeresési naplóban
3. Ellenőrizze a célpontképek minőségét
4. Végezzen új feldolgozást a megfelelő célpontok kijelölésével

### Probléma: Az NDVI értékek helytelennek tűnnek

**Várható NDVI tartományok:*** **Víz, sziklák, talaj**: -0,1 és 0,2 között
* **Ritka/rossz állapotú növényzet**: 0,2–0,4
* **Közepes növényzet**: 0,4–0,6
* **Egészséges, sűrű növényzet**: 0,6–0,9**Ha az értékek ezen tartományokon kívül esnek:**

1. Ellenőrizze, hogy elvégezték-e a reflexiós kalibrálást
2. Ellenőrizze, hogy a fényérzékelő naplófájlja szerepel-e a csomagban
3. Ellenőrizze, hogy a kalibrációs célpontokat felismerte-e a rendszer
4. Győződjön meg arról, hogy a rendszer a megfelelő kameramodellt azonosította
5. Ellenőrizze a célképek rögzítésének időzítését és körülményeit
6. Ha az indexeket saját maga számítja ki a reflektancia-fájlokból, ellenőrizze, hogy elosztotta-e a fájl `Chloros:PixelScale` értékével (lásd fent)

***

## A feldolgozott képek használata

### Fotogrammetria / ortomosaik készítéséhez

**Ajánlott munkafolyamat:**

1.**Importálja a kalibrált reflektancia-képeket** a fotogrammetriai szoftverbe:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Az EXIF-metaadatok megőrzése**: Gondoskodjon arról, hogy a GPS-adatok megmaradjanak a geotagoláshoz
3. **Kalibrált munkafolyamatok**: A tudományos pontosság érdekében használjon reflektancia-képeket — a LATTICE reflektancia-képek tartalmazzák azokat az XMP-kalibrációs címkéket, amelyeket a Pix4D olvas
4. **Indexmozaikok feldolgozása**: Készítsen NDVI ortomozaikokat az egyes indexképekből
5. **Georeferált GeoTIFF exportálása**: GIS-alkalmazásokban való felhasználáshoz

### GIS-elemzéshez

**Ajánlott munkafolyamat:**

1.**Töltse be a QGIS-be, ArcGIS-be vagy hasonló programba**

2.**Használjon 16 bites TIFF** reflektancia-képeket a többsávos elemzéshez (osztva a fájl `Chloros:PixelScale` értékével)
3. **Használja az indexképeket** (NDVI, NDRE) azonnal használható növényzetrétegekként
4. **Raszterkalkulátor**: Sávok egyesítése egyedi elemzéshez
5. **Exportálás**: Osztályozási térképek, változásérzékelés, növényzet-egészségügyi térképek készítése

### Közvetlen elemzéshez / jelentéskészítéshez

**Ajánlott munkafolyamat:**

1.**Használjon indexképeket LUT-színekkel** a vizuális jelentésekhez
2. **Statisztikák kinyerése**: Átlagos NDVI érték mezőnként/parcellánként
3. **Idősorok**: Az indexek összehasonlítása több mérési alkalom között
4. **Jelentések készítése**: Térképek, statisztikák és vizualizációk beépítése***

## Archiválás és biztonsági mentés

### Ajánlott biztonsági mentési stratégia

**Mit kell menteni:*** ✅ **Eredeti RAW/JPG képek vagy LATTICE nyers felvételek** – Arkiválja külön meghajtóra/felhőbe; a nyers adat a feldolgozási folyamat forrása, és minden más abból újra előállítható
* ✅ **`.daq` / `.csv` fényérzékelő-fájlok** – Később a reflexió újbóli kiszámításához szükségesek
* ✅ **Feldolgozott kimenetek** – A kalibrált képeket és indexeket meg kell őrizni
* ✅ **Projektmappa** (`project.json` és kiegészítői) – Tartalmazza az újrafeldolgozáshoz szükséges összes beállítást
* ✅ **Hibakeresési napló** – A feldolgozás részleteit dokumentálja
* ✅ **Kalibrációs célképek** – Ellenőrzéshez és újrafeldolgozáshoz**Tárolási ajánlások:*** **Azonnali biztonsági másolat**: Külső merevlemez
* **Hosszú távú archiválás**: Felhőalapú tárhely (Google Drive, Dropbox stb.)
* **Kritikus adatok**: Tartsunk 2–3 másolatot különböző helyeken***

## Következő feldolgozási futtatások

### A projektbeállítások újrafelhasználása

Ha a jövőben hasonló adatkészleteket dolgoz fel:

1. **Mentse el a projekt sablont** (ha még nem tette meg)
2. **Hozzon létre új projektet** a mentett sablon felhasználásával
3. **Új képek importálása**

4.**Feldolgozás**az egységesség érdekében azonos beállításokkal

### Több munkamenet kötegelt feldolgozása

Több munkamenet/adatkészlet esetén:**

1. lehetőség: Grafikus felhasználói felület (GUI) – Több projekt**

* Minden munkamenethez külön projektet kell létrehozni
* Egységes sablonbeállításokat kell használni
* Egyszerre egyet kell feldolgozni

**

2. lehetőség: Chloros CLI (csak Chloros+ esetén)**

* Automatizálja a kötegelt feldolgozást
* Több mappát dolgozzon fel szkriptekkel
* Lásd az [CLI dokumentációt](../CLI.md) és az [CLI referenciaanyagot](../reference/cli-reference.md)

**

3. lehetőség: Python SDK (csak Chloros+ esetén)**

* Programozott vezérlés
* Integráció az elemzési folyamatokba
* Lásd az [API dokumentációt](../api-python-sdk.md) és az [SDK referenciaanyagot](../reference/sdk-reference.md)

***

## Hibaelhárítás az utómunkálatok során

### Újrafeldolgozás eltérő beállításokkal

Ha az eredmények nem kielégítőek:

1. Őrizze meg az eredeti képeket (soha ne törölje őket)
2. Nyissa meg ugyanazt a projektet az Chloros-ben
3. Állítsa be a beállításokat a Projektbeállítások panelen
4. Futtassa újra a feldolgozást — a kimeneti fájlok ugyanabba a termékmappába kerülnek, így az előző futtatásból származó azonos nevű fájlok felülíródnak

### Képek egy részhalmazának feldolgozása

Ha csak bizonyos képeket szeretne újra feldolgozni:

1. Hozzon létre egy új projektet
2. Csak azokat a képeket importálja, amelyeket újra kell feldolgozni
3. Használja ugyanazt a beállítási sablont
4. Dolgozza fel a kisebb adatkészletet

### Segítségkérés

Ha problémákba ütközik:

* 📧 **E-mail**: info@mapir.camera (csatolja a hibakeresési naplót)
* 🌐 **Támogatás**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **GYIK**: [Gyakran feltett kérdések](../faq.md)
* 📖 **Dokumentáció**: [Chloros kézikönyv](../)***

## Összefoglalás: A teljes munkafolyamat

Most már teljesítette az Chloros feldolgozási munkafolyamatot:

1. ✅ **Projekt létrehozása** – Lásd: [Projektek](../projects.md)
2. ✅ **Fájlok hozzáadása** – Lásd: [Fájlok hozzáadása](adding-files-to-a-project.md)
3. ✅ **Beállítások módosítása** – Lásd: [A projekt beállításainak módosítása](adjusting-project-settings.md)
4. ✅ **Célok kijelölése** – Lásd: [Célképek kiválasztása](choosing-target-images.md)
5. ✅ **Feldolgozás elindítása** – Lásd: [A feldolgozás elindítása](starting-the-processing.md)
6. ✅ **A feldolgozás nyomon követése** – Lásd: [A feldolgozás nyomon követése](monitoring-the-processing.md)
7. ✅ **Az eredmények áttekintése** – Ez az oldal**A kalibrált, reflexióval korrigált multispektrális képei készen állnak az elemzésre!**

***

## További források

### Speciális funkciók

* [**Képnézegető**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktív megjelenítés és elemzés
* [**Index/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) – Egyéni indexek tesztelése
* [**Multispektrális indexképletek**](../project-settings/multispectral-index-formulas.md) – Teljes index-referencia

### Automatizálás és integráció

* [**CLI dokumentáció**](../CLI.md) – Parancssori kötegelt feldolgozás
* [**Python SDK**](../api-python-sdk.md) - Programozási automatizálás
* [**Chloros+ Funkciók**](../#chloros) - Fejlett feldolgozási képességek

### Támogatás és tanulás

* [**GYIK**](../faq.md) – Gyakori kérdések és válaszok
* [**Kalibrációs célpontok**](../calibration-targets.md) – A reflexiós kalibrálás megértése
* [**Támogatott kamerák**](../supported-cameras.md) – Kompatibilis hardverek
