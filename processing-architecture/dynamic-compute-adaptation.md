# Dinamikus számítási alkalmazkodás A

Chloros
1.3.0 hardverfelismerést és automatikus feldolgozási stratégia-kiválasztást alkalmaz. A feldolgozó motor manuális beállítás nélkül alkalmazkodik a hardveredhez — a Jetson Orin Nanótól a több GPU-val rendelkező munkaállomásig.

***

## Hogyan működik?

A Chloros
indításakor profilozza a rendszert:

1. **Felismeri az operációs rendszert** — Windows
vagy Linux
2. **Azonosítja a CPU-magokat és a teljes RAM-kapacitást**

3.**Felismeri a GPU jelenlétét** — NVIDIA CUDA-kompatibilitás, VRAM, modell
4. **Meghatározza a Jetson modellt** (ha van ilyen) — az `/proc/device-tree/model` azonosító alapján
5. **Ellenőrzi a hőérzékelőket** (Jetson) — a hőmérséklet-érzékeny feldolgozáshoz
6. **Kiválasztja a számítási stratégiát** — az összes felismert hardver alapján
7. **Automatikusan konfigurálja a munkaszálak számát, a feldolgozási csatorna típusát és a memóriaallokációt**

Az észlelt profil a munkamenet idejére a memóriában és a lemezen is gyorsítótárba kerül, így a későbbi futtatások gyorsabban indulnak el:

| Platform | Gyorsítótárba helyezett profil |
| --- | --- |
| **Linux
/ Jetson** | `~/.config/chloros/system_config.json` (figyelembe veszi az `XDG_CONFIG_HOME`-et) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

A fájl törlésével kényszerítheti az újbóli felismerést — ez hasznos lehet GPU vagy további RAM hozzáadása után. A Chloros
automatikusan újrafelismeri a profilt akkor is, ha a gyorsítótárat egy összeférhetetlen régebbi verzió írta.

***

## Számítási stratégiák

A Chloros
a hardver alapján három számítási stratégia közül választ egyet:

| Stratégia | Mikor választódik | Munkások | Végrehajtó | Folyamat |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| CUDA GPU, amely**12 GB-nál több VRAM-ot**jelent (Jetson egységes memórián, emellett legalább 12 GB+ összesen megosztott RAM szükséges) | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`, minimum 2 —**Jetsonon 2-re korlátozva** | Munkavállaló folyamatok (külön CUDA-kontextusok) | `fused_gpu` |
| **`GPU_SINGLE`**|**2–12 GB VRAM**-mal rendelkező CUDA GPU | 3 (I/O átfedés; a GPU-hozzáférés szemaforral sorba rendezve).**1 (szekvenciális) 12 GB-nál kevesebb RAM-mal rendelkező Jetson-okon** | Munkavégző folyamatok (külön CUDA-kontextusok); szekvenciális, folyamaton belüli feldolgozás alacsony RAM-mal rendelkező Jetson-okon | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Nincs CUDA GPU, vagy 2 GB alatti VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Példák az `GPU_PARALLEL` munkavállalói képletre: 12 GB VRAM → 2 munkavállaló, 16 GB → 3 munkavállaló, 20 GB+ → 4 munkavállaló. Minden munkás számára 3,5 GB-ot különítenek el, ami egy textúraérzékeny feladat mért igénye, a 0,5 GB-os tartalék levonása után fennmaradó VRAM 90%-án belül.

{% hint style="info" %}
**A GPU-stratégiákhoz olyan PyTorch szükséges, amely képes a CUDA használatára.** Az a gép, amelynek PyTorch-ja nem rendelkezik használható CUDA-eszközzel, az `CPU_PARALLEL` besorolást kapja, függetlenül a GPU-jától. Az 1.3.0-s Jetson-csomag a JetPack 6 CUDA-alapú PyTorch-verzióját használja, így egy Jetson akkor felel meg a követelményeknek, ha a JetPack CUDA- és cuDNN-könyvtárai telepítve vannak. A 12 GB-nál kevesebb RAM-mal rendelkező Jetson mindig egyszerre csak egy képet dolgoz fel, folyamat közben (`GPU_SINGLE` kóddal naplózva).
{% endhint %}

A GPU-stratégiák egy munkavégző folyamatokból álló csoportot futtatnak, amelyek mindegyike külön operációs rendszer-folyamat, saját CUDA-kontextussal; ezeket a Chloros
úgy indítja el, hogy újra elindítja a saját, telepített háttérprogramját (nem a Python
`multiprocessing` parancsán keresztül, amelynek munkavégző folyamatainak elindítása a csomagolt alkalmazáson belül nem működik). A CPU-stratégia szálkészletet használ (a Python
`ThreadPoolExecutor` parancsát). A Chloros
nem használ semmilyen harmadik féltől származó elosztott keretrendszert (például a Ray-t).

### Feldolgozási útvonalak

* **`fused_gpu`** — Teljes GPU-feldolgozási útvonal. A debayer, a korrekció és az indexelési műveletek egyetlen összevont lépésben futnak a GPU-n. A legnagyobb átviteli sebességet biztosítja, de a legtöbb VRAM-ot igényli.
* **`tiled_gpu`** — Memóriahatékony GPU-útvonal. A képeket csempékben dolgozza fel, hogy azok beférjenek a korlátozott GPU-memóriába. Alacsonyabb átviteli sebesség, de memóriakorlátozott eszközökön is működik.
* **`cpu_fallback`** — Kizárólag CPU-n történő feldolgozás többszálas párhuzamosság alkalmazásával. Akkor használatos, ha nincs elérhető NVIDIA GPU, valamint végső megoldásként, ha mindkét GPU-feldolgozási útvonal meghiúsul.

A futásidejű tartaléklánc mindig a következő: `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Stratégia kézi felülírása

