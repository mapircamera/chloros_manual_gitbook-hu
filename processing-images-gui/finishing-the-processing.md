# A feldolgozás befejezése

Miután az Chloros befejezte a feldolgozást, itt az ideje áttekinteni az eredményeket, ellenőrizni a kimeneti minőséget, és előkészíteni a feldolgozott képeket a munkafolyamatban való felhasználásra. Ez az oldal végigvezeti Önt a végső lépéseken és a következő teendőken.

## A feldolgozás befejezésének jelzése

A feldolgozás sikeres befejezése után több jelző is megjelenik:

* ✅ **Haladási sáv**: Eléri a 100%-os befejezést
* ✅ **Hibakeresési napló**: Megjelenik a „Feldolgozás befejezve” üzenet
* ✅ **Indítás gomb**: Ismét engedélyezetté válik (kész a következő feldolgozási futtatásra)
* ✅ **Kimeneti fájlok**: Az összes feldolgozott kép a kamera modell alkönyvtárába kerül

***

## A feldolgozott képek megkeresése

### A kimeneti mappa megnyitása

1. Kattintson a **Főmenü** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> ikonra (bal felső sarokban)
2. Válassza a **„Projektmappa megnyitása”** lehetőséget
3. A fájlkezelő megnyitja a projekt könyvtárát
4. Keresse meg a projektet a neve alapján

***

## A feldolgozott képek áttekintése

### Gyors előnézet a fájlkezelőben

**Windows beépített előnézet:**

1. Keresse meg a kamera modell alkönyvtárát
2. Válasszon ki egy képfájlt
3. Az előnézet megjelenik az Windows Explorer előnézeti ablakában
4. A nyílgombokkal böngészhet a képek között

### Előnézet külső képnézegetőkben

**Ajánlott nézegetők:**

* **QGIS** – Ingyenes GIS szoftver (legalkalmasabb georeferált multispektrális elemzéshez)
* **IrfanView** – Gyors, könnyű képnézegető (támogatja az TIFF-et)
* **Adobe Photoshop** – professzionális szerkesztés (TIFF támogatás)
* **GIMP** – ingyenes alternatíva a Photoshophoz
* **Windows Photos** – alapvető megtekintés (lehet, hogy nem támogatja a 16 bites TIFF-et)

### Előnézet az Chloros Képmegjelenítőben

Használja az Chloros beépített Képmegjelenítőjét a fejlett vizualizáláshoz:

1. Kattintson egy kép miniatűrjére a Fájlböngészőben.
2. A kép megnyílik a fő előnézeti területen.
3. Kattintson a **Képmegjelenítő** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> fülre a bal oldali sávban.
4. Használja az [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) funkciót az interaktív elemzéshez.

Részletes utasításokat az [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md) oldalon talál.

***

## A hibakeresési napló áttekintése

### Figyeljen a figyelmeztetésekre és hibákra

1. Nyissa meg a **Hibaelhárítási napló** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> fület
2. Görgessen végig az üzeneteken
3. Keresse meg a sárga figyelmeztetéseket vagy a piros hibákat
4. Tekintse át a megjegyzett problémákat
5. Forduljon az MAPIR támogatáshoz segítségért

### A napló mentése

A feldolgozás nyilvántartása vagy az MAPIR támogatáshoz való elküldéshez:

1. Kattintson a **„Másolás”** vagy **„Letöltés”** gombra.
2. Mentse szövegfájlként a projektmappába.
3. Csatolja a projekt dokumentációjához.
4. Problémák esetén küldje el az MAPIR támogatásnak.

***

## Gyakori kimeneti problémák és megoldások

### Probléma: Hiányzó kimeneti fájlok

**Lehetséges okok:**

* A fájlok nem feleltek meg a feldolgozási kritériumoknak.
* Csak célképek (kizárva az exportálásból).
* Az exportálás során elfogyott a lemezterület.
* A fájlok megsérültek a feldolgozás során.

**Megoldások:**

1. Ellenőrizze a hibakeresési naplót ugrási/hibajelentésekre.
2. Ellenőrizze, hogy elegendő lemezterület állt-e rendelkezésre.
3. Számolja meg a fájlokat: Meg kell egyeznie (eredeti szám – célszám) × (indexek + 1)
4. Importálja újra és dolgozza fel újra a hiányzó fájlokat.

### Probléma: Sötét vagy világos élek (a vignettálás továbbra is látható)

**Lehetséges okok:**

