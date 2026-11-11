# NVIDIA Jetson útmutató Az NVIDIA Jetsonon futó

Chloros lehetővé teszi a multispektrális képfeldolgozást a periférián — a terepen, drónokon és távoli telepítésekben egyaránt. A Chloros 1.3.0 indításkor felismeri a Jetson-modellt, és az észlelt hardverhez igazítja a feldolgozási stratégiát. **Nincs szükség kézi beállításra.**
{% hint style="info" %}
**Az 1.3.0-s Jetson-csomag a Jetson GPU-ján végzi a feldolgozást.** Az `chloros_1.3.0_arm64_jp6.deb` a JetPack 6-hoz készült, és a JetPack 6 CUDA-alapú PyTorch-verzióját használja, így a feldolgozás – beleértve a Texture Aware debayert is – az Orin GPU-ján fut. A csomag maga nem tartalmazza a CUDA-t: a JetPack 6 által telepített CUDA- és cuDNN-könyvtárakat használja (`sudo apt install nvidia-jetpack` olyan képen, amelyeken ezek hiányoznak). Ha nincs használható CUDA-GPU, a feldolgozás a CPU-n fut, és a Texture Aware a Standard debayert használja.
{% endhint %}

***

## Támogatott Jetson-modellek

| Modell                | RAM            | Feldolgozási stratégia                                     | Ajánlott felhasználás                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB megosztott | `GPU_PARALLEL` (2 munkavégző)                              | Maximális teljesítmény, nagy adathalmazok                      |
| **Jetson Orin NX**   | 8–16 GB megosztott | `GPU_PARALLEL` (2 munkavégző, 16 GB) / `GPU_SINGLE` (8 GB)   | Elsődleges ajánlás terepi telepítéshez              |
| **Jetson Orin Nano** | 8 GB megosztott     | `GPU_SINGLE` (1 munkaszerver, szekvenciális)                     | Belépő szintű peremszámítás                                 |

{% hint style="info" %}
A Linux arm64 csomaghoz **JetPack 6** szükséges, amely a Jetson Orin termékcsaládon elérhető. A régebbi modellek (Nano, TX2, Xavier NX) nem képesek futtatni a JetPack 6-ot, és a jelenlegi csomag nem támogatja őket.
{% endhint %}

***

## Követelmények

* **JetPack 6.x** (a legújabb verzió ajánlott)
* **NVIDIA CUDA és cuDNN** a JetPack 6-ból (`sudo apt install nvidia-jetpack`, ha a képben hiányoznak)
* **Fizetős Chloros+ csomag** — Copper szint vagy magasabb (minden CLI/SDK-hozzáféréshez szükséges; szerveroldalon érvényesítve)

## Telepítés

```bash
# Install the arm64 .deb package (JetPack 6) -- apt, not dpkg -i, so the
# Python SDK's requirements (python3-numpy, python3-requests) come too
sudo apt install ./chloros_1.3.0_arm64_jp6.deb

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.3.0"
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # the .deb installs the SDK

# Run system diagnostics
chloros-cli selftest
```

