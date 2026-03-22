# NVIDIA Jetson útmutató

XPROTX000047 Az NVIDIA Jetson rendszeren futó XPROTX lehetővé teszi a multispektrális képfeldolgozást a periférián – a terepen, drónokon és távoli telepítésekben egyaránt. XPROTX000048 Az XPROTX automatikusan felismeri a Jetson-modellt, és a hardverhez igazítja a feldolgozási stratégiát.

***

## Támogatott Jetson-modellek

| Modell                | RAM            | Feldolgozási stratégia                                   | Ajánlott felhasználás                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 GB megosztott | `GPU_PARALLEL` (4 munkavégző)                            | Maximális teljesítmény, nagy adathalmazok                      |
| **Jetson Orin NX**   | 8–16 GB megosztott  | `GPU_PARALLEL` (3 munkás, 16 GB) / `GPU_SINGLE` (8 GB) | Elsődleges ajánlás légi és terepi telepítéshez |
| **Jetson Orin Nano** | 8 GB megosztott     | `GPU_SINGLE` (1 munkavégző)                               | Belépő szintű peremhálózati számítás                                 |
| **Jetson Nano**      | 4–8 GB megosztott   | `GPU_SINGLE` (1 munkás)                               | Belépő szintű, memóriakorlátozott                          |

{% hint style="info" %}
**A régebbi Jetson modellek** (TX2, TX1, Xavier NX) esetleg nem támogatottak. A teljesítmény a rendelkezésre álló GPU-memóriától és a CUDA-képességektől függően változhat.
{% endhint %}

***

## Követelmények

* **JetPack 6.x** (a legújabb verzió ajánlott)
* **NVIDIA CUDA** (a JetPack része)
* **Chloros+ licenc** (szükséges az CLI/SDK eléréséhez)

## Telepítés

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Az Linux telepítésének általános részleteiről lásd [Linux telepítés](linux-installation.md).

***

## Dinamikus számítási adaptáció a Jetsonon

Az Chloros automatikusan felismeri a Jetson modellt, és kiválasztja az optimális feldolgozási stratégiát. **Nincs szükség kézi beállításra.**

### Hogyan működik

Indításkor az Chloros profilozza a rendszert:

1. **Felismeri a Jetson modellt** az `/proc/device-tree/model` segítségével
2. **Beolvassa a rendelkezésre álló GPU-t/megosztott memóriát**

3.**Kiválaszt egy feldolgozási stratégiát** (`GPU_PARALLEL`, `GPU_SINGLE` vagy `CPU_PARALLEL`)
4. **Automatikusan beállítja a munkavállalók számát, a folyamat típusát és a memóriaallokációt**

### Modellenkénti viselkedés

| Jetson modell                | Stratégia       | Munkások | Folyamat                       | Párhuzamosság |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8 GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (memóriahatékony) | Sorosított  |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Sorosított  |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Sorosított  |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (teljes GPU útvonal)    | Párhuzamos  |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Párhuzamos  |

{% hint style="success" %}
A **Jetson Orin NX 16 GB** az ideális választás a perifériás telepítéshez — az `GPU_PARALLEL` stratégiát alkalmazza 3 párhuzamos munkavállalóval, így valódi párhuzamos GPU-feldolgozást biztosít kompakt formában.
{% endhint %}

A platformok közötti legfőbb különbség a **memória**. A 8 GB megosztott memóriával rendelkező Jetson Nano-nak memóriatakarékos, csempés megközelítéssel kell feldolgoznia a képeket egyenként, míg a 16 GB-os Orin NX a nagyobb átviteli sebességű fúziós csővezeték segítségével egyszerre 3 képet futtathat a GPU-n.

A teljes számítási adaptációs útmutatóért lásd a [Dinamikus számítási adaptáció](../processing-architecture/dynamic-compute-adaptation.md) című részt.

***

## Hőkezelés

A Jetson eszközök hőkezelési tartaléka korlátozott, különösen zárt térben vagy repülőgépeken történő telepítés esetén. Az Chloros automatikus hőmérséklet-figyelést és fojtást tartalmaz:

