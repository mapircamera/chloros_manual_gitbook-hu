# A feldolgozás befejezése

Miután az Chloros befejezte a feldolgozást, itt az ideje áttekinteni az eredményeket, ellenőrizni a kimeneti minőséget, és előkészíteni a feldolgozott képeket a munkafolyamatban való felhasználásra. Ez az oldal végigvezeti Önt a záró lépéseken és a következő teendőkön.

## A feldolgozás befejezésének jelzése

A feldolgozás sikeres befejezésekor több jelzés is megjelenik:

* ✅ **Haladási sáv**: Eléri a 100%-os befejezést
* ✅ **Hibakeresési napló**: Megjelenik a „Feldolgozás befejezve” üzenet
* ✅ **Indítás gomb**: Újra engedélyezetté válik (kész a következő feldolgozási futtatásra)
* ✅ **Kimeneti fájlok**: Az összes feldolgozott kép elmentésre került a kamera modelljének almappájába***

## A feldolgozott képek megkeresése

### A kimeneti mappa megnyitása

1. Kattintson a **Főmenü** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> ikonra (bal felső sarokban)
2. Válassza a **„Projektmappa megnyitása”** lehetőséget
3. A fájlkezelő megnyitja a projektkönyvtárat
4. Keresse meg a projektet a neve alapján

***

## A feldolgozott képek áttekintése

### Gyors előnézet a Fájlkezelőben

**Windows beépített előnézet:**

1. Keresse meg a kamera modell alkönyvtárát
2. Válasszon ki egy képfájlt
3. Az előnézet megjelenik az Windows Explorer előnézeti ablakában
4. A nyílgombokkal lapozhat a képek között

### Előnézet külső képnézegetőkben

**Ajánlott nézegetők:*** **QGIS** - Ingyenes GIS szoftver (legalkalmasabb georeferált multispektrális elemzéshez)
* **IrfanView** - Gyors, könnyű képnézegető (támogatja az TIFF formátumot)
* **Adobe Photoshop** - Professzionális szerkesztés (TIFF támogatás)
* **GIMP** - Ingyenes alternatíva a Photoshophoz
* **Windows Photos** - Alapvető megtekintés (lehet, hogy nem támogatja a 16 bites TIFF formátumot)

### Előnézet az Chloros Képmegjelenítőben

Használja az Chloros beépített Képmegjelenítőjét a fejlett megjelenítéshez:

1. Kattintson egy kép miniatűrjére a Fájlkezelőben
2. A kép megnyílik a fő előnézeti területen
3. Kattintson az **Képmegjelenítő** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> fülre a bal oldali sávban
4. Használja az [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) funkciót az interaktív elemzéshez

Részletes utasításokat az [Image Viewer](../image-viewer-gui/opening-an-image-full-screen.md) oldalon talál.

***

## A hibakeresési napló áttekintése

### Figyelmeztetések vagy hibák ellenőrzése

1. Nyissa meg a **Hibakeresési napló** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> fület
2. Görgessen végig az üzeneteken
3. Keresse meg a sárga figyelmeztetéseket vagy a piros hibákat
4. Ellenőrizze a megjegyzett problémákat
5. Forduljon az MAPIR ügyfélszolgálathoz segítségért

### A napló mentése

A feldolgozás nyilvántartásához vagy az MAPIR ügyfélszolgálatnak való elküldéshez:

1. Kattintson a **„Másolás”**vagy**„Letöltés”** gombra
2. Mentse szöveges fájlként a projektmappába
3. Csatolja a projekt dokumentációjához
4. Ha problémák merülnek fel, küldje el az MAPIR ügyfélszolgálatnak

***

## Gyakori kimeneti problémák és megoldások

### Probléma: Hiányzó kimeneti fájlok

**Lehetséges okok:**

* A fájlok nem feleltek meg a feldolgozási kritériumoknak
* Csak célképek (kizárva az exportból)
* Az exportálás során elfogyott a lemezterület
* A fájl megsérült a feldolgozás során

**Megoldások:**

1. Ellenőrizze a hibakeresési naplót az ugrási/hibajelentésekre vonatkozóan
2. Ellenőrizze, hogy elegendő volt-e a lemezterület
3. Számolja meg a fájlokat: Meg kell egyeznie (eredeti szám - célszám) × (indexek + 1)
4. Importálja újra és dolgozza fel újra a hiányzó fájlokat

### Probléma: Sötét vagy világos élek (a vignettálás továbbra is látható)

**Lehetséges okok:**

