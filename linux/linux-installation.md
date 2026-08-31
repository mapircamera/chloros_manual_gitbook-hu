# Linux telepítése

Az Chloros az Linux számára `.deb` csomagként kerül forgalomba, amely az CLI-et és a háttérszervert telepíti. Az Python SDK egy különálló pip-csomag (amely az `.deb`-ben is megtalálható, verziójával egyező wheel formájában).

A csomagfájlok neve tartalmazza a verziót és az architektúrát: `chloros_1.2.0_amd64.deb` az x86_64-hez, és `chloros_1.2.0_arm64_jp6.deb` a JetPack 6 Jetson-összeállításokhoz. Az alábbi parancsokban helyettesítse be a ténylegesen letöltött fájl nevét.

***

## Linux amd64 (x86_64)

### Rendszerkövetelmények

| Követelmény | Minimális | Ajánlott |
| --- | --- | --- |
| **Disztribúció** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Processzor** | x86_64 (Intel/AMD) | Intel Core i7 vagy jobb |
| **Memória (RAM)** | 8 GB | 16 GB vagy több |
| **Grafikus kártya** | Nincs (CPU-feldolgozás) | NVIDIA GPU 4 GB vagy több VRAM-mal (12 GB vagy több feloldja az `GPU_PARALLEL` funkciót, 7 GB vagy több kikapcsolja a Texture Aware funkciót az egyképes útvonalon) |
| **Tárhely** | 2 GB szabad hely | SSD 10 GB vagy több szabad hellyel |
| **Python** | Python 3.7 vagy újabb (az SDK-hez) | Python 3.10+ |

> **Az Ubuntu 20.04 és a Debian 11 nem támogatott.** Az `.deb` függőségi listája
> abból származik, hogy az Chloros háttérprogram valójában mire hivatkozik, és ez magában foglalja
> az `libc6 (>= 2.34)`-et is. A Focal és a bullseye egyaránt a glibc 2.31-et tartalmazza, ezért az `apt` egyenesen elutasítja a
> telepítést, ahelyett, hogy később, futásidőben hagyja meghiúsulni azt.

### Telepítés

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
Az `dpkg -i` nem oldja meg a függőségeket. Ha hiányzó csomagokat jelzi, az `sudo apt-get install -f` (vagy az `sudo apt --fix-broken install`) befejezi a telepítést — ez a normális folyamat, nem hiba.
{% endhint %}

Ellenőrizze a telepítést:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Rendszerkövetelmények

| Követelmény | Minimális | Ajánlott |
| --- | --- | --- |
| **Platform** | NVIDIA Jetson JetPack 6-tal | Jetson Orin NX 16 GB vagy AGX Orin |
| **JetPack** | JetPack 6.x | Legújabb JetPack 6 |
| **Memória (RAM)** | 8 GB (megosztott GPU/CPU) | 16 GB+ megosztott (12 GB+ a párhuzamos GPU-munkások küszöbértéke) |
| **Tárhely** | 2 GB szabad hely | NVMe SSD 10 GB+ szabad hellyel |
| **Python** | Python 3.7+ (az SDK esetében) | Python 3.10+ |

### Telepítés

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Ugyanaz az elrendezés, mint az amd64 `.deb` esetében, a Jetson Orin / Orin NX / Orin Nano rendszerekhez hangolt CUDA-verzióval. A Jetson memóriájával, hőkezelésével és terepi telepítéssel kapcsolatos viselkedésről lásd az [NVIDIA Jetson útmutatót](nvidia-jetson-guide.md).

***

## Python SDK telepítése (minden Linux)

Az SDK egy tisztán Python alapú kliens a háttérrendszerhez, így ugyanaz a csomag működik amd64 és arm64 architektúrákon egyaránt. Két forrás:**A PyPI-ről** — a közzétett stabil kiadás:

```bash
pip install chloros-sdk
```

**A csomagban található wheel-ből** — garantáltan illeszkedik az imént telepített CLI/backendhez (használd ezt, ha az `.deb`-ed újabb, mint a PyPI-n található):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**A PEP 668-nak megfelelő disztribúciók** (Ubuntu 23.10+, Debian 12+) nem engedélyezik a rendszer szintű pip telepítéseket. Használja az `pip install --user …`-et, egy virtuális környezetet vagy az `sudo pip install --break-system-packages …`-et. A csomagtelepítő soha nem telepíti automatikusan az SDK-et a rendszer Python-ébe — ezt a döntést Önre bízza.
{% endhint %}