| Hőmérséklet         | Művelet                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Normál működés — teljes feldolgozási sebesség          |
| **70 °C** (Figyelmeztetés)  | A köteg méretének automatikus csökkentése                   |
| **80 °C** (Kritikus) | Erőteljes fojtás — alacsonyabb párhuzamosság         |
| **90°C** (Leállítás) | A GPU feldolgozásának teljes leállítása — lehűtés szükséges |

{% hint style="warning" %}
**Gondoskodjon megfelelő szellőzésről és hűtésről** a folyamatos feldolgozás érdekében, különösen zárt terepi burkolatokban vagy légi rendszerekben. A hőfojtás csökkenti a feldolgozási teljesítményt a hardver védelme érdekében.
{% endhint %}

***

## Memóriakezelés

A Jetson eszközök **egységes memóriát** használnak — a GPU és a CPU ugyanazt a fizikai RAM-ot osztja meg. Ez azt jelenti, hogy a jelentett VRAM (pl. 15,3 GB az Orin NX 16 GB-os változatán) nem dedikált GPU-memória; azt az operációs rendszer és más folyamatok is használják.

### Swap-ajánlások

Nagy adathalmazok vagy Texture Aware debayer feldolgozás esetén az Chloros swap-terület létrehozását javasolhatja:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Memóriaigény becslés képeként:**

* Standard debayer: ~10 MB képenként
* Texture Aware debayer: ~15 MB képenként

Az Chloros automatikusan kiszámítja a szükséges memóriát az adatkészlet mérete alapján, és figyelmeztet, ha swap használata ajánlott.

### OOM (Out of Memory) tartalék

Ha a feldolgozás során memóriahiányt észlel:

1. Az Chloros automatikusan csökkenti a GPU-munkások számát
2. Átáll az `fused_gpu`-ről az `tiled_gpu` folyamatra (memóriahatékonyabb)
3. A rendszer összeomlása helyett csökkentett átviteli sebességgel folytatja a feldolgozást

***

## Helyszíni telepítés

### Áramellátási szempontok

| Jetson modell     | Tipikus energiafogyasztás | Megjegyzések                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5–10 W              | USB-C vagy hengeres csatlakozó    |
| Jetson Orin Nano | 7–15 W              | DC hengeres csatlakozó          |
| Jetson Orin NX   | 10–25 W             | DC hengeres csatlakozó          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD vagy hengeres csatlakozó |

Tervezze meg az energiaigényét a folyamatos feldolgozáshoz — a csúcs energiafogyasztás a GPU-igényes 3. szál (feldolgozás) során jelentkezik.

### Tárolási ajánlások

* **NVMe SSD** erősen ajánlott arm64-es telepítésekhez
* Az SD-kártyák túl lassúak a feldolgozáshoz — csak rendszerindító adathordozóként használja őket
* A feldolgozott kimenethez a nyers képadatok méretének 2–3-szorosát tervezze be

### Headless működés az SSH segítségével

Az Chloros CLI ideális a headless Jetson-telepítésekhez:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

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
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK a Jetsonon

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
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Ajánlott Jetson rendszerek terepi használatra

Terepi és légi alkalmazásokhoz vegye fontolóra a következő Jetson Orin NX 16 GB hordozólemezeket:

* **Légi/drón**: Rezgésálló (MIL-STD), könnyű (300 g alatti), passzív hűtésű rendszerek
* **Robusztus terepi**: IP67/IP69K vízálló burkolatok PoE GigE kamera csatlakozási lehetőséggel
* **Minimális/költséghatékony**: Fejlesztői készletek kiegészítő burkolatokkal

Vegye fel a kapcsolatot az [MAPIR ügyfélszolgálattal](https://www.mapir.camera/community/contact) a telepítési forgatókönyvéhez illeszkedő konkrét hardverajánlásokért.

***

## Következő lépések

* [Linux telepítés](linux-installation.md) — Általános Linux telepítési részletek
* [Dinamikus számítási adaptáció](../processing-architecture/dynamic-compute-adaptation.md) — Teljes számítási stratégia-referencia
* [Feldolgozási folyamat](../processing-architecture/processing-pipeline.md) — A 4 szálas folyamat megértése
* [CLI : Parancssor](../CLI.md) — Teljes CLI referencia
* [API : Python SDK](../api-python-sdk.md) — Teljes SDK referencia