Az `.deb` telepíti a rendszerhez tartozó Python SDK-t (Python), amely megfelel ennek a buildnek. Csak egy virtualenv-be kell telepíteni a csomagban található wheel fájlt: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` (teljesen offline). Általános Linux-telepítési részletek, fájlhelyek és hibaelhárítás tekintetében lásd a [Linux-telepítés](linux-installation.md) című részt.

{% hint style="info" %}
**Indításkor már nincs szükség kicsomagolásra.** Az 1.3.0-s verziótól kezdve a háttérprogram és a CLI-program kicsomagolatlanul kerülnek telepítésre az `/usr/lib/chloros/` könyvtárba, így indításkor már nem csomagolják ki magukat egy ideiglenes könyvtárba. Az ideiglenes fájlok továbbra is automatikusan az `/mnt/ssd/tmp` mappába kerülnek, ha az létezik; ellenkező esetben az `TMPDIR`-et beállíthatja az NVMe-n található egy útvonalra (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Dinamikus számítási adaptáció a Jetsonon

### Hogyan működik?

Indításkor a Chloros profilozza a rendszert:

1. **Az `/proc/device-tree/model` segítségével felismeri a Jetson modellt**

2.**Beolvassa a rendelkezésre álló megosztott GPU/CPU memóriát** (a Jetson egységes memóriát használ)
3. **Megkérdezi a PyTorch-ot, hogy használhatja-e a CUDA-t** — ha a JetPack 6 CUDA-ja és cuDNN-je telepítve van, akkor igen
4. **Automatikusan kiválasztja a feldolgozási stratégiát** (`GPU_PARALLEL`, `GPU_SINGLE` vagy `CPU_PARALLEL`) és a munkavállalók számát

A döntést a **teljes megosztott RAM** határozza meg, nem a modell neve:

* **12 GB alatti teljes RAM**(minden 8 GB-os Jetson):**1 munkavégző — szándékosan szekvenciális feldolgozás**. A memória túl szűkös a párhuzamos munkavégzőkhez, ezért a képeket egyenként dolgozza fel. A**8 GB-os vagy annál kisebb** RAM-mal rendelkező Jetson-okon a 3. szál teljesen kihagyja a munkavégző-készletet, és a képekhez tartozó feladatokat a folyamaton belül hajtja végre. A háttérnapló ezt a stratégiát `GPU_SINGLE` néven jelöli.
* **12 GB vagy annál több**(Orin NX 16 GB, AGX Orin): az egységes memória alkalmas az `GPU_PARALLEL` használatára, de a munkavégzők száma**a Jetsonon 2-re van korlátozva** — a GPU, a munkavégző folyamatok RAM-ja és a munkavégzőnkénti CUDA-kontextusok mind ugyanabból a megosztott poolból merítenek, így több munkavégző esetén fennáll a memóriahiányos hibák kockázata.
* **Nincs használható CUDA** (hiányoznak a JetPack CUDA-könyvtárai): `CPU_PARALLEL` 12 GB vagy annál több memóriával, az alábbiakban egyenként egy-egy kép.

Az automatikus választást felülírhatja az `CHLOROS_STRATEGY` környezeti változóval — lásd [Dinamikus számítási adaptáció](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Modellonkénti viselkedés

| Jetson-modell                | Stratégia       | Munkások | Végrehajtás                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Szekvenciális, folyamaton belüli hurok (`tiled_gpu` memóriahiány esetén) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Szekvenciális, folyamaton belüli ciklus                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Párhuzamos munkavégző folyamatok, `fused_gpu` útvonal  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2       | Párhuzamos munkavégző folyamatok, `fused_gpu` útvonal  |

A platformok közötti legfőbb különbség a **memória**. Egy 8 GB-os Jetson nagy terhelés esetén memóriahatékony, csempés megközelítéssel, egyenként kell feldolgoznia a képeket, míg egy 16 GB-os vagy annál nagyobb Orin a nagyobb átviteli sebességű fúziós csővezeték segítségével egyszerre két képet is feldolgozhat a GPU-n.

### Modellenkénti GPU-keret

Minden Jetson-modell rendelkezik egy hardverprofiljal is, amely korlátozza, hogy a megosztott pool feldolgozása mennyi erőforrást igényelhet, és skálázza a kötegméreteket:

| Modell | GPU-keret felső határa | Kötegméret-szorzó | Rendszer/kijelző számára fenntartott |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75% | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80% | ×1,5 | 4,0 GB |

Az észlelt RAM-kapacitás módosítja a profilt: egy **16 GB-os vagy annál nagyobb** RAM-mal rendelkező Jetson esetében a kötegszorzó értéke ×1,2-re emelkedik. A szorzók alkalmazása előtti alapkötegméret 8 kép.

A teljes számítási adaptációs útmutatóért lásd a [Dinamikus számítási adaptáció](../processing-architecture/dynamic-compute-adaptation.md) című dokumentumot.

***

## A GPU frekvenciájának korlátozása a Texture Aware esetében a Nano és Orin Nano modelleken

A Texture Aware debayer GPU-alapú neurális hálózati következtetést futtat, ami **túláram-figyelmeztetéseket**válthat ki az alacsony fogyasztású Jetson modelleken (10–15 W-os osztály) teljes GPU-órajel mellett. Ha az `--debayer texture-aware` parancsot adja meg egy**Jetson Nano vagy Orin Nano** eszközön, a CLI ellenőrzi a GPU maximális frekvenciáját, és **510 MHz**-re (510000000) korlátozza azt, ha az jelenleg ennél magasabb:

* Ha a CLI írhat a GPU frekvenciájának sysfs-csomópontjába, a korlátozás **automatikusan alkalmazásra kerül**, és a rendszer megjeleníti a visszaigazolást.
* Ha nem (root jogosultság szükséges), aCLI
kijelzi a pontos `sudo` parancsot a korlátozás kézi alkalmazásához, egy pillanatig vár, hogy elolvashassa, majd folytatja a műveletet — a feldolgozás továbbra is fut, de túláram-figyelmeztetéseket jelezhet.

Ha a feldolgozás előtt saját maga szeretné alkalmazni a korlátozást:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

A nagyobb teljesítményű modellek (Orin NX 25W, AGX Orin 60W) teljes GPU-sebességgel futnak; nincs korlátozás alkalmazva. A Standard debayer egyetlen modellen sem váltja ki a korlátozást.

{% hint style="info" %}
**A Texture Aware a Jetsonon mindig egyszerre csak egy képet dolgoz fel.** Minden munkavégzőnek szüksége lenne saját CUDA-kontextusra (~1 GB), valamint a zajszűrő modell saját másolatára, amit az egységes memória nem tud biztosítani — ezért a Jetsonon a Texture Aware útvonal egyetlen munkavégzőhöz van rögzítve, a GPU-hozzáférés sorosítva. A Texture Aware várhatóan jelentősen lassabb lesz, mint a Standard bármelyik Jetsonon. A „Texture Aware” a Survey3 képekre vonatkozik: a LATTICE-felvételek mindig a „Standard” debayert kapják. Használható CUDA GPU hiányában a futtatás a „Standard” módot használja, és a háttérrendszer egy `[DEBAYER] … needs a CUDA GPU …` sorral naplózza az okot (az `chloros-cli --verbose`-szel együtt látható).
{% endhint %}

***

## Hőkezelés

A Jetson-eszközök hőkezelési tartaléka korlátozott, különösen zárt térben vagy légi alkalmazások esetén. Az Chloros figyeli a SoC hőmérsékletét, és automatikusan korlátozza a kötegméreteket:

| Hőmérséklet         | Művelet                                            |
| ------------------- | ------------------------------------------------- |
| **< 70 °C**          | Normál működés — teljes feldolgozási sebesség          |
| **70 °C** (Figyelmeztetés) | A kötegméret fokozatosan csökken (100% → 50% 70 °C és 80 °C között) |
| **80 °C** (Kritikus) | Erőteljes korlátozás (50% → 0% 80 °C és 90 °C között) |
| **90 °C** (Leállítás) | A GPU feldolgozásának teljes leállítása — lehűtés szükséges |

{% hint style="warning" %}
**Gondoskodjon megfelelő szellőzésről és hűtésről** a folyamatos feldolgozás érdekében, különösen zárt terepi burkolatokban vagy repülőgépeken. A hőterhelés miatti teljesítménycsökkentés a hardver védelme érdekében csökkenti a feldolgozási teljesítményt.
{% endhint %}

***

## Memóriakezelés

A Jetson eszközök **egységes memóriát** használnak — a GPU és a CPU ugyanazt a fizikai RAM-ot osztja meg. A jelentett VRAM (pl. ~15,3 GB egy Orin NX 16 GB-os változatán) nem dedikált GPU-memória; ez ugyanaz a RAM, amelyet az operációs rendszer és minden más folyamat is használ.

### Swap-figyelmeztetés és ajánlások

A Jetsonon történő feldolgozás előtt a CLI megszámolja a bemeneti mappában található RAW-képeket (`.tif`, `.tiff`, `.raw`, `.dng` — a JPG-előnézeteket nem számolja), megbecsüli a futtatáshoz szükséges maximális memóriaterületet, és **a futtatás megkezdése előtt figyelmeztet**, ha a RAM + swap valószínűleg nem lesz elegendő. A figyelmeztetés címe `LOW MEMORY WARNING - Jetson Detected`, kinyomtatja a képek számát, a RAM-ot, az aktuális swap-területet és a becsült csúcsértéket, majd megadja a pontos, a (soha nem kisebb, mint 8 GB). Néhány másodpercre szünetet tart, hogy az üzenet ne vesszen el a görgetési történetben, majd folytatódik a feldolgozás.**A figyelmeztetés által használt memória-becslések:**

| Debayer mód | Alap | Képenként |
| --- | --- | --- |
| Standard | ~1,5 GB | ~10 MB |
| Textúraérzékeny | ~2,5 GB (modell + Python futási környezet) | ~15 MB |

A figyelmeztetés akkor jelenik meg, ha a becsült csúcsérték meghaladja a RAM + swap méretét, levonva belőle 1 GB biztonsági tartalékot, és csak a **fájlalapú** swap-ot veszi figyelembe — egy kizárólag zram-ot használó konfiguráció esetén is megjelenik a figyelmeztetés.

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

A feldolgozás során a Chloros figyeli a GPU memóriáját, és összeomlás helyett zökkenőmentesen csökkenti a teljesítményt:

1. Amikor a GPU-memória kihasználtsága meghaladja a **85%-ot**, a kötegméreteket megelőzően csökkentik
2. Ha mégis memóriahiány lép fel, a kötegméret **felére csökken**, és minden további OOM esetén ismét felére csökken; minden következő sikeres köteg feldolgozása egy lépéssel visszavonja ezt a korlátozást
3. Tartós terhelés esetén a feldolgozási folyamat az `fused_gpu`-ről átvált a memóriatakarékos `tiled_gpu` útvonalra, végső esetben pedig a CPU-feldolgozásra

Mivel a GPU megosztja a Jetson RAM-ját, a RAM és a swap együttesen továbbra is korlátozza a futást — a fenti swap-figyelmeztetés az a jelzés, amelyre figyelni kell.

***

## Helyszíni telepítés

### Áramellátási szempontok

| Jetson-modell     | Jellemző áramfelvétel | Megjegyzések                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | DC hengeres csatlakozó          |
| Jetson Orin NX   | 10–25 W             | DC hengeres csatlakozó          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD vagy hengeres csatlakozó |

Tervezze meg az energiaigényét a folyamatos feldolgozáshoz — a csúcsáramfelvétel a GPU-igényes 3. szál (feldolgozás) során jelentkezik.

### Tárolási ajánlások

* **NVMe SSD** erősen ajánlott arm64-es telepítésekhez
* Az SD-kártyák túl lassúak a feldolgozáshoz — csak rendszerindító adathordozóként használja őket
* A feldolgozott kimenethez a nyers képadatok méretének 2–3-szorosát tervezze be

### Headless működés SSH-en keresztül A

Chloros CLI ideális a headless Jetson-telepítésekhez:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Állandóan aktív háttérszolgáltatás a LATTICE / DAQ-E időszinkronizálásához

Ha a Jetson headless módon vezérli a LATTICE kamerákat vagy a DAQ-E fényérzékelőket, engedélyezze a háttér systemd szolgáltatást **a saját felhasználói fiókjával**, hogy a PTP grandmaster folyamatosan futhasson, és az SDK-parancsfájlok működő háttérrendszert találjanak (az egység telepítve van, de alapértelmezés szerint nincs engedélyezve):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
chloros-cli time-sync status
```

