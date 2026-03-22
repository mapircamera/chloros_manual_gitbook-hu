# Linux telepítése

Az Chloros az Linux számára `.deb` csomagok formájában kerül forgalomba, amelyek az CLI-et és a háttérprogramot telepítik. Az Python SDK külön telepíthető a pip segítségével.

***

## Linux amd64 (x86_64)

### Rendszerkövetelmények

| Követelmény | Minimális | Ajánlott |
| --- | --- | --- |
| **Disztribúció** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Processzor** | x86_64 (Intel/AMD) | Intel Core i7 vagy jobb |
| **Memória (RAM)** | 8 GB | 16 GB vagy több |
| **Grafikus kártya** | Nincs (CPU-feldolgozás) | NVIDIA GPU 4 GB+ VRAM-mal |
| **Tárhely** | 2 GB szabad hely | SSD 10 GB+ szabad hellyel |
| **Python** | Python 3.7+ (az SDK-hez) | Python 3.10+ |

### Telepítés

Töltse le az `.deb` csomagot, és telepítse:

```bash
sudo dpkg -i chloros-amd64.deb
```

Ellenőrizze a telepítést:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Rendszerkövetelmények

| Követelmény | Minimális | Ajánlott |
| --- | --- | --- |
| **Platform** | NVIDIA Jetson JetPack 6-tal | Jetson Orin NX 16 GB vagy AGX Orin |
| **JetPack** | JetPack 6.x | Legújabb JetPack 6 |
| **Memória (RAM)** | 8 GB (megosztott GPU/CPU) | 16 GB+ megosztott |
| **Tárolóhely** | 2 GB szabad hely | NVMe SSD 10 GB+ szabad hellyel |
| **Python** | Python 3.7+ (SDK esetén) | Python 3.10+ |

### Telepítés

Töltse le a JetPack 6 `.deb` csomagot, és telepítse:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Ellenőrizze a telepítést:

```bash
chloros-cli --version
```

A hőkezelést és a helyszíni telepítést is magában foglaló részletes Jetson beállításokról az [NVIDIA Jetson útmutatóban](nvidia-jetson-guide.md) olvashat.

***

## Python SDK telepítés (minden Linux)

Az Python SDK külön telepíthető a pip segítségével, és mind az amd64, mind az arm64 architektúrán működik:

```bash
pip install chloros-sdk
```

Az opcionális progress streaming támogatás beépítéséhez:

```bash
pip install chloros-sdk[progress]
```

Ellenőrizze az SDK-et:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Az `.deb` csomag telepíti az Chloros CLI és a háttérprogramot. Az Python SDK egy különálló pip csomag, amely egy helyi HTTP API-en keresztül kommunikál a háttérrel.
{% endhint %}

***

## Konfigurációs könyvtárak

Az Chloros az Linux-en az [XDG alapkönyvtár specifikációt](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html) követi:

| Cél | Linux Útvonal | Windows Egyenérték |
| --- | --- | --- |
| **Konfiguráció** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Adatok / Projektek** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Gyorsítótár / Hitelesítő adatok** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## A háttérprogramok helye

Az `.deb` csomag a háttérprogramot egy szabványos helyre telepíti. Az CLI és az SDK automatikusan felismeri a háttérprogram elérési útját:

| Telepítési módszer | A háttérprogram elérési útja |
| --- | --- |
| `.deb` csomag | `/usr/lib/chloros/chloros-backend` |
| Kézi / egyéni | `/opt/mapir/chloros/backend/chloros-backend` |

A háttérprogram elérési útját felülírhatja az `--backend-exe` CLI jelzővel vagy az `backend_exe` SDK konstruktorparaméterrel.

***

## Első beállítás

### 1. Aktiválja a licencét

Az Chloros+ licenc szükséges az CLI és az SDK hozzáféréshez:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Ellenőrizze a licenc állapotát

```bash
chloros-cli status
```

### 3. Feldolgozza az első adatkészletét

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Futtassa a rendszerdiagnosztikát

Ellenőrizze, hogy a rendszere megfelelően van-e konfigurálva:

```bash
chloros-cli selftest
```

Ez 7 diagnosztikai ellenőrzést futtat, beleértve a verziót, a háttérprogram indítását, az API kapcsolatot és a CUDA/GPU rendelkezésre állását.

***

## Bash szkript példák

### Több adatkészlet feldolgozása

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Feldolgozás egyéni beállításokkal

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Automatizált feldolgozás Cron segítségével

Adja hozzá a crontab-hoz (`crontab -e`), hogy az új adatkészleteket automatikusan feldolgozza:

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK példa

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Hibaelhárítás

### CLI nem található a telepítés után

Ha az `chloros-cli` nem található az `.deb` csomag telepítése után:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Hozzáférés megtagadva

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### A háttérprogram indítása sikertelen

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

### CUDA nem észlelhető

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Hiányzó megosztott könyvtárak

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Frissítés Chloros-en Linux-en

Használja a beépített frissítési parancsot a frissítések ellenőrzéséhez és telepítéséhez:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Következő lépések

* [NVIDIA Jetson útmutató](nvidia-jetson-guide.md) — Jetson-specifikus optimalizálás és telepítés
* [CLI : Parancssor](../CLI.md) — Teljes CLI parancsreferencia
* [API : Python SDK](../api-python-sdk.md) — Teljes SDK referencia
* [Dinamikus számítási adaptáció](../processing-architecture/dynamic-compute-adaptation.md) — Hogyan alkalmazkodik az Chloros a hardveréhez
