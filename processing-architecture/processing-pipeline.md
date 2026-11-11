# Feldolgozási folyamat Az

Chloros 1.3.0 egy 4 szálas feldolgozási folyamatot használ, amely úgy működik, mint egy szakaszos futószalag. Minden szál a munkafolyamat egy-egy különálló szakaszát kezeli, így egyszerre több kép is feldolgozás alatt állhat különböző szakaszokban.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## A feldolgozási folyamat felépítése

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Minden kép sorrendben halad át mind a négy szálon. A Chloros+ többszálas feldolgozásával több kép is egyszerre foglalhatja el a különböző szálakat – miközben a 3. szál egy képet dolgoz fel, az 1. szál már a következőt észlelheti, a 2. szál egy másikat kalibrálhat, a 4. szál pedig a kész képet írhatja a lemezre.

Az előrehaladásról szálonként készül jelentés, amely a Server-Sent Events-en keresztül kerül továbbításra (a háttérrendszer ezeket az `/api/events`-en teszi közzé). A parancssori felület (CLI) élő előrehaladási kijelzőjén a négy szakasz a következőképpen van megjelölve: **Felismerés, Elemzés, Feldolgozás, Exportálás**.***

## Szálak részletei

### 1. szál: Felismerés

**Cél**: Képek betöltése és a kalibrációs célpontok felismerése.

* Képfájlokat olvas be a lemezről — Survey3 `.raw`+`.jpg` párok, LATTICE `.tif`/`.tiff` felvételek, valamint `.dng`
* Kivonja az EXIF metaadatokat (GPS, fényképezőgép-modell, időbélyegek, expozíció)
* Kalibrációs célpontok felismerése: ArUco-jelöléssel ellátott célpontgeometriák a LATTICE-felvételekhez, valamint a klasszikus panelérzékelő a Survey3 kalibrációs célpontfotókhoz
* Kimenet: képadatok + metaadatok + célpontfelismerési eredmények

Elsősorban I/O- és CPU-igényes szál.

### 2. szál: Kalibrálás

**Cél**: A kalibrációs paraméterek kiszámítása az észlelt célpontok alapján.

* Kiszámítja a reflexiós kalibrációs együtthatókat a célpontképekből
* Kiszámítja a vignettálás-korrekciós paramétereket
* Meghatározza a sávonkénti kalibrációs görbéket
* Kimenet: kalibrációs paraméterek minden egyes képhez

CPU-igényes számítási szál. Survey3-as vagy vegyes futtatás esetén a 3. szál erre vár, ha a reflexiós kalibrálás engedélyezve van, így a koefficiensei készen állnak, mielőtt bármely képet feldolgoznának. Kizárólag LATTICE-futás esetén a 3. szál az észlelés során fut: minden képkocka akkor indul el, amikor a célpont horgonya végleges. Egy képkocka az időben legközelebbi célt veszi mindkét oldalról, ezért addig vár, amíg a kamerájának minden olyan képkockája be nem lett szkennelve, amely időben közelebb van a jelenlegi legközelebbi célhoz. Az eredmények ugyanazok, csak hamarabb elkészülnek.

### 3. szál: Feldolgozás (GPU)

**Cél**: Korrekciók alkalmazása és vegetációs indexek kiszámítása.**Ez a legnagyobb számítási terhelést igénylő szál.*** **Debayering**: a RAW Bayer-adatokat többcsatornás képekké alakítja
  * Standard (gyors, közepes minőség) — alapértelmezett, `--debayer standard`
  * Texture Aware (lassú, legmagasabb minőség) — csak Chloros+ esetén, `--debayer texture-aware`, AI/ML zajszűrő modellt használ
  * A LATTICE mono (M3M) felvételek egycsatornásak: ezek esetében a demosaic és a fehéregyensúly-beállítás lépései kihagyásra kerülnek (egy soros naplóüzenettel), míg az ugyanazon futtatás során készült M3C/Bayer képek továbbra is megkapják ezeket
