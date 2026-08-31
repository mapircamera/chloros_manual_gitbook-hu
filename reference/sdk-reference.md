# Chloros Python SDK Hivatkozás

**Verzió:**

1.2.0**Létrehozás:**

2026.07.29. 19:19 ·**Frissítés:**

2026.08.30.**Csomag:** `chloros-sdk` (PyPI)**Célközönség:** Nagy nyelvi modellek (LLM) számára optimalizált; ember számára olvasható.**Hatály:** Az `import chloros_sdk` által elérhetővé tett minden nyilvános osztály, függvény és segédfüggvény, másolható példákkal a képfeldolgozás, az egykamerás vezérlés, a szinkronizált tömbök, a DAQ-érzékelők és a projekt automatizálása témakörében.

Ha csak a legfontosabb részeket szeretné megtekinteni, ugorjon ide:
- [Telepítés és gyorsindítás](#installation)
- [Smart-Connect a LATTICE-kamerákhoz](#smart-connect-for-lattice-cameras)
- [DAQ-érzékelő-munkamenetek](#daq-sensor-sessions)
- [Projekt automatizálás](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Az architektúra 60 másodpercben

Az SDK egy vékony Python réteg az Chloros háttérrendszer felett (ugyanaz a Flask szerver, amelyet az asztali grafikus felület és az CLI is használ). Az automatizáláshoz importálja az `chloros_sdk`-et, és hívja meg a magas szintű módszereket; a háttérben minden hívás egy HTTP kéréssé válik a 5000-es porton futó helyi háttérrendszer felé — `http://127.0.0.1:5000/api/...` (szándékosan nem `localhost`, amely a Windows oldalon először az `::1`-re oldódik fel, és IPv4-es háttérrendszer esetén kérésenként ~2 másodpercet vesz igénybe). A háttérrendszer rendelkezik a hardverparkkal — kamerák, DAQ-érzékelők, igazítási profilok, képkocka-pufferek — így a SDK szkriptek a grafikus felhasználói felülettel (GUI) párhuzamosan futhatnak anélkül, hogy versengeniük kellene a soros portokért vagy a hálózati kártya sávszélességéért.

Három felületet fogsz használni:

1. **`ChlorosLocal` + szabad függvények** (`process_folder`, `process_lattice_capture`) — Képfeldolgozó folyamat. Egyetlen Python hívással futtathatja le egy teljes mappa kalibrálását, debayerezését és indexexportját.
2. **Smart-connect kezelők** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Tartós háttérmunkamenet megnyitása élő hardverhez. Ugyanaz a „smart-prep” folyamat, mint a grafikus felhasználói felületen: hálózati próba, automatikus rétegválasztás, PTP, AE-kezdőérték beállítása, GPIO-trigger konfiguráció.
3. **`ChlorosProject` / `open_project`** — Mentett projekt betöltése (mappa az `cameras.json` + `sensors.json` + `project.json` fájlokkal), minden egyszerre történő csatlakoztatása, és a rögzítéseket névvel ellátott kezelőkkel vezérelje.

Az 1. és 2. felületen **automatikusan elindít egy helyi háttérprogramot**, ha még nincs ilyen futásban (ugyanazt a csomagban lévő bináris fájlt, amelyet a GUI/CLI indít el) — így egy egyszerű szkript is működik egy friss parancssorból anélkül, hogy előbb el kellene indítanod a háttérprogramot. Az `auto_start_backend=False` átadásával kikapcsolhatod ezt (pl. ha távoli háttérprogramra mutat, amelyet soha nem indítanak el). Lásd [A háttérprogram automatikus indítása](#backend-auto-start). A Surface 3 másképp viselkedik: az `open_project()` nem fogad el `auto_start_backend` paramétert, az `connect_all()` pedig soha nem indít el háttérprogramot — egyszer megpróbálja az `http://127.0.0.1:5000`-et, és ha semmi sem válaszol, csendben visszatér a közvetlen (háttérprogram nélküli) `lattice_sdk` eszközvezérlésre. Csak az `proj.process()` és az `stream(..., overlays=True)` hozza létre késleltetetten egy `ChlorosLocal()`-et (amely automatikusan elindul).

Mindhárom hitelesítéshez kötött: futtassa egyszer az `chloros-cli login` parancsot a gépen, vagy jelentkezzen be az asztali grafikus felületen keresztül. Az érvényes munkamenet nélküli SDK hívások `ChlorosAuthenticationError` hibát eredményeznek.

Követelmények:
- Python 3.7+ (a csomag leírása szerint; 3.10-en fejlesztették és tesztelték)
- Helyileg telepített Chloros Desktop (a háttérprogram a telepítőben található)
- Aktív Chloros+ bejelentkezés. A SDK / CLI szint legalább **Copper**(Copper / Bronze / Silver / Gold); az ingyenes**Iron**szint nem biztosít hozzáférést a SDK / CLI oldalhoz. Ez**szerveroldalon** kerül érvényesítésre: minden SDK / CLI jelöléssel ellátott kérésnek tartalmaznia kell mind élő munkamenetet, mind fizetett csomagot, különben a háttérrendszer `403` hibakódot ad vissza `error_code: PLAN_UPGRADE_REQUIRED` kóddal (amelyet az `ChlorosLocal` kóddal, az `connect_*` segédprogramok pedig `ChlorosConnectError` kóddal jelzik). A kijelentkezett felhasználó az `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) kódot kap — a kettő azért különbözik, mert az `chloros-cli login` újbóli futtatása az elsőt kijavítja, a másodikat viszont nem.
- Az offline használat a csomag türelmi időszakán belül támogatott: a szintet a szerver-érvényesítési gyorsítótárból (5 perc) vagy az aláírt, géphez kötött licenc-gyorsítótárból (havi csomagok esetén 30 napig, éves előfizetések esetén az előfizetés lejáratáig). A türelmi időszak lejárta után a csomag ingyenesre vált, és az SDK / CLI hozzáférés addig szünetel, amíg a gép egyszer el nem éri a szervert. Az `chloros-cli status` (`GET /api/license-status`) az ingyenes szintre váltás után is elérhető marad, így látható az ok – ez az egyetlen SDK / CLI útvonal, amely mentes a szintkorlátozás alól.
- Windows 10/11 64 bites, **Ubuntu 22.04 LTS vagy újabb**, vagy Jetson (JetPack 6). Az Ubuntu 20.04**nem** támogatott: az `.deb` függőségei abból származnak, amire a háttérprogram hivatkozik, beleértve az `libc6 (>= 2.34)`-et is, és a Focal a glibc 2.31-et szállítja.

---

## Telepítés

Az Python SDK egy vékony Python réteg a Chloros háttérprogram felett. Néhány, kizárólag adatgyűjtésre szolgáló munkafolyamaton túlmenően szükség van a **Chloros asztali csomag helyi telepítésére** (Windows telepítő vagy Linux `.deb`) — ez biztosítja a háttérprogram bináris fájlját, az Arena SDK futtatókörnyezetet a LATTICE kamerákhoz, valamint a kalibrációs csomagokat.

Legfrissebb letöltések: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### 1. lépés — Telepítse az Chloros platformcsomagot

#### Windows (.exe)

1. Töltse le az `Chloros-Setup-x.y.z.exe` fájlt a letöltési oldalról.
2. Indítsa el a telepítőt, és kövesse a varázsló utasításait. Az alapértelmezett telepítési útvonal az `C:\Program Files\MAPIR\Chloros\`.
3. Indítsa el legalább egyszer az Chloros oldalt, és jelentkezzen be a Chloros+ fiókjával.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### 2. lépés — Telepítse az Python SDK programot

**A Chloros telepítő egy megfelelő SDK wheel fájlt is tartalmaz.** Minden Windows telepítő és Linux .deb fájl egy olyan `chloros_sdk-X.Y.Z-py3-none-any.whl` fájlt helyez el a lemezen, amely pontosan megegyezik a GUI / CLI / backend verzióval. Nem kell a PyPI-t követnie a szinkronizálás érdekében.

#### Windows

A telepítő automatikusan futtatja az `pip install` fájlt a csomagban lévő wheel fájllal, a rendszer Python (előnyben részesíti az `py.exe` indítót, de visszaesik az `python -m pip`-re). Nincs szükség semmilyen beavatkozásra — az `import chloros_sdk` a sikeres telepítés után működik a Python környezetedben. Ha nincs Python a gépen, a telepítő csendben átugorjaezt a lépést, és a grafikus felület (GUI) valamint a CLI továbbra is működnek.

#### Linux (.deb)

A .deb fájl az `/usr/lib/chloros/sdk/` könyvtárba helyezi el a wheel fájlt. Az `postinst` fájl kinyomtatja a pontos parancsot — a PEP 668-nak megfelelő disztribúciók alapértelmezés szerint nem engedélyezik a globális pip-írásokat, ezért nem telepítjük automatikusan:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Air-gapped Jetson-telepítések esetén ez teljesen offline történik — a wheel fájl már a lemezen található.

#### Nyilvános PyPI

Kizárólag pip-et használó gazdagépek esetén (nincs telepítve az Chloros asztali csomag; távoli háttérrendszerrel vagy kizárólag DAQ-val működő munkafolyamatok):

```bash
pip install chloros-sdk
```

A PyPI a kiadási verziójú telepítő-összeállításoknál frissül, így a közzétett wheel megegyezik a legújabb stabil kiadással. A fejlesztői összeállítások (pl. `1.1.4.dev1`) kizárólag a csomagban szereplő telepítő-wheel-en keresztül érhetők el.

#### Ellenőrzés

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+ előfizetés szükséges.** Minden SDK híváshoz aktív Chloros+ bejelentkezés szükséges. Futtassa az `chloros-cli login user@example.com 'YourPassword'` parancsot egyszer minden gépen; a hitelesítő adatok az `~/.chloros/` fájlban kerülnek tárolásra.

### Szükségem van a Desktop csomagra?

A legtöbb munkafolyamat esetében a pip csomag önmagában **nem** elegendő. Az egyes SDK felületekhez a következőkre van szükség:

| SDK felület | Szükséges-e a Desktop Package? | Miért |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Igen** | Automatikusan elindítja a háttérprogramot az `/usr/lib/chloros/chloros-backend` (Linux) vagy az `C:\Program Files\MAPIR\Chloros\…` (Windows) címeken. |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Igen**(helyi)**/ Nem**(távoli) | Tiszta HTTP kliensek a háttérrendszeren keresztül. Helyi háttérrendszer → asztali csomag szükséges. Távoli háttérrendszer → `backend_url=`**alagúton keresztül** (lásd: Távoli háttérrendszer mód — a szállított háttérrendszerek csak a loopback-hez kapcsolódnak). |
| `ChlorosProject` / `open_project` | **Igen** | A mentett projekteket a háttérrendszeren keresztül futtatja. |
| Közvetlen LATTICE osztályok (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Igen** | Szükség van az Arena SDK natív futtatókörnyezetére, amely az asztali csomagban található. Az `CAMERA_AVAILABLE` egyébként importáláskor `False`-ként jelenik meg. |
| Közvetlen DAQ osztályok (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Nem** | Tiszta Python pyserial/bleak/zeroconf felett. Egy kizárólag pip-et használó környezet végpontok közötti DAQ-vezérlést tesz lehetővé. |

### Távoli háttérmód (kizárólag pip-et használó gazdagép, alagúton keresztül)

> **A szállított háttér nem érhető el a LAN-on keresztül.** A termelési
> verziók kizárólag a loopback-hez kapcsolódnak (mindkét loopback-család), és határozottan elutasítják a
> nem loopback módot (`CHLOROS_CLOUD_MODE`), így az
> `backend_url="http://<lan-ip>:5000"` **nem működik egy telepített
> Chloros** — ez a minta eddig csak source/dev
> háttérprogrammal működött. Ha egy másik gépen lévő háttérprogramot szeretne vezérelni, akkor saját maga továbbítsa annak loopback
> portját, és az SDK fájlt irányítsa az alagútra:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

A headless / CI / robotikai gazdagépek egy gépet tarthatnak meg a teljes asztali telepítéssel, mint „Chloros-kiszolgálót”, minden máshol pedig az `pip install chloros-sdk`-et — de a kettő közötti átvitel a fenti, felhasználó által beállított alagút, nem pedig közvetlen LAN-URLáció.

> **Ismert korlátozás — az `ChlorosLocal` nem képes kizárólag pip-en keresztül működni.** Az `ChlorosLocal(backend_url=BACKEND)` jelenleg a konstruktorában *még azelőtt* feloldja a helyi háttérprogramot, hogy megpróbálná felderíteni a URL-t, és `ChlorosBackendError` hibát jelez („Chloros-háttér nem található…”) hibaüzenetet, ha nincs telepítve asztali csomag — még akkor is, ha elérhető a távoli háttér. Csak a fenti smart-connect felület (`connect_camera` / `connect_array` / `connect_daq_sensor`, valamint az `analyze_array_network` és az `list_*` / `discover_*` segédprogramok) működnek kizárólag pip-csomaggal rendelkező gazdagépen.

### Kizárólag DAQ-ra épülő munkafolyamat (kizárólag pip-csomaggal rendelkező gazdagép)

Ha csak DAQ-érzékelőkre van szükséged, és nem használsz LATTICE-kamerákat vagy képfeldolgozást, a pip-csomag önállóan működik:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Nincs háttérprogram, nincs .deb, nincs Chloros+ bejelentkezés szükséges a közvetlen hardveres DAQ-munkához.

---

## Gyorsindítás

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Főszintű API index

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Képfeldolgozás — `ChlorosLocal`

A fő folyamatkezelő osztály. Első használatkor elindítja a háttérprogramot, létrehozza és konfigurálja a projekteket, figyelemmel kíséri a folyamatot, és futás utáni összefoglalókat ad vissza.

### Konstruktor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Módszerek

| Módszer | Leírás |
| --- | --- |
| `create_project(project_name, camera=None)` | Új projekt létrehozása (opcionálisan egy kamerasablonnal, például az `"Survey3N_RGN"`-szel). |
| `import_images(folder_path, recursive=False)` | RAW/TIF/JPG/DNG képek **és `.daq` fényérzékelő-felvételek** importálása. Visszaadja az `count` (képek) és az `scan_count` (felvételek) értékeket. Csak akkor jelez figyelmeztetést, ha a mappában egyik sem található. |
| `export_light_sensor(daq=True, csv=True)` | Minden fényérzékelő-felvételhez kalibrált `.daq` + `.csv` fájlokat ír-érzékelős felvételről a projektben az `<project>/Light Sensor/` fájlba. Lásd [Fényérzékelős felvételek](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Állítsa be a feldolgozási paramétereket. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Futtassa a feldolgozási folyamatot. Visszaadja az `{"status": "complete", "async": False}` értéket, valamint egy `summary` kulcsot, ha a háttérrendszer biztosít ilyet — lásd [Futtatás utáni összefoglaló és tippek](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | A háttérrendszer állapotának ellenőrzése. |
| `logout()` | A gyorsítótárban tárolt hitelesítő adatok törlése. |
| `shutdown_backend()` | A háttérprogram leállítása (ha SDK -started). |
| `discover_cameras()` | LATTICE kamerák felkutatása **ezen példány háttérrendszere segítségével** (`/api/camera/discover`). Szótárak listáját adja vissza (`serial`, `model`, `ip`, …) — ugyanolyan formátumban, mint amit a GUI/ CLI lát. Üres lista, ha nem található, vagy a háttérprogram nem érhető el. |
| `camera_capture(output_dir, format="tiff", **settings)` | Egyetlen képkocka rögzítése**a háttérprogramon keresztül**(automatikusindítású) úgy, hogy ugyanazt az előkészítést kapja, mint a GUI/ CLI (alapértelmezés szerint 12 bit, pool újrafelhasználás, beágyazott kalibrációs metaadatok). A célt az `serial=` vagy az `device_index=` parancsokkal határozza meg; adja át az `exposure`/`gain`/`pixel_format`/`preset` értékeket `**settings`-ként adja át. Visszaadja a régi metaadat-szótárat (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Összeállított előnézeti képkockák előállítása egy összevont kamerából — vékony MJPEG-kliens a háttérrendszer `/api/camera/<serial>/stream-annotated` útvonalán keresztül (zebra / rács / célkereszt / hisztogram / csúcsjelzés / spot szerveroldalon rajzolva). Az `decode=True` BGR-tömböket ad vissza; az `False` nyers JPEG bájtot ad vissza. Projektként is elérhető az `ChlorosProject.stream(overlays=True)` néven. |

Használja kontextuskezelőként a garantált tisztításhoz:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Fényérzékelő-felvételek — kalibrált `.daq` + `.csv`

A DAQ-U / DAQ-M / DAQ-E **a**kalibrációs csomag**nélkül** is rögzíthető. Ez az,
amit a nyilvános [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
felvevők (`record_daq.py`) alapértelmezés szerint ezt teszik: a nyers érzékelőértékeket írják ki, és megjelölik a
fájlt, így a Chloros **a sorozatszám alapján** letölti az érzékelő gyári kalibrációját — először a helyi gyorsítótárból
először, majd a MAPIR felhőből — és az importáláskor alkalmazza azt.

A Chloros az eredményt felvételenként két termékként írja vissza, az
`<project>/Light Sensor/` alatt:

| Termék | Mi ez |
| --- | --- |
| `<name>_calibrated.daq` | Az újrafeldolgozható archívum — ugyanaz a sémája, mint az élő felvételnek, most már feltüntetve azt a csomagot, amely létrehozta. Újbóli importálása **nem** kalibrálja újra. |
| `<name>_calibrated.csv` | Spektrális irsugárzás W/m²/nm-ben az érzékelő saját hullámhossz-rácsán, egy sor minden mérési értékre, valamint fotometrikus oszlopok (teljes teljesítmény, fotopikus/skotopikus lux, PPFD és annak kék/zöld/vörös felosztás, csúcs hullámhossz). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Csak csomag nélküli érzékelők (DAQ-A).** Nyers spektrális érzékelőimpulzusok — *nem* sugárzási intenzitás. Lásd alább. |

Az `process()` ezt az exportot az egyik lépéseként végzi el. Ehhez **nincs** szükség képanyagra:
az önállóan repülő fényérzékelő önmagában is elsőrendű munkafolyamatot jelent, és egy ilyen projektnek
kialakításából adódóan nincs képe.

**A DAQ-A felvételek nyers impulzusokként kerülnek exportálásra.** A DAQ-A család a sorozatszámonkénti
csomagrendszert megelőzően jött létre, és nincs letöltendő csomagja — helyette a terepen egy
reflektancia-célponttal kalibrálják, ezért soha nem is volt rá szüksége. Ezek a felvételek
`_raw` kiterjesztéssel exportálódnak, nem pedig `_calibrated` előtaggal: ez egy eltérő fájlnév, nem pedig egy
fájlon belüli jelző, mivel az azonosítónak meg kell maradnia, ha puszta névként továbbítják e-mailben. Az
`.csv` fejléc `raw spectral sensor counts (NOT irradiance)` értéket jelöl, és figyelmeztet arra, hogy az
értékek **a** fájlon belül összehasonlíthatók – pontosan erre használja őket a célalapú kalibrálás
–, és nem az érzékelők között. A teljesítményfüggő fotometrikus oszlopok (teljes teljesítmény,
fotopikus/szkotopikus lux, PPFD) **NULL** értéket adnak vissza, ahelyett, hogy a számlálásokból integrálnák őket.

Egy olyan DAQ-U / DAQ-M / DAQ-E, amelynek csomagja egyszerűen nem volt letölthető, továbbra is **kihagyásra kerül**,
nem kerülnek nyers formában írásra: ebben az esetben a csomag létezik, és a „újracsatlakozás és újrafeldolgozás” valóban jó tanács.

A régebbi **v1.01 / v1.02** felvételek (ezeket a DAQ-A-SD írja) nem tartalmaznak olvasásonkénti időpontot,
csak a fájl írási idejét. Az image↔downwelling egyeztető továbbra is elutasítja őket — egy
képkocka és az írási idő összehasonlítása láthatatlanul hibás lenne —, de az exportáló program beolvassa őket, és az
CSV kinyomtatja az `clock=daq_created_on` fájlt, így a termék jelzi, hogy melyik órára van beállítva.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Azokat a felvételeket, amelyek kalibrációs csomagja nem tölthető le (offline állapotban, vagy olyan érzékelő esetén, amelynek
nincs kalibrációs fájlja), az `skipped` kód alatt **az ok megjelölésével** kerül jelentésre. Soha nem
kerül kiírásra „kalibrált” fájlként, amely nyers számadatokat tartalmaz – csatlakozzon az internethez, és
futtassa újra a műveletet, így az export befejeződik.

### Haladási visszahívások

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Futtatás utáni összefoglaló és tippek

A befejezés után az `process()` lekérdezi az `GET /api/processing-summary` fájlt, és a testét `result["summary"]` néven csatolja. A lekérdezés a legjobb erőfeszítés alapján történik, és soha nem blokkolja a sikeres visszatérést — ha az összefoglaló nem érhető el, az `process()` visszatér az egyszerű `{"status": "complete", "async": False}` formátumra. Az `summary["hints"]` minden bejegyzése – teljes mondatok a javasolt javítással, pl. miért nem eredményezett kimenetet egy futtatás — szintén újraküldésre kerül Python `UserWarning` formátumban, így a nulla kimenetű futtatások önellenőrzőek, még akkor is, ha soha nem vizsgálja meg a szótárat:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` a géppel olvasható rész:

| Kulcs | Mit számol |
| --- | --- |
| `models` | A futtatásban szereplő kameracsoportok száma. |
| `images_in_groups` | Azokhoz a csoportokhoz tartozó forrásképek száma. |
| `targets_found` | Észlelt reflektancia-célpontok. |
| `images_calibrated` | A futtatás során kalibrált képek. |
| `exported_files` | **A futtatás során létrehozott képfájlok.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Fényérzékelő-felvételek, amelyeket szándékosan külön számoltak — ezek egy másik szakaszból származnak, és olyan futásoknál is léteznek, ahol egyáltalán nincsenek képek, így ha ezeket is beleszámolnánk, akkor egy kizárólag adatgyűjtésre szolgáló futás úgy tűnne, mintha képeket is exportált volna. |

Ezek mellett: `summary["output_dirs"]` (minden írási célpontként szolgáló könyvtár),
`summary["light_sensor_export"]`, `summary["stopped"]` (igaz, ha a felhasználó megszakította a
futást, így a részleges számlálások nem jelzik azt, hogy a futás befejeződött, de kevesebb eredményt hozott), és
`summary["groups"]` (a csoportonkénti bontás).

Az `exported_files`-et a folyamat **az írás közben** rögzíti, nem pedig utólag a
projekt képobjektumaiból. A párhuzamos és a GPU-stratégiák saját képobjektumokat hoznak létre
(a GPU-útvonalak esetében a munkavállalói alfolyamatokban), így a régi beolvasás minden ilyen futásnál
`0 file(s) written` értéket jelentett, majd kiadta a nulla exportra utaló jelzést — olyan futásoknál,
ahol minden rendben működött. Ha ezt a számot használja a szkriptjében, egy hibátlan párhuzamos futtatás most már
nem nulla értéket jelent.

A Light-sensor-kihagyások az olvasó által az egyes fájlokhoz ténylegesen megállapított okot jelentik –
olvashatatlan sémát, hiányzó csomagot, írási hibát – **duplikációmentesítve**, így húsz fájl
kihagyását egyetlen okként jeleníti meg, ahelyett, hogy húszszor ismételné azt.

> **Az `process()` nem dob hibaüzenetet, ha egy futtatás nem eredményez képeket.** Ez az egyetlen pont, ahol az SDK és
> az CLI szándékosan eltérnek egymástól: az `chloros-cli process` a „termékeket kérték, de egyet sem
> írtak” üzenetet hibaüzenetként kezeli, és nullától eltérő értékkel lép ki, míg az SDK normál módon tér vissza, és az
> állapotot az `summary` / hints-en keresztül jelenti. Ha a folyamatnak üres futás esetén le kell állnia, ellenőrizze
> saját maga — vizsgálja meg az `summary` kódot (vagy számolja meg a projektmappa alatti fájlokat), ahelyett, hogy
> a kivétel hiányára támaszkodna. A szokásos okok a következők: egy bemeneti mappa, amelyet nem ismert fel
> rögzítésként, valamint a jelen lévő kamerákra nem alkalmazható termékek kihagyása (pl. a RGB -only
> kamerák).

### Kényelmi funkciók

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Támogatott értékek

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Radiometrikus kimenet (LATTICE multispektrális feldolgozási folyamat)

Az `process` feldolgozási folyamat LATTICE multispektrális (M3C/M3M) exportszintje — `reflectance` (alapértelmezett), `radiance`, `sensor-response` vagy `all` (minden alkalmazható mód képenként) — a projekt**„Radiometrikus kimenet”** feldolgozási beállításához. Az `configure()`-hez külön kulcsszó tartozik:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

A speciális kiskapu — a projekt `"Radiometric output"` kulcsának `custom_settings`-en keresztül történő megadása — továbbra is működik, de ne feledje, hogy ez a teljes beállítási blokkot felülírja (lásd az alábbi figyelmeztetést):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

Az `reflectance` (az alapértelmezett) a kamera sugárzási intenzitását osztja az **időbélyeggel egyező DAQ lefelé irányuló sugárzással**, amelyet automatikusan határoz meg egy rögzített `.daq` (DAQ-U/M/E)**vagy egy DAQ-M natív `.csv`**adatból, amely a képek mellett található; ha bármely kamera- vagy DAQ-kalibrációs csomag hiányzik helyileg, azt az első használatkor**automatikusan letölti az AWS-ből**. Az CLI ezt típusonkénti termékkapcsolóként teszi elérhetővé az `chloros-cli process` oldalon: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> Az `custom_settings` **felülírja** a teljes számított beállítási blokkot (megkerüli az `configure()`többi kulcsszavát és az érvényesítést). Használatakor vegyen fel minden olyan `Project Settings` kulcsot, amely fontos Önnek, a fenti példához hasonlóan.

---

## Smart-Connect a LATTICE kamerákhoz

Tartós háttérmunkamenetek élő hardverekhez. Ugyanazokat a végpontokat használja, mint a grafikus felület, így a viselkedés azonos a SDK / CLI / grafikus felületen.

### Egyetlen kamera — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### `connect_camera()` aláírás

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession` módszerek

| Módszer | Leírás |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | GenICam csomópontok olvasása; visszaadja az `{nodes, errors, enums, device}` értéket. |
| `set_settings(**kwargs)` | Csomópontok írása barátságos név alapján (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | **Egyetlen** képkocka rögzítése. Visszaad egy elemből álló listát, amely a képkocka metaadat-szótárait tartalmazza. (A sorozatfelvétel/többképkockás rögzítés eltávolításra került — ha sorozatra van szüksége, hívja meg az `capture()` függvényt egy ciklusban.) |
| `disconnect()` | Felszabadítás a poolból. Nincs hatása, ha már egy megnyitott munkamenethez csatlakoztunk. |

`capture()` exportvezérlők (ugyanaz a modell, mint a tömb + GUI):

- `processing` / `levels` — `processing="all"` minden alkalmazandó exporttípust elment; `levels=["raw","radiance"]` csak azokat menti (felülírja az `processing`-et). A háttérprogram alapértelmezett beállításához hagyja ki mindkettőt.
- `force_daq=True` — a hozzárendelt DAQ/DLS-leolvasást `.daq` sidecar-ként menti el még nyers adatfelvétel esetén is, így a képkocka később újra feldolgozható reflektancia/index formátumra. Ha nincs DAQ csatlakoztatva, akkor nincs hatása.

### Szinkronizált tömb — `ArraySession` (Smart-Prep)

Az `connect_array` **az ajánlott belépési pont** a többkamerás felállásokhoz. A háttérben a teljes GUI-s smart-prep folyamatot futtatja:

1. **Hálózati elemzés** (`/api/camera/array/recommend`) — megkeresi a legnagyobb olyan képkockaméretet, amely illeszkedik a sim-emit szinthez anélkül, hogy képkockák vesznének el.
2. **Szint automatikus kiválasztása** — `sim-capture-sim-emit`, ha a hálózat képes kezelni; ellenkező esetben `sim-capture-ftd-stagger` vagy `slip-emit-and-capture`.
3. **Automatikus kicsinyítés**— észrevétlenül kicsinyíti a keretméretet / növeli a binninget, ha a vezeték nem képes fenntartani a kért felbontást.**Ez a biztonsági háló nem terjed ki az összesített túljegyzésre**: a vezeték számára túl sok kamera problémáját nem lehet a keretek kicsinyítésével megoldani — lásd [Túlterhelés](#over-subscription-the-per-cam-floor).
4. **PTP alapértelmezés szerint engedélyezve** — a kamerák közötti időbélyegek mikroszekundumon belül összehasonlíthatók.
5. **Kameránkénti pixelformátum-automatikus kiválasztás** — RGB kamerák → `BayerRG8`, multispec → `BayerRG12`.
6. **AE-beállítás** — pillanatképet készít minden kamera aktuális AE-állapotáról, így a csatlakozás nem állítja vissza az expozíciót a felvétel közben.
7. **GPIO-trigger konfiguráció** — az `connect_array` minden kamerát (`TriggerMode=On`, `TriggerSource=Line2`), így a master impulzusai az M8-as kábelen keresztül vezérlik a slave-eket. Ez egy kizárólag tömbökre vonatkozó lépés: egyetlen kamera megnyitásához az `LatticeCamera` free-parancsot használva.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### `connect_array()` aláírás

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

`force_tier` értékek:
- `"sim-capture-sim-emit"` — valódi egyidejűség (minden kamera ugyanazon az órajel-élén indít).
- `"sim-capture-ftd-stagger"` — rugalmas időtartománybeli eltolás (a kamerák kissé eltolt időpontokban sugároznak, így a csomagok sorba rendeződnek a vezetéken).
- `"slip-emit-and-capture"` — szekvenciális, kameránkénti rögzítés (nincs időbeli szinkronizálás; ez az egyetlen lehetőség, ha egyetlen keretméret sem felel meg a szimultán módnak).

Az `wire_ceiling_mbps` felülírja a **gazdagép tartós vezetékes sávszélesség-keretét** MB/s-ban — ez az az egyetlen
szám, amelytől az egész tömb allokációja függ. Hagyja `None` értékre, hogy az automatikus-felismert
értéket használja. Csökkentse, ha a tömb GVSP-sérült kereteket jelent: az automatikus érték a
hálózati kártya által jelzett kapcsolatsebességből származik, amely túlbecsüli az USB-adaptereket, a keskeny PCIe sávokat és
forgalmas megosztott hálózatok esetében — túlbecsüli a sebességet, és ez a túlbecslés sérült keretek formájában jelenik meg, nem pedig
szemmel láthatóan lassú kapcsolatként. Az érték a projekt tömb-rögzítési blokkjában kerül mentésre, így egy
újbóli megnyitás vagy egy későbbi `connect_array` parancs visszaállítja azt, akárcsak bármely más tömbbeállítást.
Lásd [Tömbállapot](#array-health--which-subsystem-is-losing-frames).

#### Túljegyzés (a kameránkénti alsó határ)

A Sim-emit ütemezés minden kamerának kioszt egy részt az ütközésbiztos vezeték-kapacitásból, amelynek alsó határa **kameraenként 8 MB/s**(`per_cam_floor_bps`). Amint az `N × floor` túllépi az ütközésmentes felső határt, a rendszer**túlterheli a vezetéket**— a hiba módja a GVSP-csomagvesztés, nem pedig alacsonyabb képkockasebesség — és nincs megoldás a képkockaméretre:**a binning és az ROI csökkenti a képkockánkénti bájtokat, nem pedig a másodpercenkénti ütemezett bájtokat**az összesített ellenőrzés ezt hasonlítja össze. Gyakorlati teljes felbontású felső határok egy 1 GbE-s gazdagépen:**6 kamera 1500 MTU-val, 9 jumbo keretekkel** (az `max_cams_collision_safe` az elemzési válaszban jelenti a vezeték felső határát). Megoldások: kevesebb kamera, végpontok közötti jumbo-keretek, vagy gyorsabb hálózati kártya.

- Az `analyze_array_network()` és `/api/camera/array/connect` válaszok az `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` és `per_cam_floor_bps` értékeket. Ha az `oversubscribed` értéke igaz, a kivetítés **nullázza az fps mezőket** (`achievable_fps_max` / `fps_bright` / `fps_dark`), ahelyett, hogy félrevezető, lassú, de működő értéket jelentené.
- Az `POST /api/camera/array/connect` elfogadja az `pin_resolution` testparamétert (**csak HTTP — nem pedig SDK kwarg**; az `connect_array` nem teszi közzé). A rögzítés eltávolítja a binning walk-down biztonsági hálót, így egy túlterhelt csatlakozás, amelynél az `pin_resolution` be van állítva,**kategorikusan elutasításra kerül** , és a hibaüzenet felsorolja az összes lehetséges megoldást. Rögzítés nélkül a csatlakozás folytatódik a walk-down-nal, de figyelmeztet, hogy a csökkentés nem tudja kiüríteni az aggregátumot.
- Kísérleti megoldás: állítsd be az `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` értéket a háttérrendszer környezetében, hogy az elutasítást hangos figyelmeztetésre minősítsd le – így a csatlakozás mégis megtörténik, és elfogadod a csomagvesztést.

#### Tömb állapota – melyik alrendszer veszít kereteket

Az `GET /api/camera/array/<array_id>/capability` egy aktív `health` blokkot hordoz egy
csatlakoztatott tömbön, amelyet egy gördülő **10 másodperces** ablakban. A keretveszteséget
két, egymással ellentétes javítást igénylő okra bontja, ahelyett, hogy egyetlen „hiányos” arányt adna meg, amely
egyiket sem nevez meg:

| Mező | Mit jelent | Melyik alrendszer |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (soronként) | A keret **megérkezett, de szerkezetileg hibás volt**— GVSP-csomagvesztés. |**Hálózat**: vezeték-kapacitás, ütemezés, NIC RX gyűrű, MTU |
| `never_arrived_rate_pct` (soronként) | A keret **egyáltalán nem érkezett meg**— a kamera nem indult el, vagy semmi nem jött ki belőle. |**Kiváltás / szinkronizálás**: M8 kábel, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | A legrosszabb kameraátviteli sebesség mindegyik esetében. | — |
| `per_cam_rate_pct` | Kameraenkénti összesített hiányossági arány (mindkét ok együtt). | — |
| `stable_for_seconds` | Mennyi ideig maradt minden kamera 0,01 % alatt. | — |

Az `health` mellett ugyanaz a rekord jelzi azt a számot is, amelyen az egész allokáció lóg:

| Mező | Mit jelent |
| --- | --- |
| `wire_ceiling_mbps` | A gazdagép érvényben lévő, fenntartott vezetékes sávszélessége, MB/s. |
| `wire_ceiling_source` | Honnan származik ez az érték, szavakkal kifejezve — pl. `USB-capped 200 MB/s (was theoretical 1062; …)` vagy `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, amikor `wire_ceiling_mbps=` beállította. |
| `nic_is_usb` | `true` egy USB-Ethernet-adapter esetén. |

Ehhez a végponthoz nincs „SDK” wrapper — olvassa ki közvetlenül:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Olvasás:** a nullától eltérő `gvsp_corrupt_rate_pct` érték, ahol az `never_arrived_rate_pct` 0-ra áll, azt jelenti, hogy
a triggerelés és a kábelszinkronizálás tökéletes, és a veszteség 100%-a a hálózati útvonalon keletkezik — csökkentse
`wire_ceiling_mbps` értéket, és csatlakozzon újra. Az ellenkező minta inkább a szinkronkábelre vagy a
triggervezetékre utal.

> **Az `target_fps` nem a sérült keretek szabályozója.** A GevSCPD ütemezését egyszer írja be a
> csatlakozáskor, így a triggerfrekvencia csökkentése a kitöltési arányt változtatja meg, nem pedig a
> egyidejű kibocsátási burst-frekvenciát. Egy mért, 5-szeres igénycsökkentés nem hozott javulást, míg
> a vezetéki felső határ 240-ről 200 MB/s-ra történő csökkentése ugyanazon berendezésnél a sérült keretek arányát 10,4 %-ról
> 0,00 %-ra csökkentette.

> **A TRI032S firmware-en a közbenső automatikus zsugorítás nem elérhető.** A futó tömb nem tudja
> ezt önállóan kijavítani; válassza le, majd csatlakoztassa újra, hogy a csatlakozási idő-beállító az új
> felső határnak megfelelően tervezzen újra.

Egy **USB-Ethernet-adaptert a szonda 200 MB/s-ra korlátoz**, függetlenül annak
névtábláján szereplő értéktől: a kapcsolati sebességet tartós értékre átalakító hatékonysági táblázat
PCIe-alapú, és egy USB hálózati kártya az Ethernet-kapcsolat sebességét hirdeti, miközben az
USB-busz és az illesztőprogram korlátozza. A korlát abszolút érték, nem pedig hányad — egy USB 1 GbE adapter
kb. 80 MB/s sebességet ér el, és ez nem érinti.

#### `ArraySession` módszerek

| Módszer | Leírás |
| --- | --- |
| `status(timeout=10.0)` | Élő `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Egy szinkronizált rögzítési csoport. Visszaad egy `CaptureResult` (keret-szótárak listája + `.skipped`) értéket ad vissza. Az exportálási vezérlők alább találhatók. |
| `capture(..., smart=True)` | **Intelligens rögzítés** — megvárja, amíg az AE minden kamerán stabilizálódik, majd elindítja a rögzítést. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Leggyorsabb rögzítés: csak nyers adatok + a hozzárendelt DAQ-érték (+ az ingyenes kombinált index). A GUI „Leggyorsabb rögzítés” gombját. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Egyszeri / Folyamatos / Intervallum egy korlátozott ciklusban. Visszaadja az `list[CaptureResult]` értéket.**Az `count` és/vagy az `duration_s`** szükséges a leállításhoz (a „SDK” nem támogatja a Ctrl+C billentyűkombinációt). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Elindítja az élő kombinált index nézet rögzítését videó/GIF formátumban → `RecorderHandle`. Minden tömbhöz egy kompozit felvevő tartozik. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Magas képsebességű nyers Bayer-sorozatfelvétel indítása → `RecorderHandle`. Offline újrafeldolgozás `build_video()` segítségével. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | A mentett nyers sorozat offline újrafeldolgozása kalibrált videó(k)vá. Blokkol, amíg be nem fejeződik (`wait=True`), majd visszaadja az `{outputs, errors, combined}` értéket. |
| `build_video_status(job_id, timeout=15.0)` | Offline összeállítási feladat lekérdezése: `{running, result, error, burst_dir}`. |
| `disconnect()` | Az egész tömb felszabadítása. |

`capture()` exportvezérlés (ugyanaz a végpont, mint a GUI/CLI):

- `processing` / `levels` — `processing="all"` (vagy `levels=["raw","radiance",…]`) minden kamera esetében elmenti az összes alkalmazható exporttípust; egy egyetlen `processing` érték csak azt a szintet menti el.
- `aligned=True` — minden tag nem nyers exportját igazítja a tömb [igazítási profiljához](#array-alignment) (együtt regisztrálva); a nyers adatok nem kerülnek eltorzításra, de a transzformációt a metaadatokban hordozzák. Ha a tömbnek nincs profilja, akkor visszatér az igazítatlan állapotra (amire figyelmeztető üzenet jelenik meg az eredmény `alignment` mezőjében).
- `render_index=False` — kihagyja a kameránkénti vegetációs index-átfedést; alapértelmezés szerint ott jeleníti meg, ahol be van állítva.
- `force_daq=True` — a hozzárendelt DAQ/DLS-leolvasást `.daq` sidecar-ként menti el akkor is, ha egyetlen kiválasztott szint sem igényli.

**TIFF-tömörítés (csak HTTP):**Az `ArraySession.capture()` nem küld `compression` kulcsot, így a háttérprogram alapértelmezett beállítása érvényes — az `POST /api/camera/array/capture` kiolvassa az `compression` testparamétert, az `"deflate"`-et olvassa be alapértelmezés szerint (veszteségmentes zlib L1 + vízszintes prediktor, ~4,1 MB teljes felbontású képkockánként). Az `"none"` tömörítetlen formában ír (~6,3 MB/képkocka),**~5× gyorsabb írási sebességgel** — mindkettő veszteségmentes, és importáláskor azonos módon olvasható. Az SDK nem tesz közzé kwarg-ot ehhez; a kiút az `chloros-cli lattice array-capture --compression none` vagy a nyers HTTP. A DEFLATE szintén tartja az Python GIL-t, így a tömörített írások nem párhuzamosíthatók akamera-író szálak között – a szenzor sebességénél történő folyamatos, 8 kamerás, teljes felbontású rögzítéshez az `compression: "none"` szükséges. Részletek: [CLI Hivatkozás → array-capture](cli-reference.md).**Elem-szintű export felülírások (csak HTTP):**ugyanaz a végpont elfogadja az `exclude_serials` parancsot is (lista — elemek eltávolítása a mentett halmazból; a tömb továbbra is egy szinkronizált csoportként működik, és a kizárt elemek az `excluded`-ben kerülnek visszaadásra), az `serial_levels` (`{serial: [level tokens]}` kameránkénti felülírások), valamint az `serial_index` (`{serial: bool}` kameránkénti index-átfedés felülírások). Ezek a GUI-paritású testparaméterek, és**még nem SDK kwargs**; a térképekből hiányzó elemek visszatérnek a tömb szintű `levels` / `render_index` értékekre.

##### Az átugrott kamerák vizsgálata — `CaptureResult.skipped`

Az `ArraySession.capture()` egy `CaptureResult`-et ad vissza, amely egy `list` alosztály: végig kell futtatni, indexelni, `len()` — minden meglévő minta továbbra is működik. Az új kód ellenőrizheti az `.skipped` attribútumot, hogy megnézze, mely bütyköket zárták ki és miért. A leggyakoribb eset az, hogyRGB-es bütyköket találunk egy vegyesszűrőtömbben, amikor `processing="radiance"` vagy `"reflectance"` kérésére – a Bayer-módszer szerinti sugárzásérték szélessávú érzékelő esetében értelmetlen, ezért a háttérrendszer inkább kihagyja ezeket a kamerákat, mintsem értelmetlen eredményeket állítson elő.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Az okjelzők az `<level>-not-applicable-to-rgb-cam` mintát követik (minden kihagyott szintre egy bejegyzés, amelyek mindegyike a `level`). A visszaverődés-specifikus kihagyások: `reflectance-skipped-no-fresh-dls` (nincs elérhető friss lefelé irányuló mérési érték), `reflectance-skipped-bound-daq-unavailable (…)` (a kapcsolt DAQ nem volt elérhető), valamint `dls-uncalibrated-band-<nm>` — a sáv nagy része a DAQ fényérzékelőjének radiometrikusan kalibrált tartományán kívül esik (~374–974 nm), ezért a DAQ-alapú abszolút reflektancia-elosztás elutasításra kerül, és a képkocka egyértelműen az érzékelő-válaszra vált át. A forgalomban lévő SKU-k közül csak az F988 váltja ki ezt; ennek a kamerának a támogatott útvonala a reflektancia-panel munkafolyamat.

`processing` szintek:

| Szint | Kimenet |
| --- | --- |
| `"raw"` | Egycsatornás Bayer (monokróm kamerák: az egyetlen sáv) közvetlenül az érzékelőből. |
| `"debayered"` *(SDK alapértelmezett)* | 3-csatornás BGR bilineáris demosaic-on keresztül (monokróm kamerák: 1-csatornás szürkeárnyalatos). |
| `"radiance"` | float32 W/m²/sr/nm a teljes radiometrikus láncon keresztül. Csak multispektrális — a „RGB” kamerák kihagyásra kerülnek. |
| `"reflectance"` | uint16 0..32768 (Pix4D-kompatibilis); abszolút referenciaértékhez élő DAQ-párosítás szükséges. Csak multispektrális. |
| `"display"` | A GUI-előnézethez illeszkedő teljes lánc (CCM + WB + gamma a kamera profilja szerint). |
| `"all"` | **Minden alkalmazható szinthez egy fájl** minden kamera esetében (megfelel a GUI „Capture All” / CLI alapértelmezésének). A visszaküldött `CaptureResult` fájl ezután `(cam, level)`-enként egy képkocka-diktumot tartalmaz, az egyes diktumokban a szinttel; a nem alkalmazható szintek az `.skipped`-ben jelennek meg. A bármely reflexiós kerethez használt DAQ-leolvasás `.daq` kiegészítő fájlként kerül mentésre. |

> **Megjegyzés — az alapértelmezett érték eltér az CLI-től.** Az `ArraySession.capture()` alapértelmezett értéke az `processing="debayered"`; az `chloros-cli lattice array-capture` parancs alapértelmezett értéke az `processing="all"`-re. Az `processing="all"` parancsot kifejezetten az SDK fájlból kell átadni, hogy tükrözze a CLI /GUI többszintű mentési beállításait.

### Felvételi módok és rögzítők

A táblázat felülete tükrözi a GUI felvételi panelt: Egyszeri / Folyamatos / Intervallum / Leggyorsabb zár üzemmódok, valamint két felvevő (élő kompozit videó és nyers sorozatfelvétel → offline újrafeldolgozás).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**az „SDK” folyamatos/intervallum hurokja. Mivel nincs olyan `Ctrl+C`, amellyel szkriptből megszakíthatnánk,**kötelező** átadni az `count` és/vagy az `duration_s` értéket (akkor áll le, ha bármelyiket eléri). Az `interval_s`-et minden futtatás elejétől mérik (a GUI-hoz igazodva). A fennmaradó kwargok közvetlenül átkerülnek az `capture()`-be.
- **`record`** *figyelési szintű*: rögzíti a megjelenített élő kombinált index-kompozitot, ezért a kombinált adatfolyamnak nyitva kell lennie ahhoz, hogy a képkockák beérkezzenek. Tömbönként egy kompozit-rögzítő (kivételt dob, ha már fut egy).
- **`burst` → `build_video`** *elemzési minőségű*: az `burst` nyers képkockákat + képkockánkénti manifesztet + egy `.daq` fájlt ír minden egyes DLS-leolvasáshoz az `<output>/bursts/<base>/` alatt a rögzítési ciklus teljes sebességén (lánc nélkül, exiftool nélkül, élő nézet nélkül). Az `build_video` minden képkockát idő szerint illeszt a legközelebbi `.daq`-hez, és újra futtatja az importáló folyamat sugárzási/visszaverődési/index láncát. Az `products` egy lista az `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}`-ekből (alapértelmezés: a kombinált index). Az `burst().stop()` emellett automatikusan elindít egy legjobb eredményre törekvő kombinált index-építést, amelyet az `build_job` néven ad vissza a leállási eredményben.

#### `RecorderHandle`

Az `ArraySession.record()` és az `ArraySession.burst()` által visszaadott érték. Használja kontextuskezelőként a hatókör kilépésekor történő automatikus leállításhoz, vagy vezérelje manuálisan.

| Tag | Leírás |
| --- | --- |
| `job_id` | Háttérfeladat-azonosító (str). |
| `kind` | `"composite"` (az `record`-ből) vagy `"raw"` (az `burst`-ből). |
| `start_stats` | Az `start` hívás által visszaadott szótár. |
| `result` | `None` futás közben; a végső leállási eredményt tartalmazó szótár a leállás után. |
| `stats(timeout=10.0)` | Élő feladatstatisztikák (leírt képkockák, tényleges fps, eltelt idő). |
| `stop(timeout=60.0)` | A felvevő leállítása; visszaadja és gyorsítótárba menti a végső eredményt. Idempotens (egy második hívás a gyorsítótárban tárolt eredményt adja vissza). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Csatlakozás egy már csatlakoztatott tömbhöz — `attach_array`

Ha a tömb már fut (a GUI megnyitotta, vagy egy korábbi SDK munkamenet meghívta az `connect_array` függvényt), akkor az újracsatlakozás helyett az `attach_array` parancsot használja a hivatkozás megszerzéséhez. Az `connect_array` ilyen esetben mindig a következő hibát jelzi: „A kamera <sn>már </sn>szerepel <sn>a tömbben<id>” hibaüzenetet ad ki ebben az esetben, mivel az `/array/connect` parancs POST-küldése egy olyan tagra, amelynem idempotens; az `attach_array` beolvassa az `/api/camera/array/list`-et, és az array_id vagy a sorozatszám alapján végez egyezéskeresést.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Minta: Azok a SDK szkriptek, amelyek a az asztali grafikus felülettel, először az `attach_array`-et kell megpróbálniuk, és csak akkor térjenek vissza az `connect_array`-re, ha még nincs tömb a poolban.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Fontos — a context-manager kilépése MEGSZAKÍTJA a kapcsolatot.**Az `ArraySession.disconnect()` mindig POST-ot küld az `/array/disconnect`-nek; nincs olyan „attached-not-owned” védelem, mint az `CameraSession` / `DAQSensorSession` esetében. Ha a GUI-val közös bérlőként működik, és nem szeretné lebontani a tömböt a hatókör kilépésekor,**ne használja az `with` blokkot** — tárold a kezelőt egy normál változóban, és hagyd ki a kifejezett `disconnect()`-et:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Hálózati elemzés segédprogram

Hasznos a tömb megnyitása előtt — megmutatja, hogy a javasolt beállítások megfelelőek-e:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

Az `status` az `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` közül (egyébként `error`). Az `auto_capped_fps` azt jelenti, hogy a kért felbontás csak korlátozott triggerfrekvencián illeszkedik az RX gyűrűhöz — tartsa meg a felbontást, és lépjen át az `target_fps=result["recommended"]["recommended_target_fps"]`-ről az `connect_array`-re (lásd a [6. példát](#6-capability-probe-before-connecting-a-4-cam-array)).

**A vetület értelmezése** (ugyanaz a modell, mint a GUI Array Settings panelen):

- **A Burst (`frame_bytes_total`) kameránként összeadódik az egyes kamerák tényleges képpontformátumában.**A**M3M**monokamerák Mono12 (2 B/px) adatfolyamot továbbítanak, függetlenül attól, hogy milyen `pixel_format` értéket adunk meg, így egy 4 kamerás, teljes felbontású képkocka mérete**~25 MB** három mono kamerával, nem pedig a ~12,6 MB, amit a teljes 8-bites feltételezés adna. A háttérrendszer a modell alapján határozza meg az egyes kamerák formátumát.
- **Az Admittance (`burst_fits_nic_ring`) figyelembe veszi a kimeneti sávszélességet**, nem pedig a teljes burst és a gyűrű közötti különbséget: a sim-emit akkor illeszkedik, amikor a gazdagép gyorsabban üríti ki az RX gyűrűt, mint ahogy a kamerák megtöltik. Egy 10G-s gazdagép + 1 GbE-s kamerák**átadják** a teljes felbontást akkor is, ha a burst túllépi a gyűrű kapacitását; egy 1 GbE-s gazdagép blokkolja (`needs_force_slip` / `auto_shrunk`).
- **Az `achievable_fps_max` egy konzervatív soros visszakeresési felső határ** — az `max(readout+emit, N×emit)` esetében a kameránkénti kibocsátás az 1 GbE-s kamerakapcsolatra van korlátozva, az expozíciótól függetlenül. Például ~2,8 fps egy 4 kamerás, teljes felbontású, 12 bites tömb esetén (megegyezik a futásidő alatt mért ~2,7–3,0 értékkel). Teljes modell: [CLI Hivatkozás → Tömb fps és sorozatfelvétel-modell](cli-reference.md#array-fps--burst-model).
- **Túlterhelés (`oversubscribed: true`) azt jelenti, hogy az N × kameránkénti alsó határ meghaladja az ütközésbiztos felső határt** — az fps mezők (`achievable_fps_max` / `fps_bright` / `fps_dark`) értéke 0, és az automatikus-zsugorítás/binning nem tudja kijavítani (ezek a képkockánkénti bájtok számát csökkentik, nem pedig a másodpercenkénti ütemezett bájtok számát). A megoldás a kamerák számának csökkentése, a jumbo keretek használata, vagy egy gyorsabb hálózati kártya; az `max_cams_collision_safe` jelzi a felső határt (6 teljes felbontású kamera 1 GbE-n @ 1500 MTU-val, 9 jumbo keretekkel). A válasz az `aggregate_demand_bps`, `collision_safe_ceiling_bps` és `per_cam_floor_bps` (8 MB/s) kódokat is tartalmazza. Lásd: [Túljegyzés](#over-subscription-the-per-cam-floor).

### Felfedezés és listázás

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

A LATTICE-tömbök a csatlakozás után azonnal elindítják a háttérben a folyamatos AE-t, de egy frissen beállított jelenet konvergenciája egy pillanatig tart. A **Smart-Capture** egy kényelmi funkció: megkérdezi az egyes kamerák expozícióját, megvárja, amíg a rendszer az egész ablakban stabilizálódik, majd elindítja a felvételt. Ez a GUI-val egyenértékű: az asztali alkalmazás „smart” rögzítési gombja ugyanazt a háttér-végpontot hívja meg.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Ha az `ChlorosProject` (következő szakasz) segítségével vezéreljük, több beállítási lehetőségünk nyílik:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

Az intelligens AE-beállítás alapértelmezés szerint konzervatív. Szigorítsuk az `exposure_tolerance_pct` értéket a szigorú radiometrikus munkákhoz; lazítsa meg a gyorsan változó jeleneteknél, ahol csak „elég közel” kell lennie.

---

## DAQ-érzékelő munkamenetek

Állandó háttérpool spektrális érzékelőkhöz (DAQ-U USB-n, DAQ-M BLE-n, DAQ-E Etherneten). Tükrözi a kamera felületét: intelligens felismerés, pool újrafelhasználás, idempotens csatlakozás.

### Intelligens felismerés (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Elsőbbségi sorrend: Ethernet → BLE → USB. Bármelyik explicit utalás átadása rögzíti az átviteli módot.

### Rögzített átviteli mód

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### `DAQSensorSession` metódusok

| Metódus | Leírás |
| --- | --- |
| `status(timeout=10.0)` | Pool-bejegyzés összefoglalása (streaming/felvétel állapota, hullámhossz-tartomány, kalibrációs sha, integrációs idő, frame_avg, AE állapot). |
| `latest(n=1, timeout=10.0)` | Legfeljebb N legutóbbi spektrumkép visszaadása. |
| `stream_start()` / `stream_stop()` | A streaming folytatása / szüneteltetése (a kezelő nyitva marad). |
| `record_start(output_dir=None, device_name=None)` | Elindít egy .daq fájl rögzítését. Visszaadja a fájl elérési útját. Elutasítja a DAQ-U/M esetében, ha nincs AWS kalibrációs csomag (a DAQ-E kivétel). |
| `record_stop()` | A felvétel leállítása. Visszaadja az `{path, rows}` értéket. |
| `disconnect()` | Felszabadítás a poolból. Nincs hatása a csatolt, de nem saját tulajdonú kezelőkre. |

> **A felső határ-korrekciós profilok (`cap_id`) nem tartoznak az „SDK” beállítás körébe.** Az `connect_daq_sensor()` / `DAQSensorSession` nem tesz közzé semmilyen `cap_id` paramétert vagy `set_cap` metódust. Válasszon ki egy flotta-kapacitás-korrekciós profilt a CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) oldalon, vagy a háttérrendszer `/api/daq` HTTP útvonalain keresztül (`/api/daq/connect` és az `/api/daq/<id>/cap-id` az `cap_id`-et fogadja).

### Felfedezés — a csatlakozáshoz szükséges cím megkeresése

Az `discover_daq_sensors()` az USB / BLE / ETH hálózatokat olyan érzékelők után kutatva, amelyeket *megnyithatna*. Ez az `discover_lattice_cameras()` DAQ-megfelelője, és az egyetlen módja a **DAQ-M BLE MAC-címének** megszerzésére — a DAQ-E-nek van gazdagépneve, a DAQ-U-nak pedig COM-portja, de a MAC-címet sem az eszközön nem nyomtatják ki, sem az operációs rendszer nem sorolja fel.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Mező | Leírás |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-port / BLE MAC / gazdagépnév — továbbítás az `connect_daq_sensor`-hez, mint `port=` / `mac=` / `eth_host=`. |
| `display` | Ember számára olvasható címke. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E`, vagy `None` olyan port esetén, amelyet a vizsgálat nem tud azonosítani (az USB soros adapterek szond nélkül nem különböztethetők meg, ezért az ismeretleneket inkább feltünteti, mint elrejti). |
| `extra` | Transzportonkénti részletek (BLE hirdetett név, USB gyártó, DAQ-E ip/fw/…). Az üres értékeket kihagyja. |

| Paraméter | Alapértelmezett | Leírás |
| --- | --- | --- |
| `transports` | mindhárom | A vizsgálatot korlátozó sorozat (vagy csv karakterlánc). Érdemes megadni, ha tudod, mit akarsz — a BLE a leglassabb láncszem. |
| `scan_timeout` | 5 | Átviteli módonkénti szkennelési ablak másodpercben; a háttérrendszer 1–20 közé korlátozza. |
| `timeout` | 60,0 | Az egész hívásra vonatkozó „HTTP” felső határ (ahogyan az „SDK” más részein is). |
| `auto_start_backend` | `True` | Helyi háttérprogramot indít, ha még nincs futó. Távoli `backend_url` esetén soha nem indít. |

> **A poolban már nyitott érzékelők nem jelennek meg.** A csatlakoztatott BLE-periféria leállítja az adását, és egy nyitott COM-portot nem lehet lekérdezni, ezért a felfedezés csak azokat sorolja fel, amelyek *csatlakozásra rendelkezésre áll*. Várható, hogy közvetlenül a csatlakozás után üres eredményt kapunk — a már meglévő eszközökhöz használjuk az `list_daq_sensors()` parancsot. Azokat a transzportokat, amelyeknél a szkennelés nem futtatható (nincs telepítve a bleak / zeroconf), a rendszer kihagyja a hibaüzenet kiadása helyett, így egy Bluetooth-nélküli gép is megkapja az USB- és ETH-válaszokat.

### Listázás

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Együttműködés a GUI-val / CLI

Ha a GUI-ban már van megnyitott érzékelő, az Python-ból az `connect_daq_sensor(port="COM3")` hívása egy `already_connected=True` jelölésű kezelőt ad vissza. A munkamenet `disconnect()` értéke ekkor no-op, így az SDK szkript nem szakítja ki az érzékelőt a GUI alól a program bezárásakor.

### Közvetlen hardverosztályok (háttérprogram nélkül)

Az `daq_sdk`-et az `chloros_sdk` újraexportálja, így az érzékelőket a folyamat elejétőlfolyamaton belül végpontok között vezérelheted a szenzorokat a háttérprogram nélkül:

> **Elérhetőség:**Az `daq_sdk` az Chloros asztali telepítővel együtt érkezik,**nem** a PyPI-csomagban — az `pip install chloros-sdk` az `lattice_sdk`-et biztosítja, de az `chloros_sdk.DAQ_AVAILABLE == False`-et kihagyja. Ezeknek az osztályoknak a használata előtt ellenőrizze ezt a jelzőt; kizárólag pip-et futtató gazdagépen inkább a [`connect_daq_sensor()`](#daq-sensor-sessions) segítségével csatlakoztassa az érzékelőt, amelyhez nincs szükség helyi transzportkönyvtárakra.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Ha megosztott tulajdonjogot szeretne a grafikus felhasználói felülettel, akkor inkább a smart-connect útvonalat (`connect_daq_sensor`) válassza; a szenzort kizárólagosan birtokló, felhasználói felület nélküli szkriptekhez használja a közvetlen osztályokat.

---

## Projekt automatizálás — `ChlorosProject`

Egy elmentett Chloros projekt egy olyan mappa, amely tartalmazza az `cameras.json` + `sensors.json` + `project.json` fájlokat. Az `open_project` betölti a manifesztet, az `connect_all` az összes mentett eszközt online állapotba hozza a mentett beállításokkal — ugyanazzal a hardverállapottal, amit a grafikus felület is létrehozna.

### Minimális példa

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Vagy kontextuskezelőként:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### `ChlorosProject` metódusok

| Metódus | Leírás |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Minden mentett eszköz felderítése és csatlakoztatása. Osztályonkénti csatlakozási jelentést ad vissza. Ha van futó háttérprogram, amely az `127.0.0.1:5000`-en hallgatózik, azt használja; ellenkező esetben csendben visszatér a közvetlen (háttérprogram nélküli) `lattice_sdk` eszközvezérlésre vált át — soha nem indít el háttérprogramot. |
| `disconnect_all()` | Minden kapcsolatot bont. |
| `capture_all(output_dir=".")` | Minden kamerából egy képkocka + minden érzékelőből egy tömb + spektrum. |
| `stream(camera, overlays=False, fps=10.0)` | Generátor, amely BGR `numpy` képkockákat állít elő egy megnevezett kamerából (vagy sorozatból). Az `overlays=False` egy közvetlen `lattice_sdk` képfelvételi hurok (a sorozatok `{serial: frame}` szótárakat eredményeznek). Az `overlays=True` az `ChlorosLocal.camera_stream()`-en keresztül irányítja → a háttérrendszer `/api/camera/<serial>/stream-annotated` MJPEG-adatfolyamát, a kameramentett `ui.overlay` blokkját lekérdezési paraméterként továbbítja. Backend módot és **önálló kamerát** szükséges: a közvetlen üzemmódú kamera `RuntimeError` hibát vet fel (a háttérprogram nem tudja megragadni a folyamat tulajdonában lévő kamerát), míg a tömb `NotImplementedError` hibát vet fel (kameránként összetett átfedéseket hoz létre — egy elemet név szerint továbbít). Egyszeri műveletnek megfelelő: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Igazítást futtat minden jelenleg csatlakoztatott tömbön. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Futtatja a kalibrációs / indexelési folyamatot a projektképein (az `ChlorosLocal.process`-et burkolja; ez a négy az **egyetlen** elfogadott kwarg — az `indices=` stb. `TypeError` hibát dob; az indexeket az `ChlorosLocal.configure()` segítségével). Lusta módon létrehoz egy `ChlorosLocal()`-et, amely automatikusan elindít egy háttérprogramot. |

Attribútumok:
- `proj.cameras` — `Dict[str, CameraHandle]`, név és sorozatszám alapján indexelve.
- `proj.arrays` — `Dict[str, ArrayHandle]`, amelynek kulcsa a név ÉS az array_id.
- `proj.sensors` — `Dict[str, SensorHandle]`, amelynek kulcsa a név ÉS a slot_id.
- `proj.config` — `project.json["config"]` szótár.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Feldolgozási szintek.** Az `capture()`, az `grab()` és az `frame_stream()` mind ugyanazt az `processing`
tokent használnak, és a lánc kumulatív – minden szint futtatja a fölötte lévőket:

| Szint | Kimenet | Megjegyzések |
| --- | --- | --- |
| `raw` | 1-csatornás Bayer, érzékelő-natív | Nincs demosaic. Ezen a szinten nincsenek átfedések. |
| `debayered` | 3-csatornás BGR (**alapértelmezett**) | Bilineáris demosaic. Az egyetlen szint, amely backend mód nélkül is működik. |
| `radiance` | float32, W/m²/sr/nm | Teljes radiometrikus lánc: demoszaik + 3×3 színszétválasztás (multispec) + DSNU + síkkép + NIST-skála, az expozíció és az erősítés kiszámításával, így az értékek abszolútak. |
| `reflectance` | uint16, 32768 = 1,0 | Sugárzás osztva a lefelé irányuló besugárzási intenzitással (ρ = π·L/E). Ehhez DLS/DAQ-leolvasásra van szükség — lásd az alábbi megjegyzést. |
| `display` | 8-bites sRGB-szerű | GUI-nek megfelelő renderelés: CCM + fehéregyensúly + gamma a kamera aktív színprofilján keresztül. |

Bármi, ami nem `debayered`-en kívül minden más backend módot igényel; a közvetlen módú kamera
`NotImplementedError` hibát dob. Az `reflectance`-hez használható lefelé irányuló sugárzási értékre van szükség — a képkocka végpontja automatikusan
beilleszti az összesített DAQ-adatot a kamera DLS-helyére, de ha nincs DAQ-hoz kötve a lánc elutasítja a
reflektancia kimenetet, és őszintén jelzi a visszaminősítést a visszaküldött metaadatokban, ahelyett, hogy csendben
visszaadna egy alacsonyabb minőségű eredményt.

> **Reflektancia DN skála — ne kódold be mereven.** A LATTICE reflektancia az `32768` = ρ 1.0 értéket, és
> XMP `Chloros:PixelScale=32768` címkét jelöl meg; az Survey3 reflektancia az `65535` = ρ 1,0 értéket használja, és nem tartalmaz
> `Chloros:*` címkéket. Olvassa be a címkét, és ossza el vele. Az uint16 tartományban van definiálva, így
> `32768` marad minden olyan formátum esetében, amely újraméretezi (16 bites TIFF, 8 bites PNG /JPG, 32 bites százalék) — először
> normalizálja vissza a tárolt adattípust uint16-ra (8-bitesből ×257, floatból ×65535). Az egyetlen kivétel:
> egy 8-bites forrásból származó rögzítés, amelyet 8-bites TIFF formátumban írtak le, *levágásra kerül*, nem pedig átméretezésre, így nincs skála, amely
> leírná — Chloros ebben az esetben teljesen kihagyja az `PixelScale`-et és a MicaSense-tupelt. A LATTICE reflektanciafájlban hiányzó
> címkét kezelje „nincs érvényes skála”ként, ne pedig alapértelmezettként.

> **Az EXIF-adatok átkerülnek az exportba.** Az `process()` a forrásfelvétel GPS-blokkját
> **és annak ExifIFD-jét** minden termékre, így az exportált fájlok tartalmazzák az `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` és `CameraSerialNumber` kódokat, valamint a
> georeferenciálást is tartalmazzák. Az `FocalLength` kód alapján határozza meg a Pix4D a talajmintavételi távolságot – ennek hiányában
> a rekonstrukció rendkívül hibás méretarányra (egy mért esetben egy 411 m-es terület
> 47,8 km-es lett). A másolat szándékosan nem az `-all:all`: az IFD0 szerkezeti címkéi megzavarják
> a LATTICE kimenetet, az `ExifImageWidth`/`Height` fájlokat pedig azért zárjuk ki, mert azok a forrás
> felvételét írják le, nem pedig az exportált rasztert.

Felvételi-fázisú aljelzők (a radiometrikus szintekre vonatkoznak — `radiance`, `reflectance`, `display`):

| Jelző | Alapértelmezett | Jelentés |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + síkkorrekció + 3x3 keverésmentesítés + NIST radiometrikus skála. |
| `apply_white_balance` | `True` | WB LUT. DLS-támogatás, ha DAQ van a kamerához kapcsolva. |
| `apply_index` | `False` | Vegetációs index értékelése. |
| `index_expression` | `None` | Képlet felülírása. Ha nem üres → automatikusan engedélyezi az indexet. |
| `annotated` | `False` | GUI-díszítések (zebra/rács/csúcsjelzés) átfedése. Nem elérhető az `raw` esetében. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **A visszatérési típus `CapturePathMap`, nem pedig `Dict[str, str]`.**
> Az `chloros_sdk.CapturePathMap` az `Dict[str, Union[str, List[str]]]`: egy szintű
> Az `processing` minden sorozatnak egy útvonalat ad, míg a többszintű (`"all"`, vagy egy
> kifejezett `levels` lista) az adott
> kamerához mentett összes termék **rendezett listáját** adja meg. Egy élő kombinált kompozíció, ha ilyen streamelés folyik, a sorozatszám helyett az extra
> `"combined"` kulcs alatt érkezik. Az a kód, amely az `str`-et feltételezi, a
> lista formátumban megszakad anélkül, hogy bármely típusellenőrző kifogást emelne — a megjegyzés szerint az `Dict[str, str]`
> volt egy ideig a listaformátum megjelenése után, ezért létezik az alias. Normalizálj
> , ha a lapos formátumot szeretnéd:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Tömb-igazítás

Az `ArrayHandle` a teljes igazítási felületet teszi elérhetővé. A profilok alapértelmezés szerint csak a munkamenetre vonatkoznak — az állandósításhoz hívd meg kifejezetten az `export_alignment()`-et.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Csatlakozáskori igazítás

Az `connect_all(align=...)` minden tömböt automatikusan igazít a csatlakozáskor:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Ha nincs megadva, akkor az `project.json["config"]["auto_align_on_connect"]`-re vált vissza.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Közvetlen hardver (háttérrendszer nélkül)

Ha teljesen független szeretne lenni a háttérrendszertől (CI, headless robotok, beágyazott rendszerek), importálja közvetlenül az `lattice_sdk` és az `daq_sdk` fájlokat — mindkettőt az-exportálja az `chloros_sdk`. Figyelem az `CAMERA_AVAILABLE` / `DAQ_AVAILABLE` esetében: az `lattice_sdk` megtalálható a PyPI-csomagban (de ehhez szükség van az Arena SDK futtatókörnyezetre), míg az `daq_sdk` kizárólag az asztali telepítővel érhető el.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Előre beállított értékek és a kioldó

A négy előre beállított érték közül három **szabad futású**: a kamera folyamatosan exponál, és egy
az `capture()` visszaadja a következő képkockát. Az `triggered` kivételt képez – a
kamerát a 2. vonalon lévő hardveres élre állítja be, így addig nem rögzít semmit, amíg az meg nem érkezik.

| Előre beállított érték | Kiváltó jel | Használata, ha |
| --- | --- | --- |
| `default` | szabad futás | általános használat |
| `high_speed` | szabad futás | 8-bites, 60 fps korlát, rövid expozíció |
| `high_quality` | szabad futás | 12 bites, nincs fps-korlát — a szokásos választás állóképekhez |
| `triggered` | **készenlét, 2. vonal** | a kamera egy M8 szinkronkábelhez van csatlakoztatva, és valami más indítja el |

Ha az `triggered`-et választja (vagy maga állítja be az `trigger_mode="On"`-et), és a 2. vonalat semmi sem
vezérli, akkor minden `capture()` időtúllépésbe kerül — helyesen, mivel
a kamerát várakozásra kérted. A SDK elmagyarázza, mi történik ilyenkor; lásd
[SC_ERR_TIMEOUT a rögzítés során](#direct-hardware-backend-free).

> **Megjegyzés — A csatlakozáskor megjelenő „GVSP probe” / `SC_ERR_TIMEOUT -1011` üzenetek nem hibák.**&gt; Csatlakozáskor a SDK megpróbál**jumbo kereteket** (9000 bájtos GVSP-csomagokat) egyeztetni a nagyobb átviteli sebesség érdekében. Közvetlen pont-pont-pont közötti NIC-kapcsolaton (pl. egy link-local `169.254.x.x` cím esetén) a hálózat általában nem képes jumbo keretek továbbítására, ezért ez a próba időtúllépésbe kerül, és olyan sorokat rögzít, mint például:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Ez a **tervezett tartalék megoldás**: az SDK automatikusan visszatér a szabványos 1500-bájtos csomagokra, és a kamera továbbra is normál módon csatlakozik (a következő `[chunk-enable …]` sorok a normál csatlakozási sorozat részét képezik). A rögzítés továbbra is működik.
>
> Ezt a próbát kihagyhatja, de **ez nem csupán a naplóbejegyzések elhallgattatására szolgál — hanem kikapcsolja a jumbo kereteket is.** A kamera a „Don&#x27;t-Fragment” pingekre legfeljebb 1500 bájtig válaszol, függetlenül attól, hogy milyen jó a hálózatod, így a ping-teszt önmagában soha nem tudja felismerni a jumbo kereteket; ezt csak ez a próba képes megtenni. Ha letiltod, a kamera örökre a szabványos 1500 bájtos csomagokat fogja használni, bármilyen hálózaton:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Csak olyan hálózaton érdemes ezt használni, amelyről *biztosan tudod*, hogy nem támogatja a jumbo-csomagokat, ahol ez kameránként körülbelül egy másodpercet spórol a csatlakozási időből. Mivel ez valódi kompromisszum, nem pedig pusztán kozmetikai változtatás, az „SDK” (Hálózati beállítások) menüpont most már jelzi ezt, ha használod:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Hagyd békén, hacsak nincs rá okod.** Ha engedélyezve marad, minden csatlakozáskor újra felméri a tényleges hálózati környezetet: csatlakozz egy jumbo-kompatibilis kapcsolóhoz, és a következő csatlakozáskor a rendszer magától felismeri a jumbo-csomagokat, konfigurálás és újraindítás nélkül.
>
> Ha *szeretnéd* a jumbo-átviteli sebességet, akkor engedélyezd a jumbo-t végpontok között (NIC MTU 9000 + egy olyan kapcsoló, amely átadja őket), vagy rögzítsd az `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` paranccsal, ha tudod, hogy a kapcsolat támogatja — bár a parancsonkénti `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` használatát részesítsd előnyben a végleges beállítással szemben, mivel a rögzített méret kihagyja a próbafolyamatot, és megakadályozza az előtte lévő hálózathoz való alkalmazkodást. Az útvonalon lévő **minden** eszköznek át kell engednie a jumbo-csomagokat — beleértve minden PoE-elosztót vagy -befecskendezőt is, ami általában az oka annak, hogy egy egyébként jumbo-kompatibilis rendszer nem tudja továbbítani őket.

> **Az `SC_ERR_TIMEOUT -1011` hiba az `capture()` / `grab*()` során egy másik probléma — ez valódi hiba.**&gt; A fenti megjegyzés kizárólag az `-1011` kódra vonatkozik, amelyet a**connect-time probe**rögzített. Ugyanez a hiba, ha**rögzítés** során jelentkezik, azt jelenti, hogy a kamera rendben csatlakozott, de nem küld képeket:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> A döntő jel az, ha a kamera *vezérlő* csatornája rendben van – az észlelés működik, a beállítások és az `[chunk-enable …]` írások mind sikeresek –, miközben *minden* képkocka időtúllépést okoz.
>
> **A szokásos ok az, hogy a kamera hardveres kioldásra van beállítva.** Az `trigger_mode="On"` és az `trigger_source="Line2"` esetében a kamera egyáltalán nem küld semmit, amíg az M8 szinkronkábelre nem érkezik elektromos jel. Ha nincs kábel, amely ezt a vonalat vezérli, minden képfelvétel örökké vár. A kamera nem romlott el, és a hálózat is rendben van — pontosan azt teszi, amit megparancsoltak neki.
>
> Az `CameraSettings()` és az `default` / `high_speed` / `high_quality` beállítások szabad futást engedélyeznek, és egy élesítés alatt időtúllépés miatt megszakadó rögzítés magyarázatot ad a helyzetre, ahelyett, hogy pusztán egy `-1011` kódot írna ki. Az `PRESETS["triggered"]` a tervezésnek megfelelően élesíti a a 2. vonalat, a tervezésnek megfelelően.
>
> Bármely kamera szabad futásának kikényszerítéséhez:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Ha az `trigger_mode="Off"` parancs mellett is időtúllépés történik, akkor a kamera valóban nem továbbít adatokat — küldje el nekünk a naplófájlt és az `ip link show` parancsot.

#### Színprofilok (RGB élő előnézet) — `set_color_profile`

Az `LatticeCamera.set_color_profile(profile, custom_cct_k=None)` beállítás kiválasztja a kijelző színprofilját a **élő előnézethez** a RGB kamerákon (a multispec kamerák figyelmen kívül hagyják ezt a beállítást):

| Profil | Jelentés |
| --- | --- |
| `raw` | A radiometrikus lánc teljes mellőzése. |
| `linear` | DSNU + flat + WB, nincs CCM, nincs gamma. |
| `natural` | Lineáris + mért CCM + sRGB gamma, kizárólag az egyszerű utómunkával (színárnyalat-simítás + fényes részek telítettségének csökkentése) — a valósághű alapbeállítás. |
| `enhanced` | `natural` plusz a teljes hub-parity utómunka (színszegélyek eltávolítása, élénkség, CLAHE helyi kontraszt). Gazdagabb megjelenés, nagyjából **kétszeres képkockánkénti utómunkaköltséggel**, tehát alacsonyabb élő képkockasebességgel. |
| `custom_temp` | `natural`, de a fehér egyensúly (WB) rögzítve az `custom_cct_k` Kelvin-értékre (a DLS figyelmen kívül hagyva; a háttér oldalon 2000–10 000 K-ra korlátozva a háttérrendszer oldalán). |

A profil **kizárólag élő előnézetre** szolgáló sebesség-/megjelenés-szabályozó: a mentett felvételek mindig a teljes, gazdag végső megjelenést kapják, függetlenül a kiválasztott profiltól, így az `natural` kiválasztása a képkocka-idő visszanyerése érdekében nem rontja a lemezre kerülő anyag minőségét. Egy ismeretlen profil az `ValueError` értéket emeli; amikor egy chloros háttérprogram elérhető, a változás POST-kéréssel továbbításra kerül hozzá is, így a következő előnézeti képkocka már tükrözi azt (a direct-SDK felhasználók háttérrendszer nélkül is megkapják a beállításváltozást).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Mono (M3M) kamerák és `Calibration`

Egy mono **M3M** kamera (`M3M-<lens>-F<wavelength>`) egysávos: egy szürkeárnyalatos sík, nincs Bayer-mozaik, nincs 3×3 spektrális-átviteli mátrix. Az `Calibration` felismeri ezt, és egy `is_mono` jelzőt tesz közzé. A reflexió továbbra is sávonkénti radiometrikus térképként érvényes (a szétkeverés identitásmátrix), de az egyetlen kamerán végzett többsávos számítások értelmes eredményt adnak, nem pedig értelmetlen értékeket:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Ha monokróm hardverrel szeretnénk növényzetindexet készíteni, egyesítsünk több, különböző hullámhosszúságú M3M kamerát egy igazított többsávos rétegbe (lásd [Tömbigazítás](#array-alignment)), és az indexet ne egy kamerán, hanem ezen a rétegen számítsuk ki.

DAQ közvetlen mód:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **`apply_sensor_settings` elfogadott kulcsok**— pontosan `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; elavult, helyette az `cap_id` használata ajánlott), `filter_model` (DAQ-M) és `cap_id` (minden DAQ-típus; `None`/`""`/`"none"` = puszta érzékelő, nincs sapkakorrekció). Az ismeretlen kulcsokat**figyelmen kívül hagyja** — pl. az `{"integration_time": 64}` nem hajt végre semmilyen műveletet (`integration_time_ms`-nek kell lennie). Visszaadja az `{"applied": [...], "errors": {...}}` értéket, és soha nem dob kivételt.

Az `chloros_sdk` csak a fent használt alapfelületet exportálja újra. A teljes `daq_sdk` nyilvános API (22 név) a következőket adja hozzá — ezeket közvetlenül az `daq_sdk`-ból kell importálni:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Kivételek

Fogja meg az alaposztályt, hogy kezelje a „Chloros-ban bármi rosszul sült el” helyzeteket:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> Az `ChlorosAuthenticationError` és az `ChlorosConfigurationError` a többi elemmel együtt a legfelső szinten kerülnek exportálásra; az ábrán látható módon az `chloros_sdk.exceptions`-ből is importálhatók.

Hierarchia:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Teljes folyamatot bemutató példák

### 1. Mappa feldolgozása egyéni haladási sávval

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Élő LATTICE-tömb → Reflektancia + DAQ-referencia

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Projektvezérelt rögzítési kampány

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Többkamerás képkocka-folyam → NumPy-feldolgozási folyamat

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Fej nélküli, közvetlenül a hardverhez kapcsolódó (háttérrendszer nélküli) rögzítési szkript

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Képességvizsgálat egy 4-kamerás rendszer csatlakoztatása előtt

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Felvételi recept-egyenértékű (tiszta Python)

Az CLI recept-DSL-jének van egy közvetlen Python-egyenértékűje:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Backend Auto-Start

A smart-connect belépési pontok — `connect_camera`, `connect_array`, `connect_daq_sensor` és `discover_lattice_cameras` — vékony HTTP kliensek , amelyek feltételezik, hogy egy háttérszerver figyel az `127.0.0.1:5000` porton (a smart-connect felület alapértelmezett URL). Ha a grafikus felület (GUI) vagy a CLI már fut, akkor az egyik biztosan elérhető. Egy puszta szkriptből indítva viszont előfordulhat, hogy nincs — ezért ezek a függvények **automatikusan elindítják a csomagban szereplő háttérprogramot** (ablak nélkül, ugyanúgy, ahogyan az `ChlorosLocal` is teszi) az első hívásuk előtt, majd legfeljebb `backend_startup_timeout` ideig várnak, amíg az elindul.

Szabályok:

- **Csak a helyi URL futtatódik el.** A `backend_url`, amely az `localhost` / `127.0.0.1` / `[::1]`-re mutató `backend_url` megengedett; bármely más gazdagépet másé gépének tekintünk, és azt soha nem indítjuk el.
- **A háttérprogram újrafelhasználás céljából futva marad** (CLI-hez hasonlóan) — a szkript leállásakor nem történik implicit leállítás. A szkript újbóli futtatásakor a futó háttérprogramot újra felhasználja.
- **Az `auto_start_backend=False` paranccsal le lehet tiltani** bármelyik ilyen hívásnál (pl. ha távoli háttérprogramra mutat, vagy maga kezeli a háttérprogram életciklusát).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Ha a csomagban szereplő bináris fájl nem található meg vagy nem indítható el, a következő HTTP hívás egy kezelhető, **platformfüggő** `ChlorosConnectError` hibajelentést generál ahelyett, hogy egyszerűen csak a kapcsolat megtagadását jelentené — az Windows oldalon az asztali alkalmazásra vagy egy `chloros-cli` parancsra irányít; Linux-en (GUI nélkül) pedig egy `chloros-cli` parancsra vagy az `.deb`-re irányít.

---

## Környezet és fejlécek

Az SDK minden háttérHTTP-hívást `X-Chloros-Client: sdk`-szel jelöljelöléssel látja el. A háttérrendszer a SDK / CLI licencelési szabályokat alkalmazza (bejelentkezés **és** fizetős Chloros+ csomag szükséges), a GUI ingyenes szintje helyett. Ez az importáláskor automatikusan beállítódik — Önnek semmit sem kell tennie.

Az `http://localhost` és az `http://127.0.0.1` a helyi háttérrendszerként kerülnek felismerésre. Más gazdagépekhez (pl. a saját elemzési szolgáltatásodhoz) változatlanul maradnak.

A háttérszolgáltatást (URL) felülírhatod az `backend_url=` (vagy `api_url=` az `ChlorosLocal` esetén) megadásával:

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(A nem loopback `backend_url` csak a source/dev háttérrendszert éri el — a szállított háttérrendszerek csak a loopbackhoz kapcsolódnak; az alagútmintát lásd a Távoli háttérrendszer módnál.)

---

## Verziókezelés és kompatibilitás

- A SDK verziója `chloros_sdk.__version__` néven érhető el.
- Az SDK a mellékelt háttérprogram-verzióhoz igazítja a viselkedést. A régebbi SDK és az újabb háttérprogramok keverése általában működik (előre kompatibilis végpontok), de az újabb SDK és a régebbi háttérprogramok keverése `404` hibákat okozhat az új végpontokon — frissítse az asztali alkalmazást, hogy megfeleljen.
- A smart-connect felület (`connect_camera` / `connect_array` / `connect_daq_sensor`) és a hálózatelemző végpont stabil JSON sémákat ad vissza; az új mezők kiegészítő jellegűek.

---

## Hibaelhárítási tippek

- **`ChlorosAuthenticationError: Login required`** → Futtassa egyszer az `chloros-cli login EMAIL PASSWORD` parancsot ezen a gépen, vagy jelentkezzen be az Chloros asztali alkalmazáson keresztül.
- **`ChlorosConnectError: No Chloros backend is running …`** → A smart-connect hívások automatikusan elindítanak egy helyi háttérprogramot, ezért ez a hibaüzenet csak akkor jelenik meg, ha a csomagban szereplő bináris fájl nem található vagy nem indítható el (pl. egy kizárólag pip-et futtató, asztali csomaggal nem rendelkező gazdagépen). Az üzenet platform-függő: az Windows rendszeren nyissa meg az asztali alkalmazást, vagy futtasson bármely `chloros-cli` parancsot; az Linux rendszeren futtasson egy `chloros-cli` parancsot (nincs grafikus felület), vagy telepítse az `.deb` programot. Távoli háttérprogram esetén adja meg a `backend_url=` (és `auto_start_backend=False`) parancsot.
- **`CAMERA_AVAILABLE == False`** importáláskor → Az `lattice_sdk` nem tudta betölteni (általában az Arena SDK futásidejű DLL-jei nincsenek telepítve). A nem kamerás felület továbbra is működik.
- **Az Array connect nem natív felbontást ad vissza**→ A háttérprogram smart-prep funkciója automatikusan kicsinyíti a képkocka méretét, hogy illeszkedjen a vezetékhez. Használja az `analyze_array_network()` parancsot az ok megállapításához, majd vagy frissítse a kapcsolatot, fogadja el a méretcsökkentést, vagy adja meg az `force_tier="slip-emit-and-capture"` parancsot a szekvenciális rögzítéshez. A méretcsökkentés biztonsági hálója**nem** fedezi az összesített túljegyzést (`oversubscribed: true`, fps mezők 0): a vezetékre vonatkozó túl sok kamera problémáját nem lehet binninggel/ROI-val megoldani — csökkentse a kamerák számát, engedélyezze a jumbo kereteket, vagy váltson gyorsabb hálózati kártyára (lásd [Túljegyzés](#over-subscription-the-per-cam-floor)).
- **Az `analyze_array_network()` jelzi, hogy a hálózati kártya RX gyűrűje rendkívül kicsi (~0,26 MB)-nek jelzi / a csatlakozási kapukon „FRAMES WILL DROP” felirat jelenik meg** → A gazdagép hálózati kártyájának vételi gyűrűje az alapértelmezett értéken van (gyakran 32-re áll vissza a hálózati kártya illesztőprogram frissítése után). Realtek USB 10GbE adapter esetén állítsa be az `ReceiveBufferLen=256` és az `PendingReceives=64` (emelt szintű) értékeket, majd indítsa újra a háttérprogramot, hogy újraolvassa a gyűrűt. A teljes eljárás: [CLI Hivatkozás → Gazdagép hálózati kártyájának beállítása és hangolása](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **A gazdagép lefagy újraindításkor/leállításkor, később WMI `Invalid class` hibák jelentkeznek / a hálózati kártya nem aktiválódik** → Elavult USB 10GbE illesztőprogram okozza az `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Frissítse az adapter illesztőprogramját a legújabb verzióra (≥ 2026), majd-állítsa be újra a fogadógyűrű beállításait. Lásd [CLI Referencia → Gazdagép NIC beállítása és hangolása](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Reflektancia elutasítva** → Egy aktív DAQ-t kell hozzárendelni a kamerához (vagy sorozathoz) az abszolút skálájú reflektancia érdekében. A társítást végezze el a grafikus felhasználói felületen keresztül, vagy használja az `processing="radiance"` (W/m²/sr/nm) beállítást, amelyhez nincs szükség párosított érzékelőre.
- **Az `smart=True` rögzítése a vártnál hosszabb ideig tart** → Az AE konvergenciája a jelenet dinamikájától függ; szigorítsa az `exposure_tolerance_pct` értéket, vagy rövidítse az `stability_window_s` értéket, ha gyorsabb (kevésbé stabil) kiváltást szeretne.

---

## Lásd még

- [CLI Referencia](cli-reference.md) — minden CLI alparancs egy SDK hívást tükröz.
- [DAQ-érzékelő útmutató](../daq/README.md) — érzékelőspecifikus bekötési, kalibrálási és rögzítési szabályok.
- Online dokumentáció: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