* A vignettálás-korrekció ki van kapcsolva.
* A kamera/objektív nincs az Chloros profil adatbázisban.
* A vignettálás olyan mértékű, hogy a korrekció nem képes kijavítani.

**Megoldások:**

1. Ellenőrizze, hogy a vignettázás korrekció engedélyezve van-e a Projektbeállításokban.
2. Ellenőrizze, hogy a kamera modellje helyesen lett-e felismerve.
3. Ha a vignettázás továbbra is fennáll, vegye fel a kapcsolatot az MAPIR ügyfélszolgálattal.

### Probléma: Helytelen színek vagy értékek

**Lehetséges okok:**

* Nincs kalibrációs célpont felismerve.
* Helytelen kalibrációs célpont modell lett kiválasztva.
* A reflektancia kalibráció le van tiltva.
* Rossz minőségű célképek.

**Megoldások:**

1. Ellenőrizze, hogy a reflektancia kalibrálás engedélyezve van-e.
2. Ellenőrizze a „Célpont megtalálva” üzeneteket a hibakeresési naplóban.
3. Ellenőrizze a célképek minőségét.
4. Ismételje meg a feldolgozást a megfelelő célpontok megjelölésével.

### Probléma: Az NDVI értékek helytelennek tűnnek.

**Várható NDVI tartományok:**

* **Víz, sziklák, talaj**: -0,1 és 0,2 között
* **Ritka/egészségtelen növényzet**: 0,2 és 0,4 között
* **Közepes növényzet**: 0,4 és 0,6 között
* **Egészséges, sűrű növényzet**: 0,6 és 0,9 között

**Ha az értékek nem esnek ezekbe a tartományokba:**

1. Ellenőrizze, hogy a reflektancia kalibrálása megtörtént-e.
2. Ellenőrizze, hogy a fényérzékelő naplója szerepel-e.
3. Ellenőrizze, hogy a kalibrációs célpontok felismerésre kerültek-e.
4. Győződjön meg arról, hogy a megfelelő kameramodell lett felismerve.
5. Ellenőrizze a célképek rögzítésének időzítését és feltételeit.

***

## A feldolgozott képek használata

### Fotogrammetria / ortomosaik készítéshez

**Ajánlott munkafolyamat:**

1. **Importálja a kalibrált visszaverődési képeket** a fotogrammetriai szoftverbe:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Tartsa meg az EXIF metaadatokat**: Győződjön meg arról, hogy a GPS-adatok megmaradtak a földrajzi címkézéshez.
3. **Kalibrált munkafolyamatok**: Használjon visszaverődési képeket a tudományos pontosság érdekében.
4. **Indexmozaikok feldolgozása**: Készítsen NDVI ortomosaikokat az egyes indexképekből
5. **Exportálja a georeferált GeoTIFF**-et: GIS-alkalmazásokban való felhasználáshoz

### GIS-elemzéshez

**Ajánlott munkafolyamat:**

1. **Töltse be a QGIS, ArcGIS vagy hasonló programba**
2. **Használjon 16 bites TIFF** reflektancia képeket többsávos elemzéshez
3. **Használjon index képeket** (NDVI, NDRE) használatra kész növényzet rétegekként
4. **Raszter számológép**: Kombinálja a sávokat egyéni elemzéshez
5. **Exportálás**: osztályozási térképek, változásérzékelés, növényzet-egészségügyi térképek létrehozása

### Közvetlen elemzéshez / jelentéskészítéshez

**Ajánlott munkafolyamat:**

1. **Használjon indexképeket LUT színekkel** vizuális jelentésekhez
2. **Statisztikák kivonása**: Átlagos NDVI mezőnként/parcellánként
3. **Idősorok**: indexek összehasonlítása több munkamenet között
4. **Jelentések létrehozása**: térképek, statisztikák és vizualizációk beépítése

***

## Archiválás és biztonsági mentés

### Ajánlott biztonsági mentési stratégia

**Mit kell menteni:**

* ✅ **Eredeti RAW/JPG képek** – archiválás külön meghajtón/felhőben
* ✅ **Feldolgozott kimenetek** – Kalibrált képek és indexek megőrzése
* ✅ **Projektfájl** – Szükség esetén tartalmazza az újrafeldolgozáshoz szükséges összes beállítást
* ✅ **Hibakeresési napló** – Dokumentálja a feldolgozás részleteit
* ✅ **Kalibrációs célképek** – Ellenőrzés és újrafeldolgozás céljára

