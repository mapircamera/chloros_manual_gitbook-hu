# Linux-telepítés

A Chloros a Linux rendszerre `.deb` csomagok formájában kerül forgalomba, amelyek telepítik a CLI-t, a háttérszervert és a Python SDK-t (az `python3` rendszerhez, valamint egy verzióhoz illeszkedő wheel fájlt a virtualenvs-hez).

A csomagfájlok neve tartalmazza a verziót és az architektúrát: `chloros_1.3.0_amd64.deb` az x86_64-hez, az `chloros_1.3.0_arm64_jp6.deb` pedig az arm64-hez (NVIDIA Jetson, JetPack 6). Az alábbi parancsokban helyettesítse be a ténylegesen letöltött fájl nevét.

{% hint style="info" %}
**Újdonság az 1.3.0-ban: a programok kicsomagolt állapotban kerülnek szállításra.** A háttérprogram és a CLI mappaként kerülnek telepítésre, `/usr/lib/chloros/backend/` és `/usr/lib/chloros/cli/` néven, nem pedig önkicsomagoló fájlként. Már nem csomagolják ki magukat ideiglenes mappába minden indításkor, így sokkal gyorsabban indulnak el (x86_64-en a háttérprogram korábban minden indításkor körülbelül 100 másodpercet töltött a kicsomagolással) és már nincs szükség több GB szabad helyre az `/tmp` mappában. Az amd64 `.deb` mérete is körülbelül a felére csökkent.
{% endhint %}

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
| **Python** | Python 3.7 vagy újabb (az SDK-hez) | Python 3.10 vagy újabb |

> **Az Ubuntu 20.04 és a Debian 11 nem támogatott.** Az `.deb` függőségi listája
> abból származik, amire a Chloros háttérprogram ténylegesen hivatkozik, és ez magában foglal egy
> a focal és a bullseye kiadásokban szereplő glibc 2.31-nél újabb `libc6`-et is, ezért az `apt` egyenesen elutasítja a
> telepítést, ahelyett, hogy később, futásidőben hagyja meghiúsulni azt.

### Telepítés

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Használja az `apt install ./…`-et, ne az `dpkg -i`-et.** Az `apt` megoldja a csomag függőségeit, és telepíti az általa ajánlott elemeket is: a Python SDK követelményeit, az `python3-numpy`-et és az `python3-requests`-et. Az `dpkg -i` egyikét sem telepíti. A parancssori felület (CLI) és a háttérprogram önállóak, és ezek nélkül is futnak, de az `import chloros_sdk`-nek szüksége van rájuk; a telepítés kimenete felsorolja az esetleg hiányzó elemeket. Tartsa meg az `./`-et, hogy az `apt` telepítse a fájlt, ahelyett, hogy egy ilyen nevű csomagot keresne. A csomag az `libgl1`-et is deklarálja, amelyet a csomagban lévő OpenCV betölt, és amely egy minimális szerverképből hiányozhat.
{% endhint %}

Ellenőrizze a telepítést:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Rendszerkövetelmények

| Követelmény | Minimális | Ajánlott |
| --- | --- | --- |
| **Platform** | NVIDIA Jetson JetPack 6-tal | Jetson Orin NX 16 GB vagy AGX Orin |
| **JetPack** | JetPack 6.x | Legújabb JetPack 6 |
| **Memória (RAM)** | 8 GB (megosztott GPU/CPU) | 16 GB+ megosztott (12 GB+ a párhuzamos feldolgozás küszöbértéke) |
| **Tárolókapacitás** | 2 GB szabad hely | NVMe SSD 10 GB+ szabad hellyel |
| **Python** | Python 3.7+ (az SDK-hez) | Python 3.10+ |

### Telepítés

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Ugyanaz az elrendezés, mint az amd64 `.deb` esetében (Python SDK, vételi pufferek és háttérszolgáltatások is benne vannak), a **JetPack 6** (L4T R36) rendszeren futó Jetson Orin / Orin NX / Orin Nano készülékekhez. A PyTorch a JetPack 6 Orinra készült CUDA-verziója, így a GPU-feldolgozás – beleértve a Texture Aware debayert is – a Jetson GPU-ján fut. A csomag maga nem tartalmazza a CUDA-t: a JetPack 6 által telepített CUDA- és cuDNN-könyvtárakat használja (`sudo apt install nvidia-jetpack` olyan képen, amelyeken ezek hiányoznak). Ha nincs használható CUDA-GPU, a feldolgozás a CPU-n fut, és az `--debayer texture-aware` a Standard debayert használja (a háttérprogram egy `[DEBAYER] … needs a CUDA GPU …` sorban naplózza, hogy miért). A Jetson memóriájával, hőkezelésével és terepi telepítésével kapcsolatos viselkedésről lásd az [NVIDIA Jetson útmutatót](nvidia-jetson-guide.md).***