Állítsa be az `CHLOROS_STRATEGY` környezeti változót egy adott stratégia kényszerítéséhez — ez egy szakértői menekülési útvonal arra az esetre, ha az automatikus felismerés a helyzetéhez nem megfelelő megoldást választ (például azért, hogy a GPU-t más feladatokra szabadon tartsa):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

A változó nagy- és kisbetűk közötti különbséget nem tesz; mindent, ami nem a három név egyikéhez tartozik, figyelmen kívül hagy, és az automatikus felismerés normál módon folytatódik. Felülírás esetén a Chloros
továbbra is kiválasztja az Ön számára a munkavállalók számát:

| Felülírás | Használt munkavállalók száma |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Egy olyan gépen, amelyen nincs használható CUDA-eszköz, az `GPU_SINGLE` vagy `GPU_PARALLEL` felülírás elutasításra kerül, és a naplóban megjelenik egy `[STRATEGY]` sor, és a futtatás az `CPU_PARALLEL` értéket használja.

Javasolt a beállítást parancsonként elvégezni, nem pedig állandóan, így a normál futtatások továbbra is automatikusan alkalmazkodnak.

***

## Platformspecifikus viselkedés

| Platform | Stratégia | Munkások | Folyamat | Megjegyzések |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (szekvenciális) | Memóriahatékony mód, egyszerre egy kép |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (szekvenciális) | A 12 GB alatti megosztott RAM szekvenciális feldolgozást kényszerít |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (párhuzamos) | Ajánlott peremkészülék — a Jetson 2 munkavégzőre korlátozza |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (párhuzamos) | Maximális peremhálózati teljesítmény (Jetson esetén szintén 2 munkavégzőre korlátozva) |
| **Asztali számítógép 8 GB-os GPU-val** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 munkavégző átfedi egymás I/O-műveleteit, miközben egy szemafor sorba rendezi a GPU-hozzáférést |
| **12 GB-os vagy annál nagyobb GPU-val rendelkező asztali számítógép** | `GPU_PARALLEL` | 2–4 | `fused_gpu` (párhuzamos) | Optimális asztali teljesítmény: 12 GB → 2 munkavégző, 16 GB → 3, 20 GB+ → 4 |
| **Kizárólag CPU-val rendelkező rendszer** | `CPU_PARALLEL` | fizikai magok − 1 (min. 2) | `cpu_fallback` | Nincs szükség GPU-ra, szálkészletet használ |

{% hint style="info" %}
**Jetson egységes memória**: A Jetson eszközök megosztják a GPU és a CPU memóriáját. Egy 16 GB-os Jetson Orin NX ~15,3 GB VRAM-ot jelez, de ez megegyezik az operációs rendszer és a CPU-folyamatok által használt fizikai RAM-mal. Ezért a 16 GB-os vagy annál nagyobb memóriájú Jetson-készülékek ugyanúgy megfelelnek az `GPU_PARALLEL` követelményeinek, mint egy 12 GB-os vagy annál nagyobb memóriájú asztali GPU, mégis 2 munkavégzőre vannak korlátozva – a GPU, a munkavégző folyamatok és a munkavégzőnkénti CUDA-kontextusok mind ugyanabból a megosztott poolból merítenek.
{% endhint %}

### GPU-keret a VRAM alapján (diszkrét GPU-k)

Diszkrét NVIDIA GPU-val rendelkező x86_64-es gazdagépeken az észlelt VRAM határozza meg azt is, hogy a kártya feldolgozási kapacitásának mekkora részét igényelheti, és mekkora méretűre nőhetnek a kötegek:

