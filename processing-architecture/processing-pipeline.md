# Feldolgozási folyamat

ChlorosAz 1.2.0-s verzió egy 4 szálas feldolgozási folyamatot használ, amely úgy működik, mint egy szakaszos futószalag. Minden szál a munkafolyamat egy-egy különálló szakaszát kezeli, így egyszerre több kép is feldolgozás alatt állhat különböző szakaszokban.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## A feldolgozási folyamat felépítése

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Minden kép sorrendben halad át mind a négy szálon. Az Chloros+ többszálas feldolgozásával több kép is egyidejűleg foglalhatja el a különböző szálakat — miközben a 3. szál egy képet dolgoz fel, az 1. szál már a következőt észlelheti, a 2. szál egy másikat kalibrálhat, a 4. szál pedig a kész képet írja a lemezre.

Az előrehaladásról szálonként készül jelentés, és az adatok Server-Sent Events-en keresztül kerülnek továbbításra (a háttérrendszer az `/api/events`-en teszi közzé őket). Az CLI élő előrehaladási kijelzőjén a négy szakasz a következőképpen van megjelölve: **Felismerés, Elemzés, Feldolgozás, Exportálás**.***

## Szálak részletei

### 1. szál: Felismerés

**Cél**: Képek betöltése és a kalibrációs célpontok felismerése.

* Képfájlokat olvas be a lemezről — Survey3 `.raw`+`.jpg` párok, LATTICE `.tif`/`.tiff` felvételeket, valamint az `.dng`
* Kivonja az EXIF metaadatokat (GPS, fényképezőgép-modell, időbélyegek, expozíció)
* Kalibrációs célpontok felismerése: ArUco-jelöléssel ellátott célgeometriák a LATTICE-felvételekhez, valamint a klasszikus panelérzékelő az Survey3 kalibrációs célpontfotóihoz
* Kimenet: képadatok + metaadatok + célpont-felismerési eredmények

Elsősorban I/O- és CPU-igényes szál.

### 2. szál: Kalibrálás

**Cél**: A kalibrációs paraméterek kiszámítása az észlelt célpontok alapján.

* Kiszámítja a reflexiós kalibrációs együtthatókat a célképekből
* Kiszámítja a vignettálás-korrekciós paramétereket
* Meghatározza a sávonkénti kalibrációs görbéket
* Kimenetek: kalibrációs paraméterek minden egyes képhez

CPU-igényes számítási szál. A 3. szál erre vár, ha a reflexiós kalibráció engedélyezve van, így az együtthatók készen állnak, mielőtt bármely képet feldolgoznának.

### 3. szál: Feldolgozás (GPU)

**Cél**: Korrekciók alkalmazása és vegetációs indexek kiszámítása.**Ez a legnagyobb számítási terhelést igénylő szál.*** **Debayering**: a RAW Bayer-adatokat többcsatornás képekké alakítja át
  * Standard (gyors, közepes minőség) — alapértelmezett, `--debayer standard`
  * Texture Aware (lassú, legmagasabb minőség) — csak Chloros+ esetén, `--debayer texture-aware`, AI/ML zajszűrési modellt használ
  * A LATTICE mono (M3M) felvételek egycsatornásak: ezeknél a demosaikolás és a fehéregyensúly-beállítás lépései kihagyásra kerülnek (egy soros naplóüzenettel), míg az ugyanazon futtatás során készült M3C/Bayer képek továbbra is megkapják ezeket
* **Vignettakorrekció**: a kép egészére alkalmazza a lencse vignettakorrekcióját
* **Reflektancia-kalibrálás**: kalibrációs együtthatókat alkalmaz a reflektanciaértékekre történő átalakításhoz
* **Indexszámítás**: kiszámítja a vegetációs indexeket (NDVI, NDRE, GNDVI, …)
* Kimenet: exportálásra kész, feldolgozott képadatok

Ez a szál profitál leginkább a GPU-gyorsításból, és ez az a szál, amelyet a [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) hangol.

### 4. szál: Export

**Cél**: A feldolgozott képek lemezre írása.

* A kimeneti fájlokat a kiválasztott formátumban írja — `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Metadatákat ágyaz a kimeneti fájlokba (GPS, időbélyegek, feldolgozási paraméterek)
* A kimenetet a projektmappa alatt `<camera>/<format>/<Product>_Images/` néven rendezi el — például `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Az exportált fájlok megtartják a forrásfájl nevét; a mappa azonosítja a terméket.**
* LATTICE-felvételek esetén egy forráskép több termékre is felbontható (Debayered, Preview, Radiance, Reflectance, Index), amelyek mindegyike a saját termékmappájában található
* Kimenet: végleges fájlok a lemezen

Elsősorban I/O-korlátozott szál — az SSD-tároló észrevehetően javítja a teljesítményt.

***

## A háttérben: végrehajtók

A 3. szálon belül a képekhez tartozó munkát a Python szabványos `concurrent.futures`-ével párhuzamosítják:

