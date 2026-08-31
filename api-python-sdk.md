# API : Python SDK

{% hint style="info" %}
**A teljes API-et keresi?** Ez az oldal egy gyakorlati útmutató. Minden nyilvános osztály, metódus, pontos szignatúra és másolható példa megtalálható az [SDK Referenciában](reference/sdk-reference.md), amely AI-asszisztensek számára van optimalizálva.**AI-asszisztenssel dolgozol?** Illeszd be ezt az URL szöveget a csevegőbe, hogy az rendelkezzen a teljes, aktuális Chloros 1.2.0 API verzióval:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

A kézikönyv minden oldala nyers Markdown formátumban elérhető a kisbetűs slug + `.md` formában, a teljes kézikönyv pedig az `https://mapir.gitbook.io/chloros/llms.txt` címen található meg az indexben.
{% endhint %}

A **Chloros Python SDK** (`chloros-sdk` a PyPI-n) vezérli az asztali alkalmazás minden funkcióját az Python-től kezdve: a képek tömeges feldolgozását, az élő LATTICE kamera és mátrix vezérlését, a DAQ fényérzékelő-munkameneteket, valamint a mentett projektek automatizálását. Ez egy vékony réteg ugyanazon a helyi háttérrendszeren, amelyet a GUI és az CLI is használ (HTTP az `127.0.0.1:5000`-en), így a viselkedés mindhárom felületen azonos.

## Telepítés

A telepítés két lépésből áll: először az Chloros asztali csomagot kell telepíteni (ez biztosítja a feldolgozási háttérrendszert és a hardveres futtatókörnyezetet), majd az Python csomagot.

**

