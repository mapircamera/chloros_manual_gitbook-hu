# NVIDIA Jetson útmutató Az NVIDIA Jetsonon futó „

Chloros

” lehetővé teszi a multispektrális képfeldolgozást a periférián – a terepen, drónokon és távoli telepítésekben egyaránt. Az „Chloros

” 1.2.0 verziója indításkor felismeri a Jetson modellt, és az észlelt hardverhez igazítja a feldolgozási stratégiát. **Nincs szükség kézi beállításra.**

***

## Támogatott Jetson-modellek

| Modell                | RAM            | Feldolgozási stratégia                                     | Ajánlott felhasználás                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB megosztott | `GPU_PARALLEL` (2 munkavégző)                              | Maximális teljesítmény, nagy adathalmazok                      |
| **Jetson Orin NX**   | 8–16 GB megosztott | `GPU_PARALLEL` (2 munkavégző, 16 GB) / `GPU_SINGLE` (8 GB)   | Elsődleges ajánlás légi és terepi alkalmazásokhoz |
| **Jetson Orin Nano** | 8 GB megosztott     | `GPU_SINGLE` (1 munkaszerver, szekvenciális)                     | Belépő szintű peremszámítás                                 |

{% hint style="info" %}
Az „Linux

” arm64 csomaghoz **JetPack 6** szükséges, amely a Jetson Orin termékcsaládon elérhető. A régebbi modellek (Nano, TX2, Xavier NX) nem képesek futtatni a JetPack 6-ot, és a jelenlegi csomag nem támogatja őket.
{% endhint %}

***

## Követelmények

* **JetPack 6.x** (a legújabb verzió ajánlott)
* **NVIDIA CUDA** (a JetPack része)
* **FizetősChloros

+ csomag** — Copper szint vagy magasabb (mindenCLI

/SDK

hozzáféréshez szükséges; szerveroldalon érvényesítve)

## Telepítés

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

ALinux

általános telepítési részleteiről, a fájlok helyéről és a hibaelhárításról lásd a [Linux

Telepítés](linux-installation.md) című részt.

{% hint style="info" %}
**Helyezze a kicsomagolási könyvtárat gyors tárolóra.** A lefordított bináris fájlok minden indításkor kicsomagolják magukat egy ideiglenes könyvtárba — ez SD-kártyáról rendkívül lassú. AzChloros