* **A GPU-stratégiák**(`GPU_SINGLE`, `GPU_PARALLEL`) egy `ProcessPoolExecutor`-et használnak**spawn** indítási módszerrel — minden munkavégző egy külön folyamat, saját CUDA-kontextussal (az `fork` örökölné a szülő inicializált CUDA-állapotát, és megrongálná a gyermekfolyamatokat)
* **`CPU_PARALLEL`** egy `ThreadPoolExecutor`-et használ — a NumPy és az OpenCV felszabadítja a GIL-t, így a szálak elegendőek
* A 8 GB-os vagy annál kisebb megosztott RAM-mal rendelkező Jetson-eszközök teljesen kihagyják az executort, és folyamaton belül, szekvenciálisan dolgoznak
* A Texture Aware 7 GB alatti VRAM-mal rendelkező GPU-n szintén szekvenciálisan fut — a zajszűrő modell nem fér el többször

Chlorosnem használ semmilyen harmadik féltől származó elosztott keretrendszert (például Ray-t). A stratégia és a munkavállalók számának kiválasztásáról lásd a [Dinamikus számítási adaptáció](dynamic-compute-adaptation.md) című részt.

***

## Szekvenciális vs. csővezetékes feldolgozás

### Szabad mód (szekvenciális)

Az Chloros ingyenes verziójában a képeket **egyszerre egyenként**, szekvenciálisan dolgozzák fel a négy szakasz mindegyikén:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

A grafikus felhasználói felület (GUI) egyszerűsített haladási sávot jelenít meg szabad módban; a szekvenciális fázisok **Célfelismerés**, majd**Feldolgozás** néven jelennek meg.

### „Chloros”+ mód (csővezetékes)

A „Chloros”+ licenccel mind a négy szál **egyidejűleg** dolgozik különböző képeken:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

A grafikus felhasználói felület (GUI) haladási sávja a négy szakaszt mutatja; ha az egérmutatót ráhelyezi, láthatja az egyes szálak haladását. A „CLI” alkalmazásban ugyanezek a négy szakaszok élőben jelennek meg **Detecting, Analyzing, Processing, Exporting** néven.

{% hint style="info" %}
**Egy címke, két név.** A CLI a 3. szakaszt _Feldolgozás_ néven jelöli. A háttérrendszer prémium módú haladási adatfolyama — amelyet a GUI haladási sávja jelenít meg — ugyanezt a szakaszt _Kalibrálás_ néven jelöli. Ugyanaz a szál végzi ugyanazt a munkát (3. szál: debayer, korrekciók, indexek).
{% endhint %}

{% hint style="success" %}
**A „Chloros+” funkcióval történő csővezetékes feldolgozás** a hardvertől és az adatkészlet méretétől függően 3–5-ször gyorsabb lehet, mint a szekvenciális feldolgozás. A sebességnövekedés a gyors GPU-kkal és SSD-kkel rendelkező rendszereken a legnagyobb.
{% endhint %}

***

## 4. szál: Exportálás előrehaladása

Az exportáló szálnak saját előrehaladás-követése van, amelyet külön lekérdezhet:

**CLI:**

```bash
chloros-cli export-status
```

**SDK:**

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

A feldolgozás akkor fejeződik be, amikor a 4. szál eléri a 100%-ot.

{% hint style="info" %}
**Az a futtatás, amely nem ír ki képeket, sikertelennek minősül.**Siker esetén az `chloros-cli process` jelenti, hogy hány képterméket írt ki (`Image products written: N`). Ha termékeket kértek, de**egyet sem**írtak — csak az `project.json` és az `calibration_data.json` —, akkor a CLI kinyomtatja az `Processing finished but wrote no image products.` értéket, és**nem nulla értékkel lép ki**, megnevezve a projektmappát és a szokásos okokat (a bemeneti mappát nem ismerte fel rögzítésként — ellenőrizze az elrendezést és az `--input-level` fájlt — vagy az összes kért termék alkalmatlan volt az adott kamerákra). A szkriptek támaszkodhatnak a kilépési kódra.
{% endhint %}

***

## Kapcsolat a dinamikus számítási adaptációval

A [dinamikus számítási adaptáció](dynamic-compute-adaptation.md) elsősorban a **

3. szálat (feldolgozás)** érinti:

* **`GPU_PARALLEL`**: A 3. szál az `fused_gpu` feldolgozási csatornát használva több képet futtat egyszerre a GPU-n
* **`GPU_SINGLE`**: A 3. szál szemaforral sorba rendezi a GPU-hozzáférést, miközben a munkavégző folyamatok átfedésben végzik az I/O-műveleteket, az `fused_gpu` vagy a memóriahatékony `tiled_gpu` folyamatvezeték használatával
* **`CPU_PARALLEL`**: A 3. szál többszálú párhuzamossággal CPU-alapú feldolgozást használ

A 3. szál GPU-memória-allokációja is növekszik, ahogy az 1. és 2. szál befejezi a munkát — lásd [Dinamikus GPU-memória-allokáció](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Következő lépések

* [Dinamikus számítási adaptáció](dynamic-compute-adaptation.md) — Hogyan választja ki az Chloros az Ön hardveréhez legmegfelelőbb stratégiát
* [NVIDIA Jetson útmutató](../linux/nvidia-jetson-guide.md) — Platformspecifikus feldolgozási folyamat a Jetsonon
* [A feldolgozás figyelemmel kísérése](../processing-images-gui/monitoring-the-processing.md) — A folyamat előrehaladásának figyelemmel kísérése a grafikus felhasználói felületen
* [Az CLI-referencia](../reference/cli-reference.md) — `process`, `export-status`, kilépési kódok és kimeneti elrendezés
