# API: Python SDK

{% hint style="info" %}
**A teljes API-t keresed?** Ez az oldal egy gyakorlati útmutató. Minden nyilvános osztály, metódus, pontos szignatúra és másolható példa megtalálható az [SDK-referenciában](reference/sdk-reference.md), amely az AI-asszisztensek számára lett optimalizálva.**Mesterséges intelligencia-asszisztenssel dolgozol?** Illeszd be ezt az URL-t a csevegőbe, hogy a teljes, aktuális Chloros 1.3.0 API-t tartalmazza:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

A kézikönyv minden oldala nyers Markdown formátumban elérhető a kisbetűs slug + `.md` alatt, a teljes kézikönyv pedig az `https://mapir.gitbook.io/chloros/llms.txt` alatt van indexelve.
{% endhint %}

A **Chloros Python SDK** (`chloros-sdk` a PyPI-n) Pythonból vezérli az asztali alkalmazás összes funkcióját: a képek tömeges feldolgozását, az élő LATTICE kamera és sorozat vezérlését, a DAQ fényérzékelő-munkameneteket, valamint a mentett projektek automatizálását. Ez egy vékony réteg ugyanazon a helyi háttérrendszeren, amelyet a grafikus felhasználói felület (GUI) és a parancssori felület (CLI) is használ (HTTP az `127.0.0.1:5000` alatt), így a viselkedés mindhárom felületen azonos.

## Telepítés

Az SDK-t a Chloros asztali csomag tartalmazza, amely a feldolgozási háttérrendszert és a hardveres futtatókörnyezeteket is biztosítja, és verziója mindig egyezik a csomagéval.

**