* A vignettálás-korrekció ki van kapcsolva
* A kamera/objektív nem szerepel az Chloros profiladatbázisban
* A vignettálás olyan mértékű, hogy a korrekció nem képes kezelni

**Megoldások:**

1. Ellenőrizze, hogy a vignettálás-korrekció engedélyezve van-e a Projektbeállításokban
2. Ellenőrizze, hogy a kamera modellje helyesen lett-e felismerve
3. Ha a vignettálás továbbra is fennáll, vegye fel a kapcsolatot az MAPIR ügyfélszolgálattal

### Probléma: Helytelen színek vagy értékek

**Lehetséges okok:**

* Nincs kalibrációs célpont észlelve
* Helytelen kalibrációs célpont modell lett kiválasztva
* A fényvisszaverődés-kalibrálás ki van kapcsolva
* Rossz minőségű célképek

**Megoldások:**

1. Ellenőrizze, hogy a reflexiós kalibrálás engedélyezve van-e
2. Ellenőrizze a „Célpont megtalálva” üzeneteket a hibakeresési naplóban
3. Ellenőrizze a célképek minőségét
4. Végezze el újra a feldolgozást a megfelelő célpontok megjelölésével

### Probléma: Az NDVI értékek helytelennek tűnnek

**Várható NDVI tartományok:*** **Víz, sziklák, talaj**: -0,1 és 0,2 között
* **Ritka/egészségtelen növényzet**: 0,2 és 0,4 között
* **Közepes növényzet**: 0,4 és 0,6 között
* **Egészséges, sűrű növényzet**: 0,6 és 0,9 között**Ha az értékek ezen tartományokon kívül esnek:**

1. Ellenőrizze, hogy a reflexiós kalibrálás alkalmazásra került-e
2. Ellenőrizze, hogy a fényérzékelő naplója szerepel-e
3. Ellenőrizze, hogy a kalibrációs célpontok felismerésre kerültek-e
4. Győződjön meg arról, hogy a megfelelő kameramodell került felismerésre
5. Ellenőrizze a célképek rögzítésének időzítését és körülményeit

***

## A feldolgozott képek használata

### Fotogrammetria / ortomosaik készítéséhez

**Ajánlott munkafolyamat:**

1.**Importálja a kalibrált reflektancia képeket** a fotogrammetriai szoftverbe:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Tartsa meg az EXIF metaadatokat**: Győződjön meg arról, hogy a GPS-adatok megmaradnak a geotaggeléshez
3. **Kalibrált munkafolyamatok**: Használjon reflektancia képeket a tudományos pontosság érdekében
4. **Feldolgozza az indexmozaikokat**: Készítsen NDVI ortomosaikokat az egyes indexképekből
5. **Exportálja a georeferált GeoTIFF**: GIS-alkalmazásokban való felhasználáshoz

### GIS-elemzéshez

**Ajánlott munkafolyamat:**

1.**Töltse be a QGIS-be, ArcGIS-be vagy hasonló programba**

2.**Használjon 16 bites TIFF** reflektancia képeket többsávos elemzéshez
3. **Használjon indexképeket** (NDVI, NDRE) készre használható növényzetrétegekként
4. **Raszterkalkulátor**: Sávok kombinálása egyedi elemzéshez
5. **Exportálás**: Hozzon létre osztályozási térképeket, változásérzékelést, növényzet-egészségügyi térképeket

### Közvetlen elemzéshez / jelentéskészítéshez

**Ajánlott munkafolyamat:**

1.**Használjon indexképeket LUT színekkel** a vizuális jelentésekhez
2. **Statisztikák kivonása**: Átlagos NDVI érték mezőnként/parcellánként
3. **Idősor**: Indexek összehasonlítása több munkamenet között
4. **Jelentések létrehozása**: Térképek, statisztikák és vizualizációk hozzáadása***

## Archiválás és biztonsági mentés

### Ajánlott biztonsági mentési stratégia

**Mit kell menteni:*** ✅ **Eredeti RAW/JPG képek** – Archiválás külön meghajtón/felhőben
* ✅ **Feldolgozott eredmények** – A kalibrált képeket és indexeket meg kell őrizni
* ✅ **Projektfájl** – Tartalmazza az összes beállítást az esetleges újrafeldolgozáshoz
* ✅ **Hibakeresési napló** – Dokumentálja a feldolgozás részleteit
* ✅ **Kalibrációs célképek** – Ellenőrzéshez és újrafeldolgozáshoz**Tárolási ajánlások:*** **Azonnali biztonsági mentés**: Külső merevlemez
* **Hosszú távú archiválás**: Felhőalapú tárolás (Google Drive, Dropbox stb.)
* **Kritikus adatok**: Tarts 2–3 másolatot különböző helyeken***

