# Dinamikus számítási alkalmazkodás

Az Chloros 1.1.0 verzió intelligens hardverfelismerést és automatikus feldolgozási stratégia-kiválasztást vezet be. A feldolgozó motor manuális beállítás nélkül alkalmazkodik a hardverhez – a Jetson Nanótól a több GPU-val rendelkező munkaállomásokig.

***

## Hogyan működik

Az Chloros indításakor automatikusan profilozza a rendszert:

1. **Felismeri az operációs rendszert** — Windows vagy Linux
2. **Meghatározza a CPU magokat és a teljes RAM-ot**

3.**Észleli a GPU jelenlétét** — NVIDIA CUDA képesség, VRAM, modell
4. **Meghatározza a Jetson modellt** (ha alkalmazható) — az `/proc/device-tree/model` segítségével
5. **Ellenőrzi a hőérzékelőket** (Jetson) — a hőmérséklet-érzékeny feldolgozáshoz
6. **Kiválasztja az optimális számítási stratégiát** — az összes észlelt hardver alapján
7. **Automatikusan konfigurálja a munkavállalók számát, a folyamat típusát és a memóriaallokációt**Az eredményt gyorsítótárba menti, így a következő futtatások gyorsabban indulnak. Ha a hardver megváltozik (pl. GPU kerül hozzáadásra), az Chloros a következő indításkor új profilozást végez.***

## Számítási stratégiák

Az Chloros a hardver alapján a három számítási stratégia egyikét választja ki:

| Stratégia | Szükséges GPU | Munkások | Folyamat | Legalkalmasabb |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Igen (12 GB+ VRAM vagy 16 GB+ megosztott) | 3-4 | `fused_gpu` | 12 GB+ memóriájú asztali GPU-k, Jetson Orin NX 16 GB, AGX Orin |
| **`GPU_SINGLE`** | Igen (&lt; 12 GB VRAM) | 1-3 | `tiled_gpu` | Belépő szintű GPU-k, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | Nem | magok - 1 | `cpu_fallback` | NVIDIA GPU nélküli rendszerek |

### Pipeline típusok

* **`fused_gpu`** — Teljes GPU feldolgozási útvonal. Minden debayer, korrekciós és indexelési művelet egyetlen összevont lépésben fut a GPU-n. A legnagyobb átviteli sebesség, de több VRAM-ot igényel.
* **`tiled_gpu`** — Memóriahatékony GPU-útvonal. A képeket csempékben dolgozza fel, hogy azok beférjenek a korlátozott GPU-memóriába. Alacsonyabb átviteli sebesség, de memóriakorlátozott eszközökön is működik.
* **`cpu_fallback`** — Kizárólag CPU-n történő feldolgozás többszálas párhuzamosság használatával. Akkor használható, ha nincs elérhető NVIDIA GPU.***

## Platformspecifikus viselkedés

| Platform | Stratégia | Munkások | Folyamat | Megjegyzések |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (soros) | Memóriahatékony mód, egyszerre egy képet dolgoz fel |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (párhuzamos) | Ajánlott peremkészülék — valódi párhuzamos GPU-feldolgozás |
| **Jetson AGX Orin 64 GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (párhuzamos) | Maximális peremteljesítmény |
| **Asztali számítógép 8 GB-os GPU-val** | `GPU_SINGLE` | 3 | `tiled_gpu` | Jó asztali teljesítmény memóriahatékony csempékkel |
| **Asztali számítógép 12 GB-os vagy annál nagyobb GPU-val** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Optimális asztali teljesítmény |
| **Csak CPU-val rendelkező rendszer** | `CPU_PARALLEL` | magok - 1 | `cpu_fallback` | Nincs szükség GPU-ra, ThreadPool-t használ |

{% hint style="info" %}
**Jetson egységes memória**: A Jetson eszközök megosztják a GPU és a CPU memóriáját. Egy Jetson Orin NX 16 GB ~15,3 GB VRAM-ot jelez, de ez ugyanaz a fizikai RAM, amelyet az operációs rendszer és a CPU-folyamatok is használnak. Az Chloros ezt figyelembe veszi a memóriaallokációs küszöbértékek beállításakor.
{% endhint %}

***

## Dinamikus GPU-memóriaallokáció

Az Chloros egy [4 szálas feldolgozási folyamatot](processing-pipeline.md) használ:

* **

1. szál** (Észlelés) — Kép betöltése, EXIF-elemzés, célpont-észlelés
* **