## Python SDK telepítése (minden Linux-verzió)**Az `.deb` automatikusan telepíti az SDK-t.** Mindkét csomag (amd64 és Jetson) a rendszerre telepíti az `python3`-et, így az `import chloros_sdk` a telepítés után azonnal működik, pip és hálózat nélkül. A csomagjai (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` és a projektbetöltők) az `/usr/lib/python3/dist-packages`-be kerülnek, amely a csomag tulajdonában van: egy frissítés felülírja őket, a csomag eltávolítása pedig eltávolítja őket. A verzió mindig megegyezik a telepített CLI-vel és háttérprogrammal.

Az SDK saját követelményei, az `python3-numpy` és az `python3-requests`, **ajánlottak, nem kötelezőek** a csomag által: az `sudo apt install ./chloros_<version>_<arch>.deb` telepíti őket, az `dpkg -i` viszont nem. A telepítési kimenet jelenti az `python3` által importált SDK-verziót, az esetleges hiányzó követelményeket, valamint az `chloros_sdk` elérési útjának más pontján található, a csomag példányát felülíró elemeket (például egy korábban az `pip install --user`-szel telepítettet).**Egy virtualenv-ben**, amely nem látja a rendszer csomagjait, telepítse bele a csomagban lévő wheel fájlt (teljesen offline módban):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**A PyPI-ről** — kizárólag olyan gépre, amelyre nincs telepítve a Chloros csomag, és amelynek SDK-je egy másik gépen futó háttérprogramot vezérel (lásd [Távoli háttérprogram mód](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Opcionális kiegészítők pip telepítéshez (virtuális környezetben vagy a PyPI-ről):

| Kiegészítő | Parancs | Hozzáad |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` az élő haladás-streameléshez |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` BLE (DAQ-M) adatátvitelhez |

Ellenőrizze az SDK-t:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
A Python SDK kommunikál a háttérrel, amelyet az `.deb` telepít egy helyi HTTP API-n (`http://127.0.0.1:5000`) keresztül. Az `ChlorosLocal` és a csatlakozási segédprogramok szükség esetén automatikusan elindítják azt; egy mentett projekt `connect_all()`-je soha nem indítja el, ezért egy headless gépen futtassa a háttérprogramot [szolgáltatásként](#always-on-ptp-for-headless-hosts). Mindig a konkrét IPv4-címet használja az `localhost` helyett — az `localhost` az `::1`-re oldódhat fel, és kérésenként körülbelül két másodpercbe kerül.
{% endhint %}

***

## Első beállítás

### 1. Bejelentkezés

A CLI-hez és az SDK-hez való hozzáféréshez fizetős Chloros+ szint (**Copper** vagy magasabb) szükséges, amelyet szerveroldalon érvényesítenek: a kijelentkezett hívó az `401 AUTH_REQUIRED` címet kapja, míg az ingyenes szintű (Iron) hívó az `403 PLAN_UPGRADE_REQUIRED` címet kapja.

```bash
chloros-cli login your@email.com 'your-password'
```

A hitelesítő adatok az `~/.chloros/user_session.json` fájlban kerülnek tárolásra.

{% hint style="info" %}
**A frissítések megőrzik a bejelentkezési adatait; az eltávolítás törli azokat.** Ha egy újabb `.deb` verziót telepít egy meglévő telepítésre (beleértve az `chloros-cli update`-en keresztül történő telepítést is), az `~/.chloros/user_session.json` és a gyorsítótárban tárolt licenc megmarad, így a headless CLI/SDK feladatok továbbra is működnek. A csomag (`sudo apt remove chloros`) eltávolítása törli a gépen minden felhasználó számára a mentett bejelentkezési adatokat és a licenc-gyorsítótárat, ezért egy új telepítéshez ismét szükség van az `chloros-cli login`-re.
{% endhint %}

### 2. Ellenőrizze a licencállapotát

```bash
chloros-cli status
```

Az `chloros-cli status` bármely szinten (beleértve az ingyeneset is) működik, így mindig láthatja, miért érhető el vagy nem érhető el a hozzáférés.

### 3. Futtassa a rendszerdiagnosztikát

```bash
chloros-cli selftest
```

Hét ellenőrzés fut sorban, és a parancs nem nulla kóddal lép ki, ha bármelyikük sikertelen:

| # | Ellenőrzés | Mit igazol |
| --- | --- | --- |
| 1 | **Verzió** | A CLI jelenti a verzióját (`v1.3.0`). |
| 2 | **Port elérhető** | Az 5000-es port szabad, *vagy* már válaszolt rá egy működőképes Chloros-háttérprogram (ez sikernek számít). |
| 3 | **Háttérprogram indítása** | A háttérprogram bináris fájlja elindul. |
| 4 | **API-teszt (`/api/test`)** | A háttérprogram `status: ok`-et válaszol. |
| 5 | **Rendszerinformáció** | Kinyomtatja az `GPU: <name>, CUDA: <bool>, PyTorch: <version>` értéket az `/api/system-info`-ből. |
| 6 | **Zajszűrő modellek** | Megtalálja az `*.pth.enc` modelleket (Linux rendszeren: `/usr/lib/chloros/models`). |
| 7 | **CUDA + zajszűrő**| A Texture Aware valóban használható — CUDA-ra**és** legalább egy modellfájlra van szükség. |

A futtatás az `N/7 checks passed`-szel zárul, felsorolva az esetleges hibákat név szerint.

### 4. Az első adatkészlet feldolgozása

```bash
chloros-cli process ~/datasets/flight001
```

***

## Fájlok és könyvtárak

### A felhasználónkénti

Chloros a hitelesítő adatait és a CLI-konfigurációját egyetlen, platformfüggetlen könyvtárban, az **`~/.chloros/`**-ben (Windows rendszeren: `%USERPROFILE%\.chloros\`). Két Linux-specifikus gyorsítótár viszont az XDG-konvenciókat követi — ezek az `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` neveket használják, ha be vannak állítva.

| Útvonal | Cél |
| --- | --- |
| `~/.chloros/user_session.json` | Az `chloros-cli login` által írt bejelentkezési munkamenet-gyorsítótár (a csomagfrissítések során megmarad; a csomag eltávolításakor törlődik) |
| `~/.chloros/working_directory.txt` | Alapértelmezett projektmappa felülírása (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | A CLI-nyelvi beállítás (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | A nyelvi beállítás megosztása a Windows-felülettel — az itt megadott `language` beállítás elsőbbséget élvez az `cli_language.json` beállítással szemben |
| `~/.chloros/update_cache.json` | Egyórás gyorsítótár a Linux/Jetson indításkori frissítés-ellenőrzéshez |
| `~/.chloros/backend.log` | Háttérnapló, amikor a háttérprogramot a CLI indította el |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Kameránkénti LATTICE kalibrációs csomagok gyorsítótárban, sorozatszám és csomag-hash alapján indexelve |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Opcionális felhasználói felülírások a DAQ cap-korrekciós profilokhoz |
| `~/.config/chloros/system_config.json` | A Dynamic Compute Adaptation által tárolt hardverprofil – törölje, ha új hardverfelismerést szeretne kényszeríteni |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Háttérszerver-naplók, indításonként egy fájl |
| `~/Documents/MAPIR/Chloros Projects/` | Alapértelmezett projektmappa, ha nincs felülírás beállítva |

### Rendszer-szintű

| Elérési út | Cél |
| --- | --- |
| `/usr/bin/chloros-cli` | Wrapper szkript — beállítja az `LD_LIBRARY_PATH` értéket a csomagban lévő natív könyvtárakhoz (és az `TMPDIR` értéket `/mnt/ssd/tmp`-re, ha az a mappa létezik), majd futtatja a valódi bináris fájlt |
| `/usr/bin/chloros-backend` | Wrapper szkript — ugyanez vonatkozik a háttérprogramra is, amelyet telepített termelési verzióként jelöl meg |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | A lefordított CLI és háttérprogram, kicsomagolt állapotban telepítve (újdonság az 1.3.0-ban); a programok nevei: `cli/chloros-cli` és `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Két soros indító szkriptek, amelyek elindítják a fenti valódi programokat — ezeken az elérési útvonalakon vannak tárolva, mert az SDK, a CLI és a régebbi szkriptek itt keresik őket |
| `/usr/lib/chloros/arena_runtime/` | A LATTICE kamerákhoz szükséges Arena SDK-futásidejű környezet |
| `/usr/lib/chloros/models/*.pth.enc` | A Texture Aware debayer által használt titkosított zajszűrő modellek |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (és `lattice_sdk`, `lattice_cam`, `arena_api`, a projektbetöltők) | A Python SDK, amely az `python3` rendszerhez van telepítve, és a csomaghoz tartozik |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | A virtuális környezethez tartozó, pontosan ehhez a verzióhoz illeszkedő Python SDK wheel |
| `/usr/lib/chloros/exiftool` | A csomagban található ExifTool indító (csak akkor van szimbolikus linkkel összekötve az `/usr/local/bin/exiftool`-hez, ha nincs rendszerbeli exiftool). Az 1.3.0-s verziótól kezdve az ExifTool szkript saját Perl-könyvtárral érkezik az `/usr/lib/chloros/exiftool-dist/`-ben, így a rendszer `libimage-exiftool-perl` csomagja nélkül is fut |
| `/etc/chloros/update.conf` | Az `chloros-cli update` által olvasott frissítési csatorna konfiguráció |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Beállítja az `net.ipv4.ip_unprivileged_port_start = 319`-et, hogy a háttérprogram root jogosultság nélkül tudjon kapcsolódni a PTP-portokhoz |
| `/etc/sysctl.d/60-chloros-gige.conf` | A LATTICE kamerák adatfolyamaihoz 32 MB-ra növeli a kernel socket-fogadó puffereit (`net.core.rmem_max`, `net.core.rmem_default`) 32 MB-ra a LATTICE kamerák adatfolyamaihoz, és ezeket a telepítéskor alkalmazza (a már magasabbra beállított értéket nem módosítja). A kernel alapértelmezett beállításainál a kamera-csomagok elvésznek a kernelben, és a rendszer képkockákat és képkockasebességet veszít; az `chloros-cli lattice network` jelzi, ha a pufferek szintje alacsony. Lásd [Linux-gépek — socket-fogadó pufferek](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | A dinamikus betöltőt az `/usr/lib/chloros/arena_runtime`-re irányítja |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Hozzáférést biztosít a bejelentkezett felhasználónak a DAQ-U USB soros hídhoz (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Beállítható, állandóan futó háttérszolgáltatás, amely a háttérprogramot az Ön által megadott felhasználóként futtatja (telepítve, **nem engedélyezve**) — ezt kell engedélyezni, lásd [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Ugyanaz a szolgáltatás, amely a háttérszolgáltatást **root**ként futtatja (telepítve,**nem engedélyezve**), azért tartják meg, hogy az a gép, amelyen engedélyezték, továbbra is működjön |
| `/var/log/chloros/` | A root szolgáltatás részletes naplója, amelyet az `adm` csoport olvashat (a csomag eltávolításakor törlődik) |
| `/usr/share/applications/chloros-cli.desktop` | „Chloros CLI” alkalmazásmenü-bejegyzés, amely megnyit egy terminált |

**Tűzfal (új a 1.3.0-ban).** Ha a telepítéskor az `ufw` vagy az `firewalld` aktív, a csomag megnyitja az `5002` (DAQ-E adatok), az `5353` (mDNS-felfedezés), az `319` és az `320` (PTP) portokat, és a csomag eltávolításakor eltávolítja ezeket a szabályokat. Ha tűzfal van telepítve, de nem aktív, a csomag nem ad hozzá semmit, és kinyomtatja az engedélyezendő portokat arra az esetre, ha később engedélyezné a tűzfalat. Nem nyit meg TCP-portot: a DAQ-E firmware-frissítés, amelynek során az érzékelő TCP-n keresztül csatlakozik vissza ehhez a számítógéphez a kép begyűjtése érdekében, `… device never connected back on TCP <port> …` hibával végződik, ha egy aktív tűzfal blokkolja azt a kapcsolatot.

## A háttérprogram helye

A CLI és az SDK automatikusan felismeri a háttérprogramot:

| Komponens | Útvonal |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Háttér | `/usr/lib/chloros/chloros-backend` (az `/usr/lib/chloros/backend/chloros-backend` indítója) |

A háttérprogram elérési útját az `--backend-exe` CLI-flaggel vagy az `backend_exe` SDK-konstruktorparaméterrel, a portot pedig az `--port` parancssor-opcióval (alapértelmezés: `5000`).

{% hint style="info" %}
Az `CHLOROS_BACKEND_URL` a **`lattice`**,**`project`**és**`daq pool-*`** parancscsaládokra egy távoli háttérrendszerben. Az alapvető parancsok (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) szándékosan figyelmen kívül hagyják ezt, és mindig az `http://127.0.0.1:<port>`-et célozzák meg.
{% endhint %}

***

## LATTICE kamerák és DAQ fényérzékelők Linux rendszeren

A live-hardware parancscsaládok mindegyike működik Linux rendszeren (amd64 és Jetson):

* **`chloros-cli lattice`** — LATTICE kamerák és szinkronizált tömbök felismerése, csatlakoztatása, konfigurálása és felvétele. Az `.deb` csomagja tartalmazza a szükséges Arena SDK futásidejű környezetet, és regisztrálja azt a dinamikus betöltőnél.
* **`chloros-cli daq pool-*`** — DAQ-U/M/E fényérzékelők csatlakoztatása a háttérpoolon keresztül, kalibrált spektrumok streamelése, `.daq` fájlok rögzítése és cap-profilok cseréje, valamint DAQ-E esetén a hálózati beállítások módosítása, újraindítása és a firmware frissítése. Az `chloros-cli daq discover` megkeresi a csatlakoztatható érzékelőket, az `chloros-cli daq --help` pedig felsorolja az `pool-*` alparancsokat.
* **`chloros-cli project`** — egy mentett projekt (kamerái, érzékelői és feldolgozási beállításai) vezérlése headless módban.
* **`chloros-cli time-sync`** — ellenőrzi a PTP grandmastert, amelyet a Chloros háttérprogram futtat a LATTICE kamerák és a DAQ-E érzékelők számára.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

Az `--sensor-id` parancs az `pool-latest`, az `pool-stream`, az `pool-record` és az `pool-set-cap` számára; az `pool-list` a poolban jelenleg található azonosítókat jeleníti meg.

{% hint style="info" %}
**Az `--eth-host` kihagyja a felderítést.** Az `pool-connect --eth` mDNS segítségével talál egy DAQ-E-t, olyan ARP-visszaeséssel, amely egy üres ARP-gyorsítótárból is működik; az érzékelő IP-címének vagy gazdagépnevének az `--eth-host`-szel történő átadásával közvetlenül csatlakozik hozzá.
{% endhint %}

**A DAQ-U soros engedélyeket** a telepített udev-szabály kezeli (`uaccess` + csoport `dialout`). Ha egy már csatlakoztatott érzékelő továbbra sem érhető el, töltse be újra a szabályokat, vagy csatlakoztassa újra:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

A teljes parancssorozatot lásd a [CLI-referenciában](../CLI.md).

### Folyamatosan aktív PTP fej nélküli gazdagépekhez

A csomag két systemd szolgáltatást telepít a háttérrendszerhez, amelyek egyike sem engedélyezett. Olyan kijelző nélküli Jetson-on vagy szerveren, ahol a DAQ-E érzékelők és a LATTICE kamerák számára folyamatosan futnia kell a PTP időszinkronizálásnak, vagy ahol az SDK-parancsfájlok csatlakoznak a mentett projektekhez (az `connect_all()` soha nem indítja el a háttérrendszert), engedélyezze a szolgáltatást **a saját felhasználói fiókjával**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

Az `chloros-backend@<user>.service` az adott felhasználóként futtatja a háttérrendszert: az általa írt rögzítések az ő tulajdonát képezik, a részletes naplója az ő `~/.cache/chloros/logs` fájljában található, és az ő `chloros-cli login` (`~/.chloros`) fájlt használja, ezért először jelentkezzen be egyszer azzal a felhasználóval. A csomag telepítésekor a rendszer kinyomtatja ezt a parancsot, a felhasználónevével kitöltve.

Szolgáltatás nélkül a háttérprogram (és vele együtt a PTP grandmaster) csak akkor fut, ha valami elindítja – egy `chloros-cli` parancs vagy egy SDK-parancsfájl.

Az egység a háttérprogramot az `127.0.0.1:5000` (az egység belsejében található `CHLOROS_HOST` / `CHLOROS_PORT` környezeti beállítások; felülírható az `sudo systemctl edit chloros-backend@.service` paranccsal), és hiba esetén 5 másodperc múlva újraindítja.

**A root szolgáltatás.**Az `chloros-backend.service` a háttérprogramot**root**ként futtatja. Továbbra is telepítve van, így az a gép, amelyen engedélyezték, továbbra is működik, de minden, amit a root háttérprogram ír, a root tulajdonát képezi, így a felhasználó nem tudja törölni a saját rögzítéseit; a naplója az `/var/log/chloros` fájlban található, amelyet az `adm` csoport olvashat. A saját felhasználói fiókra való átváltáshoz:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**A szolgáltatás háttérprogramja megosztott, az ügyfél soha nem állíthatja le.** Az `chloros-cli` és az SDK szkriptek úgy használják, ahogyan megtalálják: az `process` ezen fut, és a legutóbbi helyett a saját projektjét tölti be. Ahol a CLI-nek egyébként újra kellene indítania a háttérrendszert (`--restart`, vagy egy `process`, amelynek importja nem talált semmit), megnevezi az egységet, kinyomtatja az `sudo systemctl restart <unit>` parancsot, majd kilép az `1`-ből, mert egy ilyen módon leállított háttérprogram leállt állapotban maradna. Indítsa újra saját maga az `sudo systemctl restart chloros-backend@$USER.service` parancs segítségével. Egy szolgáltatás háttérfolyamata soha nem áll le magától tétlenség esetén, és az `chloros-cli backend stop` elutasítja ezt, megnevezve az `systemctl stop` parancsot.**Hogyan szerzi meg a PTP a portjait?** A PTP az UDP 319/320 portokat használja, amelyek mindkettő a szokásos 1024-es kiváltságos portok alsó határánál alacsonyabbak. A csomag `postinst` parancsa az `/etc/sysctl.d/60-chloros-ptp.conf` fájlt írja az `net.ipv4.ip_unprivileged_port_start = 319` parancs segítségével, ami lehetővé teszi a háttérprogram számára, hogy az Ön felhasználójaként futva hozzájuk kössön. Az 1.3.0-s verziótól kezdve a csomag nem állít be fájlkapacitásokat (`setcap`) a háttérprogramon, és az `libcap2-bin` már nem függőség: a kicsomagolt háttérprogramon egy fájlkapacitás megakadályozná az indítását, ezért ne adjon hozzá ilyet saját maga.***

## Bash szkriptelési példák

{% hint style="info" %}
**Szkriptelésre alkalmas kilépési kódok.**Az `chloros-cli process` sikeres futás esetén az `0` kóddal lép ki,**hibás futás esetén pedig nullától eltérő értékkel — ideértve azokat a futtatásokat is, amelyek képtermékeket kértek, de egyet sem írtak ki** (kiírja az `Processing finished but wrote no image products.` kódot, megnevezi a projektmappát és a szokásos okokat). A sikeres futtatások jelentik, hogy hány képtermék került írásra (`Image products written: N`, valamint `(plus M clamp masks)`, ha maszkok is íródtak). Kilépési kódok: `0` siker, `1` hiba, `2` argumentumhiba, `130` megszakítás.
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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Az érvényes `--format` értékek száma pontosan négy, és szóközöket tartalmaznak — mindig idézőjelek közé kell tenni őket:

| `--format` érték | Kimeneti mappa |
| --- | --- |
| `TIFF (16-bit)` *(alapértelmezett)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

Az `--debayer` elfogadja az `standard` (alapértelmezett) vagy az `texture-aware` (Chloros+) értéket.

### Automatizált feldolgozás Cron segítségével

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Példa a Python-SDK-re

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Hibaelhárítás

### A telepítés után nem található a CLI

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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### A háttérprogram nem indul el a fájljogosultságok hozzáadása után

Ne futtassa az `setcap` parancsot a háttérprogramban. Az 1.3.0-s verziótól kezdve a csomag nem állít be fájlképességeket, és a PTP 319/320-as portjai a [Hogyan kapja meg a PTP a portjait](#always-on-ptp-for-headless-hosts) című részben leírt `sysctl` beállításból származnak. A fájlkapacitás a kicsomagolt háttérprogramot biztonságos végrehajtási módba helyezi, amelyben nem találja meg saját könyvtárait, így nem indul el. Ha hozzáadott ilyet, távolítsa el:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### A PTP nem indul el / nem tud csatlakozni a 319-es porthoz

Ellenőrizze, hogy a nem privilegizált portok alsó határértékét csökkentették-e, és ha nem, akkor alkalmazza újra az aktuális indításra:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Ezután ellenőrizze a grandmastert:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### `lattice` Parancsok hibaüzenettel buknak: „LATTICE kamera illesztőprogramok nem találhatók”

A háttérprogram nem tudta betölteni a vele együtt telepített LATTICE kamera futásidejű környezetet. Minden `lattice` parancs a háttérprogramban fut; maga a CLI nem tartalmaz kamera-illesztőprogramokat. Telepítse újra a Chloros-t (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): az `.deb` a futásidejű környezetet a háttérprogram mellé telepíti az `/usr/lib/chloros/arena_runtime` mappába, és regisztrálja azt a dinamikus betöltőben. A regisztráció ellenőrzéséhez:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
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

A sikertelen indításhoz tartozó háttérszolgáltatási naplófájlok az `~/.cache/chloros/logs/` mappában találhatók. Ha az 5000-es port egy háttérszolgáltatáshoz tartozik (`systemctl status chloros-backend@$USER.service`), akkor a folyamat leállítása helyett indítsa újra a szolgáltatást.

### CUDA nem észlelve

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

Az `chloros-cli selftest` ugyanezt jelenti egy sorban: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Hiányzó megosztott könyvtárak

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Lassú indítás SD-kártyás rendszereken

Az 1.3.0-s verziótól kezdve a háttérszolgáltatás és a CLI-program kicsomagolatlanul kerül telepítésre, így már nem csomagolják ki magukat egy ideiglenes könyvtárba minden indításkor. Az ideiglenes fájlok automatikusan az `/mnt/ssd/tmp` mappába kerülnek, ha az létezik; ellenkező esetben egy gyors fájlrendszerre irányíthatja őket:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## A Chloros frissítése Linux rendszeren

Az `update` parancs kizárólag Linux/Jetson rendszereken használható. Ellenőrzi az `/etc/chloros/update.conf` alatt konfigurált frissítési csatornában közzétett verziót, és felajánlja a megfelelő `.deb` letöltését és telepítését:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

Linux/Jetson rendszeren a CLI minden indításkor elvégzi a frissítések blokkolásmentes ellenőrzését is (az eredményt egy órán át tárolja az `~/.chloros/update_cache.json`), és kinyomtatja az `Update available: vX.Y.Z` üzenetet, ha újabb verzió áll rendelkezésre. A beállítások, projektek és bejelentkezési adatok a frissítés után is megmaradnak.

A csomag frissítése a telepítés során minden háttérszolgáltatást leállít, így egy futó szkript elveszíti a kameráit és érzékelőit, majd újraindítja az összes engedélyezett szolgáltatást (azt, amelyet kézzel indítottak el, de soha nem engedélyeztek, leállt állapotban marad, akárcsak újraindítás után).

## Eltávolítás

```bash
sudo apt remove chloros
```

Az eltávolítás leállítja és letiltja az összes háttérszolgáltatást, visszaállítja az alapértelmezett, nem privilegizált port alsó határértékét (1024), eltávolítja a receive-buffer beállítási fájlt (a futó értékek az újraindításig megmaradnak), a csomagban található exiftool szimbolikus linket, az Arena loader konfigurációját, a csomag által hozzáadott tűzfalszabályokat és a telepített Python SDK-t, valamint törli a gyorsítótárban tárolt hitelesítő adatokat. A projektjei és az `~/.chloros/` adatfájlok érintetlenek maradnak. Az `sudo apt purge chloros` az `/var/log/chloros` fájlt is törli.

***

## Következő lépések

* [NVIDIA Jetson útmutató](nvidia-jetson-guide.md) — Jetson-specifikus optimalizálás és telepítés
* [CLI: Parancssor](../CLI.md) — a CLI útmutató
* [API: Python SDK](../api-python-sdk.md) — az SDK útmutató
* [CLI-referencia](../reference/cli-reference.md) és [SDK-referencia](../reference/sdk-reference.md) — a 1.3.0-s verzió teljes parancs- és API-listája
* [Dinamikus számítási alkalmazkodás](../processing-architecture/dynamic-compute-adaptation.md) — hogyan alkalmazkodik aChlorosa a hardveréhez