## Következő feldolgozási futtatások

### A projektbeállítások újrafelhasználása

Ha a jövőben hasonló adatkészleteket dolgozol fel:

1. **Mentse el a projekt sablont** (ha még nem tette meg)
2. **Hozzon létre új projektet** a mentett sablon felhasználásával
3. **Importálja az új képeket**

4.**Feldolgozza**azonos beállításokkal a konzisztencia érdekében

### Több munkamenet kötegelt feldolgozása

Több munkamenet/adatkészlet esetén:**

1. lehetőség: GUI – Több projekt**

* Hozzon létre külön projektet minden munkamenethez
* Használjon konzisztens sablonbeállításokat
* Feldolgozza őket egyenként

**

2. lehetőség: Chloros CLI (csak Chloros+)**

* Automatizálja a kötegelt feldolgozást
* Feldolgozzon több mappát szkriptekkel
* Lásd az [CLI dokumentációt](../CLI.md)

**

3. lehetőség: Python SDK (csak Chloros+ esetén)**

* Programozott vezérlés
* Integráció elemzési folyamatokkal
* Lásd: [API dokumentáció](../api-python-sdk.md)

***

## Hibaelhárítás az utómunkálatok során

### Újrafeldolgozás eltérő beállításokkal

Ha az eredmények nem kielégítőek:

1. Őrizze meg az eredeti képeket (soha ne törölje őket)
2. Nyissa meg ugyanazt a projektet az Chloros-ben
3. Állítsa be a beállításokat a Projektbeállítások panelen
4. Futtassa újra a feldolgozást – az eredmények felülírják a korábbiakat

### Képek részhalmazának feldolgozása

Ha csak bizonyos képeket szeretne újra feldolgozni:

1. Hozzon létre új projektet
2. Csak azokat a képeket importálja, amelyeket újra kell feldolgozni
3. Használja ugyanazt a beállítási sablont
4. Feldolgozza a kisebb adathalmazt

### Segítség

Ha problémákba ütközik:

* 📧 **E-mail**: info@mapir.camera (csatolja a hibakeresési naplót)
* 🌐 **Támogatás**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **GYIK**: [Gyakran ismételt kérdések](../faq.md)
* 📖 **Dokumentáció**: [Chloros kézikönyv](../)***

## Összefoglalás: Teljes munkafolyamat

Most már teljesítette a teljes Chloros feldolgozási munkafolyamatot:

1. ✅ **Projekt létrehozása** - Lásd [Projektek](../projects.md)
2. ✅ **Fájlok hozzáadása** - Lásd [Fájlok hozzáadása](adding-files-to-a-project.md)
3. ✅ **Beállítások módosítása** - Lásd [Projektbeállítások módosítása](adjusting-project-settings.md)
4. ✅ **Célok kijelölése** - Lásd [Célképek kiválasztása](choosing-target-images.md)
5. ✅ **Feldolgozás elindítása** - Lásd [A feldolgozás elindítása](starting-the-processing.md)
6. ✅ **A folyamat nyomon követése** - Lásd [A feldolgozás nyomon követése](monitoring-the-processing.md)
7. ✅ **Eredmények áttekintése** - Ez az oldal**A kalibrált, reflektancia-korrigált multispektrális képei készen állnak az elemzésre!**

***

## További források

### Speciális funkciók

* [**Képmegjelenítő**](../image-viewer-gui/opening-an-image-full-screen.md) - Interaktív megjelenítés és elemzés
* [**Index/LUT Sandbox**](../image-viewer-gui/index-lut-sandbox.md) - Egyéni index tesztelés
* [**Multispektrális indexképletek**](../project-settings/multispectral-index-formulas.md) – Teljes index-referencia

### Automatizálás és integráció

* [**CLI dokumentáció**](../CLI.md) – Parancssori kötegelt feldolgozás
* [**Python SDK**](../api-python-sdk.md) - Programozási automatizálás
* [**Chloros+ Funkciók**](../#chloros) - Fejlett feldolgozási képességek

### Támogatás és tanulás

* [**GYIK**](../faq.md) – Gyakori kérdések és válaszok
* [**Kalibrációs célpontok**](../calibration-targets.md) – A fényvisszaverődés-kalibrálás megértése
* [**Támogatott kamerák**](../supported-cameras.md) - Kompatibilis hardver