1. lépés — Az Chloros telepítése.** Windows: futtassa az asztali telepítőt (alapértelmezett elérési út: `C:\Program Files\MAPIR\Chloros\`) a [Letöltés](download.md) oldalról. Linux: telepítse az `.deb` csomagot ([Linux telepítés](linux/linux-installation.md)).**

2. lépés — Telepítse az SDK-et** (Python 3.7+):

```bash
pip install chloros-sdk
```

Lehet, hogy nincs is szüksége a pip parancsra: minden telepítő tartalmaz egy megfelelő SDK wheel fájlt. Az Windows telepítő automatikusan telepíti azt a rendszer Python mappájába; az Linux `.deb` a `/usr/lib/chloros/sdk/` helyre helyezi, és kinyomtatja a pontos `pip install --user` parancsot. A PyPI a kiadási verziók megjelenésekor frissül, így az `pip install chloros-sdk` megegyezik a legújabb stabil kiadással.

**

3. lépés — Jelentkezzen be egyszer minden gépen:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

A hitelesítő adatok az `~/.chloros/` fájlban kerülnek tárolásra (mindkét platformon). Az Windows rendszeren a bejelentkezés az asztali alkalmazás „Felhasználó” (User) <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> fülén keresztül is elvégezhető. Az SDK használatához fizetős Chloros+ csomag szükséges — lásd az alábbi [Licenckövetelményeket](#license-requirement).

| Követelmény | Részletek |
| --- | --- |
| **Chloros telepítve** | Windows: asztali telepítő; Linux: `.deb` csomag (tartalmazza a háttérprogram bináris fájlját) |
| **Python** | 3.7 vagy újabb (3.10-es verzión fejlesztve/tesztelve) |
| **Operációs rendszer** | Windows 10/11 64 bites, Ubuntu 22.04 LTS vagy újabb, vagy NVIDIA Jetson (JetPack 6) |
| **Licenc** | Aktív Chloros+ bejelentkezés, bármely fizetős szint (Copper vagy magasabb) |

## 60 másodperc alatt kész

Egyetlen parancs létrehoz egy projektet, importál egy mappát, beállítja a feldolgozást, és futtatja a feldolgozási folyamatot — automatikusan elindítva a háttérprogramot, ha az még nem fut:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(Linux esetén használja az Linux útvonalakat: `/home/user/drone_images/flight001`. Az SDK mindkét platformon azonos módon működik.)

LATTICE rögzítési mappát dolgoz fel? Használja a LATTICE-kompatibilis wrappert — ez a megfelelő alapértelmezéseket alkalmazza (nincs panelcél-felismerés, standard debayer):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — teljes folyamatirányítás

Bármilyen, egy soros parancsnál bonyolultabb feladatra használja az `ChlorosLocal` parancsot. Ez az első használatkor elindítja a háttérprogramot (`auto_start_backend=True`), létrehozza és konfigurálja a projekteket, figyelemmel kíséri a folyamatot, és futás utáni összefoglalót ad vissza.

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
Maradjon az alapértelmezett `http://127.0.0.1:5000` használatánál, ne cserélje le `localhost`-re — az Windows esetén az `localhost` először az `::1`-re oldódik fel, és kérésenként körülbelül 2 másodpercet vesz igénybe a kizárólag IPv4-et támogató háttérrendszer esetében.
{% endhint %}

Használja kontextuskezelőként a garantált tisztítás érdekében:

```python
import chloros_sdk

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

Az `configure()` a következő kulcsszavakat fogadja el: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` és `custom_settings`. A legfontosabb értékek:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

A LATTICE-specifikus vezérlők (`input_level`, `radiometric_output`, valamint az `array_alignment*` család) teljes értéktáblázataikkal együtt a [SDK Referencia](reference/sdk-reference.md#supported-values) című dokumentumban találhatók a teljes értéktábláikkal együtt.

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

### A futás utáni összefoglaló olvasása – és az üres futások észlelése

A befejezés után az `process()` a háttérrendszer feldolgozási összefoglalóját `result["summary"]` néven csatolja. Az `summary["hints"]` minden egyes bejegyzése egy teljes mondat, amely elmagyarázza a figyelemre méltó eseményeket — például, miért nem eredményezett kimenetet egy futtatás — és minden utalást újra kiad Python `UserWarning` formájában, így az üres futtatások önmagukban is felismerhetők, még akkor is, ha soha nem vizsgálja meg a szótárat:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**Az `process()` nem lép fel, ha egy futtatás nem hoz létre képeket.** Ez az egyetlen pont, ahol az SDK és az CLI szándékosan eltér egymástól: Az `chloros-cli process` a „termékeket kérték, de egyet sem írtak” állapotot hibaként kezeli, és nem nulla kóddal lép ki, míg az SDK normál módon tér vissza, és az állapotot az `summary` / hints segítségével jelenti. Ha a folyamatnak üres futás esetén le kell állnia, ellenőrizze ezt saját maga — vizsgálja meg az `summary` fájlt (vagy számolja meg a projektmappa alatti fájlokat), ahelyett, hogy kivételre támaszkodna.
{% endhint %}

## Smart Connect — élő hardver

Három segédprogram nyit állandó munkameneteket a háttérrendszer hardverpooljában — ugyanabban a poolban, amelyet a grafikus felület is használ, így az SDK szkriptek a soros portokért vagy a hálózati sávszélességért való versengés nélkül létezhetnek együtt az asztali alkalmazással. Mindhárom automatikusan elindít egy helyi háttérrendszert, ha még nem fut ilyen.

### Egyetlen LATTICE kamera — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Szinkronizált kamerasor — `connect_array`

Az `connect_array` az ajánlott kiindulási pont a többkamerás rendszerekhez. Ugyanazt az intelligens előkészítési folyamatot futtatja, mint a grafikus felhasználói felület: hálózati elemzés, szinkronizációs szint automatikus kiválasztása, PTP időszinkronizálás, kameránkénti pixelformátum-kiválasztás, AE-beállítás és GPIO-trigger aktiválása. Az **első soros eszköz a master** (ez adja le a hardveres kioldó impulzust); a többi slave.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Adja hozzá az `smart=True`-et bármely tömbfelvételhez, hogy a kioldás előtt megvárja, amíg az automatikus expozíció minden kamerán beáll. A rögzítési módokról (Egyedi / Folyamatos / Intervallum / Leggyorsabb), a rögzítőkről, a burst-to-video funkcióról és a mátrix-igazításról lásd az [SDK Referencia](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep) című dokumentumot.

### DAQ fényérzékelő — `connect_daq_sensor`

Argumentumok nélkül az `connect_daq_sensor()` automatikusan felismeri az átviteli módot (elsőbbségi sorrend: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Minden keret tartalmazza a 135 pontos `spectrum` értéket (kalibrálás után W/m²/nm), egy `is_saturated` jelzőt, valamint a CIE `x`, `y`, `z` értékeket. Egy adott érzékelő vagy átviteli mód kijelöléséhez — ami megbízható választás olyan gazdagépeken, ahol több hálózati interfész is van, és az Ethernet automatikus felismerése az első kísérlet során elmulaszthat egy működőképes DAQ-E-t — adjon meg egy kifejezett utalást:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Ne feledje, hogy a felső határ-korrekciós profilok (`cap_id`) **nem** egy SDK vezérlőgombot jelentenek – ezeket inkább az `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap` segítségével válassza ki őket.

### Mentett projektek — `open_project`

Egy elmentett Chloros projekt megőrzi a hozzá csatlakoztatott hardvereket (`cameras.json` + `sensors.json` az `project.json` mellett), és az `chloros_sdk.open_project(path)` egyszerre tud mindent újra csatlakoztatni, valamint eszköznév alapján vezérli a rögzítéseket. Lásd a [Projekt automatizálás](reference/sdk-reference.md#project-automation--chlorosproject) című részt a referenciaanyagban.

## Mit kap egy kizárólag pip-tel történő telepítés

A hardverfelületek használata előtt ellenőrizze a modulszintű elérhetőségi jelzőket:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Olyan gazdagépen, amelyen **csak** az `pip install chloros-sdk` van, és nincs Chloros asztali csomag:

* Az `ChlorosLocal`, az `process_folder` és az `process_lattice_capture` **nem** működnek — ezekhez szükség van az asztali telepítőben található háttér bináris fájlra.
* A smart-connect segédprogramok (`connect_camera`, `connect_array`, `connect_daq_sensor`) tisztán HTTP kliensek, így egy másik gépen futó háttérprogrammal is működnek — azonban a mellékelt háttérprogramok csak a loopback-címre kötődnek, ezért a portot magának kell átirányítania (pl. `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`), és az `backend_url="http://127.0.0.1:5000"`-et az `auto_start_backend=False`-szel együtt kell átadnia. Lásd a [Távoli háttérrendszer mód](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel) című részt.
* A közvetlen hardveres LATTICE osztályok (`LatticeCamera`, `CameraPool`, …) importálhatók, de szükségük van az asztali csomagból származó Arena SDK futásidejű környezetre — ennek hiányában az `CAMERA_AVAILABLE` az `False`.
* Az `daq_sdk` (a közvetlen DAQ-osztályok) az asztali telepítővel együtt érkezik, nem a PyPI-csomaggal, így az `DAQ_AVAILABLE` egy kizárólag pip-et használó gépen az `False`-nek felel meg — a DAQ-érzékelőket inkább az `connect_daq_sensor()` segítségével vezérelje egy (alagúton keresztül elérhető) háttérrendszerhez.

## Licenckövetelmény

Az SDK hozzáféréshez aktív Chloros+ bejelentkezés szükséges bármely fizetős csomagban — **Copper vagy magasabb**(Copper / Bronze / Silver / Gold); az ingyenes Iron csomag nem biztosít SDK/CLI hozzáférést. Az érvényesítés**szerveroldali**: minden SDK kérésnek tartalmaznia kell egy aktív munkamenetet és egy fizetős csomagot is, ellenkező esetben a háttérrendszer `403` / `PLAN_UPGRADE_REQUIRED` hibakódot ad vissza kódot (amelyet az `ChlorosLocal` kód `ChlorosLicenseError` kódként, az `connect_*` segédprogramok pedig `ChlorosConnectError` kódként generálnak). A kijelentkezett hívó a következő hibakódokat kapja: `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) kódot kap – az `chloros-cli login` újrafuttatása az első esetet megoldja, a másodikat azonban nem.

Az offline használat a terv türelmi időszakán belül működik: a szintet a szerver-érvényesítési gyorsítótárból (5 perc) vagy az aláírt, géphez kötött licenc-gyorsítótárból olvassa be (havi tervek esetén 30 napig; éves előfizetések esetén az előfizetés lejáratáig). A türelmi időszak lejárta után a csomag ingyenesre vált, és az SDK hozzáférés leáll, amíg a gép egyszer kapcsolatba nem lép a szerverrel. Az `chloros-cli status` az ingyenes szinten továbbra is elérhető marad, így az ok mindig látható. Lásd [Chloros+ Bejelentkezés](chloros+-login.md).

## Kivételek

Fogja meg az alaposztályt, hogy kezelje a „bármi Chloros rosszul sült el” helyzeteket:

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

Minden pipeline-kivétel (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) az `ChlorosError`-ből származnak. Egy apró bökkenő: az `ChlorosConnectError` hibaüzenetet kizárólag az `connect_camera`, az `connect_array` és az `connect_daq_sensor` hibaüzenetek váltják ki — a sima `Exception`-ből származik, **nem** az `ChlorosError`-ből, így az `except ChlorosError` nem fogja elkapni. A teljes hierarchia az [SDK hivatkozásban](reference/sdk-reference.md#exceptions) található.

## Lásd még

* [SDK hivatkozás](reference/sdk-reference.md) — a teljes API felület, amely mesterséges intelligencia-asszisztensekhez lett optimalizálva.
* [CLI Referencia](reference/cli-reference.md) — minden CLI alparancs egy SDK hívást tükröz.
* [Letöltés](download.md) — telepítőprogramok az Windows és az Linux verziókhoz.