| Észlelt VRAM | GPU-keret felső határa | Kötegméret-szorzó |
| --- | --- | --- |
| **8 GB felett** | 90% | ×2,0 |
| **6–8 GB** | 85% | ×1,75 |
| **3,5–6 GB** | 80% | ×1,5 |
| **2–3,5 GB** | 75% | ×1,25 |
| **2 GB alatt** | 70% | ×1,0 |

A diszkrét GPU-k csak 0,5 GB-ot tartanak fenn a rendszer számára, mivel nem osztoznak a rendszer RAM-ján. A Jetson-profilok sokkal többet tartanak fenn, és alacsonyabb a felső határuk — lásd az [NVIDIA Jetson útmutatót](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Dinamikus GPU-memória-allokáció

A Chloros
egy [4 szálas feldolgozási folyamatot](processing-pipeline.md) használ:

* **

1. szál** (Észlelés) — Kép betöltése, EXIF-adatok elemzése, célpont-észlelés
* **

2. szál** (Kalibrálás) — Reflektancia-kalibrációs számítás
* **

3. szál** (Feldolgozás) — GPU-alapú debayer, vignettakorrekció, indexszámítás
* **

4. szál** (Exportálás) — Fájlírás, metaadatok beágyazása

Az 1., 2. és 4. szálak kevés GPU-erőforrást igényelnek; a 3. szál viszont nagy terhelést jelent. Ahogy a korábbi feldolgozási szálak befejezik munkájukat, a GPU-kapacitásuk **átosztásra kerül a fennmaradó aktív szálak között**, így a 3. szál a futás előrehaladtával egyre több memóriát kap.

### Allokációs szakaszok

| Szakasz | Aktív szálak | GPU-memória elosztás |
| --- | --- | --- |
| **Korai** | 1, 2, 3, 4 | Az összes szál között felosztva, a legtöbb a 3. szálhoz kerül |
| **Közép-korai** | 2, 3, 4 | Az 1. szál részesedése újraelosztásra kerül |
| **Közép-késő** | 3, 4 | Az 1. és 2. szál részesedése a 3. és 4. szálhoz kerül |
| **Késő** | 3 vagy 4 | Az utolsó aktív szál megkapja a maximális allokációt |

Két szabály szabályozza az értékeket:

* Az a szál, amely **az egyetlen** aktív szál, megkapja a profiljában megadott maximális allokációt.
* Ha egynél több *igényes* GPU-feladat aktív, az egyes igényes feladatok alapallokációját felosztják közöttük (soha nem kerülhet a beállított minimum alá).

A futásidőben ténylegesen használt érték a platformprofil allokációja és a GPU-memóriamonitor élő ajánlása közül a **kisebb**, így egy leterhelt kártya mindig elsőbbséget élvez egy optimista profillal szemben.***

## Textúra-tudatos feldolgozás

A Texture Aware debayer (**csak Chloros
+** — `--debayer texture-aware`) egy olyan AI/ML zajszűrő modellt futtat, amely másolatonként körülbelül 1,75 GB VRAM-ot igényel FP16-ban, így sokkal több GPU-memóriát használ, mint a Standard módszer:

* A **7 GB alatti VRAM-mal**rendelkező rendszerek a textúraérzékeny feldolgozást**szinkron ciklusban, egyszerre egy kép** feldolgozásával végzik — több modellmásolat nem fér el, és a munkavállalói pool csak további versengést okozna
* A **7 GB-nál több VRAM-mal** rendelkező rendszerek párhuzamosan tudják feldolgozni a Texture Aware-t, bár a Standard módszerhez képest csökkentett munkavállalói számmal
* A **Jetson** esetében a Texture Aware mindig egyetlen munkavégzőhöz van rendelve
* **Használható CUDA GPU** hiányában a futtatás a Standard debayert használja, és a háttérrendszer egy `[DEBAYER] … needs a CUDA GPU …` sorral jelzi az okot
* Az alacsony fogyasztású Jetson-készülékeken (Nano, Orin Nano) a CLI
GPU-frekvenciakorlátozást alkalmaz, ha a Texture Aware-t kéri — lásd az [NVIDIA Jetson útmutatót](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)

***

## Hőkezelés (Jetson)

A Jetson-eszközök hőmérsékleti korlátozásoknak vannak kitéve, különösen zárt térben vagy repülőgépeken történő telepítés esetén. Az Chloros
figyeli a Jetson beépített hőmérséklet-érzékelőit, és automatikusan méretezi a kötegméreteket:

| Hőmérséklet | Reakció |
| --- | --- |
| **&lt; 70 °C** | Normál működés — teljes sebesség |
| **70 °C** (Figyelmeztetés) | A kötegméret fokozatosan csökken (100% → 50% 70 °C és 80 °C között) |
| **80 °C** (Kritikus) | Erőteljes teljesítménycsökkentés (50% → 0% 80 °C és 90 °C között) |
| **90 °C** (Leállítás) | A GPU-feldolgozás teljes leállítása |

A megfelelő hűtéssel rendelkező asztali rendszereken a hőterhelés miatti teljesítménycsökkentés ritkán lép működésbe.

***

## Memóriaigény kezelése

A Chloros
a feldolgozás során folyamatosan figyeli a GPU memóriáját, és három szinten reagál.

**Kötegméret meghatározása.** Egy köteg a fenti táblázatokban szereplő platform-szorzóval megszorozott 8 képből indul. A Chloros
ezután ellenőrzi a szabad VRAM-ot, annak 20%-át a PyTorch saját overheadjére tartalékolja, és 12 MP-os képenként körülbelül 100 MB GPU-memóriával számol — a köteg mérete a memóriából származó határérték és a platform alapértéke közül a kisebbik. Soha nem csökken 1 alá.

**Megelőző csökkentés.** **85%-os VRAM-kihasználtság** felett a kötegméreteket csökkentik, mielőtt bármilyen hiba lépne fel.**Szálankénti allokáció-csökkentés.** Ahogy az aktuális kihasználtság emelkedik, az egyes szálak GPU-keretét visszafogják: 80%-os kihasználtság felett ×0,75-re, 90%-os felett ×0,5-re. A monitor sávjai: 70% (konzervatív), 85% (normál működési határ) és 95% (OOM-kockázat).**OOM-visszavonulás és helyreállítás.** Ha mégis memóriahiány lép fel:

* a kötegméret **felére csökken**, és minden egymást követő memóriahiány esetén ismét felére csökken — minden egyes sikeres köteg futtatása egy lépéssel visszavonja ezt a korlátozást
* az aktív szálak allokációi a jelenlegi értékük 70%-ára csökkennek, és az allokátor átvált konzervatív stratégiájára, amely sikeres allokációk sorozata után ismét enyhül
* súlyos terhelés esetén a feldolgozási folyamat az `fused_gpu`-ről az `tiled_gpu`-re vált, végső esetben pedig az `cpu_fallback`-re

**Gazdagép RAM (Jetson).** A feldolgozás előtt a CLI
a képek számából és a debayer módból becsüli a gazdagép memóriájának csúcsigényét, és figyelmeztet, ha a RAM és a fájlalapú swap valószínűleg nem lesz elegendő, kinyomtatva a swap hozzáadásához szükséges pontos parancsokat — lásd az [NVIDIA Jetson útmutatót](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).

***

## A számítási adaptáció figyelemmel kísérése

### Rendszerdiagnosztika

Az `chloros-cli selftest` a leggyorsabb módja annak, hogy ellenőrizzük, mit lát a számítási réteg:

```bash
chloros-cli selftest
```

A 7 ellenőrzés a verziót, a portok elérhetőségét, a háttérrendszer indítását, az `/api/test`-et, a rendszerinformációkat, a zajszűrő modell jelenlétét, valamint a CUDA és a zajszűrő készenlétét fedi le. Az 5. ellenőrzés közvetlenül kinyomtatja a hardver sorát:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

A 7. ellenőrzés kinyomtatja az `CUDA: <bool>, Denoiser: <bool>` értéket — mindkettőnek igaznak kell lennie ahhoz, hogy a Texture Aware egyáltalán használható legyen.

### Háttérrendszer-naplók

A stratégiát és a munkavállalók számát a háttérrendszer választja ki minden futtatás elején — ezeket nem jelzi semmilyen CLI
-banner. Ha valami váratlanul viselkedik (egy GPU-útvonal visszaesése, memóriahiány, vagy egy nem betöltődő zajszűrő), akkor az adott munkamenet háttérnaplójában jelenik meg:

| Platform | Napló helye |
| --- | --- |
| **Linux
/ Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (indításonként egy fájl) |
| **Linux
, CLI
-ből indított háttérrendszer** | szintén `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Élő haladás

A futás során a CLI
szálankénti élő haladást jelenít meg (Észlelés, Elemzés, Feldolgozás, Exportálás), amelyet Server-Sent Events-en keresztül továbbít — ez ad gyakorlati képet arról, hogy a 3. szál jelent-e szűk keresztmetszetet. Lásd [Feldolgozási folyamat](processing-pipeline.md).

***

## Következő lépések

* [Feldolgozási folyamat](processing-pipeline.md) — A 4-szálas folyamatarchitektúra megértése
* [NVIDIA Jetson útmutató](../linux/nvidia-jetson-guide.md) — Jetson-specifikus telepítés és optimalizálás
* [CLI
: Parancssor](../CLI.md) — A CLI
útmutató
* [CLI
-referencia](../reference/cli-reference.md) — A 1.3.0-s verzió parancsainak teljes listája