**Tárolási ajánlások:**

* **Azonnali biztonsági mentés**: Külső merevlemez
* **Hosszú távú archívum**: Felhőalapú tároló (Google Drive, Dropbox stb.)
* **Kritikus adatok**: 2-3 másolatot tároljon különböző helyeken

***

## Következő feldolgozási futtatások

### A projektbeállítások újrahasznosítása

Ha a jövőben hasonló adatkészleteket dolgoz fel:

1. **Projekt sablon mentése** (ha még nem tette meg)
2. **Új projekt létrehozása** a mentett sablon felhasználásával
3. **Új képek importálása**
4. **Feldolgozás** azonos beállításokkal a konzisztencia érdekében

### Több munkamenet kötegelt feldolgozása

Több munkamenet/adatkészlet esetén:

**1. lehetőség: GUI – Több projekt**

* Hozzon létre külön projektet minden munkamenethez
* Használjon konzisztens sablonbeállításokat
* Feldolgozás egyenként

**2. lehetőség: Chloros CLI (csak Chloros+)**

* Automatizálja a kötegelt feldolgozást
* Feldolgozás több mappában szkriptekkel
* Lásd [CLI dokumentáció](../CLI.md)

**3. lehetőség: Python SDK (csak Chloros+)**

* Programozási vezérlés
* Integráció az elemzési folyamatokkal
* Lásd [API dokumentáció](../api-python-sdk.md)

***

## Hibaelhárítás Utómunka

### Újrafeldolgozás különböző beállításokkal

Ha az eredmények nem kielégítőek:

1. Tartsa meg az eredeti képeket (soha ne törölje őket)
2. Nyissa meg ugyanazt a projektet az Chloros programban
3. Állítsa be a beállításokat a Projektbeállítások panelen
4. Futtassa újra a feldolgozást – az eredmények felülírják a korábbi eredményeket

### Képek részhalmazának feldolgozása

Csak bizonyos képek újrafeldolgozásához:

1. Hozzon létre egy új projektet
2. Csak az újrafeldolgozásra szoruló képeket importálja
3. Használja ugyanazt a beállítási sablont
4. Feldolgozza a kisebb adathalmazt

### Segítség

Ha problémákba ütközik:

* 📧 **E-mail**: info@mapir.camera (tartalmazza a hibakeresési naplót)
* 🌐 **Támogatás**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **GYIK**: [Gyakran ismételt kérdések](../faq.md)
* 📖 **Dokumentáció**: [Chloros kézikönyv](../)

***

## Összefoglalás: Teljes munkafolyamat

Most már teljesítette a teljes Chloros feldolgozási munkafolyamatot:

1. ✅ **Létrehozott projekt** - Lásd [Projektek](../projects.md)
2. ✅ **Fájlok hozzáadása** – Lásd [Fájlok hozzáadása](adding-files-to-a-project.md)
3. ✅ **Beállítások módosítása** – Lásd [Projektbeállítások módosítása](adjusting-project-settings.md)
4. ✅ **Célok megjelölése** - Lásd [Célképek kiválasztása](choosing-target-images.md)
5. ✅ **Feldolgozás elindítása** - Lásd [A feldolgozás elindítása](starting-the-processing.md)
6. ✅ **Figyelt előrehaladás** - Lásd [A feldolgozás figyelése](monitoring-the-processing.md)
7. ✅ **Ellenőrzött eredmények** - Ez az oldal

**A kalibrált, reflektancia-korrigált multispektrális képei készen állnak az elemzésre!**

***

## További források

### Speciális funkciók

* [**Képmegjelenítő**](../image-viewer-gui/opening-an-image-full-screen.md) – Interaktív megjelenítés és elemzés
* [**Index/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) – Egyéni index tesztelés
* [**Multispektrális index képletek**](../project-settings/multispectral-index-formulas.md) - Teljes index referencia

### Automatizálás és integráció

* [**CLI dokumentáció**](../CLI.md) - Parancssori kötegelt feldolgozás
* [**Python SDK**](../api-python-sdk.md) – Programozási automatizálás
* [**Chloros+ funkciók**](../#chloros) – Fejlett feldolgozási képességek

### Támogatás és tanulás

* [**GYIK**](../faq.md) – Gyakori kérdések és válaszok
* [**Kalibrációs célok**](../calibration-targets.md) – A visszaverődés kalibrálásának megértése
* [**Támogatott kamerák**](../supported-cameras.md) – Kompatibilis hardver
