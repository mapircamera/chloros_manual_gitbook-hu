# A feldolgozás befejezése

Miután a Chloros befejezte a feldolgozást, itt az ideje áttekinteni az eredményeket, ellenőrizni a kimeneti minőséget, és előkészíteni a feldolgozott képeket a munkafolyamatban való felhasználásra. Ez az oldal végigvezeti Önt a záró lépéseken és a következő teendőkön.

## A feldolgozás befejezésének jelzése

Amikor a feldolgozás sikeresen befejeződik, több jelzés is megjelenik:

* ✅ **Haladási sáv**: Eléri a 100%-os befejezést
* ✅ **Hibakeresési napló**: Megjeleníti az utolsó `[RUN-SUMMARY]` sorokat a számadatokkal (képek, kameracsoportok, célpontok, kalibrált képek, írt fájlok)
* ✅ **Indítás gomb**: Újra aktiválódik (kész a következő feldolgozási futtatásra)
* ✅ **Kimeneti fájlok**: Az összes feldolgozott kép elmentésre került a projekt kimeneti mappájába (lent)

{% hint style="warning" %}
**Ha a futtatás során nem kerülnek mentésre képek, az a folyamat sikertelenségét jelenti.** Ha képtermékeket kért, és a futtatás nem írt ki egyet sem, a Chloros hibaüzenetet jelez – az `[RUN-SUMMARY]` a naplónevében utal a valószínűsíthető okra (semmit sem importáltak, nem észleltek célpontot, vagy minden kért terméket kihagytak, mert nem volt alkalmazható). A CLI-megfelelője nem nulla kóddal lép ki. A szándékosan csak metaadatokra korlátozott futtatás (minden exporttermék kikapcsolva, nincs index) továbbra is sikeresnek minősül. Lásd [a CLI-referenciát](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## A feldolgozott képek megkeresése

### A kimeneti mappa megnyitása

1. Kattintson a **Főmenü** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">
ikonra (bal felső sarokban)
2. Válassza a **„Projektmappa megnyitása”** lehetőséget
3. A fájlkezelő megnyílik a projektkönyvtárban
4. Keresse meg a projektet a neve alapján

### A kimeneti fa

A termékek **a projektmappa alatt, kameránként, majd fájlformátum szerint csoportosítva** kerülnek elmentésre:

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

* **Kamera mappa**: `LATT-<sensor>-<lens>-F<filter>` a LATTICE esetében (megegyezik a felvétel EXIF-adatával: `Model`), `<model>_<filter>` a Survey3-hoz (pl. `Survey3N_RGN`). Két olyan kamera, amely ugyanazt az érzékelőt és szűrőt használja, de az objektívük eltérő, külön fa struktúrákat tart fenn — a vignettálás, a látómező és a torzítás eltérő.
* **Formátummappa**: az exportformátum-beállításnak megfelelően — `tiff16`, `tiff8`, `png8`, `jpg8`, vagy `tiff32` TIFF esetén (32 bites, százalékos). A sugárzás értéke mindig float32 típusú, és mindig az `tiff32` mappába kerül.
* **Termékmappák**:
  * `Reflectance_Calibrated_Images/` — kalibrált reflexió
  * `Debayered_Images/` — lineáris debayering (LATTICE): a sötét eltolás eltávolítva és demosaicingelve, flat field vagy unmix nélkül, az érzékelő natív tájolásában
  * `Preview_Images/` — megjelenítési előnézet (LATTICE)
  * `Radiance_Images/` — float32 spektrális sugárzási intenzitás, W/m²/sr/nm (LATTICE multispektrális)
  * `Vignette_Corrected_Images/` **vagy** `Sensor_Response_Images/` — a kalibrálatlan tartalékfájl a reflexiós referencia nélküli képkockákhoz; futtatásonként pontosan az egyik létezik, amelyet a Vignette korrekciós beállítás határoz meg
  * `<INDEX>_Index_Images/` — egy mappa a kiválasztott indexenként (pl. `NDVI_Index_Images`)
* **`clamp_masks/`** (újdonság az 1.3.0-s verzióban, kizárólag LATTICE multispektrális M3C kamerák esetén): egy formátummappában jelenik meg a termékmappák mellett, amikor a sávszétválasztás a sugárzási vagy reflexiós termékben a negatív értékeket nullára vágja. Minden érintett termékhez egy pixelenkénti maszkot tartalmaz, pl. `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif`. A maszkok soha nem találhatók a termékmappában, így a termékmappa változatlan formában átadható a fotogrammetriai szoftvernek. Lásd [Clamp maszkok](../output-image-formats.md#clamp-masks).

{% hint style="info" %}
**Minden exportált termék megtartja a FORRÁS fájl nevét.**Az `capture_..._raw.tif` sugárzási értékének exportja továbbra is `capture_..._raw.tif` néven szerepel — csupán az `tiff32/Radiance_Images/` mappában található.**A mappa azonosítja a terméket, nem a fájlnév**, ezért az `*radiance*.tif` keresése nem ad eredményt; inkább a könyvtár alapján keressen.
{% endhint %}

<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-M3C multispectral camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images, clamp_masks) and tiff32 (Radiance_Images, clamp_masks) subfolders visible -->


### Hány fájlnak kell lennie?

Ne számoljon képlet alapján — a kimenetek száma attól függ, hogy mely termékeket engedélyezték, és melyek vonatkoznak az egyes kamerákra (pl. az RGB-kamerák nem kapnak sugárzási/visszaverődési adatokat). A hiteles szám a naplóban található: az utolsó `[RUN-SUMMARY]` sor pontosan jelzi, hány fájl került írásra, a magyarázó sorok pedig elmagyarázzák, mi maradt ki.

***

## A feldolgozott képek áttekintése

### Gyors előnézet a Fájlkezelőben

**A Windows be épített előnézet:**

1. Keresse meg a termékmappát (pl. `tiff16/Reflectance_Calibrated_Images/`)
2. Válasszon ki egy képfájlt
3. Az előnézet megjelenik a Windows-fájlkezelő előnézeti ablakában
4. A nyílgombokkal lapozhat a képek között

### Előnézet külső képnézegetőkben

**Ajánlott képnézegetők:**

* **QGIS** – Ingyenes GIS-szoftver (legalkalmasabb georeferált multispektrális elemzéshez)
* **IrfanView** – Gyors, könnyű képnézegető (támogatja a TIFF formátumot)
* **Adobe Photoshop** – Professzionális képszerkesztő (TIFF támogatás)
* **GIMP** – Ingyenes alternatíva a Photoshophoz
* **Windows Photos** – Alapvető képnézegetés (lehet, hogy nem támogatja a 16 bites TIFF formátumot)

### Előnézet a Chloros képnézegetőben

Használja a Chloros beépített képnézegetőjét a fejlett megjelenítéshez:

1. Kattintson egy kép miniatűrjére a Fájlkezelőben
2. A kép megnyílik a fő előnézeti területen
3. Kattintson a bal oldali oldalsávon az **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
fülre
4. Használja az [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) funkciót interaktív elemzéshez

Részletes utasításokat az [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md) oldalon talál.

***

## A reflexiós pixelértékek kiolvasása (GIS / Pix4D / Szkriptek)

A reflexió egész számú DN-ként van tárolva, és **az a DN-érték, amely ρ = 1,0-t jelent, a forráskamerától függ**:

| Forrás          | ρ = 1,0 értéke | Hogyan lehet megállapítani                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (tartalék ρ 2,0-ig) | Az XMP-címke `Chloros:PixelScale=32768` szerepel a fájlban |
| Survey3          | **65535** (1,0-nál levágva)     | Nincs `Chloros:PixelScale` címke — ez a hiánya jelzi a helyzetet (a Survey3 termékek egyéb `Chloros:*` feldolgozási címkéket viselnek) |

**Olvassa be az `Chloros:PixelScale` címkét, és ossza el vele**, ahelyett, hogy általánosan 65535-öt feltételezne — a LATTICE-reflektancia 65535-tel történő osztása minden értéket észrevétlenül felére csökkenti. Az 1.3.0-ás verzió előtti, 8 bites formátumban írt reflektanciafájl nem lett átméretezve, hanem levágva, és nincs benne skála-címke: az osztás helyett exportálja újra 16 bites vagy 32 bites formátumban. A teljes leírásért lásd a [Kimeneti képformátumok](../output-image-formats.md) részt.

***

## Az exportált fájlokba átvitt metaadatok

Minden termék megőrzi a forrásfelvétel **GPS-blokkját**és**EXIF al-IFD-jét**, így az
exportálás során az `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` és
`CameraSerialNumber` azonosítókat, valamint a georeferenciát.

{% hint style="warning" %}
**Ha egy ortomozaj abszurd méretarányban jelenik meg, először ellenőrizze az `FocalLength` fájlt.**
A Pix4D a gyújtótávolság és a magasság alapján számítja ki a földi mintavételi távolságot. A címke hiányában
rendkívül hibás méretarányra áll vissza — egy 49 felvételből álló, lemért repülés során egy 411 m × 160 m-es
narancsültetvényt 47,8 km × 13 km-esnek rekonstruált, ami egy 455 megapixeles, többnyire
üres területet ábrázoló ortomosaikot eredményezett. A lassú mozaikozás és a váratlanul hatalmas fájlméret ennek a tünetei, nem pedig különálló
problémák.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Nem *minden* címke kerül másolásra. Az IFD0 szerkezeti címkéit szándékosan kihagyjuk (másolásuk
megrontja a LATTICE kimenetét), és az `ExifImageWidth` / `ExifImageHeight` kizárásra kerülnek,
mert az eredeti rögzítést írják le — egy átméretezett export egyébként
olyan méreteket jelölne meg, amelyek ellentmondanak a saját raszterének.

A Chloros emellett saját `Chloros:*` XMP-címkéket is ír, amelyek leírják, hogy mit végeztek az egyes termékekkel
(például `Chloros:ProcessingLevel`, `Chloros:CalibrationApplied`,
`Chloros:VignetteCorrected`). Az 1.3.0-s verziótól kezdve a Survey3-termékek minden feldolgozási
módban hordozzák őket; az indextermékek magukat az indexeket írják le (`Chloros:ProcessingLevel=index`, a
képletet, valamint azt, hogy a reflexióból vagy a sugárzásból számították-e ki), ahelyett, hogy megismételnék a
forrás nyersadatok címkéit; továbbá a 260 karakternél mélyebb mappákban található termékek is megőrzik a metaadataikat. Lásd [Kimeneti képformátumok](../output-image-formats.md#index-products) című részt.

***

## A hibakeresési napló áttekintése

### Figyelmeztetések és hibák ellenőrzése

1. Nyissa meg a **Hibakeresési napló** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">
fület
2. Görgessen végig az üzeneteken
3. Keresse meg a sárga figyelmeztetéseket vagy a piros hibákat
4. Olvassa el az `[RUN-SUMMARY]` sorokat és az esetleges tippeket
5. Forduljon a MAPIR ügyfélszolgálathoz segítségért

### A napló mentése

A feldolgozás nyomon követése vagy a MAPIR ügyfélszolgálatnak való elküldés céljából:

1. Kattintson a **„Másolás”**vagy a**„Letöltés”** gombra
2. Mentse szöveges fájlként a projektmappába
3. Csatolja a projekt dokumentációjához
4. Probléma esetén küldje el a MAPIR ügyfélszolgálatnak

Ha egy problémát közvetlenül a MAPIR-nek szeretne elküldeni, használja a Hibaelhárítási napló fül tetején található hibaikonot (**Hiba jelentése**) — lásd [Hiba jelentése a napló fülről](monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

***

## Gyakori kimeneti problémák és megoldások

### Probléma: Hiányzó kimeneti fájlok

**Lehetséges okok:**

* A termék nem alkalmazható az adott kamerára (pl. sugárzás/visszaverődés RGB-kamerák esetén — ezt a napló is jelzi)
* Hiányzott egy szükséges hivatkozás (pl. reflektancia célpont és `.daq` lefelé irányuló sugárzás nélkül)
* A termék exportálási jelölőnégyzete le volt tiltva a Projektbeállításokban
* Az exportálás közben elfogyott a lemezterület

**Megoldások:**

1. Ellenőrizze a hibakeresési naplóban az `[RUN-SUMMARY]` tippeket és az `[EXPORT-CHECK]` sorokat — ezek magyarázzák a kameránkénti kihagyásokat
2. Ellenőrizze az exportált termékre vonatkozó jelölőnégyzeteket a [Projektbeállítások](adjusting-project-settings.md) menüpontban
3. Ellenőrizze, hogy elegendő-e a lemezterület
4. A hiba okának kijavítása után futtassa újra a feldolgozást

### Probléma: Sötét vagy világos élek (a vignettálás továbbra is látható)

**Lehetséges okok:**

* A vignettálás-korrekció ki van kapcsolva
* A kamera/objektív nem szerepel a Chloros profiladatbázisban
* A vignettálás olyan mértékű, hogy a korrekció nem képes kijavítani

**Megoldások:**

1. Ellenőrizze, hogy a projektbeállításokban engedélyezve van-e a vignettálás-korrekció
2. Ellenőrizze, hogy a kamera modellje helyesen lett-e felismerve
3. Ha a vignettálás továbbra is fennáll, vegye fel a kapcsolatot a MAPIR ügyfélszolgálattal

### Probléma: Helytelen színek vagy értékek

**Lehetséges okok:**

* Nem észlelt kalibrációs célpontokat
* Helytelen kalibrációs célpont-modell választása
* A reflexiós kalibráció ki van kapcsolva
* Rossz minőségű célpontképek

**Megoldások:**

1. Ellenőrizze, hogy a reflexiós kalibráció be van-e kapcsolva
2. Ellenőrizze a „Célpont megtalálva” üzeneteket a hibakeresési naplóban
3. Ellenőrizze a célképek minőségét
4. Végezze el újra a feldolgozást a megfelelő célpontok kijelölésével

### Probléma: Az NDVI-értékek helytelennek tűnnek

**Várható NDVI-tartományok:**

* **Víz, sziklák, talaj**: -0,1 és 0,2 között
* **Ritka/egészségtelen növényzet**: 0,2 és 0,4 között
* **Közepes sűrűségű növényzet**: 0,4 és 0,6 között
* **Egészséges, sűrű növényzet**: 0,6–0,9**Ha az értékek ezen tartományokon kívül esnek:**

1. Ellenőrizze, hogy elvégezték-e a reflexiós kalibrálást
2. Ellenőrizze, hogy a fényérzékelő naplója szerepel-e a fájlban
3. Ellenőrizze, hogy a kalibrációs célpontokat felismerte-e a rendszer
4. Győződjön meg arról, hogy a rendszer a megfelelő kameramodellt azonosította
5. Ellenőrizze a célképek rögzítésének időzítését és körülményeit
6. Ha az indexeket saját maga számítja ki a reflektanciafájlokból, győződjön meg arról, hogy elosztotta a fájl `Chloros:PixelScale` értékével (lásd fent)

***

## A feldolgozott képek használata

### Fotogrammetria / ortomosaik készítése

**Ajánlott munkafolyamat:**

1.**Importálja a kalibrált reflektancia-képeket** a fotogrammetriai szoftverbe:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Az EXIF-metaadatok megőrzése**: Gondoskodjon arról, hogy a GPS-adatok megmaradjanak a geotagoláshoz
3. **Kalibrált munkafolyamatok**: Használjon reflektancia-képeket a tudományos pontosság érdekében — a LATTICE reflektancia-képek tartalmazzák azokat az XMP-kalibrációs címkéket, amelyeket a Pix4D olvas
4. **Indexmozaikok feldolgozása**: Készítsen NDVI-ortomozaikokat az egyes indexképekből
5. **GeoTIFF fájlok exportálása földrajzi hivatkozással**: GIS-alkalmazásokban való felhasználáshoz

### GIS-elemzéshez

**Ajánlott munkafolyamat:**

1.**Töltse be a QGIS-be, az ArcGIS-be vagy hasonló programba**

2.**Használjon 16 bites TIFF** reflektancia-képeket többsávos elemzéshez (ossza el a fájl `Chloros:PixelScale` értékével)
3. **Használjon indexképeket** (NDVI, NDRE) azonnal használható növényzetrétegekként
4. **Raszterkalkulátor**: Sávok kombinálása egyedi elemzéshez
5. **Exportálás**: Osztályozási térképek, változásérzékelés, növényzet-egészségügyi térképek készítése

### Közvetlen elemzéshez / jelentéskészítéshez

**Ajánlott munkafolyamat:**

1.**Használjon LUT-színekkel ellátott indexképeket** vizuális jelentésekhez
2. **Statisztikák kivonása**: Átlagos NDVI érték mezőnként/parcellánként
3. **Idősorok**: Indexek összehasonlítása több munkamenet között
4. **Jelentések készítése**: Térképek, statisztikák és vizualizációk beépítése***

## Archiválás és biztonsági mentés

### Ajánlott biztonsági mentési stratégia

**Mit kell menteni:**

* ✅ **Eredeti RAW/JPG képek vagy LATTICE nyers felvételek** – Archiválja külön meghajtóra/felhőbe; a nyers adat a feldolgozási folyamat forrása, és minden más abból újra előállítható
* ✅ **`.daq` / `.csv` fényérzékelő-fájlok** – Később a reflexió újbóli kiszámításához szükségesek
* ✅ **Feldolgozott kimenetek** – A kalibrált képeket és indexeket meg kell őrizni
* ✅ **Projektmappa** (`project.json` és kiegészítői) – Tartalmazza az újrafeldolgozáshoz szükséges összes beállítást, ha szükséges
* ✅ **Hibakeresési napló** – Dokumentálja a feldolgozás részleteit
* ✅ **Kalibrációs célképek** – Ellenőrzéshez és újrafeldolgozáshoz

**Tárolási ajánlások:**

* **Azonnali biztonsági mentés**: Külső merevlemez
* **Hosszú távú archiválás**: Felhőalapú tárhely (Google Drive, Dropbox stb.)
* **Kritikus adatok**: Tartsunk 2–3 másolatot különböző helyeken***

## Következő feldolgozási futtatások

### A projektbeállítások újrafelhasználása

Ha a jövőben hasonló adatkészleteket dolgoz fel:

1. **Mentse el a projekt sablont** (ha még nem tette meg)
2. **Hozzon létre új projektet** a mentett sablon felhasználásával
3. **Importálja az új képeket**

4.**Feldolgozza**azonos beállításokkal a konzisztencia érdekében

### Több munkamenet kötegelt feldolgozása

Több munkamenet/adatkészlet esetén:**

1. lehetőség: Grafikus felület (GUI) – Több projekt**

* Hozzon létre külön projektet minden munkamenethez
* Használjon egységes sablonbeállításokat
* Feldolgozza őket egyenként

**

2. lehetőség: Chloros CLI (csak Chloros+ esetén)**

* Automatizálja a kötegelt feldolgozást
* Több mappát dolgozzon fel szkriptekkel
* Lásd a [CLI-dokumentációt](../CLI.md) és a [CLI-referenciát](../reference/cli-reference.md)

**

3. lehetőség: Python SDK (csak Chloros+)**

* Programozási vezérlés
* Integráció elemzési folyamatokba
* Lásd az [API-dokumentációt](../api-python-sdk.md) és az [SDK-referenciát](../reference/sdk-reference.md)

***

## Hibaelhárítás az utómunkálatok során

### Újrafeldolgozás eltérő beállításokkal

Ha az eredmények nem kielégítőek:

1. Őrizze meg az eredeti képeket (soha ne törölje őket)
2. Nyissa meg ugyanazt a projektet a Chloros-ban 3. Állítsa be a beállításokat a Projektbeállítások panelen
4. Végezze el újra a feldolgozást — a kimeneti fájlok ugyanabba a termékmappába kerülnek, így az előző futtatásból származó azonos nevű fájlok felülíródnak

### Képek egy részhalmazának feldolgozása

Ha csak bizonyos képeket szeretne újra feldolgozni:

1. Hozzon létre egy új projektet
2. Csak azokat a képeket importálja, amelyeket újra kell feldolgozni
3. Használja ugyanazt a beállítási sablont
4. Dolgozza fel a kisebb adathalmazt

### Segítségkérés

Ha problémákba ütközik:

* 📧 **E-mail**: info@mapir.camera (csatolja a hibakeresési naplót)
* 🌐 **Támogatás**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **GYIK**: [Gyakran feltett kérdések](../faq.md)
* 📖 **Dokumentáció**: [Chloros-kézikönyv](../)

***

## Összefoglalás: Teljes munkafolyamat

Most már végigjártad a teljes Chloros-feldolgozási munkafolyamatot:

1. ✅ **Projekt létrehozása** – Lásd: [Projektek](../projects.md)
2. ✅ **Fájlok hozzáadása** – Lásd: [Fájlok hozzáadása](adding-files-to-a-project.md)
3. ✅ **Beállítások módosítása** – Lásd: [A projektbeállítások módosítása](adjusting-project-settings.md)
4. ✅ **Célok kijelölése** – Lásd: [Célképek kiválasztása](choosing-target-images.md)
5. ✅ **Feldolgozás elindítása** – Lásd: [A feldolgozás elindítása](starting-the-processing.md)
6. ✅ **A folyamat nyomon követése** – Lásd: [A feldolgozás nyomon követése](monitoring-the-processing.md)
7. ✅ **Eredmények áttekintése** – Ez az oldal**A kalibrált, reflexióval korrigált multispektrális képei készen állnak az elemzésre!**

***

## További források

### Speciális funkciók

* [**Képnézegető**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktív megjelenítés és elemzés
* [**Index/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) – Egyéni indexek tesztelése
* [**Multispektrális indexképletek**](../project-settings/multispectral-index-formulas.md) – Teljes index-referencia

### Automatizálás és integráció

* [**CLI-dokumentáció**](../CLI.md) – Parancssori kötegelt feldolgozás
* [**Python SDK**](../api-python-sdk.md) – Programozási automatizálás
* [**A Chloros+ funkciói**](../#chloros) – Fejlett feldolgozási lehetőségek

### Támogatás és oktatás

* [**GYIK**](../faq.md) – Gyakori kérdések és válaszok
* [**Kalibrációs célpontok**](../calibration-targets.md) – A reflexiós kalibráció megértése
* [**Támogatott kamerák**](../supported-cameras.md) – Kompatibilis hardverek