Opcionális kiegészítők:

| Kiegészítő | Parancs | Hozzáad |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` az élő előrehaladás-közvetítéshez |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` BLE (DAQ-M) átvitelhez |

Ellenőrizze az SDK fájlt:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Az `.deb` telepíti az Chloros CLI-et és a háttérprogramot. Az Python SDK egy helyi HTTP API (`http://127.0.0.1:5000`) hálózaton keresztül kommunikál azzal a háttérszolgáltatással, és szükség esetén automatikusan elindítja azt. Mindig a szó szerinti IPv4-címet használja az `localhost` helyett — az `localhost` feloldódhat az `::1`-re, és kérésenként körülbelül két másodpercbe kerül.
{% endhint %}

***

## Első beállítás

### 1. Bejelentkezés

Az CLI és az SDK eléréséhez fizetős Chloros+ szint (**Copper** vagy magasabb), amelyet szerveroldalon érvényesítenek: a kijelentkezett felhasználó az `401 AUTH_REQUIRED` kódot kapja, az ingyenes (Iron) csomaggal rendelkező felhasználó pedig az `403 PLAN_UPGRADE_REQUIRED` kódot.

```bash
chloros-cli login your@email.com 'your-password'
```

A hitelesítő adatok az `~/.chloros/user_session.json` fájlban kerülnek tárolásra.

{% hint style="warning" %}
**Minden telepítés vagy frissítés után újra be kell jelentkeznie.** A csomag `prerm` szkriptje szándékosan törli az `~/.chloros/user_session.json` fájlt és a gyorsítótárban tárolt licencet a gépen lévő minden felhasználó esetében, így egy új verzió mindig újraérvényesíti a licencet, ahelyett, hogy egy elavult gyorsítótárra támaszkodna.
{% endhint %}

### 2. Ellenőrizze a licenc állapotát

```bash
chloros-cli status
```

Az `chloros-cli status` minden szinten (beleértve az ingyeneset is) működik, így mindig láthatja, miért érhető el vagy nem érhető el a hozzáférés.

### 3. Futtassa a rendszerdiagnosztikát

```bash
chloros-cli selftest
```

Hét ellenőrzés fut sorban, és a parancs nem nulla kóddal lép ki, ha bármelyikük sikertelen:

| # | Ellenőrzés | Mit igazol |
| --- | --- | --- |
| 1 | **Verzió** | Az CLI jelenti a verzióját (`v1.2.0`). |
| 2 | **Port elérhető** | Az 5000-es port szabad, *vagy* már válaszolt rá egy működőképes Chloros háttérprogram (ez sikernek számít). |
| 3 | **Háttérprogram indítása** | A háttérprogram elindul. |
| 4 | **API teszt (`/api/test`)** | A háttérprogram válaszol az `status: ok`-re. |
| 5 | **Rendszerinformáció** | Kinyomtatja az `GPU: <name>, CUDA: <bool>, PyTorch: <version>` értéket az `/api/system-info`-ből. |
| 6 | **Zajszűrő modellek** | Megtalálja az `*.pth.enc` modelleket (az Linux-en: `/usr/lib/chloros/models`). |
| 7 | **CUDA + zajszűrő**| A Texture Aware valóban használható — CUDA-ra**és** legalább egy modellfájlra van szükség. |

A futtatás az `N/7 checks passed`-szel zárul, felsorolva az esetleges hibákat név szerint.

### 4. Az első adatkészlet feldolgozása

```bash
chloros-cli process ~/datasets/flight001
```

***

## Fájlok és könyvtárak

### Felhasználónként