* **Vignettakorrekció**: az egész képre alkalmazza a lencse vignettakorrekcióját
* **Reflektancia-kalibrálás**: kalibrációs együtthatókat alkalmaz a reflektanciaértékekre történő átalakításhoz
* **Indexszámítás**: kiszámítja a vegetációs indexeket (NDVI, NDRE, GNDVI, …)
* Kimenetek: exportálásra kész, feldolgozott képadatok

Ez a szál profitál a leginkább a GPU-gyorsításból, és ez az a szál, amelyet a [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) hangol.

### 4. szál: Exportálás

**Cél**: A feldolgozott képek lemezre írása.

* Kimeneti fájlokat ír a kiválasztott formátumban — `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Metadatákat ágyaz be a kimeneti fájlokba (GPS, időbélyegek, feldolgozási paraméterek) — beleértve az `Chloros:*` feldolgozási címkéket is, amelyeket az 1.3.0-s verziótól kezdve minden feldolgozási módban írnak, mind a Survey3, mind a LATTICE esetében
* A kimeneti fájlokat a projektmappa alatt `<camera>/<format>/<Product>_Images/` néven rendezi — például `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Az exportált fájlok megtartják a forrásfájl nevét; a mappa azonosítja a terméket.**
* LATTICE-felvételek esetén egy forráskép több termékre is felbontható (Debayered, Preview, Radiance, Reflectance, Index), amelyek mindegyike a saját termékmappájában található. Az Index termékeket a futtatás által létrehozott reflektancia-termékből számítják ki (ha ilyen nincs, akkor a futtatás által létrehozott egy másik termékből, például a sugárzási értékből). A multispektrális M3C kamerák sugárzási és visszaverődési termékei az `<format>/clamp_masks/` fájlban pixelenkénti korlátozó maszkot is kaphatnak (lásd [Kimeneti képformátumok](../output-image-formats.md#clamp-masks))
* Kimenetek: végleges fájlok a lemezen

Elsősorban I/O-korlátozott szál — az SSD-tároló észrevehetően javítja a teljesítményét.

***

## A háttérben: végrehajtók

A 3. szálon belül a képekhez tartozó feladatok a következőképpen párhuzamosítódnak:

* **A GPU-stratégiák**(`GPU_SINGLE`, `GPU_PARALLEL`) egy**munkavégző folyamatok**ból álló poolt használnak, amelyek mindegyike külön operációs rendszer-folyamat, saját CUDA-kontextussal. A Chloros minden munkavégzőt úgy indít el, hogy újra elindítja a saját, telepített háttérprogramját — nem a Python `multiprocessing` parancsán keresztül, amelynek munkavégző-indítása a csomagolt alkalmazáson belül nem működik
* **`CPU_PARALLEL`** szálkészletet használ (a Python `ThreadPoolExecutor` funkciója) — a NumPy és az OpenCV felszabadítja a GIL-t, így a szálak elegendőek
* A 8 GB-os vagy annál kisebb megosztott RAM-mal rendelkező Jetson-eszközök teljesen kihagyják az executort, és folyamaton belül, szekvenciálisan dolgoznak fel
* A 7 GB-nál kevesebb VRAM-mal rendelkező GPU-n a Texture Aware szintén szekvenciálisan fut — a zajszűrő modell nem fér el többször A

Chloros nem használ semmilyen harmadik féltől származó elosztott keretrendszert (például a Ray-t). A stratégia és a munkavállalók számának kiválasztásáról lásd a [Dinamikus számítási adaptáció](dynamic-compute-adaptation.md) című részt.

***

## Szekvenciális vs. csővezetékes feldolgozás

### Szabad mód (szekvenciális)

A Chloros ingyenes verziójában a képeket **egyszerre egyenként**, szekvenciálisan dolgozzák fel mind a négy szakaszban:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

A grafikus felhasználói felület (GUI) egyszerűsített haladási sávot jelenít meg szabad módban; a szekvenciális fázisokat **Célfelismerés**, majd**Feldolgozás** néven jelzi.

### Chloros+ mód (csővezetékes)

Chloros+ licenc esetén mind a négy szál **párhuzamosan** dolgozik különböző képeken:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

A grafikus felhasználói felület (GUI) haladási sávja a négy szakaszt mutatja; ha az egérmutatót ráhelyezzük, láthatóvá válik az egyes szálak haladása. A parancssorban (CLI) ugyanezek a négy szakaszok **Detecting, Analyzing, Processing, Exporting** néven jelennek meg valós időben.

{% hint style="info" %}
**Egy címke, két név.** A CLI-ben a 3. szakaszt _Feldolgozás_ néven jelöli. A háttérprogram prémium módú haladási jelzése – amelyet a GUI haladási sávja jelenít meg – ugyanezt a szakaszt _Kalibrálás_ néven jelöli. Ugyanaz a szál végzi ugyanazt a munkát (3. szál: debayer, korrekciók, indexek).
{% endhint %}

{% hint style="success" %}
A **Chloros+ segítségével végzett csővezetékes feldolgozás** a hardvertől és az adatkészlet méretétől függően 3–5-ször gyorsabb lehet, mint a szekvenciális feldolgozás. A sebességnövekedés a gyors GPU-kkal és SSD-kkel rendelkező rendszereken a legnagyobb.
{% endhint %}

***

## 4. szál: Exportálás előrehaladása

Az exportáló szálnak saját előrehaladás-követése van, amelyet külön lekérdezhet:

**Parancssori felület (CLI):**

```bash
chloros-cli export-status
```

**Fejlesztői készlet (SDK):**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

A feldolgozás akkor fejeződik be, amikor a 4. szál eléri a 100%-ot.

{% hint style="info" %}
**Az a futtatás, amely nem ír ki képfájlokat, sikertelennek minősül.**Siker esetén az `chloros-cli process` jelenti, hogy hány képfájlt írt ki (`Image products written: N`). Ha termékeket kértek, de**egyet sem**írtak — csak az `project.json` és az `calibration_data.json` parancsok futottak —, a CLI kinyomtatja az `Processing finished but wrote no image products.` parancsot, és**nem nulla értékkel lép ki**, megnevezve a projektmappát és a szokásos okokat (a bemeneti mappát nem ismerte fel rögzítésként – ellenőrizze az elrendezést és az `--input-level` fájlt –, vagy az összes kért termék alkalmazhatatlan volt az adott kamerákra). A szkriptek támaszkodhatnak a kilépési kódra.
{% endhint %}

***

## Kapcsolat a dinamikus számítási adaptációval

A [dinamikus számítási adaptáció](dynamic-compute-adaptation.md) elsősorban a **

3. szálat (feldolgozás)** érinti:

* **`GPU_PARALLEL`**: A 3. szál az `fused_gpu` csővezetéket használva egyszerre több képet futtat át a GPU-n
* **`GPU_SINGLE`**: A 3. szál szemaforral sorba rendezi a GPU-hozzáféréseket, miközben a munkavégző folyamatok átfedésben végzik az I/O-műveleteket, az `fused_gpu` vagy a memóriahatékony `tiled_gpu` folyamatvezeték használatával
* **`CPU_PARALLEL`**: A 3. szál többszálas párhuzamossággal CPU-alapú feldolgozást használ

A 3. szál GPU-memória-allokációja is növekszik, ahogy az 1. és 2. szál befejezi a feladatát — lásd [Dinamikus GPU-memória-allokáció](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Következő lépések

* [Dinamikus számítási adaptáció](dynamic-compute-adaptation.md) — Hogyan választja ki a Chloros az Ön hardveréhez legmegfelelőbb stratégiát
* [NVIDIA Jetson útmutató](../linux/nvidia-jetson-guide.md) — Platformspecifikus folyamatviselkedés a Jetsonon
* [A feldolgozás figyelemmel kísérése](../processing-images-gui/monitoring-the-processing.md) — A folyamat előrehaladásának figyelemmel kísérése a grafikus felhasználói felületen
* [A CLI-referencia](../reference/cli-reference.md) — `process`, `export-status`, kilépési kódok és a kimeneti elrendezés
