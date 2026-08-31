# CLI : Parancssor

> **Teljes referencia:**[CLI Referencia](reference/cli-reference.md)**minden alparancs minden jelzőjét** dokumentálja, és AI-asszisztensekhez van optimalizálva — illessze be az URL kódot az asszisztensébe, és kérjen egy működő parancsot: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Tipp az AI-eszközökhöz:** a kézikönyv bármely oldala nyers Markdown formátumban is elérhető, ha az URL (pl. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`) kiegészítésével elérhető nyers Markdown formátumban, az `https://mapir.gitbook.io/chloros/llms.txt` pedig az egész kézikönyvet indexeli az LLM-ek számára.

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->
## Mi az az CLI?

Az `chloros-cli` ugyanazon feldolgozó motor parancssori felülete, amelyet az Chloros asztali alkalmazás is használ. Ez egy vékony HTTP kliens az Chloros háttérrendszer felett (egy helyi szerver az `127.0.0.1:5000`-en) — a legtöbb parancs automatikusan elindítja a háttérrendszert, így egy szkriptnek csupán egyetlen `chloros-cli process …` hívásra van szüksége.

**Windows 10/11 (x64)**és**Linux (x86_64, valamint NVIDIA Jetson arm64 a JetPack 6-on)** rendszereken fut, bármely terminálon, grafikus felület nélkül. Ellenőrizze a telepítést a következővel:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

A parancscsaládok áttekintése:

* **Feldolgozás és fiók** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 nyelv — lásd [Támogatott nyelvek](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (Linux/csak Jetson)
* **Élő hardver** — `lattice` (LATTICE kamera vezérlés, 45+ alparancs), `daq pool-*` (DAQ fényérzékelők), `time-sync` (PTP)
* **Automatizálás** — `project` (mentett Chloros projekt vezérlése headless módban, beleértve a YAML rögzítési receptjeit is)

Érdemes tudni a globális beállításokat: `--port N` (háttérport, alapértelmezett: `5000`), `-v/--verbose`, `--restart` (a háttér erőltetett újraindítása), `--backend-exe PATH`. A teljes listát lásd az [CLI-referenciában](reference/cli-reference.md).

***

## Telepítés

Az CLI **minden platformon az Chloros telepítőben található** – nincs külön letölthető CLI. A telepítőt a [Letöltés](download.md) oldalról töltheti le.

### Windows

A telepítő az CLI fájlt a következő helyre menti:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

mappába, és hozzáadja ezt a mappát a rendszerhez: `PATH` — **nyisson meg egy új terminált**a telepítés után, hogy a rendszer felismerje a frissített `PATH` fájlt. A telepítő emellett elhelyezi a futtató szkripteket (`Chloros_CLI.bat` / `Chloros_CLI.ps1`), valamint egy**Chloros CLI** Start menü parancsikont is, amelyek mindegyike megnyit egy terminált, ahol az `chloros-cli` már használatra kész állapotban van.

### Linux

Telepítse az architektúrájához megfelelő `.deb` fájlt:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Ezzel az `chloros-cli` fájlt telepíti az `/usr/bin/chloros-cli` verzióig (már `PATH`-en van), valamint a háttérprogramot `/usr/lib/chloros/chloros-backend`-re, a LATTICE kamerákhoz szükséges Arena SDK futtatókörnyezettel együtt. A részleteket lásd az [Linux telepítés](linux/linux-installation.md) című részben.

### Ellenőrzés

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Bejelentkezés és licencelés

Az CLI (és az Python, valamint az SDK) eléréséhez **fizetős Chloros+ csomag**szükséges — bármely fizetős csomag tartalmazza; az ingyenes csomag nem. A korlátozást**szerveroldalon** a háttérrendszer érvényesíti, nem az CLI bináris fájl: a kijelentkezett felhasználó hívását `401 AUTH_REQUIRED` hibakóddal utasítja el a rendszer, míg az ingyenes csomagban bejelentkezett felhasználó hívását `403 PLAN_UPGRADE_REQUIRED` hibakóddal, függetlenül attól, hogy az `chloros-cli`-ből, az SDK-ről, vagy egy saját fejlesztésű HTTP kliensről érkezik. Frissítsen a [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) oldalon.

Jelentkezzen be **gépenként egyszer**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->
{% hint style="warning" %}
**Különleges karaktereket tartalmazó jelszavak**(az `$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` jelszót a parancssor eltorzítja; az CLI ezt 401-es hibakóddal jelzi és automatikusan újrapróbálja, de az egyszerű idézőjelek használatával a probléma teljesen elkerülhető).
{% endhint %}

A munkamenet az `~/.chloros/user_session.json`-ben kerül tárolásra, és a csomag türelmi ideje alatt (havi csomagok esetén 30 napig, éves csomagok esetén a lejáratig) offline módban is működik. Az `chloros-cli status` fizetett terv nélkül is működik, így az elutasítás oka mindig látható.

{% hint style="danger" %}
**Fej nélküli feladatot szeretne ütemezni? Először jelentkezzen be.**A háttérfolyamatot indító parancsok (`process`, `status`, `export-status`, …)**gyorsan nem**bukik meg — interaktív `Email:` / `Password:` parancssorba vált át a stdin-en. Egy felügyelet nélküli cron-feladat vagy CI-lépés ezért**a bemenet várakozása közben lefagy**. Mielőtt bármit is ütemeznél, futtasd le egyszer az `chloros-cli login EMAIL 'PASSWORD'` parancsot a gépen.
{% endhint %}

***

## Az első feldolgozási futtatás

Irányítsa az `process` parancsot a rögzítések mappájára — a program automatikusan felismeri az Survey3 (`.raw` + `.jpg`), a LATTICE-t (`.tif`/`.tiff`), az `.dng`-et, vagy ezek kombinációját:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

A folyamatok állapota csővezeték-szálonként valós időben követhető (felismerés, elemzés, feldolgozás, exportálás), és a sikeres futás befejeztével a rendszer jelentést készít arról, hogy hány képterméket írt ki (`Image products written: N`).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### A kimenetek helye

Az `process` egy **projektmappába** ír, nem a bemeneti mappába:

* `-o` hiányában: a projekt az alapértelmezett projektmappában jön létre (amelyet a grafikus felülettel oszt meg; kezelése az `get-project-folder` / `set-project-folder` parancsokkal, tartalék megoldásként az `~/Chloros Projects` parancs), amelynek neve az `-n/--project-name`, vagy ha elhagyják, akkor egy időbélyeg (`YYYYMMDD_HHMMSS`).
* Az `-o PATH` használatával: ez a mappa **kifejezetten** a projektmappa. Ha már tartalmaz egy `project.json` fájlt, akkor a felülírás helyett egy `_1`/`_2`… utótaggal ellátott testvérfájl jön létre.

A projekten belül a termékek **kameránként, majd fájlformátum szerint** vannak csoportosítva:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

A kamera mappa neve LATTICE esetén `LATT-<sensor>-<lens>-F<filter>` (megfelel a felvétel EXIF-adatainak: `Model`), az `<model>_<filter>` (pl. `Survey3N_RGN`) az Survey3 esetében. A formátummappa az `--format`-et követi: `tiff16`, `tiff8`, `png8`, `jpg8`, vagy `tiff32` az `TIFF (32-bit, Percent)` esetében.

{% hint style="info" %}
**Minden exportált termék megtartja a SOURCE fájl nevét.**Az `capture_..._raw.tif` radiance-exportja továbbra is `capture_..._raw.tif` néven szerepel — csupán az `tiff32/Radiance_Images/` mappában található.**A mappanév azonosítja a terméket, nem a fájlnév**, ezért a könyvtárat keresse globális kifejezéssel, ne pedig az `*radiance*` kiterjesztést.
{% endhint %}

### A ténylegesen használni fogott opciók

| Opció | Alapértelmezett | Mit csinál |
| --- | --- | --- |
| `-o, --output PATH` | alapértelmezett projektmappa | A projektmappa helye (lásd fent). |
| `-n, --project-name NAME` | időbélyeg | Projektnév. |
| `--format FMT` | `TIFF (16-bit)` | Az `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` közül az egyik. |
| `--indices NAME [NAME ...]` | nincs | Exportálandó vegetációs indexek (lásd [Vegetációs indexek](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neurális debayer, lassabb, legmagasabb minőség (Chloros+, NVIDIA GPU). |
| `--vignette / --no-vignette` | be | Vignett-korrekció. |
| `--reflectance / --no-reflectance` | be | Reflektancia-kalibrálás; a LATTICE esetében ez egyben a reflektancia-termék kapcsolója is. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | A feldolgozási folyamat belépési pontjának kényszerítése LATTICE TIFF-fájlok esetén. |

Minden egyéb beállítás – célpont-felismerés hangolása, PPK, expozíciós pontok, tömb-igazítási jelzők – tekintse meg az [`process` szakaszt az CLI Referenciában](reference/cli-reference.md) című részét.

***

## Az exportálandó elemek kiválasztása (LATTICE termékek)

A LATTICE-feldolgozás **egy lépésben**minden alkalmazható termékre kiterjed. A termékenkénti négy kapcsoló**alapértelmezés szerint mind be van kapcsolva**; az `--no-` űrlapot használja, ha egyet ki szeretne kapcsolni:

| Kapcsoló | Termék |
| --- | --- |
| `--debayered` | Lineáris demosaic → `Debayered_Images/` |
| `--preview` | Előnézet megjelenítése (fehér egyensúly + gamma; hamis színű kiterjesztés multispektrális képekhez) → `Preview_Images/` |
| `--radiance` | float32 sugárzás, W/m²/sr/nm → `Radiance_Images/` (mindig `tiff32/`) |
| `--reflectance` | uint16 fényvisszaverődés, Pix4D-kompatibilis → `Reflectance_Calibrated_Images/` |

RGB A master kamerák kizárólag debayered + előnézeti értékeket adnak ki — a sávonkénti sugárzás/reflektancia szélessávú érzékelő esetében nincs értelme, ezért ezek a kapcsolók számukra hatástalanok. Survey3 `.raw` figyelmen kívül hagyja a kapcsolókat, és a szabványos reflektancia/célútvonalat követi.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (alapértelmezés: `auto`) a reflexió-referenciát választja: az `auto` létrehoz egy minőségbiztosítási követelményeknek megfelelő, a képkockán belüli [kalibrációs célt](calibration-targets.md) az abszolút referencia, és cél hiányában a DAQ fényérzékelő lefelé irányuló megosztására (ρ = π·L/E) tér vissza; az `target` szigorú (nincs DAQ-helyettesítés); az `daq` a DAQ-ra támaszkodik. Az egységre vonatkozóan mért célpontok szkennelései az `--target-reflectance-dir` segítségével adhatók meg.

{% hint style="info" %}
**Reflektancia-pixelek olvasása:**a ρ = 1,0 értékű DN**forrásonkénti** — A LATTICE-fájlok az XMP-be az `Chloros:PixelScale=32768` címkét írják be; az Survey3 fájlok a 65535 értéket használják (és nem tartalmaznak `Chloros:*` címkéket). Olvassa be a címkét, és osztja el azzal, ahelyett, hogy állandónak venné. A részletek és az egyetlen szándékos, nem skálázott szélsőséges eset a [CLI Referenciában](reference/cli-reference.md) találhatók.
{% endhint %}

**A feldolgozás mindig az `raw` fájlból indul.** A származtatott termékek (debayered/radiance/reflectance exportok) soha nem kerülnek vissza a feldolgozási folyamatba — újbóli importálásuk és feldolgozásuk a kalibrációs számítások kétszeri alkalmazását jelentené, ezért az Chloros kihagyja őket, és erről tájékoztatást is ad. Az `--input-level` egy szándékosan beépített menekülési útvonal arra az esetre, ha valóban szükség van egy belépési pont kényszerítésére.***

## Ha egy futtatás sikertelen

Az 1.2.0-s verziótól kezdve az `process` hangosan jelzi a hibát, ahelyett, hogy látszólag „sikeresen” futna, de semmit sem adna ki:

* Az a futtatás, amely **termékeket kért, de egyet sem írt ki**— csak az `project.json` és az `calibration_data.json` — kinyomtatja az `Processing finished but wrote no image products.` üzenetet, és**nem nulla kóddal lép ki**, így a szkriptek képesek ezt észlelni. A szokásos okok: a bemeneti mappát nem ismerte fel rögzítésként (ellenőrizze az elrendezést és az `--input-level` kódot), vagy az összes lekért termék alkalmatlan volt az adott kamerákhoz (pl. sugárzás/reflektancia kérése kizárólag RGB kamerákról).
* Egy **szándékos, kizárólag metaadatokra irányuló futtatás** (minden termék kikapcsolva, nincs `--indices`) továbbra is sikeresnek minősül — ebben az esetben az üres képkimenet a helyes eredmény.
* Futtassa újra az `--verbose`-szel, és ellenőrizze a háttérnaplóban az `[LATTICE-EXPORT]` / `[EXPORT-CHECK]` sorokat, amelyek magyarázatot adnak a kameránkénti kihagyásokra.

Kilépési kódok: `0` siker · `1` általános hiba · `2` argumentumhiba · `130` a Ctrl+C billentyűkombinációval megszakítva.

***

## Növényzetindexek

Futtassa az `--indices` parancsot egy vagy több előre beállított névvel; minden index a saját `<INDEX>_Index_Images/` mappájába kerül:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

A 22 előre beállított név, amelyet az `process --indices` elfogad:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Három indexlista létezik — ne keverje össze őket!**A grafikus felhasználói felület (GUI) Projektbeállítások legördülő menüjében 27 képlet található (ide tartoznak az `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — ez az öt kizárólag a grafikus felületen használható, és**nem** érvényes az `--indices`-re). Az élő/offline `lattice index --preset` parancs saját, külön 22 előre beállított listát használ. A képletek és a sávszámítások a [Multispektrális indexképletek](project-settings/multispectral-index-formulas.md) című dokumentumban találhatók.
{% endhint %}

***

## DAQ fényérzékelők: Rövid áttekintés

Az `daq pool-*` család az MAPIR DAQ spektrális érzékelőket (DAQ-U USB-n keresztül, DAQ-M BLE-n keresztül, DAQ-E Etherneten keresztül) a háttérrendszer állandó poolján keresztül — a grafikus felhasználói felület, az CLI és az SDK mind egy élő kezelőt használnak. **Az `pool-*` a szállított CLI-ben támogatott DAQ-útvonal**; az egyéb, esetleg hivatkozott `daq` alparancsok a MAPIR belső, kizárólag forrásként szolgáló felületei, és explicit hibával lépnek ki, amely a `pool-*`-re utal.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` `--duration` nélkül fut az `pool-record --stop`-ig; az alapértelmezett kimeneti könyvtár **a háttérrendszer gépén** az `~/Documents/DAQ Live View/`. A kap-korrekciós profilt a csatlakozáskor választják ki (`--cap-id`, háttérrendszer alapértelmezett: `sunshine_cosine`), és futás közben is kicserélhető az `pool-set-cap` — a felső határ profilokról és az érzékelő kalibrált tartományáról a kézikönyv DAQ fejezetei tárgyalnak.

{% hint style="warning" %}
**DAQ-E több hálózati kártyával rendelkező gazdagépen:** az indítás utáni első `pool-connect --eth` automatikus felismerés még működőképes érzékelő esetén is sikertelen lehet. Az `--eth-host <ip-or-hostname>` a megbízható megoldás — válassza ezt, ha a felismerés nem jár sikerrel.
{% endhint %}

***

## LATTICE kamerák, PTP és projekt-automatizálás

Az `lattice` család (több mint 45 alparancs) a LATTICE kamerák teljes körű kezelését biztosítja: felismerés, egyedi felvételek, állandó szinkronizált tömbök a grafikus felhasználói felület intelligens előkészítési csatlakozási folyamatával, élő böngésző-előnézet, igazítás, indexszámítás és gazdagép-hálózati kártya diagnosztika. Ízelítő:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Mellé: az `chloros-cli time-sync` jelentést készít a PTP grandmasterről, amelyet az Chloros gazdagép futtat (a LATTICE kamerák és a DAQ-E érzékelők ehhez kapcsolódnak az eszközök közötti időbélyegek érdekében), az `chloros-cli project` pedig megnyit egy elmentett Chloros projektet, és vezérli a kameráit, szenzortömbjeit és érzékelőit headless módban — beleértve a szkriptelt YAML-felvételi receptjeit is.

Ez a három család (`lattice`, `project`, `daq pool-*`) az egyetlenek, amelyek támogatják az `CHLOROS_BACKEND_URL` parancsot **távoli** háttérrendszer vezérléséhez; az alapvető parancsok mindig a helyi gépet célozzák meg.

A teljes útmutatók a kézikönyv LATTICE fejezeteiben találhatók; minden jelzőt a [CLI Referencia](reference/cli-reference.md) tartalmaz.

***

## Hibaelhárítás: Az 5 leggyakoribb hiba

| Tünet | Megoldás |
| --- | --- |
| Az `Login required` vagy egy ütemezett feladat lefagy az `Email:` parancssorban | Futtassa egyszer az `chloros-cli login EMAIL 'PASSWORD'` parancsot ezen a gépen — a gyors hibaüzenet helyett a gyorsítótárban nem tárolt munkamenet parancsai interaktív módon futnak. |
| `backend unreachable` | Indítsa el az Chloros asztali alkalmazást, vagy futtassa közvetlenül a háttérprogramot (`chloros-backend`). Ha az `lattice`/`project`/`daq pool-*`-et egy távoli háttérprogramra irányítja, ellenőrizze az `CHLOROS_BACKEND_URL`-et. |
| A tömbcsatlakozás blokkolva: `FRAMES WILL DROP` / `Reduce ROI to enable` | A gazdagép hálózati kártyájának vételi gyűrűje alapértelmezett értékekre állt vissza — ez a leggyakoribb oka annak, hogy egy korábban működő rendszer nem hajlandó csatlakozni, jellemzően egy hálózati kártya-illesztőprogram frissítése után. Futtassa az `chloros-cli lattice network --fix` parancsot egy **feljogosított** terminálról (vagy állítsa be az `ReceiveBufferLen=256`, `PendingReceives=64` értékeket); lásd a kézikönyv *Host NIC Setup &amp; Tuning* című részét. |
| Az `daq` alparancs kilép: „a teljes DAQ-csomag szükséges…” | Ez a szállított verzióknál várható — a lefordított CLI csak az `daq pool-*` családot tartalmazza, amely a csatlakozást, az adatfolyamot, a rögzítést és a csatornaválasztást fedi le. Használja az `pool-*` parancsot (vagy az Python-ből származó `chloros_sdk.connect_daq_sensor()`-et). |
| A Jetson nagy mappák esetén swap-figyelmeztetést jelenít meg | Adjon hozzá fájlalapú swap-ot — az CLI pontosan kinyomtatja az `fallocate`/`swapon` parancsokat, amelyeket futtatni kell. |

***

## Súgó

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Minden kapcsoló, minden alparancs:** [CLI Referencia](reference/cli-reference.md)
* **Python megfelelője:** [Python SDK](api-python-sdk.md) és a [SDK Referencia](reference/sdk-reference.md)
* **Támogatás:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