1. lépés — Telepítse a Chloros-t.** Windows: futtassa az asztali telepítőt (alapértelmezett elérési út: `C:\Program Files\Chloros\`) a [Letöltés](download.md) oldalról. Linux: telepítse az `.deb` csomagot az `sudo apt install ./chloros_1.3.0_<arch>.deb` paranccsal ([Linux telepítés](linux/linux-installation.md)).**

2. lépés — Az SDK ellenőrzése** (Python 3.7+). Általában nincs mit telepíteni:

* **Windows:** a telepítő a csomagban lévő SDK wheel fájlt telepíti a rendszer Pythonjébe (ha nincs telepítve Python, akkor ezt a lépést kihagyja).
* **Linux:** az `.deb` a rendszer `python3`-hez tartozó SDK-t telepíti az `/usr/lib/python3/dist-packages`-be. A csomag által ajánlott követelmények: az `python3-numpy` és az `python3-requests`; az `apt install` tartalmazza őket, az `dpkg -i` viszont nem.
* **A virtualenv** nem látja a rendszer csomagjait; telepítse a csomagban található wheel fájlt a virtualenv-be: Linux esetén az `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` fájlt, vagy a Windows telepítési mappájának `sdk` mappájában található `chloros_sdk-*.whl` fájlt (alapértelmezés szerint az `C:\Program Files\Chloros\sdk\`).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

Chloros-csomag nélküli gépen, amelynek SDK-je egy másik gépen futó háttérprogramot vezérel (lásd [Mit kapunk egy kizárólag pip-pel végzett telepítéssel](#what-a-pip-only-install-gets)), telepítse a PyPI-ről: `pip install chloros-sdk`. A PyPI a kiadási verziók megjelenésekor frissül, így mindig a legújabb stabil kiadással egyezik meg.

**

3. lépés — Jelentkezzen be egyszer minden gépen:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

A hitelesítő adatok az `~/.chloros/` fájlban kerülnek tárolásra (mindkét platformon). Windows-en ugyanúgy bejelentkezhet az asztali alkalmazás „Felhasználó” <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> fülén keresztül is. Az SDK-hez fizetős Chloros+ csomag szükséges — lásd az alábbi [Licenckövetelményeket](#license-requirement).

| Követelmény | Részletek |
| --- | --- |
| **A Chloros telepítése** | Windows: asztali telepítő; Linux: `.deb` csomag (tartalmazza a háttérprogramot és az SDK-t) |
| **Python** | 3.7 vagy újabb (3.10-es verzión fejlesztve/tesztelve) |
| **Operációs rendszer** | Windows 10/11 64 bites, Ubuntu 22.04 LTS vagy újabb, vagy NVIDIA Jetson (JetPack 6) |
| **Licenc** | Aktív Chloros+ bejelentkezés, bármely fizetős csomag (Copper vagy magasabb) |

## 60 másodperc alatt kész

Egyetlen hívással létrehozhatunk egy projektet, importálhatunk egy mappát, beállíthatjuk a feldolgozást és futtathatjuk a feldolgozási folyamatot — automatikusan elindítva a háttérprogramot, ha az még nem fut:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(Linux rendszeren használjon Linux elérési utakat: `/home/user/drone_images/flight001`. Az SDK mindkét platformon azonos módon működik.)

LATTICE-felvételi mappát dolgoz fel? Használja a LATTICE-kompatibilis wrappert – ez a megfelelő alapértelmezéseket alkalmazza (nincs panelcél-felismerés, standard debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — teljes folyamatvezérlés

Ha egy soros parancsnál bonyolultabb feladatot szeretne végrehajtani, használja az `ChlorosLocal` parancsot. Ez a már futó háttérprogramot (az asztali alkalmazásét) használja, vagy az első használatkor elindítja a telepített háttérprogramot (`auto_start_backend=True`; Windows rendszeren a Chloros telepítőmappájának `backend` mappájából, alapértelmezés szerint az `C:\Program Files\Chloros\backend\`-et), létrehozza és konfigurálja a projekteket, figyelemmel kíséri a folyamatot, és futás utáni összefoglalót ad vissza.

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

{% hint style="info" %}
Tartsa meg az alapértelmezett `http://127.0.0.1:5000`-et, ne cserélje ki `localhost`-re — Windows rendszeren az `localhost` először az `::1`-re oldódik fel, és kérésenként körülbelül 2 másodpercet vesz igénybe a kizárólag IPv4-et támogató háttérrendszer esetében.
{% endhint %}

Használja kontextuskezelőként. A blokkból való kilépés véget vet ennek az instanciának a háttérrendszer használatának, nem pedig magának a háttérrendszernek: az továbbra is fut a következő szkript számára, és 30 perc használat nélkül magától leáll. Az `shutdown_backend()` leállítja azt az instanciát, amelyet ez indított el.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` a következő kulcsszavakat fogadja el: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` és `custom_settings`. A fő értékek:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

A LATTICE-specifikus beállítók (`input_level`, `radiometric_output`, valamint az `array_alignment*` család) teljes értéktáblázataival együtt az [SDK-referencia](reference/sdk-reference.md#supported-values) című dokumentumban találhatók.

### A folyamat nyomon követése

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### A futás utáni összefoglaló olvasása — és az üres futások észlelése

A befejezéskor az `process()` a háttérrendszer feldolgozási összefoglalóját `result["summary"]` néven csatolja. Az `summary["hints"]` minden bejegyzése egy teljes mondat, amely elmagyarázza a figyelemre méltó eseményeket — például, hogy egy futtatás miért nem eredményezett kimenetet — és minden hibaüzenet Python `UserWarning` formátumban is újraküldésre kerül, így az üres futtatások önmagukban is diagnosztizálhatók, még akkor is, ha soha nem vizsgálod meg a dict-et:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**Az `process()` nem dob kivételt, ha egy futtatás nem eredményez képeket.** Ez az egyetlen pont, ahol az SDK és a CLI szándékosan eltér egymástól: Az `chloros-cli process` a „termékeket kérték, de egyet sem írtak” állapotot hibaként kezeli, és nullától eltérő értékkel lép ki, míg az SDK normál módon tér vissza, és az állapotot az `summary` / hints segítségével jelenti. Ha a folyamatnak üres futás esetén le kell állnia, ellenőrizze ezt saját maga — vizsgálja meg az `summary` kódot (vagy számolja meg a projektmappa alatti fájlokat), ahelyett, hogy a kivételre hagyatkozna.
{% endhint %}

## Smart Connect — élő hardver

Három segédprogram nyit állandó munkameneteket a háttérrendszer hardverpooljában — ugyanabban a poolban, amelyet a CLI, más szkriptek és a GUI kamerafüljei is használnak, így az SDK-szkriptek velük együtt működhetnek anélkül, hogy versengeniük kellene a soros portokért vagy a hálózati sávszélességért. Mindhárom automatikusan elindít egy helyi háttérrendszert, ha még nem fut ilyen. A GUI DAQ-panele nem része a poolnak: zárjon be egy fényérzékelőt a GUI-ban, mielőtt egy szkriptből megnyitná.

### Egyetlen LATTICE kamera — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**A relatív útvonalak a szkript futási helyére mutatnak (változás a 1.3.0-s verzióban).** Helyi háttérrendszer esetén egy relatív `output_dir` (beleértve az alapértelmezett `"output"`-et is) — és a többi útvonal, amelyet a háttérrendszer megnyit (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — a szkript munkakönyvtárához viszonyítva kerülnek feloldásra, így az `cam.capture("output/")` az `<current folder>/output/`-ba ír. Az 1.2.0-s verzióban egy telepített háttérprogram a saját telepítési mappájában értelmezte őket, és „Hozzáférés megtagadva” hibaüzenettel bukott meg. Távoli háttérprogram esetén az elérési útvonalakat a beírt formában küldi el.
{% endhint %}

### Szinkronizált tömb — `connect_array`

Az `connect_array` a többkamerás rendszerekhez ajánlott belépési pont. Ugyanazt az intelligens előkészítési folyamatot futtatja, mint a grafikus felhasználói felület: hálózati elemzés, szinkronizációs szint automatikus kiválasztása, PTP időszinkronizálás, kameránkénti képpontformátum-kiválasztás, AE-beállítás és GPIO-kioldó aktiválása. Az **első soros eszköz a master** (ez indítja el a hardveres trigger impulzust); a többi slave.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Adja hozzá az `smart=True` parancsot bármely tömbfelvételhez, hogy a kioldás előtt megvárja, amíg az automatikus expozíció minden kamerán beáll.

Az 1.3.0-s verzió újdonsága, hogy az `ArraySession` parancs fájlok írása nélkül is közvetlenül szinkronizált képkockákat ad vissza:

* Az `arr.grab()` kameránként egy képkockát ad vissza, mindet egyetlen kioldásból, numpy-tömbök formájában (`{serial: array}`); az `.group_id` a közös indító, az `.errors` pedig megnevezi a kihagyott kamerákat;
* az `arr.frame_stream(fps=5.0)` tickenként egy ilyen készletet ad vissza, a saját feldolgozási folyamatodban történő élő feldolgozáshoz;
* Az `arr.continuous("output/")` futtatja a háttérprogram saját folyamatos rögzítését (a GUI „Folyamatos” módját), és visszaad egy `ContinuousHandle`-et az `status()`-szel, `wait()` és `stop()` kódokat — használja ezt a rögzítési ciklus helyett, ha minden triggert nagy sebességgel kell mentenie.

A rögzítési módokról (Egyszeri / Folyamatos / Intervallum / Leggyorsabb), a rögzítőkről, a burst-to-video funkcióról és a tömb-igazításról lásd az [SDK-referenciát](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### DAQ fényérzékelő — `connect_daq_sensor`

Argumentumok nélkül az `connect_daq_sensor()` automatikusan felismeri az átviteli módot (elsőbbségi sorrend: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Minden keret tartalmazza a 135 pontos `spectrum` értéket (kalibrálás után W/m²/nm), egy `is_saturated` jelzőt, valamint a CIE `x`, `y`, `z` értékeket. Egy adott érzékelő vagy átviteli mód rögzítéséhez adjon meg egy kifejezett utalást:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Ne feledje, hogy a felső határ-korrekciós profilok (`cap_id`) **nem** tartoznak az SDK-beállítások közé — ezeket inkább az `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap` parancsokkal válassza ki.

### Mentett projektek — `open_project`

Egy mentett Chloros-projekt megőrzi a hozzá kapcsolt hardvereket (`cameras.json` + `sensors.json`, valamint `project.json`), az `chloros_sdk.open_project(path)` pedig egyszerre tudja újra csatlakoztatni kameráit, érzékelőit és fényérzékelőit, és eszköznév alapján vezérelheti a rögzítéseket. A fényérzékelők a futó Chloros-háttérrendszeren keresztül csatlakoznak újra, a mentett beállításaikkal és felvételi korlátjukkal együtt. Lásd a [Projekt automatizálás](reference/sdk-reference.md#project-automation--chlorosproject) című részt a referenciaanyagban.

## Mit kap egy kizárólag pip-telepítés

A hardverfelületek használata előtt ellenőrizze a modulszintű elérhetőségi jelzőket:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Egy olyan gazdagépen, amelyen **csak** az `pip install chloros-sdk` van, és nincs Chloros asztali csomag:

* Az `ChlorosLocal`, `process_folder` és `process_lattice_capture` **nem** működnek — ezekhez szükség van a desktop telepítőben található háttérbináris fájlra.
* A smart-connect segédprogramok (`connect_camera`, `connect_array`, `connect_daq_sensor`) tisztán HTTP-kliensek, így egy másik gépen futó háttérprogrammal is működnek — azonban a mellékelt háttérprogramok csak a loopback-címre kapcsolódnak, ezért a portot magának kell átirányítania (pl. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`), és az `backend_url="http://127.0.0.1:5000"`-et az `auto_start_backend=False`-szel együtt kell átadnia. Lásd [Távoli háttérszerver mód](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* A közvetlen hardveres LATTICE osztályok (`LatticeCamera`, `CameraPool`, …) importálhatók, de szükségük van az asztali csomagból származó Arena SDK futásidejére — ennek hiányában az `CAMERA_AVAILABLE` az `False`.
* A DAQ-érzékelők kizárólag egy (alagúton keresztül elérhető) háttérrendszeren keresztül vezérelhetők — `connect_daq_sensor()`, vagy egy projekt `proj.sensors`-je; a háttérrendszer a saját gépén nyitja meg az érzékelőt.

## AI-asszisztensek — `chloros-mcp`

Az SDK tartalmazza az `chloros-mcp`-et, egy MCP-kiszolgálót, amely lehetővé teszi a Claude Desktop, a Claude Code, a Cursor és más MCP-kliensek számára a Chloros vezérlését: képek feldolgozását, valamint a LATTICE kamerák, érzékelősorok és DAQ fényérzékelők működtetését. Eszközei lefedik az SDK funkcióit. Az SDK-val együtt települ, így a Claude Code esetében a beállítás egyetlen sor:

```bash
claude mcp add chloros -- chloros-mcp
```

Egyéb kliensek: [Chloros használata AI-asszisztensekkel](ai-assistants.md). Beállítások és az összes eszköz: [MCP-kiszolgáló (AI-asszisztensek)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Licenckövetelmények

Az SDK-hez való hozzáféréshez aktív Chloros+ bejelentkezés szükséges bármely fizetős csomagban — **Copper vagy annál magasabb**(Copper / Bronze / Silver / Gold); az ingyenes Iron csomag nem biztosít SDK/CLI-hozzáférést. Az ellenőrzés**szerveroldali**: minden SDK-kérésnek tartalmaznia kell egy aktív munkamenetet és egy fizetős csomagot, ellenkező esetben a háttérrendszer `403` / `PLAN_UPGRADE_REQUIRED` hibakódot ad vissza (amelyet az `ChlorosLicenseError` segédfüggvény az `ChlorosLocal`-re, az `ChlorosConnectError` segédfüggvény pedig az `connect_*`-re emel). A kijelentkezett hívó a következő hibakódokat kapja: `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) kódot kap – az `chloros-cli login` újbóli futtatása az első esetet megoldja, a másodikat azonban nem.

Az offline használat a csomag türelmi időszakán belül működik: a szintet a szerver-érvényesítési gyorsítótárból (5 perc) vagy az aláírt, gépre kötött licencgyorsítótárból olvassa ki (havi csomagok esetén 30 napig; éves előfizetés esetén az előfizetés lejáratáig). A türelmi időszak lejárta után a csomag ingyenesre vált, és az SDK-hozzáférés leáll, amíg a gép egyszer kapcsolatba nem lép a szerverrel. Az `chloros-cli status` az ingyenes szinten is elérhető marad, így az ok mindig látható. Lásd: [Chloros+ bejelentkezés](chloros+-login.md).

## Kivételek

Fogja meg az alaposztályt a „Chloros bármilyen hibájának” kezeléséhez:

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

Minden pipeline-kivétel (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) az `ChlorosError`-ből származnak. Egy apró bökkenő: az `ChlorosConnectError` — amelyet kizárólag az `connect_camera` / `connect_array` / `connect_daq_sensor` — a sima `Exception`-ből származik, **nem** az `ChlorosError`-ből, így az `except ChlorosError` nem fogja elkapni. A teljes hierarchia az [SDK-referenciában](reference/sdk-reference.md#exceptions található.

## Lásd még

* [SDK-referencia](reference/sdk-reference.md) — a teljes API-felület, AI-asszisztensekre optimalizálva.
* [CLI-referencia](reference/cli-reference.md) — minden CLI-alparancs egy SDK-hívást tükröz.
* [Chloros használata AI-asszisztensekkel](ai-assistants.md) — `chloros-mcp` beállítási és prompt-receptek.
* [Letöltés](download.md) — telepítőprogramok Windows és Linux rendszerekhez.