automatikusan az `/mnt/ssd/tmp` könyvtárat használja, ha az létezik; ellenkező esetben állítsd be az `TMPDIR`-et az NVMe-n található elérési útra (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dinamikus számítási adaptáció a Jetsonon

### Hogyan működik?

Induláskor azChloros

profilozza a rendszert:

1. **Felismeri a Jetson modellt** az `/proc/device-tree/model` segítségével
2. **Beolvassa a rendelkezésre álló megosztott GPU/CPU memóriát** (a Jetson egységes memóriát használ)
3. **Kiválaszt egy feldolgozási stratégiát** (`GPU_PARALLEL`, `GPU_SINGLE` vagy `CPU_PARALLEL`)
4. **Automatikusan beállítja a munkaszálak számát, a feldolgozási csatorna típusát és a memóriaallokációt**A döntést a**teljes megosztott RAM** határozza meg, nem a modell neve:

* **12 GB alatti teljes RAM**(minden 8 GB-os Jetson): `GPU_SINGLE`**1 munkavégzővel — szándékosan szekvenciális feldolgozás**. A memória túl szűkös a párhuzamos munkavégzők számára, ezért a képeket egyenként dolgozzák fel. A**8 GB-os vagy annál kisebb** Jetson-készülékeken a 3. szál teljesen kihagyja a munkavégzői poolt, és a képekhez tartozó feladatokat a folyamaton belül hajtja végre.
* **12 GB vagy annál több**(Orin NX 16 GB, AGX Orin): az egységes memória alkalmas az `GPU_PARALLEL` használatára, de a munkavégzők száma**a Jetsonon 2-re van korlátozva** — a GPU, a munkavégző folyamatok RAM-ja és a munkavégzőnkénti CUDA-kontextusok mind ugyanabból a megosztott poolból merítenek, így több munkavégző esetén fennáll a memóriahiányos hibák kockázata.

Az automatikus választást felülírhatja az `CHLOROS_STRATEGY` környezeti változóval — lásd [Dinamikus számítási adaptáció](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Modell-specifikus viselkedés

| Jetson-modell                | Stratégia       | Munkavállalók | Végrehajtás                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Szekvenciális, folyamaton belüli ciklus (`tiled_gpu` memóriahiány esetén) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Szekvenciális, folyamaton belüli ciklus                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Párhuzamos munkavégző folyamatok, `fused_gpu` útvonal  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2       | Párhuzamos munkavégző folyamatok, `fused_gpu` útvonal  |

A platformok közötti legfőbb különbség a **memória**. Egy 8 GB-os Jetson nagy terhelés esetén memóriahatékony, csempés megközelítéssel, egyenként kell feldolgoznia a képeket, míg egy 16 GB-os vagy annál nagyobb Orin a nagyobb átviteli sebességű, összevont feldolgozási csatornát használva egyszerre két képet is feldolgozhat a GPU-n.

### Modellenkénti GPU-keret

Minden Jetson-modell rendelkezik egy hardverprofiljal is, amely korlátozza, hogy a megosztott memóriakészletből mennyi feldolgozási kapacitást igényelhet, és skálázza a kötegméreteket:

| Modell | GPU-keret felső határa | Kötegméret-szorzó | Rendszer/kijelző számára fenntartott |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75% | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80% | ×1,5 | 4,0 GB |

Az észlelt RAM-kapacitás alapján a profil módosul: ha egy Jetson **16 GB-ot vagy annál többet** jelent, a kötegszorzója ×1,2-re emelkedik. A szorzók alkalmazása előtti alapkötegméret 8 kép.

A számítási adaptációval kapcsolatos teljes útmutatót lásd a [Dinamikus számítási adaptáció](../processing-architecture/dynamic-compute-adaptation.md) című dokumentumban.

***

## A Texture Aware GPU-frekvencia-korlátozása a Nano és az Orin Nano esetében

A Texture Aware debayer GPU-alapú neurális hálózati következtetést futtat, ami **túláram-figyelmeztetéseket**válthat ki az alacsony fogyasztású Jetson-modelleken (10–15 W-os osztály) teljes GPU-órajel-sebesség mellett. A**Jetson Nano vagy Orin Nano** eszközön a Texture Aware feldolgozás megkezdése előtt aChloros

ellenőrzi a GPU maximális frekvenciáját, és ha az jelenleg magasabb, akkor **510 MHz**-re (510000000) korlátozza:

* Ha aCLI

képes írni a GPU frekvenciájának sysfs-csomópontjába, a korlátozás **automatikusan alkalmazásra kerül**, és a rendszer megjeleníti a megerősítést.
* Ha nem (root jogosultság szükséges), azCLI

kinyomtatja a pontos `sudo` parancsot a korlátozás kézi alkalmazásához, vár egy pillanatot, hogy elolvashassa, majd folytatja a feldolgozást — a feldolgozás továbbra is fut, de túláram-figyelmeztetéseket jeleníthet meg.

Ha a feldolgozás előtt saját maga szeretné beállítani a korlátot:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

A nagyobb teljesítményű modellek (Orin NX 25W, AGX Orin 60W) teljes GPU-sebességgel futnak; ezeknél nincs korlát. A Standard debayer egyetlen modellen sem váltja ki a korlátot.

{% hint style="info" %}
**A Jetsonon a Texture Aware mindig egyszerre csak egy képet dolgoz fel.** Minden munkavégzőnek szüksége lenne saját CUDA-kontextusra (~1 GB), valamint a zajszűrő modell saját példányára, amit az egységes memória nem tud biztosítani — ezért a Jetsonon a Texture Aware útvonal egyetlen munkavégzőhöz van rögzítve, a GPU-hozzáférés sorosítva. Bármely Jetsonon a Texture Aware várhatóan jelentősen lassabb lesz, mint a Standard.
{% endhint %}

***

## Hőkezelés

A Jetson-eszközök hőkezelési tartaléka korlátozott, különösen zárt térben vagy repülőgépeken történő alkalmazás esetén. A „Chloros

” funkció figyeli a SoC hőmérsékletét, és automatikusan korlátozza a kötegméreteket:

| Hőmérséklet         | Művelet                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normál működés — teljes feldolgozási sebesség          |
| **70 °C** (Figyelmeztetés) | A kötegméret fokozatosan csökken (100% → 50% 70 °C és 80 °C között) |
| **80 °C** (Kritikus) | Erőteljes teljesítménycsökkentés (50% → 0% 80°C és 90°C között) |
| **90°C** (Leállás) | A GPU feldolgozásának teljes leállítása — lehűtés szükséges |

{% hint style="warning" %}
**Gondoskodjon megfelelő szellőzésről és hűtésről** a folyamatos feldolgozás érdekében, különösen zárt terepi burkolatokban vagy repülőgépeken. A hőterhelés miatti teljesítménycsökkentés a hardver védelme érdekében csökkenti a feldolgozási teljesítményt.
{% endhint %}

***

## Memóriakezelés

A Jetson-eszközök **egységes memóriát** használnak — a GPU és a CPU ugyanazt a fizikai RAM-ot osztja meg. A jelentett VRAM (pl. ~15,3 GB egy Orin NX 16 GB-os változatán) nem dedikált GPU-memória; ez ugyanaz a RAM, amelyet az operációs rendszer és minden más folyamat is használ.

### Swap-figyelmeztetés és ajánlások

A Jetsonon történő feldolgozás előtt azCLI

megszámolja a bemeneti mappában található RAW-képeket (`.tif`, `.tiff`, `.raw`, `.dng` — a JPG-előnézeteket nem számolja), megbecsüli a futtatáshoz szükséges maximális memóriaterületet, és **a futtatás megkezdése előtt figyelmeztet**, ha a RAM és a swap együttesen valószínűleg nem lesz elegendő. A figyelmeztetés címe `LOW MEMORY WARNING - Jetson Detected`, kinyomtatja a képek számát, a RAM-ot, az aktuális swap-ot és a becsült csúcsértéket, majd megadja a projekthez méretezett pontos `fallocate` / `chmod` / `mkswap` / `swapon` parancsokat adja meg, amelyek mérete a projekthez igazodik (soha nem kisebb, mint 8 GB). Néhány másodpercre szünetet tart, hogy az üzenet ne vesszen el a görgetési történetben, majd folytatódik a feldolgozás.**A figyelmeztetés által használt memória-becslések:**

| Debayer mód | Alap | Képenként |
| --- | --- | --- |
| Standard | ~1,5 GB | ~10 MB |
| Textúraérzékeny | ~2,5 GB (modell + „Python

” futási idő) | ~15 MB |

A figyelmeztetés akkor lép működésbe, ha a becsült csúcsérték meghaladja a RAM + swap méretét, levonva belőle 1 GB-os biztonsági tartalékot, és csak a **fájlalapú** swap-ot veszi figyelembe — egy kizárólag zram-ot használó beállítás esetén is megjelenik a figyelmeztetés.

A swap manuális hozzáadása (példa: 8 GB):

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

### OOM (Out of Memory) kezelése

A feldolgozás során azChloros

figyeli a GPU memóriáját, és összeomlás helyett fokozatosan csökkenti a teljesítményt:

1. Amikor a GPU-memória kihasználtsága meghaladja a **85%**-ot, a kötegméreteket megelőzően csökkentik
2. Ha mégis memóriahiány lép fel, a kötegméret **felére csökken**, és minden további OOM esetén ismét felére csökken; minden azt követő sikeres köteg feldolgozása egy lépéssel visszavonja ezt a korlátozást
3. Tartós terhelés esetén a feldolgozási folyamat az `fused_gpu` útvonalról a memóriatakarékos `tiled_gpu` útvonalra vált át, végső esetben pedig a CPU-n történő feldolgozásra

***

## Helyszíni telepítés

### Teljesítménybeli szempontok

| Jetson-modell     | Jellemző energiafogyasztás | Megjegyzések                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | DC hengeres csatlakozó          |
| Jetson Orin NX   | 10–25 W             | DC hengeres csatlakozó          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD vagy hengeres csatlakozó |

Tervezze meg az energiaigényét a folyamatos feldolgozáshoz — a csúcs energiafogyasztás a GPU-igényes 3. szál (feldolgozás) során jelentkezik.

### Tárolási ajánlások

* **NVMe SSD** erősen ajánlott arm64-es telepítésekhez
* Az SD-kártyák túl lassúak a feldolgozáshoz — kizárólag rendszerindító adathordozóként használja őket
* A feldolgozott kimeneti adatok méretét a nyers képadatok méretének 2–3-szorosára tervezze

### Headless üzemeltetés aSSH

oldalon keresztül

Chloros

ACLI

ideális a headless Jetson-telepítésekhez:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Állandóan aktív háttérszolgáltatás a LATTICE / DAQ-E időszinkronizáláshoz

Ha a Jetson fej nélküli üzemmódban vezérli a LATTICE kamerákat vagy a DAQ-E fényérzékelőket, engedélyezze a systemd háttérszolgáltatást, hogy a PTP grandmaster folyamatosan futhasson (az egység telepítve van, de alapértelmezés szerint nincs engedélyezve):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

A részleteket, beleértve azt is, hogy a csomag hogyan teszi a 319/320-as PTP-portokat root jogosultság nélkül is hozzárendelhetővé, a [Linux

telepítési útmutatóban](linux-installation.md#always-on-ptp-for-headless-hosts) találja meg.

### Automatizált feldolgozás a systemd segítségével

Hozzon létre egy systemd szolgáltatást az automatizált feldolgozáshoz:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

Az `chloros-cli process` nem nulla kóddal lép ki, ha egy termékeket kérő futtatás nem ír képfájlokat, így a systemd hibaállapota hasznos a felügyelet szempontjából.

Párosítsa egy systemd időzítővel az ütemezett feldolgozáshoz:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Példa munkafolyamatokra

### Alapvető Jetson-feldolgozás

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI
```

###Python

SDK

a Jetsonon

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Több repülés kötegelt feldolgozása

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Ajánlott Jetson-rendszerek terepi használatra

Terepi és légi alkalmazásokhoz vegye fontolóra a következő Jetson Orin NX 16 GB hordozólemezeket:

* **Légi/drón**: Rezgésálló (MIL-STD), könnyű (300 g alatt), passzív hűtésű rendszerek
* **Robusztus terepi használat**: IP67/IP69K vízálló burkolatok PoE GigE kameracsatlakozással
* **Minimális/költséghatékony**: Fejlesztői készletek kiegészítő burkolatokkal

Vegye fel a kapcsolatot a [MAPIR

ügyfélszolgálattal](https://www.mapir.camera/community/contact), ha konkrét hardverajánlásra van szüksége a telepítési forgatókönyvéhez.

***

## Következő lépések

* [Linux

telepítése](linux-installation.md) — Általános részletek aLinux

telepítéséről
* [Dinamikus számítási adaptáció](../processing-architecture/dynamic-compute-adaptation.md) — Teljes számítási stratégia-referencia
* [Feldolgozási folyamat](../processing-architecture/processing-pipeline.md) — A 4 szálas folyamat megértése
* [CLI

: Parancssor](../CLI.md) — Az „CLI

” útmutató
* [API

:Python

SDK

](../api-python-sdk.md) — Az „SDK

” útmutató
* [CLI

Referencia](../reference/cli-reference.md) és [SDK

Referencia](../reference/sdk-reference.md) — A 1.2.0-s verzióhoz tartozó teljes parancs- ésAPI

-lista