A háttérszolgáltatás ezután az Ön nevében fut: az általa írt rögzítések az Önéi, és az Ön `chloros-cli login`-jét használja. A részleteket, beleértve a root `chloros-backend.service`-et és azt, hogy a csomag hogyan teszi a 319/320-as PTP-portokat root jogosultság nélkül is hozzárendelhetővé, lásd a [Linux-telepítés](linux-installation.md#always-on-ptp-for-headless-hosts) című részben.

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

Az `chloros-cli process` nem nulla kóddal lép ki, ha egy termékeket kérő futtatás nem ír képeket, így a systemd hibaállapota értelmezhető a felügyelet szempontjából.

Párosítsuk egy systemd időzítővel az ütemezett feldolgozáshoz:

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
    --indices NDVI GNDVI OSAVI
```

### Python SDK a Jetsonon

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "GNDVI", "OSAVI"],
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
        --indices NDVI OSAVI
    echo "Completed $name"
done
```

***

## Ajánlott Jetson rendszerek terepi használatra

Terepi és légi alkalmazásokhoz vegye fontolóra a következő Jetson Orin NX 16 GB hordozólemezeket:

* **Légi/drón**: Rezgésálló (MIL-STD), könnyű (300 g alatt), passzív hűtésű rendszerek
* **Robusztus terepi használat**: IP67/IP69K vízálló burkolatok PoE GigE kameracsatlakozással
* **Minimális/költséghatékony**: Fejlesztői készletek kiegészítő burkolatokkal

Vegye fel a kapcsolatot a [MAPIR ügyfélszolgálattal](https://www.mapir.camera/community/contact), ha konkrét hardverajánlásra van szüksége a telepítési forgatókönyvéhez.

***

## Következő lépések

* [Linux telepítés](linux-installation.md) — Általános Linux telepítési részletek
* [Dinamikus számítási adaptáció](../processing-architecture/dynamic-compute-adaptation.md) — Teljes számítási stratégia-áttekintés
* [Feldolgozási folyamat](../processing-architecture/processing-pipeline.md) — A 4-szálas folyamat megértése
* [CLI: Parancssor](../CLI.md) — A CLI útmutató
* [API: Python SDK](../api-python-sdk.md) — Az SDK útmutató
* [CLI-referencia](../reference/cli-reference.md) és [SDK-referencia](../reference/sdk-reference.md) — A 1.3.0-s verzió teljes parancs- és API-listája