2. szál** (Kalibrálás) — Reflektancia-kalibrálás számítása
* **

3. szál** (Feldolgozás) — GPU-alapú debayer, vignettás korrekció, indexszámítás
* **

4. szál** (Exportálás) — Fájlírás, metaadatok beágyazása

Ahogy a korábbi folyamat szálak befejezik munkájukat (pl. az összes kép felismerése), a GPU-memória-allokációjuk felszabadul, és **újraelosztásra kerül a fennmaradó aktív szálak között**. Ez azt jelenti, hogy a 3. szál (a GPU-igényes szakasz) a folyamat előrehaladtával egyre több memóriát kap, javítva ezzel a leginkább számításigényes feladatok átviteli sebességét.

### Allokációs szakaszok

| Szakasz | Aktív szálak | GPU-memória elosztás |
| --- | --- | --- |
| **Korai** | 1, 2, 3, 4 | Az összes szál között felosztva |
| **Középső-korai** | 2, 3, 4 | Az 1. szál memóriája újraelosztásra kerül |
| **Középső-késői** | 3, 4 | Az 1. és 2. szál memóriája a 3. és 4. szálhoz kerül |
| **Késői** | 3 vagy 4 | Maximális memória a fennmaradó szál számára |***

## Textúraérzékeny feldolgozás

A textúraérzékeny debayer módszer (csak Chloros+) az AI/ML zajszűrési modell miatt lényegesen több GPU-memóriát használ, mint a standard módszer:

* A **&lt; 7 GB VRAM**-mal rendelkező rendszerek kénytelenek szinkron feldolgozási ciklusba lépni a textúraérzékeny módban (egyszerre egy kép)
* A **7 GB-nál nagyobb VRAM-mal** rendelkező rendszerek párhuzamosan tudják feldolgozni a textúraérzékeny módot, bár a Standard módhoz képest csökkentett munkavállalói létszámmal***

## Hőkezelés (Jetson)

A Jetson eszközöknek hőmérsékleti korlátai vannak, különösen zárt vagy repülőgépes telepítések esetén. Az Chloros figyeli a GPU és a CPU hőmérsékletét, és automatikusan szabályozza a feldolgozást:

| Hőmérséklet | Reakció |
| --- | --- |
| **&lt; 70°C** | Normál működés — teljes sebesség |
| **70°C** (Figyelmeztetés) | A köteg méretének csökkentése |
| **80°C** (Kritikus) | Erőteljes fojtás — a párhuzamosság és a munkavállalók számának csökkentése |
| **90°C** (Leállítás) | A GPU-feldolgozás teljes leállítása |

A hőmérséklet-figyelés a Jetson platformokon az `tegrastats`-et használja. Megfelelő hűtéssel rendelkező asztali rendszereken a hőmérséklet-szabályozás ritkán lép működésbe.

***

## Memóriaigény kezelése

Az Chloros a feldolgozás során figyeli a rendszer memóriaterhelését:

* **Memória küszöbérték**: 85%-os kihasználtság esetén konzervatív viselkedés lép életbe
* **OOM-csökkentés**: Memóriahiány esetén az allokáció 25%-kal csökken (0,75-szeres szorzó)
* **Pipeline-visszaállás**: Súlyos memória terhelés esetén a pipeline automatikusan visszatér az `fused_gpu`-ről az `tiled_gpu`-re
* **Swap-ajánlások**: A Jetsonon az Chloros figyelmeztet, ha a swap-terület nem elegendő az adatkészlet méretéhez***

## A számítási adaptáció figyelése

### CLI állapotkimenet

A feldolgozás megkezdésekor az CLI megjeleníti az észlelt hardverprofilt:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Rendszerdiagnosztika

Futtassa az `chloros-cli selftest` parancsot a teljes hardverprofil megtekintéséhez és a számítási képességek ellenőrzéséhez:

```bash
chloros-cli selftest
```

Ez ellenőrzi a CUDA elérhetőségét, a GPU memóriát, a zajszűrő modelleket és a háttérkapcsolatot.

***

## Következő lépések

* [Feldolgozási folyamat](processing-pipeline.md) — A 4-szálas folyamatarchitektúra megértése
* [NVIDIA Jetson útmutató](../linux/nvidia-jetson-guide.md) — Jetson-specifikus telepítés és optimalizálás
* [CLI : Parancssor](../CLI.md) — Teljes CLI referencia
