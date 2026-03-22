# Feldolgozási folyamat

Az Chloros 1.1.0 verzió egy négy szálas feldolgozási folyamatot használ, amely szakaszos futószalagként működik. Minden szál a feldolgozási munkafolyamat egy-egy különálló szakaszát kezeli, így több kép is párhuzamosan feldolgozható a különböző szakaszokban.

***

## A folyamat felépítése

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Minden kép sorrendben halad át mind a négy szálon. Az Chloros+ többszálas feldolgozásával több kép is lehet egyszerre különböző szálakon – miközben a 3. szál egy képet dolgoz fel, az 1. szál a következőt észlelheti, a 2. szál egy másikat kalibrálhat, a 4. szál pedig egy korábban feldolgozott képet írhat a lemezre.

***

## Szálak részletei

### 1. szál: Felismerés

**Cél**: Képek betöltése és kalibrációs célpontok felismerése.

* Képfájlok olvasása a lemezről (RAW, JPG)
* Kivonja az EXIF metaadatokat (GPS, fényképezőgép modell, időbélyegek, expozíció)
* Felismeri az ArUco kalibrációs célpontokat a megjelölt célképeken
* Kimenet: képadatok + metaadatok + célpont-felismerési eredmények

Ez elsősorban egy I/O- és CPU-igényes szál.

### 2. szál: Kalibrálás

**Cél**: A kalibrációs paraméterek kiszámítása a felismert célpontokból.

* Kiszámítja a reflektancia-kalibrációs együtthatókat a célképekből
* Kiszámítja a vignettálás-korrekciós paramétereket
* Meghatározza a sávonkénti kalibrációs görbéket
* Kimenet: kalibrációs paraméterek minden képhez

Ez egy CPU-igényes számítási szál.

### 3. szál: Feldolgozás (GPU)

**Cél**: Korrekciók alkalmazása és növényzetindexek kiszámítása.**Ez a leginkább számításigényes szál.*** **Debayering**: A RAW Bayer-minta adatokat többcsatornás képekké alakítja
  * Standard (Gyors, Közepes minőség) — alapértelmezett
  * Texture Aware (Lassú, Legmagasabb minőség) — csak Chloros+, AI/ML zajszűrést használ
* **Vignette-korrekció**: Lencse-vignette-korrekciót alkalmaz a képen
* **Reflektancia-kalibrálás**: Kalibrációs együtthatókat alkalmaz a reflektanciaértékekre történő konvertáláshoz
* **Indexszámítás**: Növényzetindexeket számol (NDVI, NDRE, GNDVI stb.)
* Kimenet: exportálásra kész feldolgozott képadatok

Ez a szál profitál a legjobban a GPU-gyorsításból. A [Dynamic Compute Adaptation](dynamic-compute-adaptation.md) rendszer elsősorban ennek a szálnak a viselkedését optimalizálja.

### 4. szál: Export

**Cél**: A feldolgozott képek lemezre írása.

* Kimeneti fájlokat ír a kiválasztott formátumban (TIFF 16 bites, TIFF 32 bites %, PNG, JPG)
* EXIF metaadatokat ágyaz a kimeneti fájlokba (GPS, időbélyegek, feldolgozási paraméterek)
* A kimenetet kameramodell szerinti almappákba rendezi
* Kimenet: végleges fájlok a lemezen

Ez elsősorban egy I/O-korlátozott szál. Az SSD-tároló jelentősen javítja a 4. szál teljesítményét.

***

## Szekvenciális vs. csővezetékes feldolgozás

### Ingyenes mód (szekvenciális)

Az Chloros ingyenes verziójában a képeket **egyszerre egyenként**, szekvenciálisan dolgozza fel a program mind a négy szakaszban:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

A GUI haladási sávja 2 szakaszt mutat: Célfelismerés és Feldolgozás.

### Chloros+ mód (csővezetékes)

Chloros+ licenccel mind a négy szál **egyidejűleg** működik különböző képeken:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

A GUI haladási sávja 4 szakaszt jelenít meg: Észlelés, Elemzés, Kalibrálás, Exportálás. Vigye az egérmutatót a haladási sáv fölé a szálak szerinti haladás megtekintéséhez.

{% hint style="success" %}
**A Chloros+ segítségével végzett pipelined feldolgozás** a hardver és az adatkészlet méretétől függően 3-5-ször gyorsabb lehet, mint a szekvenciális feldolgozás. A sebességnövekedés a gyors GPU-kkal és SSD-kkel rendelkező rendszereken a legnagyobb.
{% endhint %}

***

## 4. szál exportálási előrehaladása

Az Chloros 1.1.0 verzióban az exportáló szál (4. szál) saját, dedikált előrehaladási nyomon követéssel rendelkezik. Az exportálás előrehaladását külön is figyelemmel kísérheti:**CLI:**
```bash
chloros-cli export-status
```

**SDK:**
```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

A feldolgozás akkor fejeződik be, amikor a 4. szál eléri a 100%-ot.

***

## Kapcsolat a dinamikus számítási adaptációval

A [dinamikus számítási adaptáció](dynamic-compute-adaptation.md) rendszer elsősorban a **

3. szálat (feldolgozás)** érinti:

* **`GPU_PARALLEL`** stratégia: A 3. szál az `fused_gpu` csővezetéket használva egyszerre több képet futtat a GPU-n
* **`GPU_SINGLE`** stratégia: A 3. szál a memóriatakarékos `tiled_gpu` csővezetéket használva egyszerre egy képet dolgoz fel
* **`CPU_PARALLEL`** stratégia: A 3. szál többszálas párhuzamossággal CPU-alapú feldolgozást használ

A 3. szál GPU-memória-allokációja is dinamikusan változik, ahogy az 1. és 2. szál befejezi a feldolgozást — lásd [Dinamikus GPU-memória-allokáció](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Következő lépések

* [Dinamikus számítási adaptáció](dynamic-compute-adaptation.md) — Hogyan választja ki az Chloros a hardveréhez legmegfelelőbb stratégiát
* [NVIDIA Jetson útmutató](../linux/nvidia-jetson-guide.md) — Platformspecifikus pipeline viselkedés a Jetsonon
* [A feldolgozás figyelése](../processing-images-gui/monitoring-the-processing.md) — A folyamat előrehaladásának figyelése a grafikus felhasználói felületen