Az Chloros hitelesítő adatait és az CLI konfigurációját egyetlen, platformfüggetlen könyvtárban tárolja, az **`~/.chloros/`** (az Windows rendszeren: `%USERPROFILE%\.chloros\`) könyvtárban tárolja hitelesítő adatait és az CLI konfigurációját. Két Linux-specifikus gyorsítótár viszont az XDG-konvenciókat követi – ezek az `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` beállításokat veszik figyelembe, ha azok meg vannak adva.

| Útvonal | Cél |
| --- | --- |
| `~/.chloros/user_session.json` | Az `chloros-cli login` által írt bejelentkezési munkamenet-gyorsítótár (minden csomag telepítésekor/frissítésekor törlődik) |
| `~/.chloros/working_directory.txt` | Alapértelmezett projektmappa felülírása (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | CLI nyelvi beállítás (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Az Windows grafikus felhasználói felülettel megosztott nyelvi beállítás — az itt megadott `language` érték elsőbbséget élvez az `cli_language.json` értékkel szemben |
| `~/.chloros/update_cache.json` | Egy órás gyorsítótár az Linux/Jetson indításkori frissítés-ellenőrzéshez |
| `~/.chloros/backend.log` | Háttérnapló, amikor a háttérprogramot az CLI indította el |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Kameránkénti LATTICE kalibrációs csomagok gyorsítótárban, sorozatszám és csomag-hash alapján indexelve |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Opcionális felhasználói felülírások a DAQ kap-korrekciós profilokhoz |
| `~/.config/chloros/system_config.json` | A Dynamic Compute Adaptation rendszerből származó, gyorsítótárba mentett hardverprofil — törölje, hogy új hardverfelismerést kényszerítsen |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Háttérszerver-naplók, indításonként egy fájl |
| `~/Chloros Projects/` | Alapértelmezett projektmappa, ha nincs felülírás beállítva |

### Rendszer-szintű

| Útvonal | Cél |
| --- | --- |
| `/usr/bin/chloros-cli` | Wrapper szkript — beállítja az `LD_LIBRARY_PATH` értéket a csomagban lévő natív könyvtárakhoz, majd futtatja a valódi bináris fájlt |
| `/usr/bin/chloros-backend` | Wrapper szkript — ugyanaz, plusz az `CHLOROS_PRODUCTION=1` beállítás, így a háttérrendszer hitelesítési kapuja soha nem tudja magát észrevétlenül letiltani |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | A lefordított bináris fájlok |
| `/usr/lib/chloros/arena_runtime/` | A LATTICE kamerák által igényelt Arena SDK futási környezet |
| `/usr/lib/chloros/models/*.pth.enc` | A Texture Aware debayer által használt titkosított zajszűrő modellek |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK – ehhez a konkrét összeállításhoz illeszkedő csomag |
| `/usr/lib/chloros/exiftool` | Csomagban szereplő exiftool (csak akkor szimbolikus linkkel kapcsolódik az `/usr/local/bin/exiftool`-hez, ha nincs rendszerbeli exiftool) |
| `/etc/chloros/update.conf` | Az `chloros-cli update` által beolvasott frissítési csatorna konfiguráció |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Beállítja az `net.ipv4.ip_unprivileged_port_start = 319` fájlt, hogy a háttérprogram root jogosultság nélkül tudja lekötni a PTP-portokat |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | A dinamikus betöltőt az `/usr/lib/chloros/arena_runtime`-re irányítja |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Hozzáférést biztosít a bejelentkezett felhasználónak a DAQ-U USB soros hídhoz (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Az állandóan futó háttérszolgáltatás bekapcsolása (telepítve, **nem engedélyezve**) |
| `/usr/share/applications/chloros-cli.desktop` | „Chloros CLI” alkalmazásmenü-bejegyzés, amely megnyit egy terminált |

## A háttérprogram végrehajtható fájljának helye

Az CLI és az SDK automatikusan felismeri a háttérprogramot:

| Komponens | Útvonal |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Háttérprogram | `/usr/lib/chloros/chloros-backend` |

A háttér elérési útját felülírhatja az `--backend-exe` CLI jelzővel vagy az `backend_exe` SDK konstruktorparaméterrel, a portot pedig az `--port` értékkel (alapértelmezés: `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` a **`lattice`**,**`project`**, és**`daq pool-*`** parancscsaládokra egy távoli háttérrendszerben. Az alapvető parancsok (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) szándékosan figyelmen kívül hagyják azt, és mindig az `http://127.0.0.1:<port>`-et célozzák meg.
{% endhint %}

***

## LATTICE kamerák és DAQ fényérzékelők az Linux-en

A live-hardware parancscsaládok mindegyike működik az Linux-en (amd64 és Jetson):

* **`chloros-cli lattice`** — LATTICE kamerák és szinkronizált tömbök felismerése, csatlakoztatása, konfigurálása és felvétele. Az `.deb` tartalmazza a szükséges Arena SDK futásidejű környezetet, és regisztrálja azt a dinamikus betöltőben.
* **`chloros-cli daq pool-*`** — DAQ-U/M/E fényérzékelők csatlakoztatása a háttérpoolon keresztül, kalibrált spektrumok streamelése és `.daq` fájlok rögzítése. A lefordított CLI kizárólag az `pool-*` családot tartalmazza: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — egy mentett projekt (kamerái, érzékelői és feldolgozási beállításai) vezérlése headless módban.
* **`chloros-cli time-sync`** — a PTP-grandmaster ellenőrzése, amelyen az Chloros háttérprogram fut a LATTICE kamerák és a DAQ-E érzékelők számára.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

Az `--sensor-id` szükséges az `pool-latest`, az `pool-stream`, `pool-record` és az `pool-set-cap` számára szükséges; az `pool-list` a poolban jelenleg található azonosítókat jeleníti meg.

{% hint style="info" %}
**Több hálózati kapcsolattal rendelkező gépen az első DAQ-E-kapcsolat létrehozásához az `--eth-host` azonosítót részesítse előnyben.** Az automatikus felismerés az mDNS-t vizsgálja át, és előfordulhat, hogy a hideg ARP-gyorsítótár miatt nem találja meg az érzékelő interfészét, ezért a rendszerindítás utáni első `pool-connect --eth` meghívás sikertelen lehet, még akkor is, ha az érzékelő tökéletesen működik. Az érzékelő IP-címének vagy gazdagépnevének megadása esetén a felismerés teljesen kihagyásra kerül.
{% endhint %}

**A DAQ-U soros engedélyeket** a telepített udev-szabály kezeli (`uaccess` + csoport `dialout`). Ha egy már csatlakoztatott érzékelő továbbra sem érhető el, töltse be újra a szabályokat, vagy csatlakoztassa újra:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

A teljes parancssorozatot lásd az [CLI referencia](../CLI.md) alatt.

### Állandóan aktív PTP fej nélküli gazdagépekhez

Az első telepítéskor a rendszer létrehozza az `chloros-backend.service` systemd egységet, de **nem aktiválja**. Egy kijelző nélküli Jetsonon vagy szerveren, ahol a PTP időszinkronizálásnak folyamatosan futnia kell a DAQ-E érzékelők és a LATTICE kamerák számára, aktiválja azt:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Enélkül a PTP csak akkor fut, amíg az Chloros háttérprogram fut — vagyis egy aktív CLI/SDK munkamenet alatt.

A készülék a háttérprogramot az `127.0.0.1:5000` -hez (az egység belsejében található `CHLOROS_HOST` / `CHLOROS_PORT` környezeti beállítások; felülírható az `sudo systemctl edit chloros-backend.service` beállítással), és hiba esetén 5 másodperc elteltével újraindítja.

**Hogyan szerzi meg a PTP a portjait?** A PTP az UDP 319/320 portokat használja, amelyek mindkettő a szokásos 1024-es kiváltságos portok alsó határánál alacsonyabbak. A csomag `postinst` parancsa az `/etc/sysctl.d/60-chloros-ptp.conf` fájlt az `net.ipv4.ip_unprivileged_port_start = 319` értékkel írja fel, ami lehetővé teszi a háttérprogram számára, hogy a felhasználó nevében futva ezekhez a portokhoz kapcsolódjon. Biztonsági intézkedésként az `setcap cap_net_bind_service,cap_net_raw=+ep`-et is alkalmazza a háttérprogram bináris fájljára — ezért szerepel az `libcap2-bin` a csomag deklarált függőségei között.***

## Bash szkriptelési példák

{% hint style="info" %}
**Szkriptelésre alkalmas kilépési kódok.**Az `chloros-cli process` siker esetén az `0` értéket adja vissza,**hibás futás esetén pedig nullától eltérő értéket – ideértve azokat a futtatásokat is, amelyek képtermékeket kértek, de egyet sem írtak ki** (kiírja az `Processing finished but wrote no image products.` kódot, megnevezi a projektmappát és a szokásos okokat). A sikeres futtatások jelentik, hogy hány képtermék került írásra (`Image products written: N`). Kilépési kódok: `0` siker, `1` hiba, `2` argumentumhiba, `130` megszakítás.
{% endhint %}

### Több adatkészlet feldolgozása

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Feldolgozás egyéni beállításokkal

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Az érvényes `--format` értékek száma pontosan négy, és szóközöket tartalmaznak — mindig idézőjelek közé kell tenni őket:

| `--format` érték | Kimeneti mappa |
| --- | --- |
| `TIFF (16-bit)` *(alapértelmezett)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

Az `--debayer` elfogadja az `standard` (alapértelmezett) vagy az `texture-aware` (Chloros+) értéket.

### Automatizált feldolgozás Cron segítségével

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

### Az CLI nem található a telepítés után

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Hozzáférés megtagadva

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### „setcap failed” hiba a telepítés során

Az `.deb` az `cap_net_bind_service`-et alkalmazza az `/usr/lib/chloros/chloros-backend`-re, hogy root jogosultság nélkül is képes legyen a 319/320-as PTP-portokat root jogosultság nélkül is lekötözhesse. Ha a telepítéskor hiányzott az `libcap2-bin`, akkor a hívás kihagyásra kerül. Telepítse azt, majd telepítse újra a csomagot:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### A PTP nem indul el / nem tudja lekötni a 319-es portot

Ellenőrizze, hogy a nem privilegizált portok alsó határértéke csökkent-e, és ha nem, akkor alkalmazza újra az aktuális indításra:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Ezután ellenőrizze a grandmastert:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### „A LATTICE kamera illesztőprogramjai nem találhatók”

Az Arena SDK futásideje nem oldódik meg. Ellenőrizze, hogy a csomag által írt betöltőkonfiguráció jelen van-e és frissült-e:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
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

A sikertelen indítás háttérprogram-naplói az `~/.cache/chloros/logs/` fájlban találhatók.

### CUDA nem észlelve

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

Az `chloros-cli selftest` ugyanezt jelenti egy sorban: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Hiányzó megosztott könyvtárak

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Lassú indítás SD-kártyás rendszereken

A lefordított bináris fájlok minden indításkor kibontják magukat egy ideiglenes könyvtárba. Ha az `/mnt/ssd/tmp` létezik, az Chloros automatikusan azt használja; ellenkező esetben állítsa be az `TMPDIR`-et egy gyors fájlrendszerre:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Az Chloros frissítése az Linux rendszeren

Az `update` parancs kizárólag az Linux/Jetson rendszereken használható. Ellenőrzi az `/etc/chloros/update.conf`-ben konfigurált frissítési csatornán közzétett verziót, és felajánlja a megfelelő `.deb` verzió letöltését és telepítését:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Az Linux/Jetson rendszeren az CLI parancs minden indításkor elvégzi a nem blokkoló frissítésellenőrzést is (az eredmény egy órán át tárolódik az `~/.chloros/update_cache.json`-ben), és az `Update available: vX.Y.Z` üzenetet jeleníti meg, ha újabb verzió áll rendelkezésre. A beállítások és a projektek a frissítés után is megmaradnak; utána azonban újra be kell jelentkezni.

## Eltávolítás

```bash
sudo apt remove chloros
```

Az eltávolítás leállítja az `chloros-backend.service` futását, visszaállítja az alapértelmezett, nem privilegizált port alsó határértékét (1024), eltávolítja a mellékelt exiftool szimbolikus linket és az Arena betöltő konfigurációját, valamint törli a gyorsítótárban tárolt hitelesítő adatokat. A projektjei és az `~/.chloros/` adatfájlok érintetlenek maradnak.

***

## Következő lépések

* [NVIDIA Jetson útmutató](nvidia-jetson-guide.md) — Jetson-specifikus optimalizálás és telepítés
* [CLI : Parancssor](../CLI.md) — az CLI útmutató
* [API : Python SDK](../api-python-sdk.md) — az SDK útmutató
* [CLI Referencia](../reference/cli-reference.md) és [SDK Referencia](../reference/sdk-reference.md) — a 1.2.0-s verzió teljes parancs- és API-listája
* [Dinamikus számítási alkalmazkodás](../processing-architecture/dynamic-compute-adaptation.md) — hogyan alkalmazkodik az Chloros a hardveréhez
