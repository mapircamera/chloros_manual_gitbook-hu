# Chloros CLI Referencia

**Verzió:**

1.2.0**Létrehozás:**

2026.07.29. 19:19 ·**Felülvizsgálat:**

2026.08.30.**Célközönség:** LLM-felhasználásra optimalizálva; ember számára olvasható.**Hatály:** Az `chloros-cli` minden felhasználói alparancsa, opciókkal és másolható példákkal.

Ez a dokumentum a teljes referencia az `chloros-cli` parancshozparancssori eszközhöz, amely az MAPIR Chloros csomagban található. Szándékosan kimerítő, hogy egy LLM (vagy ember) a forráskód áttekintése nélkül is összeállíthassa az alábbi listákból bármely támogatott munkafolyamatot.

Ha csak a legfontosabb részekre van szüksége, ugorjon ide:
- [Ötperces gyorsindítás](#five-minute-quickstart)
- [LATTICE kamera első csatlakozási munkafolyamat](#lattice-camera-first-connect-workflow)
- [DAQ-érzékelő első csatlakozási munkafolyamat](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Rögzítési módok, rögzítők és offline újrafeldolgozás](#capture-modes-recorders--offline-reprocess)

---

## Konvenciók

- Minden parancs előtagja `chloros-cli`. Windows-en a bináris fájl neve `chloros-cli.exe`; Linux /Jetson-on pedig `chloros-cli`.
- Az opcionális argumentumok `--flag` formában szerepelnek. A kötelező pozíciós argumentumok zárójelek nélkül szerepelnek.
- Ha alapértelmezett érték van megadva, a jelző kihagyása esetén az a érték kerül felhasználásra.
- Az CLI egy „thin” HTTP kliens, amely az Chloros háttérrendszeren (Flask szerver az `127.0.0.1:5000`-en) fut. A háttérrendszert a legtöbb parancs automatikusan elindítja. Az `CHLOROS_BACKEND_URL=<url>` az **`lattice`**,**`project`**és**`daq pool-*`** parancscsaládokat egy távoli háttérrendszerre irányítja — az alapvető parancsok (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) szándékosan rögzítik az `http://127.0.0.1:<port>` parancsot, és figyelmen kívül hagyják (az IPv4-literál elkerüli a Windows oldalon található `localhost`→`::1` ~2 másodperces kérésenkénti büntetést). Lásd [Környezeti változók](#environment-variables).
- Minden SDK / CLI híváshoz Chloros+ fiókkal való bejelentkezés szükséges (futtassa az `chloros-cli login` parancsot gépenként egyszer; az eredmény az `~/.chloros/` fájlban kerül tárolásra).
- A példák az Linux útvonalakat használják; az Windows oldalon cserélje ki az `/home/user/...`-et az `C:/Users/.../...`-reX000067-re.

---

## Legfelső szintű áttekintés

```
chloros-cli [global options] COMMAND [command options]
```

### Globális beállítások

| Jelző | Leírás |
| --- | --- |
| `--backend-exe PATH` | Felülírja az automatikusan felismert háttérprogramot. |
| `--port N` | A háttérprogram HTTP portja (alapértelmezett: `5000`). |
| `-v, --verbose` | Részletes kimenet engedélyezése. |
| `--restart` | A háttérprogram kényszerű újraindítása (minden futó `backend_server.py` folyamatot leállít). |
| `--version` | Verziószám kinyomtatása (`Chloros CLI 1.2.0`). |
| `--help` | A legfelső szintű súgó megjelenítése. |

### Parancsok indexe

| Parancs | Cél |
| --- | --- |
| [`process`](#chloros-cli-process) | Egy mappa feldolgozása, amely „Survey3” vagy „LATTICE” rögzítéseket tartalmaz végpontok között. |
| [`login`](#chloros-cli-login) | A gép hitelesítése egy Chloros+ fiókkal. |
| [`logout`](#chloros-cli-logout) | A gyorsítótárban tárolt hitelesítő adatok törlése. |
| [`status`](#chloros-cli-status) | Az aktuális licenc- és hitelesítési állapot megjelenítése. |
| [`export-status`](#chloros-cli-export-status) | A Thread-4 exportálásának élő állása egy `process` futás során. |
| [`language`](#chloros-cli-language) | A CLI megjelenítési nyelvének beállítása vagy felsorolása (38 támogatott). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Alapértelmezett projektmappa (megosztva a grafikus felhasználói felülettel). |
| [`update`](#chloros-cli-update) | A CLI frissítéseinek ellenőrzése és telepítése (Linux /Jetson). |
| [`selftest`](#chloros-cli-selftest) | Rendszerdiagnosztika + füsttesztek. |
| [`time-sync`](#chloros-cli-time-sync) | PTP grandmaster állapot / vezérlés. |
| [`lattice`](#chloros-cli-lattice) | LATTICE kamera vezérlés és felvétel (több mint 45 alparancs). |
| [`daq`](#chloros-cli-daq) | DAQ spektrális érzékelő vezérlése (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Mentett Chloros-projekt megnyitása és futtatása (kamerák + DAQ-ok). |

---

## Telepítés

Az `chloros-cli` minden támogatott platformon az Chloros asztali telepítőjében található — nincs külön CLI letöltés. A platformcsomag telepítésével az `chloros-cli` hozzáadódik az `PATH`-hez az asztali alkalmazás és az általa vezérelt háttér bináris fájl mellett.

Legfrissebb letöltések: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> A telepítő kényelmi indító szkripteket is tartalmaz (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), amelyek egy azonnal használható CLI parancssort nyitnak meg; ezekről a [CLI Felhasználói útmutató](../CLI.md) tartalmaz részletes leírást, ezért itt nem ismételjük meg őket.

### Windows (.exe)

1. Töltse le az Windows telepítőt a letöltési oldalról.
2. Futtassa az `Chloros-Setup-x.y.z.exe` fájlt, és kövesse a varázsló utasításait. Az alapértelmezett telepítési útvonal az `C:\Program Files\Chloros\` (a CLI az `C:\Program Files\Chloros\cli\` mappába kerül, amelyet a telepítő hozzáad a PATH változóhoz).
3. Nyisson meg egy új terminált (`cmd.exe`, PowerShell vagy Windows terminál), hogy a rendszer felismerje a frissített `PATH` fájlt.

```powershell
chloros-cli --version
```

A telepítő automatikusan hozzáadja az `chloros-cli.exe` fájlt a rendszer `PATH` változójához, és csomagolja az LATTICE kamerákhoz szükséges Arena SDK futtatókörnyezetet.

### Linux amd64 (.deb)

Ubuntu 22.04 LTS vagy újabb verziókhoz / Debian-alapú x86_64 munkaállomásokhoz.

> **Az Ubuntu 20.04 nem támogatott.** A csomag függőségi listája abból származik,
> amire a háttérprogram ténylegesen hivatkozik, és ez magában foglalja az `libc6 (>= 2.34)`-et is;
> a focal a glibc 2.31-et szállítja. Az `apt` inkább elutasítja a telepítést, mintsem hogy
> futásidőben.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

A .deb a következőket telepíti:
- `chloros-cli`-et `/usr/bin/chloros-cli`-re
- A lefordított háttérprogramot `/usr/lib/chloros/chloros-backend`-re-re
- Az Arena SDK futásidejű környezetet (LATTICE kamerákhoz)
- Zajszűrő modelleket, kalibrációs csomagokat és frissítési csatorna konfigurációt

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Ugyanaz az elrendezés, mint az amd64 .deb fájlnál, a Jetson Orin / Orin NX / Orin Nano készülékekhez hangolt CUDA-verzióval.

### Egyszeri hitelesítés gépenként

Minden platformon egyszeri bejelentkezés szükséges a Chloros+ oldalon, mielőtt a SDK / CLI hívások működni kezdenének:

```bash
chloros-cli login user@example.com 'YourPassword'
```

A hitelesítő adatok az `~/.chloros/user_session.json` fájlban kerülnek tárolásra.

### A telepítés ellenőrzése

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Chloros+ előfizetés szükséges.**A CLI használatához aktív Chloros+ tervezetre van szükség. A**Copper**a belépő szintű Chloros+ csomag – minden fizetős Chloros+ csomaghoz tartozik CLI / SDK hozzáférés; csak az ingyenes**Iron** csomag nem. (Csomag-azonosító táblázat: `0`=Iron/ingyenes, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Frissítés a [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Ezt a korlátot nem csupán az CLI, hanem a háttérrendszer is érvényesíti: a fizetett csomaggal nem rendelkező, SDK / CLI jelöléssel ellátott kéréseket `403 PLAN_UPGRADE_REQUIRED` kóddal utasítják el, függetlenül attól, hogy azok az `chloros-cli`, az Python SDK vagy egy saját fejlesztésű HTTP kliensről érkeznek-e. A kijelentkezett felhasználó helyette az `401 AUTH_REQUIRED` hibakódot kapja. Az elérhetőség a csomag türelmi ideje alatt (havi csomag esetén 30 napig, éves csomag esetén a lejáratig) offline módban is működik, és a türelmi idő lejárta után megszűnik; az `chloros-cli status` hibakód továbbra is működik, így az ok látható marad (ez ez az egyetlen SDK / CLI útvonal, amely mentesül a szintkorlátozás alól — `GET /api/license-status`).

---

## Ötperces gyorsútmutató

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Feldolgozza a képeket tartalmazó mappát a teljes „Chloros” folyamaton keresztül (célpont-felismerés → kalibrálás → vignettálás → fényvisszaverődés → index exportálás).

### Összefoglalás

```
chloros-cli process INPUT [OPTIONS]
```

### Pozíciós argumentumok

| Argumentum | Leírás |
| --- | --- |
| `INPUT` | Az `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) vagy `.dng` fájlokat tartalmazó bemeneti mappa elérési útja. |

### Általános beállítások

| Jelző | Alapértelmezett | Leírás |
| --- | --- | --- |
| `-o, --output PATH` | egy új, időbélyeggel ellátott mappa az alapértelmezett projektmappa alatt (`~/Chloros Projects`, ha nincs másként beállítva) | Létrehozandó vagy újrahasznosítandó projektmappa. Ha a mappában már található egy `project.json` fájl, akkor a felülírás helyett egy `_1`/`_2` nevű testvérmappa jön létre. |
| `-n, --project-name NAME` | automatikus (időbélyeg) | Projektnév. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` Chlorosés neurális debayer-t használ; lassabb, de jobb minőségű. |
| `--vignette / --no-vignette` | `--vignette` | Vignettakorrekció. |
| `--reflectance / --no-reflectance` | `--reflectance` | Reflektancia-kalibrálás (ha található, a panelcélpontot használja, LATTICE esetén NIST sorozatszámonkénti kalibrálást). A LATTICE multispektrális esetében ez egyben a reflexió **termék** kapcsolója is — lásd [Termékenkénti exportkapcsolók](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | ki | PPK GNSS-korrekciók alkalmazása a sidecar fájlokból. |
| `--exposure-pin-1 MODEL` | ki | A „Survey3” kettős kamerás rendszer „pin-1” modelljét. |
| `--exposure-pin-2 MODEL` | ki | A „pin-2” modell rögzítése. |
| `--recal-interval SECONDS` | 0 | A kalibrációs számítások újrafuttatásának kényszerítése a felvételkészítés ideje alatt N másodpercenként. |
| `--timezone-offset HOURS` | helyi | Felülírja a kimeneti metaadatokba beépített időzóna-eltolást. |
| `--format FORMAT` | `TIFF (16-bit)` | Az `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` közül az egyik. |
| `--indices NAME [NAME ...]` | nincs | Vegetációs indexek (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | A LATTICE TIFF-ek esetében (a Survey3 .raw fájl nem érintett). Ezenkívül a menekülési út, amely lehetővé teszi, hogy a **raw nélküli** rögzítéseket egyáltalán feldolgozzák — lásd [Hogyan néz ki egy captures mappa](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | be | A lineáris debayered termék kimenete (`Debayered_Images`). Lásd [Termékenkénti exportkapcsolók](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | be | A képernyőn megjelenő előnézet kimenete (`Preview_Images`): RGB = fehér egyensúly (DAQ-megvilágító, ha elérhető, egyébként szürke-világ) + gamma; multispec = hamis színű kiterjesztés. |
| `--radiance / --no-radiance` | be | Float32 sugárzási érték kibocsátása (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referencia a LATTICE reflexiós termékhez: `auto` = a minőségbiztosításon átesett, a keretben lévő célpont az abszolút referencia, DAQ-lefelé irányuló (ρ = π·L/E) tartalékérték; `target` = szigorú (nincs DAQ-helyettesítés); `daq` = DAQ-meghatározó. Lásd [Termékenkénti exportkapcsolók](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | nincs | Az egységenkénti **mért** célreflektancia-letapogatások könyvtára (`<serial>.csv`); hiány esetén a névleges T3/T4P spektrumokra vált vissza. |
| `--array-alignment / --no-array-alignment` | be | LATTICE-tömbök: az egyes felvételek `Chloros:Alignment*` XMP-fájljában rögzített modulok közötti igazítást alkalmazza minden feldolgozott termékre (debayered / preview / radiance / reflectance / index). A címkék nélküli képek esetében nincs művelet. |
| `--array-alignment-crop / --no-array-alignment-crop` | crop | Az igazított exportokat a rács közös átfedési területére vágja, így minden modul ugyanazt a lábnyomot használja; az `--no-…` megtartja a teljes érzékelő felületét (fekete kitöltés a forráson kívül fekete kitöltéssel). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Újramintavétel az igazítási torzításhoz. Az `nearest` megőrzi a forrás pontos dinamikatartományát (a radiometrikus értékek pixelek közötti keveredése nélkül). |

### Célfelismerési beállítások

| Jelző | Leírás |
| --- | --- |
| `--min-target-size PIXELS` | A detektor minimális panel-célmérete (px). |
| `--target-clustering 0-100` | Klaszterezési érzékenység. |
| `--target / --targets` | A bemeneti mappát kizárólag célpaneleként kezeli (a felmérési felismerést kihagyja). |

### Példák

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Termékenkénti exportkapcsolók (LATTICE multispektrális)

A LATTICE feldolgozás **egy lépésben** **minden alkalmazható termékre**kiterjed. A négy típusonkénti kapcsoló — `--debayered`, `--preview`, `--radiance`, `--reflectance` —**alapértelmezés szerint mind be vannak kapcsolva**; az `--no-<type>` formátummal lehet egyet kikapcsolni. A RGB mesterkamerák kizárólag debayered + előnézeti (nincs sávonkénti sugárzás/visszaverődés), így az `--radiance`/`--reflectance` ezeknél hatástalanok. A kapcsolókat figyelmen kívül hagyja az Survey3 `.raw` (amely a szabványos reflektancia/cél útvonalat követik). *(A régi `--radiometric-output {reflectance,radiance,sensor-response}` jelzőt **eltávolították**, és ezekkel a kapcsolókkal helyettesítették; már nincs `sensor-response` szint.)*

| Termék | Kimenet | Szükséges-e DAQ lefelé irányuló jel? |
| --- | --- | --- |
| `--debayered` | Lineáris demosaic (`Debayered_Images`). | Nem. |
| `--preview` | Előnézet megjelenítése (`Preview_Images`): RGB = WB + gamma; multispec = hamis színű kiterjesztés. | Nem. |
| `--radiance` | float32 W/m²/sr/nm a teljes radiometrikus láncból (`Radiance_Images`). | Sz. |
| `--reflectance` | uint16 reflexió ρ (`32768` = 1,0), Pix4D-kompatibilis. | **Igen**, kivéve, ha egy minőségbiztosításon átesett, a képkockán belüli célpont rögzíti (lásd alább). |

`--reflectance-source` kiválasztja a reflexiós referenciaértéket:**`auto`**(alapértelmezett) egy minőségbiztosításon átesett, a képkockán belüli célt tesz**abszolút referenciává**— a célponthoz rögzített empirikus vonalláncokat a kihagyott paneleken keresztértékelik, és a mért győztes értéket alkalmazzák — visszatérve a DAQ lefelé irányuló osztásához (ρ = π·L/E) ha nincs célpont, vagy a minőségbiztosítás sikertelen; a**`target`**szigorú (nincs DAQ-helyettesítés); a**`daq`**a DAQ-vezérelt viselkedést választja. A célpont geometriája (ArUco / rögzített ROI / csík) a projekt célkonfigurációjából származik; az `--target-reflectance-dir DIR` egységenkénti**mért** letapogatásokat (`<serial>.csv`) tárol, amelyeket a célegység sorozatszáma/QR-kódja alapján lehet lekérdezni, a névleges T3/T4P spektrumok pedig tartalékértékekként szolgálnak.

A DAQ-reflektancia útvonal automatikusan meghatározza az **időbélyeggel egyező lefelé irányuló sugárzást**egy rögzített**`.daq`**(DAQ-U/M/E)**vagy a képek mellett található DAQ-M natív `.csv`**fájlból. Ha a kameránkénti vagy DAQ-kalibrációs csomag nincs helyileg tárolva, a feldolgozási folyamat**az első használatkor automatikusan letölti azt az AWS-ről** (ehhez egyszer szükség van internetkapcsolatra; az adat az `~/.chloros/` név alatt kerül tárolásra).

#### Reflektancia-pixelek olvasása (Pix4D / Metashape / saját szkriptek)

A reflexió egész számú DN-ként van tárolva, és **az a DN-érték, amely ρ = 1,0-t jelent, a forráskamerától függ**:

| Forrás | ρ = 1,0 értéke | Hogyan lehet megkülönböztetni |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (tartalék ρ 2,0) | Az XMP fájlban szerepel az `Chloros:PixelScale=32768` címke. |
| Survey3 | `65535` (ρ 1,0-nál levágva) | Nincs `Chloros:*` XMP-címke — ez a hiánya *mag* a jel. |

**Olvassa be az `Chloros:PixelScale` értéket, és ossza el vele**, ahelyett, hogy állandót feltételezne. A címke az uint16 tartományban van definiálva, így az `32768` érték marad az olyan kimeneti formátumokban is, amelyek újraméreteznek — az `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` és `TIFF (32-bit, Percent)` mind önleíróakleíróak (először normalizáljuk vissza a tárolt adattípust uint16-ra: 8-bites érték esetén ×257, float esetén ×65535).

> **Egy eset tervezés szerint nem hordoz méretarányt.** Amikor egy 8-bites forrásból származó felvételt (BayerRG8) 8-bites TIFF formátumban írnak, a feldolgozási folyamat *levágja* az értékeket 0,.255-re, ahelyett, hogy újraméretezne, így minden ρ≈0,008 feletti érték 255-re laposodik, és a fájlt nem jellemzi semmilyen skála. A Chloros szándékosan kihagyja mind az `Chloros:PixelScale`, mind az `MicaSense:RadiometricCalibration` tupelt , és naplózza az okát. **Ha a címke hiányzik egy LATTICE reflektanciafájlból, ne feltételezzünk skálát — inkább exportáljuk újra 16-bites vagy 32-bites formátumban**, ahelyett, hogy olyan pixeleket osztanánk, amelyek soha nem voltak oszthatók.

#### Az exportba átvitt EXIF-adatok

Az `process` minden termékre átmásolja a forrásfelvétel **GPS-blokkját és az ExifIFD-jét** minden termékre, így az
exportálás során az `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` és
`CameraSerialNumber` fájlokat a georeferenciálás mellett.

**Az `FocalLength` nem opcionális a fotogrammetriaimetriában.** A Pix4D a földi mintavételi távolságot a
fókusztávolság és a magasság alapján számítja ki; a címke hiányában rendkívül téves méretarányra áll át. Egy
49 felvételből álló narancssárganarancsültetvény felett végzett repülés során a hiányzó címke egy 411 m × 160 m-es területet egy rekonstruált
47,8 km × 13 km-es területté alakított – egy 455 MP-os ortofotó, amely többnyire „nodata” értékeket tartalmazott, amit aztán mozaikproblémának és
BigTIFF-problémának tulajdonítottak, mielőtt bárki is ellenőrizte volna a GSD-t. Ha az Ön ortofotója hihetetlen
méretarányú lesz, először futtasd le az `exiftool -FocalLength` parancsot az exportált terméken.

A másolat szándékosan **nem** `-all:all`: az IFD0 szerkezeti címkéi megszakítják a LATTICE kimenetét, amikor
másolják őket, az `ExifImageWidth` / `ExifImageHeight` pedig azért vannak kizárva, mert a
*forrás* felvételt írják le – egy korábban átméretezett export egyébként olyan méreteket hordozna,
amelyek ellentmondanak a saját raszterének. Az XMP-t közvetlenül írjuk, nem másoljuk, mert az ExifTool
elveti az azonos hívású XMP-címkéket, amikor az XMP-blokkot másolják (ami elhagyná az MAPIR
kalibrációs címkéket).

### A kimenetek helye

A termékek **a projektmappa alá kerülnek, kameránként, majd fájlformátum szerint csoportosítva**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

A LATTICE kameramappája `LATT-<sensor>-<lens>-F<filter>` (megegyezik a felvétel EXIF-jével:
`Model`), a `<model>_<filter>` az Survey3 esetében — két kamera, amely ugyanazt az érzékelőt és szűrőt használja, de
objektívjük eltérő, külön fa struktúrában van tárolva, mivel a vignettálás, a látómező és a torzítás eltérő. A formátum
mappa az `--format`-et követi: `tiff16`, `tiff8`, `png8`, `jpg8` vagy `tiff32` az
`TIFF (32-bit, Percent)` esetében.

> **Minden exportált termék megtartja a FORRÁS fájl nevét.** A
> `capture_…_raw.tif` továbbra is `capture_…_raw.tif` néven szerepel — csupán az
> `tiff32/Radiance_Images/` mappában található. **A mappanév azonosítja a terméket, nem a fájlnév**, ezért az
> az `*radiance*.tif` fájlra nem talál semmit; inkább a könyvtár alapján keressen.

### Fényérzékelő-felvételek — kalibrált `.daq` + `.csv`

Az `process` a bemeneti mappában található `.daq` felvételeket is feldolgozza, és ehhez **nincs**
szüksége semmilyen képre: egy önállóan repült DAQ-U / DAQ-M / DAQ-E már önmagában is teljes
rögzítő, és egy kizárólag `.daq` fájlokat tartalmazó mappa is érvényes bemenetnek számít.

A DAQ-ot **kalibrálás nélkül** is lehet rögzíteni — ezt végzik alapértelmezés szerint a nyilvánosan elérhető
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) felvevők
(`record_daq.py`) alapértelmezés szerint ezt teszik: a nyers érzékelőértékeket írják ki, és megjelölik a fájlt, így
a Chloros **soros kapcsolaton keresztül** (először a helyi gyorsítótárból,
majd a MAPIR felhőből), és alkalmazza azt. Az `process` visszírja az eredményt:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

Az `.csv` minden leolvasáshoz egy sort tartalmaz: UTC időbélyeg, integrációs idő, teljes teljesítmény,
fotopikus/skotopikus lux, PPFD (és annak kék/zöld/vörös bontása), csúcs hullámhossz, majd a
teljes spektrumot az érzékelő saját hullámhossz-rácsán. Az `.daq` újraimportálja az adatokat anélkül, hogy
másodszor is kalibrálná őket.

Siker esetén a futtatás az `Light-sensor products written: N (calibrated .daq + .csv)` eredményt jelenti.
A zárójelben szereplő szöveg leírja, hogy mi került ténylegesen írásba, így
`(RAW COUNTS — this sensor has no calibration bundle)` olvasható egy csomag nélküli érzékelő esetében,
`(N calibrated, M raw counts)` pedig egy mindkettőt tartalmazó mappa esetében. A háttérrendszersaját
`[DAQ-EXPORT]` és `[RUN-SUMMARY]` címsorai ugyanúgy alakulnak ki – a
három közül sem nevezheti kalibráltnak a nyers exportot.

Egy DAQ-U / DAQ-M / DAQ-E felvétel, amelynek kalibrációs csomagja nem tölthető le – mert
offline állapotban van, vagy az adott érzékelőhöz nincs kalibrációs fájl – **ok megjelölésével kihagyásra kerül** egy
`[DAQ-EXPORT]` sorban, és soha nem kerül kiírásra „kalibrált” fájlként, amely nyers számlálási adatokat tartalmaz.
Csatlakozzon az internethez, és futtassa újra a programot. Az ok az, amelyet az olvasó ténylegesen
állapított meg az adott fájlra vonatkozóan (olvashatatlan sémakészlet, nincs csomag, írási hiba), és a futtatás
összefoglalója **különálló** okokat sorol fel — húsz fájl kihagyása egy ok miatt egyetlen
okként jelenik meg, nem pedig annak húsz ismétléseként.

#### A DAQ-A felvételek nyers számértékekként exportálódnak

A **DAQ-A** család a sorozatszámonkénti csomagrendszer bevezetése előtt készült, és nincs kalibrációs csomagja,
amit letölthetne — ehelyett a helyszínen egy reflektancia-célponttal kalibrálják, ezért
soha nem is volt rá szüksége. E felvételek elutasítása miatt nem volt módjuk a
számokat, ezért **más név** alatt exportálják őket:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Egy másik fájlnév, nem pedig a fájlon belüli jelölés, mert az adatnak meg kell maradnia
akkor is, ha puszta névként továbbítják e-mailben. Az `.csv` fejlécben az
`raw spectral sensor counts (NOT irradiance)` szerepel, és figyelmeztet arra, hogy az értékek
**a** fájlon belül – ami pontosan az, amire a célalapú kalibrálás használja őket –, és
nem érzékelők között. A teljesítményfüggő fotometrikus oszlopok (teljes teljesítmény, fotopikus és
skotopikus lux, PPFD) **NULL** értékkel kerülnek rögzítésre, ahelyett, hogy a számlálásokból integrálnák őket, és a futtatási
összefoglalóban az `RAW COUNTS` szerepel, így a naplóba „exportált” értékek nem értelmezhetők besugárzási intenzitásként.

A régebbi **v1.01 / v1.02** felvételek (amelyeket egy DAQ-A-SD ír) nem tartalmaznak olvasásonkénti időpontot,
csak a fájl írási idejét. A kép↔lefelé irányuló sugárzás-egyeztető továbbra sem fogadja el őket — egy
képkocka és az írási idő összeillesztése láthatatlanul hibás lenne — de az exportáló program beolvassa őket, és
a „CSV” kimenetben az `clock=daq_created_on` jelenik meg, így a termék jelzi, hogy melyik órára van beállítva.

### Megjegyzések

- Az `process` automatikusan felismeri, hogy a mappád „Survey3”, „LATTICE” vagy vegyes típusú-e.
- Az előrehaladás a Server-Sent Events-en keresztül történik; az CLI oldalon szálankénti élő előrehaladási állapot látható (felismerés, elemzés, feldolgozás, exportálás).
- A Linux /Jetson esetében az CLI ellenőrzi a swap-területet, és nagy mappák feldolgozása előtt figyelmeztetést jelezhet. A textúraérzékeny debayer emellett automatikusan GPU-frekvenciakorlátozást alkalmaz az alacsony fogyasztású Jetson-készülékeken (Nano, Orin Nano) automatikusan GPU-frekvenciakorlátozást alkalmaz.
- Siker esetén a futtatás jelentést ad arról, hogy hány képi terméket írt ki (`Image products written: N`).

#### A képeket nem író futtatás sikertelen

Ha termékeket kértél, és a futtatás **egyet sem** írt ki — csak `project.json` és
`calibration_data.json` —, az `process` ezt hibaként kezeli: kinyomtatja az
`Processing finished but wrote no image products.` üzenetet ír ki, és **nem nulla értékkel lép ki**, így egy szkript
felismerheti a hibát. Az üzenet megnevezi a projektmappát és a szokásos okokat:

- a bemeneti mappát nem ismerte fel rögzítésként (ellenőrizze az elrendezést és az `--input-level`-et), vagy
- minden kért terméket kihagyott, mivel azok nem alkalmazhatók az adott kamerákra (pl.
  RGB-only kameráktól kért sugárzási/visszaverődési értékek).

Futtassa újra az `--verbose` parancsot, és ellenőrizze a háttérnaplóban az `[LATTICE-EXPORT]` / `[EXPORT-CHECK]` sorokat,
amelyek kameraenkénti kihagyásokat magyarázzanak, amelyek egyébként nem kerülnek be a CLI kimenetét.

Egy szándékosan kizárólag metaadatokra korlátozott futtatás — minden termék kikapcsolva és nincs `--indices` — még mindig
**sikeres**, mert ebben az esetben az üres képkimenet a helyes eredmény.

Ugyanígy **a kizárólag fényérzékelővel végzett futtatás** is: egy `.daq` felvételeket tartalmazó mappa definíció szerint nem tartalmaz exportálható képeket, és a futtatást a helyette írt, kalibrált `.daq` / `.csv` alapján értékelik.

---

## `chloros-cli login`

Hitelesítse ezt a gépet egy Chloros+ felhőfiókkal. A hitelesítő adatok biztonságosan tárolódnak az `~/.chloros/user_session.json` fájlban.

```
chloros-cli login EMAIL PASSWORD
```

### Példák

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (a jelszó egyes részeinek elhagyásával vagy duplikálásával). 401-es hiba esetén a CLI automatikusan újrapróbálja a bejelentkezést az `$$` újra hozzáfűzésével, majd a jelszó duplikált felének elhagyásával ; ha az újrakísérlet sikeres, bejelentkezik, és kinyomtatja a következő alkalommal használandó helyes egyszerű idézőjel-szintaxist.

> **Headless/parancsfájl-alapú használat: a gyorsítótárban tárolt munkamenet hiánya interaktív parancssort jelent, nem pedig gyors hibaüzenetet.** Bármely háttérfolyamatot indító parancs (`process`, `status`, `export-status`, `time-sync`, …), amelyet gyorsítótárban tárolt licenc/munkamenet nélkül futtatnak, a folytatás előtt interaktív `Email:` / `Password:` parancssorba vált át a stdin-en. Egy gyorsítótárban tárolt munkamenet nélküli felügyelet nélküli feladat ezért bemenetre várva lefagy — futtassa az `chloros-cli login EMAIL PASSWORD` parancsot, mielőtt headless feladatokat ütemez.

---

## `chloros-cli logout`

Törli a gyorsítótárban tárolt munkamenetet, és a következő híváskor új bejelentkezést kényszerít.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Megjeleníti az aktuális licencszintet (Iron/Copper/Bronze/Silver/Gold), a hitelesített felhasználót és az eszközhoz kötött fiókok számát.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

A Thread-4 exportálás aktuális állapotának lekérdezése. Biztonságosan hívható **a**`process` parancs futása**alatt** egy másik parancssorból.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Beállítja az CLI megjelenítési nyelvét (38 nyelv támogatott, beleértve a CJK, RTL és indiai nyelveket is). Azoknál a régebbi konzoloknál, amelyek nem tudják megjeleníteni a szkriptet, zökkenőmentesen visszatér az angol nyelvre.

```
chloros-cli language [LANG_CODE] [--list]
```

### Példák

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Projektmappa-parancsok

Ezek az alapértelmezett projektmappa helyét kezelik (a grafikus felhasználói felülettel megosztva).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux/ Csak Jetson. Ellenőrzi az `version_url` fájlt az `/etc/chloros/update.conf` fájlból, és felajánlja a megfelelő `.deb` fájlt.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

Linux-en / Jetsonon az CLI **minden indításkor automatikus frissítésellenőrzést** is futtat (nem blokkoló, soha nem késlelteti a parancsot): beolvassa az `/etc/chloros/update.conf` fájlt, az eredményt 1 órára az `~/.chloros/update_cache.json` fájlba tárolja, és kinyomtatja az `Update available: vX.Y.Z / Run: chloros-cli update` üzenetet, ha újabb verzió létezik. Hiba esetén és a Windows parancs esetén csendben kihagyja a műveletet.

---

## `chloros-cli selftest`

7 lépésből álló gyors tesztet futtat: verzió, port elérhetősége, háttérprogram indítása, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), zajszűrő modell jelenléte, CUDA+zajszűrő készenléte.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

PTP grandmaster állapota és vezérlése. A Chloros gazdagép futtatja a PTP grandmastert; a LATTICE kamerák és a DAQ-E egységek ehhez kapcsolódnak az eszközök közötti időbélyegek érdekében.

| Alparancs | Leírás |
| --- | --- |
| `status` | A grandmaster állapotának, a BMCA prioritásoknak és az óraazonosítónak a megjelenítése. |
| `peers` | A Delay_Req-en keresztül észlelt slave-ek listája (kamerák + DAQ-E érzékelők). |
| `cameras` | Kameránkénti PTP-állapot (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | A grandmaster folyamat újraindítása. |
| `set-priority --priority1 N --priority2 N` | A BMCA prioritások felülírása. |

### Példák

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

LATTICE kamera vezérlés. Minden alparancs a Chloros háttérprogramon keresztül fut; a háttérprogram birtokolja a kamerakészletet, így a későbbi CLI hívások ugyanazt a nyitott kezelőt használják újra.

### Általános opciók (a legtöbb alparancsban közös)

| Jelző | Leírás |
| --- | --- |
| `-d, --device N` | Kamera index (alapértelmezett: 0). |
| `-s, --serial SN` | Konkrét sorozatszám; felülírja az `--device` értéket. |
| `--serials SN1,SN2,…` | Többkamerás működéshez vesszővel elválasztott sorozatszámok. |
| `--all` | Minden felismert kamerán végrehajtás. |
| `--exposure US` | Expozíciós idő mikroszekundumokban. |
| `--gain DB` | Erősítés dB-ben. |
| `--pixel-format FMT` | pl.g. `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Képméretek. |
| `--preset {default,high_quality,high_speed,triggered}` | Beállítási sablon alkalmazása. Mindegyik szabadon fut, kivéve az `triggered`-et, amely a 2. vonalon lévő hardveres élre érzékenyen állítja be a kamerát – ha semmi sem vezérli azt a vonalat, akkor a kamera örökké várni fog, ahelyett, hogy felvételt készítene. |
| `-o, --output DIR` | Kimeneti könyvtár (alapértelmezett: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | GVSP-csomagméret. Az `auto` ICMP+GVSP-próbákat futtat; `jumbo` = 9000; `standard` = 1500. |

### LATTICE kamera első csatlakozási munkafolyamat

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Alparancsok áttekintése

#### Felderítés és információk

| Alparancs | Cél |
| --- | --- |
| `lattice info` | A csatlakoztatott kamerák listája (gyártó, modell, sorozatszám, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | A gazdaszerver elemzése az optimális kamerakonfiguráció érdekében. Az `--no-discover` kihagyja a kamerák felderítését (gyorsabb, kizárólag a hálózati kártyát elemzi). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Hálózati kártya beállításainak ellenőrzése/javítása; sávszélesség/FPS becslése. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Stabil sémájú háttérrendszer hálózati képessége + tömbajánlás (visszaadja: `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). Az `auto_capped_fps` megtartja a kért felbontást, de korlátozza a cél fps-t — olvassa be az `recommended.recommended_target_fps` értéket, és adja át csatlakozási célként; kezelje sikernek, ne hibának. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | „Mi lenne, ha” elemzés a kamerák megnyitása nélkül. **Az `--n-active` a hálózaton lévő kamerák teljes száma, nem csak ennek a tömbnek a száma**— növelje meg, ha önálló kamerák egyszerre streamelnek, vagy a hálózati sávszélességet olyan igény alapján számolják ki, amely alulbecsüli azok számát (alapértelmezett: `len(--models)`). Mindig kinyomtatja az összesített `Wire budget:` (igényelt MB/s vs. ütközésmentes felső határ) és `Max cameras:` sorokat, és jelzi az `** OVER-SUBSCRIBED**` értéket, amikor a tömb túllépialuljegyzi a vezetéket — lásd [Tömb fps és burst modell](#array-fps--burst-model). |
| `lattice gpu` | A GPU állapotának megjelenítése. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | A kamera firmware-jének ellenőrzése vagy frissítése. Helyi `.fwa` kiválasztás rögzítve van: az `firmware/<MODEL_PREFIX>/` mappában található, a build `MIN_FIRMWARE_VERSION`-jéhez illeszkedő fájl flashelésre kerül, ha jelen van (a legmagasabb verzió csak tartalékként), így a lemezen tárolt újabb gyártói kép addig inaktív marad, amíg azt a rögzítést fel nem oldják — a szándékosan újabb kiadások az aláírt AWS-manifeszt révén jutnak el az egységekhez, ami az újabb verziók esetén előnyösebb. |
| `lattice presets [--apply NAME]` | A kamera előre beállított értékeinek felsorolása vagy alkalmazása. |
| `lattice status` | Élő kameraállapot. |

#### Felvétel

| Alparancs | Cél |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Egyetlen képkocka. **Alapértelmezés szerint minden exporttípust elment** (`--processing all`); lásd [Rögzítési exportszintek](#capture-export-levels-the-all-default). Az `--levels` egy kifejezett részhalmazt ment (felülírja az `--processing` parancsot); az `--force-daq` a hozzárendelt DAQ-értéket `.daq` sidecar fájlként írja, még nyers formátumú rögzítés esetén is. `--jpeg-quality` = JPEG minőség 1–100 (alapértelmezett 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Folyamatos írás a lemezre a Ctrl+C billentyűkombinációig. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Böngészőalapú élő MJPEG-előnézet. Az `--ae-damping` beállítja az automatikus expozíciócsillapítást (0,4–100). |

#### Érzékelő beállítása

| Alparancs | Cél |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Bármely GenICam-csomópont olvasása/írása. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Expozíció és automatikus expozíció (AE). |
| `lattice gain [--auto] [--off] [--set DB]` | Erősítés és automatikus erősítés. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | Érzékelő ROI és binning. |
| `lattice format [--set FMT] [--list]` | Pixelformátum. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Hardveres/szoftveres trigger. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (flag nélkül = egyszeri fehér egyensúly) | Fehér egyensúly műveletek. Csak RGB/Bayer kamerák esetében; monokróm M3M esetén nem hajtódik végre (kihagyásra kerül). |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | RGB színkezelési folyamat. `natural` (alapértelmezett) az olcsó élő feldolgozás; `enhanced` hozzáadja a színszegély-eltávolítást, az élénkséget és a CLAHE helyi kontrasztot a teljes hub-paritású megjelenéshez, körülbelül kétszeresképkockánkénti feldolgozási költség mellett, így alacsonyabb **élő** képfrissítési sebességet eredményez – a mentett felvételek mindkét esetben mindig a teljes feldolgozást kapják. Csak RGB/Bayer kamerák esetén; mono M3M esetén kihagyva. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | A telítettség/kontraszt megjelenítése (szűrős kamerák: RGB). Mono M3M esetén kihagyva. |
| `lattice filter [--set NAME] [--list]` | A kamera szűrőmodelljének beállítása (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Teljesítmény- és hőérzékelők ellenőrzése; aaz alacsony fogyasztású készenléti állapot beállításához. |

#### Kalibrálás és érzékelők

| Alparancs | Cél |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Kalibrálás fényvisszaverő célpont alapján. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Beépített-beépített lefelé irányuló fényérzékelő parancsok. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Vignettakorrekció alkalmazása a meglévő képekre. |

#### Többkamerás (átmeneti munkamenetek)

| Alparancs | Cél |
| --- | --- |
| `lattice multi-info` | Az összes szinkronizáló szerepkörrel rendelkező kamera felsorolása. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Egy szinkronizált képkocka minden kamerából. **Alapértelmezés szerint minden exporttípust**elment, ha állandó tömb csatlakozik; az átmeneti, tömb nélküli tartalék megoldás**csak debayeringelt** (a többihez először futtassa az `array-connect` parancsot). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Szinkronizált képkockák streamelése (átmeneti). |
| `lattice multi-test [--count N]` | GPIO szinkronizációs időzítés tesztje. |
| `lattice multi-detect [--line LINE] [--json]` | GPIO mester/szolga bekötés automatikus felismerése. |

#### Igazítás

| Alparancs | Cél |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — plusz detektor/illesztő gombok `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, RANSAC gombok `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, többképkombináció `[--averaging mean\|median\|inlier_weighted]`, geometriai korlátok `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, térbeli korlátozás `[--roi X0,Y0,X1,Y1] [--mask PATH]`, valamint szolga-specifikus felülírások `[--per-cam-override SN:KEY=VALUE]` (ismételhető) | Igazítási profil kiszámítása élő kamerákból. Az `--prefilter` alapértelmezés szerint az `gradient` (éltérkép; megegyezik a GUI/tömb-igazítóval — az élek a spektrális sávokon át megmaradnak). Az `--matcher flann` ~5000 jellemző felett térül meg; az `--averaging median` robusztus egy hibás felvétel esetén, az `inlier_weighted` az egyezések száma alapján súlyoz; az `--lock-scale` a legközelebbi forgatásra vetíti (méretarány nélkül), az `--lock-axis` egy transzlációs komponenst nullára állít; az `--mask` minden kamerára alkalmazható (kameránkénti beállításokhoz használja az `--per-cam-override`-et, pl. `--per-cam-override 214701292:method=phase`). Az `--rms-threshold-px` nem olyan kalibrációt menteni, amelynek újraprojektálási RMS-értéke meghaladja a küszöbértéket. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Egy igazított többsávos képkocka rögzítése. Az `--bit-depth` alapértelmezés szerint a kamerához igazodik; az `--no-crop` a teljes képkockát megtartja (feketével kitöltve); az `--interpolation` (alapértelmezett `linear`) és az `--border-mode`/`--border-value` (alapértelmezett: `constant`/0) a CPU-torzítást szabályozzák — a GPU-útvonal minden esetben bilineáris marad. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Stream-igazított többsávos keretek (ugyanazok a warp-szabályozók, mint az `align-apply`-nél). |
| `lattice align-info --profile PATH [--json]` | Profiladatok megjelenítése. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | A rétegek sorrendjének módosítása. |

#### Index / Növényzet-matematika

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Teljes jelzőkészlet: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI / NDRE / EVI / SAVI / GNDVI /…), `--formula EXPR`, `--channel SYM=BAND` (ismételhető), `--capture-level raw|debayered|radiance|reflectance|unknown` (felülírja a forrásban rögzített rögzítési szintet TIFF; alapértelmezés: a TIFF metaadatokból olvas), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Az `--live` esetén az igazító torzító gombok is érvényesek: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **Az `--channel` szimbólumoknál a kis- és nagybetűk megkülönböztetése fontos.** A szimbólumnak pontosan meg kell egyeznie a preset csatornáinak nevével (a presetek kisbetűket használnak, pl. NDVI = `red`,`nir` — ellenőrizze az `--list-presets`-et), a sávoldalnak pedig meg kell egyeznie az igazított sorozatban szereplő sávnévvel (vagy 0-tól kezdődő sávindexnek kell lennie offline módban). Az `--channel red=Red_660 --channel nir=NIR_850` működik; az `--channel RED=660` hibaüzenettel bukik: „`channel_map missing entries`”.

#### Állandó kapcsolatok (Smart-Prep, GUI-vel egyenértékű folyamat)

Ezek a parancsok a háttérpoolban nyitva tartják a kamerákat az CLI-hívások között.

| Alparancs | Cél |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Egy kamera hozzáadása a poolhoz (egy kamera, nem tömb). |
| `lattice cam-disconnect [--serial SN] [--all]` | Felszabadítás. |
| `lattice cam-list` | A poolban lévő kamerák listázása. |
| **`lattice array-connect`**|**Tartós, szinkronizált tömb csatlakoztatása (AZ ajánlott belépési pont).** A teljes GUI smart-prep folyamatot futtatja. |
| `lattice array-disconnect [--array-id ID] [--all]` | Tömb felszabadítása. |
| `lattice array-list` | A csatlakoztatott tömbök listája. |
| `lattice array-status [--array-id ID]` | Élő fps, PTP, utolsó hiba. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Egy szinkronizált felvétel az élő tömbből — Egyszeri / Folyamatos / Intervallum / Leggyorsabb. **Alapértelmezés szerint `all`** (kameraenként egy fájl az adott exporttípusra). A kihagyott kamerák (pl. a sugárzás/visszaverődésből kizárt RGB) az `Skipped: SN:<serial> (<reason>)` kóddal kerülnek jelentésre; a reflexióhoz használt DAQ-értékeket pedig a fájl mellett mentik el, és `DAQ: <path>` kóddal jelzik. Lásd [Rögzítési módok, rögzítők és offline újrafeldolgozás](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Az élő kombinált index nézet rögzítése videóba/GIF-be (megfigyelési minőségű; a kombinált adatfolyam megnyitása szükséges). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Magas képsebességű nyers Bayer-sorozat (elemzési minőségű; offline újrafeldolgozás szükséges). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | A mentett nyers sorozat újrafeldolgozása kalibrált videó(k)vá. |

##### `array-connect` beállítások

| Jelző | Alapértelmezett | Leírás |
| --- | --- | --- |
| `--serials SN1,SN2,…` | az összes LATTICE kamera automatikus felismerése (legalább 2 szükséges) | Az első sorozatszámú a MASTER. Ha elhagyják, az azonosítás a LATTICE (`TRI032*`) modellekre szűri a keresést, és mindet csatlakoztatja. |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO szinkronizáló vonal. |
| `--target-fps F` | auto | A MASTER trigger tűzgyakorisága. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | A szintválasztó felülírása. |
| `--wire-ceiling-mbps MB_PER_S` | automatikusan felismerve | **A gazdagép folyamatos sávszélesség-kerete, MB/s-ban — ez az érték határozza meg a teljes tömb allokációját.** Csökkentse ezt az értéket, ha a tömb GVSP-sérült kereteket jelent: az automatikus érték a hálózati kártya által jelzett kapcsolat sebességéből származik, amely túlbecsüli az USB-adaptereket, a keskeny PCIe sávokat és a leterhelt megosztott hálózati struktúrákat. A projekttömb rögzítési blokkjában, így egy újbóli megnyitás / CLI / SDK újracsatlakozás visszaállítja. Lásd [Tömb állapota](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | automatikus | Hardveres binning. |
| `--no-recommend` | ki | A hálózati elemzés lépésének kihagyása. |
| `--no-ptp` | ki | A PTP letiltása (ekkor a kamerák közötti időbélyegek **nem** összehasonlíthatók). |

### Smart-AE / Smart-Capture

A LATTICE-tömbök a csatlakozás után azonnal folyamatos AE-t futtatnak a háttérben, de egy frissen beállított jelenet konvergenciája egy pillanatig tart. Az `array-capture --smart` egy **beépített kényelmi funkció**: megvárja, amíg az AE minden kamerán stabilizálódik a rendszerben, majd elindítja a felvételt. Használja, ha a munkamenet közben vált jelenetet.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Az állandósodási szabály alapértelmezés szerint konzervatív: 5 másodperces időkorlát, 1,5 másodperces stabilitási ablak, ±5 %-os expozíciós szórás-tolerancia. Ha az automatizálástól eltérő viselkedésre van szükséged, az SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) oldalon állítsd be, ha az automatizálástól eltérő viselkedésre van szükséged.

### Felvétel-export szintek (az `all` alapértelmezett beállítás)

Ettől a kiadástól kezdve az `lattice capture`, az `lattice multi-capture` és az `lattice array-capture` **alapértelmezés szerint az `--processing all`-et** használják — exporttípusonként egy mentett fájl, amely minden kamerára vonatkozik, megegyezve a grafikus felhasználói felület „Capture All” (Minden rögzítése) viselkedésével. A szintek a következők:

| Szint | Kimenet | Vonatkozik |
| --- | --- | --- |
| `raw` | Egycsatornás Bayer (monokróm kamerák: az egyetlen sáv) közvetlenül az érzékelőből. | Minden kamera. |
| `debayered` | 3-csatornás BGR demosaic (mono kamerák: 1-csatornás szürkeárnyalatú). | Minden kamera. |
| `radiance` | float32 W/m²/sr/nm a teljes radiometrikus láncon keresztül. | Csak multispektrális (M3C/M3M) — **RGB-szűrős kamerák esetén kihagyva**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), Pix4D-kompatibilis. | Csak multispektrális, és **csak akkor, ha DAQ van hozzárendelve + a kamera kalibrálva van**; ellenkező esetben kihagyva. |
| `preview` / `display` | Teljes GUI-előnézeti lánc (CCM + WB + gamma a kamera profilja szerint). Az `lattice capture` ezt nevezi el `preview`-nek; az `array-capture`/`multi-capture` az `display`-et használja. | Minden kamera. |

Adjon meg egyetlen szintet, hogy csak azt mentse el (`--processing debayered`). Ha az `all`-et kéri, az adott kamerára nem vonatkozó szinteket a rendszer kihagyja (és jelenti), de nem jelenti hibaként — egy csatlakoztatlan vagy nem kalibrált kamera továbbra is megkapja az `raw` / `debayered` / `preview` kódot kap.

Bármely reflektancia-képkocka esetében a ténylegesen felhasznált DAQ lefelé irányuló mérési érték a kép mellé egy **`.daq`** kiegészítő fájlba kerül (így a felvétel később újra feldolgozható), és egy `DAQ:` sorban kerül jelentésre.

### Hogyan néz ki egy rögzítési mappa?

Minden exporttípus a **saját almappájába** kerül az `-o` alatt, így egy többszintű rögzítésben soha nem keverednek a típusok:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

Az `<ts>` a rögzítés időbélyege, az `<serial>` pedig a kamera sorozatszáma, így egy szinkronizált csoport
időbélyegét a kamerák között megosztják. **Figyeljünk az egyetlen eltérésre:** az `display` szint egy
`preview/` nevű mappában kerül tárolásra, míg maguk a fájlok az `_display` névvel szerepelnek – a mappa és az utótag
csak ezen a szinten tér el egymástól. Az ismeretlen szintek a saját nevüket viselő mappába kerülnek, és ha az almappa
nem hozható létre, a fájl a kimeneti gyökérkönyvtárba kerül, ahelyett, hogy elveszne.

**A rögzítések mappájának újrafeldolgozása:**irányítsa az `chloros-cli process` fájlt a**captures gyökérmappára**
(`output/`). Az `process` általában csak a megadott mappát importálja, de ha abban a mappában nincsenek
képek, viszont vannak almappái, akkor automatikusan lejut az almappákba — így a gyökérmappaszintű almappái és a
gyökér `.daq` is egy lépésben felvételre kerülnek. A felvétel minden szintje egyetlen képként kerül importálásra,
a többi szint pedig módként érhető el, nem pedig szintenként egy-egy képként.

A **szint almappa** közvetlen megnevezése (pl. `output/raw/`) szintén működik. Ezzel a gyökér
`.daq` hátramarad, ezért másolja át vagy mutasson rá a DAQ-leolvasásra, amikor radiometrikus
terméket állít elő az `raw/` mappából — ellenkező esetben az időbélyeg-egyeztetésnek nincs mihez viszonyítania.

**A feldolgozás mindig az `raw` mappából indul.** Minden egyes rögzítésen belül a nyers képkocka a feldolgozási folyamat forrása;
az `debayered`, `radiance`, `reflectance` és `preview` megjeleníthető módokként jelennek meg, de soha nem kerülnek
vissza a feldolgozási folyamatba. Egy származtatott termék újrafeldolgozása újra alkalmazná a vignettázást, a CCM-et és
a sugárzási számításokat, amelyek már be vannak építve a képpontjaiba, ezért az Chloros elutasítja ahelyett, hogy
kétszer feldolgozná. Két tudnivaló következmény:

- Az `index/` és `composite/` renderelések **soha** nem kerülnek feldolgozásra. Ezek kimenetek, nem rögzítések —
  egy „NDVI” LUT-renderelésnek nincs értelmes sugárzási értelmezése.
- Az `raw` **nélkül** exportált rögzítések mappája (pl. `array-capture --processing reflectance`)
  nem rendelkezik érvényes pipeline-forrással. Ezek a rögzítések normálisan importálhatók és megjeleníthetők, de az `process` kihagyja
  és ezt jelzi:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Ha valóban át kell nyomnod egy származtatott terméket – például egy olyan hub-munkamenetet, amelyet
  `demosaic` bekapcsolt állapotban rögzítettek, vagy egy régi mappát –, az `--input-level {raw,debayered,processed}` kényszeríti a bejegyzést
  és felülírja az átugrást. Ez a jelző egy szándékosan beépített menekülési út; az `auto` (az alapértelmezett)
  soha nem dolgozza fel azokat a rögzítéseket, amelyekhez nincs nyers adat.

### Kihagyott rögzítések vegyes szűrőjű sorozatokban

Ha egy sorozatban kevered az RGB-es és multispektrális kamerákat, az `array-capture --processing radiance` (vagy az `reflectance`) elmenti a multispektrális képkockákat, és **kihagyja** a RGB kamerákat — a Bayer-mátrixonkénti sugárzásérték szélessávú érzékelő esetében nem értelmezhető. Az CLI kimenet minden mentett fájlt (az exportszintjével együtt), minden `.daq` fájlt, és minden kihagyást kifejezetten kinyomtat, így a fájlok száma nem meglepő:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

A kihagyás-reason tokenek az `<level>-not-applicable-to-rgb-cam` mintát követik. A reflektancia az `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)` jelekkel is kihagyható, valamint az `dls-uncalibrated-band-<nm>` jelekkel is, ha a sáv túlnyomórészt a DAQ fényérzékelő radiometrikusan kalibrált tartományán (~374–974 nm) kívül esik — a forgalomban lévő SKU-k közül csak az F988, amely támogatja a reflexiós panel munkafolyamatot.

Használja az `--processing debayered` (vagy `display`) parancsot, hogy minden kamerát bevonjon a szűrőtípustól függetlenül, vagy az alapértelmezett `all` parancsot, hogy egy lépésben megkapja az összes alkalmazható szintet kameránként.

---

## Felvételi módok, rögzítők és offline újrafeldolgozás

Ezek mindegyike **tartós tömbön** működik (először futtassa az `array-connect` parancsot). Ezek a grafikus felhasználói felület (GUI) felvételi paneljét tükrözik.

### `array-capture` módok

Az `array-capture` egy egyetlen parancs, amely négy exponálási módot és egy sor exportálási kapcsolót tartalmaz:

| Mód | Jelző | Viselkedés |
| --- | --- | --- |
| **Egyszeri** *(alapértelmezett)* | (nincs) | Egy szinkronizált rögzítési csoport, majd kilépés. |
| **Folyamatos** | `--continuous` | Egymást követő futtatások az `Ctrl+C`, `--count N` vagy `--duration S` parancsokig. |
| **Intervallum** | `--interval S` | Egy átfutás minden `S` másodpercenként (az egyes átfutások kezdetétől számítva), ugyanazok a határok. |
| **Leggyorsabb** | `--fastest` | Csak nyers-csak + a hozzárendelt DAQ-leolvasás + a kombinált indexű kompozit; kihagyja a sugárzási/visszaverődési/megjelenítési számításokat, így a képkocka gyorsan megjelenik. Ez magában foglalja az `--processing raw --force-daq`-et. A mentett `.daq`-et később kalibrált termékekké dolgozza át. |

Exportálási kapcsolók (bármely móddal kombinálhatók; mindegyik a GUI/SDK végpontot használja):

| Jelző | Hatás |
| --- | --- |
| `--processing LEVEL` | Egyetlen exportszint, vagy `all` (alapértelmezett). |
| `--levels L1,L2,…` | Exporttípusok kifejezett részhalmaza (pl. `raw,radiance,reflectance`); **felülírja az `--processing`-et**. |
| `--aligned` / `--no-aligned` | Minden tag nem nyers exportját igazítja a tömb [igazítási profiljához](#alignment) (co-regisztrált). A nyers adatok nem kerülnek átalakításra, de a transzformációt a metaadatokban hordozzák. Ha a tömbnek nincs profilja, akkor visszatér az igazítatlan állapotra (figyelmeztetéssel). |
| `--index` / `--no-index` | A kameránkénti növényzet-index-átfedést, ha az be van állítva. Alapértelmezés: megjelenítése. |
| `--force-daq` | A hozzárendelt DAQ/DLS-érték mentése `.daq` sidecar fájlként akkor is, ha egyetlen kiválasztott szint sem igényli (pl. nyersfelvétel esetén is), így a képkockák offline módon újrafeldolgozhatók reflektancia/index értékekké. |
| `--smart` | Várja meg, amíg az AE minden kamerán stabilizálódik, mielőtt elindítaná a rögzítést (lásd [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | TIFF pixeltömörítés. `deflate` (alapértelmezett) = veszteségmentes zlib L1 + vízszintes prediktor, ~4,1 MB teljes felbontású képkockánként; `none` = tömörítetlen, ~5× gyorsabb írás ~6,3 MB/képkockánként — használja a maximális folyamatos sebesség elérése érdekében, ha a lemez kapacitása megengedi. Mindkettő veszteségmentes, és importáláskor azonos módon olvasható. |

> **EgyszeriTIFF + a folyamatos sebességű modell.**A felvételek**egy**TIFF-fájlba kerülnek írásra, amely tartalmazza a pixeleket, az XMP-t és az IFD0 gyártmány/modellt (teljes felbontású Mono12-en mért: 36 ms tömörítve / 6,5 ms tömörítetlenül, szemben a régi „írás, majd ExifTool-lal történő újraírás” módszer ~148 ms-os értékével); az egyetlen fennmaradó ExifTool-feladat (az EXIF al-IFD finomítása) egy aszinkron háttérfolyamaton fut, és egy képkocka akkor is kész és importálásra kész, ha ez a folyamat soha nem fut le. Ne feledje, hogy a DEFLATE tömörítés az Python GIL-t tartja fenn, így a tömörített írások**nem**párhuzamosodnak a kameránkénti írószálak között — a 8 kamerás, teljes felbontású, érzékelői sebességű (~10,4 fps) folyamatos rögzítéshez `--compression none`**és** NVMe-osztályú lemez szükséges (~500 MB/s folyamatos írási sebesség). Ugyanez a beállító gomb `compression` néven érhető el az `POST /api/camera/array/capture`-en.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — kombinált indexű videó/GIF (felügyeleti minőségű)

Rögzíti mindazt, amit az **élő kombinált index nézet** mutat egy `.avi`-re (és opcionálisan egy `.gif`-re). Mivel az élő kompozit jelből vesz mintát, a kombinált adatfolyamnak nyitva kell lennie (pl. a tömb előnézete látható a grafikus felhasználói felületen), hogy a képkockák beérkezzenek. 2 másodpercenként ellenőrzi az előrehaladást, és leáll az `--duration`-en, az `Ctrl+C`-en, vagy amikor a felvevő önmagától befejezi a működést.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Jelző | Alapértelmezett | Leírás |
| --- | --- | --- |
| `--array-id ID` | csak tömb | Cél tömb (hagyja ki, ha csak egy van csatlakoztatva). |
| `-o, --output DIR` | `output` | Kimeneti könyvtár (backend-helyi). |
| `--fps F` | `10` | Felvételi képkocka sebesség. |
| `--duration S` | Ctrl+C-ig | Automatikus leállítás `S` másodperc elteltével. |
| `--gif` | ki | Animált GIF-fájl írása is. |
| `--gif-only` | ki | Csak GIF-et írjon (nincs `.avi`). |

### `array-burst` — nyers-Bayer nagy képsebességű sorozatfelvétel (elemzési minőségű)

Közvetlenül olvassa be a felvételi ciklus szinkronizált csoportjának pufferét — **nincs szükség kalibrációs láncra, exiftoolra vagy élőképre** — így a kamera teljes felvételi sebességén fut. Raw képkockákat ír + képkockánkénti manifesztet + egy `.daq`-et minden egyes DLS-leolvasáshoz az `<output>/bursts/<base>/` alatt. Offline újrafeldolgozás (következő parancs), vagy adja meg az `--build` parancsot, hogy a leállás után azonnal elvégezze.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Jelző | Alapértelmezett | Leírás |
| --- | --- | --- |
| `--array-id ID` | csak tömb | Cél tömb. |
| `-o, --output DIR` | `output` | Kimeneti könyvtár (a burst az `<DIR>/bursts/<base>/`-be kerül). |
| `--duration S` | Ctrl+C-ig | Automatikus leállás `S` másodperc után. |
| `--max-frames N` | korlátlan | Automatikus leállás `N` nyers képkocka után. |
| `--build` | ki | Leállás után azonnal dolgozza fel újra a sorozatot (ugyanúgy, mint az `array-build-video`). |
| `--products …` | `combined:index` | Az `--build` beállítással: mely videó(k) készüljenek (lásd alább). |
| `--fps F` | `10` | Az `--build` használatával: a kimeneti videó képkockasebessége. |
| `--save-tiffs` | ki | `--build` használatával: képkockánként kalibrált TIFF-fájlokat is ment. |
| `--gif` | ki | `--build` használatával: animált GIF-fájlokat is ír. |

### `array-build-video` — mentett sorozat offline újrafeldolgozása

Időben összehangolja az egyes nyers képkockákat a legközelebbi mentett `.daq` értékkel, és **ugyanazon a sugárzási / visszaverődési / index láncon keresztül**, és egy vagy több videót renderel.

Az `--products` egy vesszővel elválasztott lista `kind:level` elemekből, ahol `kind` ∈ `per_cam` | `combined` és `level` ∈ `radiance` | `reflectance` | `index`. Egy önálló `level` (`kind:` nélkül) alapértelmezés szerint `per_cam`-re van állítva. Az alapértelmezett érték `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Jelző | Alapértelmezett | Leírás |
| --- | --- | --- |
| `--burst-dir DIR` | (kötelező) | A burst mappa elérési útja (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | `kind:level` lista, a fentihez hasonlóan. |
| `--fps F` | `10` | Kimeneti videó képkockasebessége (fps). |
| `--save-tiffs` | ki | A videó(k) mellett képkockánként kalibrált TIFF-fájlokat is ment |. |
| `--gif` | ki | Animált GIF-eket is írjon. |

> **Válassza ki a megfelelő rögzítőt.** `array-record` *megfigyelési minőségű* — a kijelzőn látható élő kompozíciót rögzíti, és ehhez nyitott adatfolyamra van szüksége. `array-burst` → `array-build-video` *elemzési minőségű* — teljes sebességgel menti a nyers érzékelőadatokat, majd utólag rekonstruálja a kalibrált sugárzási/visszaverődési/index videókat, élő kép megtekintése nélkül.

### Mono (M3M) egysávos kamerák

Az **M3M**sorozat a Bayer**M3C**monokróm változata: kameránként egy keskenysávú interferencia-szűrő (`M3M-<lens>-F<wavelength>`, pl. `M3M-L87-F685`), így az érzékelő**egyetlen szürkeárnyalatos sávot** szolgáltat, Bayer-mozaik nélkül. Nincs mit demozikálni, nincs csatornák közötti átvitel, amit szét kellene választani, és nincs fehér egyensúlyt sem kell beállítani — az egész RGB -kijelzős színfeldolgozási folyamat egyszerűen nem alkalmazandó.

Mit jelent ez az CLI esetében:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**felismerik a monokróm kamerát, és**egy soros üzenettel átugorják** a feldolgozást, ahelyett, hogy értelmetlen beállításokat alkalmaznának. Ugyanebben a munkamenetben az RGB /Bayer M3C kamerával továbbra is normálisan működnek.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** továbbra is működnek — a sugárzás és a visszaverődés *sávonkénti* radiometrikus térképek, és tökéletesen-egy sávra vonatkozóan. A monokróm képkockák **azonos** érzékelő-válaszmátrixot hordoznak (nincs 3×3-as keverés), így a sík érintetlenül halad át a kalibrációs számításon.
- **Egyetlen monokróm kamera nem képes növényzetindexet előállítani.**Az NDVI / NDRE / stb. legalább két sávot igényelnek (pl. Red + NIR). Ahhoz, hogy monokróm hardverből indexet kapjunk,**több**, különböző hullámhosszon működő M3M kamerát kell különböző hullámhosszokra irányítani, azokat egy többsávos képcsomagba összehangolni, majd *azt* indexelni:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` szimbólumoknak **pontosan**meg kell egyezniük** (a kis- és nagybetűk megkülönböztetése érvényes; az NDVI-esek kisbetűs `red`,`nir` — lásd `--list-presets`), és a sávoldali neveknek egy sávot kell jelölniük az igazított veremben (offline módban 0-tól kezdődő sávindexeket is elfogad, pl. `--channel red=0 --channel nir=1`).

A verem egészében a megkülönböztető jel a modell karakterláncban szereplő `M3M` token (ez soha nem jelenik meg egy `M3C` karakterláncban), amely a GUI-ban/SDK-ban `is_mono` néven jelenik meg.

---

## Gazdagép hálózati kártyájának beállítása és hangolása (LATTICE-rendszerek)

A LATTICE kamerák a gazdagép Ethernet-adapterén keresztül továbbítják a GVSP-t, ezért többkamerás rendszerek esetén az adapter **illesztőprogramja**és**fogadási gyűrű mérete** ugyanolyan fontos, mint a kapcsolat sebessége. A helytelen beállítások `FRAMES WILL DROP` / `Reduce ROI to enable` kapuként jelennek meg a Rendszerbeállítások panelen (valamint az `lattice network-analysis` / az SDK`analyze_array_network()`) hibaüzenetként jelennek meg, még akkor is, ha maguk a kamerák rendben vannak.

### USB 10GbE adapterek — Realtek RTL8157 („Realtek USB 10GbE Family Controller”)

| Elem | Szükséges érték | Miért fontos |
| --- | --- | --- |
| **Illesztőprogram-verzió**|**≥ v10.67 (2026. január)**, INF `rtump64x64sta.inf` | A régi**2016-os**illesztőprogram (v10.65, `rtump64x64.inf`) nem kezeli megfelelően az energiaellátás leállását és a**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**hibával a leállítás/újraindítás/alvó mód esetén. Az átmenet lefagy (~5 perces időtúllépés), a felhasználó kénytelen erőszakosan kikapcsolni a rendszert, és az ismételt nem megfelelő leállítások**megrongálják a WMI-tárat**(a PowerShell és az eszközök az `Invalid class` hibával kezdenek meghibásodni), és**megakadályozzák az USB-stack működését** a következő indításkor (a hálózati kártya nem aktiválódik; az USB-meghajtók nem sorolják fel magukat). Frissítsd a realtek.com webhelyről (vagy a dongle gyártójától), mielőtt a tiszta újraindításokra támaszkodnál. |
| **Fogadó pufferek**— kulcsszó: `ReceiveBufferLen` |**256**(az illesztőprogram maximuma) | A hálózati kártya RX-gyűrűje. Az illesztőprogram alapértelmezett**32**értéke csak ~0,26 MB használható gyűrűt hagy — ami túl kicsi egy többkamerás adatcsomaghoz —, így a tömbpanel `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` hibát jelez, és blokkolja a csatlakozást.**256**értéknél a gyűrű nagy (**~13,5 MB, a laboratóriumi 10GbE-s gazdagépen mért érték**), ami valódi tartalékot biztosít az RX-csatornának a többkamerás GVSP-burstokhoz. (Hogy egy adott konfiguráció ténylegesen *csatlakozik-e*, azt két ellenőrzés határozza meg — a **drain-aware**befogadási ellenőrzés és az**aggregált túljegyzés** ellenőrzés —, nem pedig a nyers sorozat és a gyűrű közötti összehasonlítás; lásd [Tömb fps és sorozat modell](#array-fps--burst-model).) |
| **Fogadó URB-k**— kulcsszó `PendingReceives` |**64** (max.) | Folyamatban lévő USB-kérésblokkok; a rohamok felszívásához a fogadó pufferekkel együtt növelendő. |
| **Jumbo Frame** — kulcsszó `*JumboPacket` | **9014** | Szükséges a 9000 bájtos GVSP csomagokhoz (6× kevesebb csomag/keret, mint 1500-nál). |

> ⚠️ **A hálózati kártya-illesztőprogram frissítése VISSZAÁLLÍTJA ezeket a speciális tulajdonságokat az alapértelmezett értékekre.**Az adapter illesztőprogramjának frissítése vagy cseréje után**újra kell alkalmazni** az `ReceiveBufferLen=256` és az `PendingReceives=64` beállításokat, különben a tömbpanel újra korlátozni fogja a forgalmat, még akkor is, ha „a hardverben semmi sem változott”. Ez a legfőbb oka annak, hogy egy korábban működő rendszer hirtelen nem hajlandó csatlakozni.

Alkalmazza **feljogosított** PowerShell-ből (cserélje ki a hálózati kártya nevére, pl. `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **Az `lattice network --fix` az USB 10 GbE hálózati kártyákat fedi le.** Ez most felismeri az adapter típusát, és beállítja a megfelelő receive-ring kulcsszót: `*ReceiveBuffers`→2048 a PCIe hálózati kártyákhoz (Intel I219 stb.), vagy `ReceiveBufferLen`→256 + `PendingReceives`→64 a Realtek **USB** 10GbE vezérlő esetében (amely nem teszi elérhetővé az `*ReceiveBuffers` értéket). A célértékek az egyes illesztőprogramok által jelentett maximális értékhez (`NumericParameterMaxValue`) vannak korlátozva, így soha nem írnak tartományon kívüli értéket. Futtassa **feljogosított** terminálból; mint bármelyik rendszerleíró adatbázis-alapú beállításnál, a módosítás az adapter újraindítása vagy a rendszer újraindítása után lép hatályba. A fenti manuális `Set-NetAdapterAdvancedProperty` parancsok továbbra is kiváló alternatívát jelentenek — azok újraindítás nélkül, azonnal hatályba lépnek (az adaptert újra kötik).

### Hálózati alapismeretek (minden LATTICE-kapcsolat)

- **Címzés:** link-local `169.254.0.0/16` (GigE Vision LLA). A gazdagép statikus `169.254.x.x/16` címet kap; a kamerák és a DAQ-E ugyanebben a tartományban osztják ki maguknak a címeket. Nincs szükség DHCP-re vagy átjáróra.
- **Csomagméret:**előnyben részesítse a jumbo (9000) méretet, de hagyja, hogy az automatikus felderítés határozza meg — minden csatlakozáskor újra méri a méretet, és egy GVSP-próbával már túllépi a kamera 1500 bájtos ICMP-korlátját, így jumbo méretet állít be, bárhol is bírja el a vezeték. Csak akkor állítsd be rögzítve az `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` értéket, ha jobban tudod, mint a próba, és inkább parancsonkénti beállítást válassz az állandó helyett: a rögzített beállítás kihagyja a próbát, így ha az útvonalvalójában nem képes 9000-et továbbítani,**minden** rögzítés időtúllépésbe kerül az `SC_ERR_TIMEOUT -1011` beállítással (lásd [Környezeti változók](#environment-variables)).
- **Az RX gyűrű mérete az `ReceiveBufferLen` értékével arányos:**az alapértelmezett `32` értéknél a használható gyűrű mérete ~0,26 MB (túl kicsi bármilyen többkamerás sorozatfelvételhez); a maximális `256` értéknél ez nagy (~13,5 MB, a laboratóriumi 10 GbE-s gazdagépen mért érték), ami valódi tartalékot biztosít. Hogy egy konfiguráció csatlakozik-e, azt a sávszélesség-figyelembe vevő belépési ellenőrzés**és** az alábbi összesített túljegyzés-ellenőrzés határozza meg – nem pedig a nyers sorozat és a gyűrű közötti összehasonlítás.

### Tömb fps és sorozatmodell

Hogyan kell értelmezni a „Array Settings” panelt (és az `lattice analyze-array` / a „SDK” `analyze_array_network` elemét):

- **A sorozatfelvétel kameránként, az egyes kamerák tényleges pixelformátumában kerül összeadásra.**A**M3M**kamerák**Mono12 (2 B/px)**adatfolyamot adnak;**M3C**Bayer kamerák 8- vagy 12-bites adatfolyamot adnak (a TRI032S csendben BayerRG12-t ad ki, még akkor is, ha BayerRG8-at kérnek). Így egy 4 kamerás, teljes felbontású képkocka mérete**~12,6 MB, ha mind 8 bites, de ~25 MB, ha három 12 bites mono kamera van**. A vetítés a modellből (azonosító gyorsítótárból) határozza meg az egyes kamerák formátumát, így a burst pontosan megfelel annak, amit a kábel ténylegesen továbbít — nem pedig egy általános „BayerRG8” feltételezésnek.
- **Egy USB Ethernet-adapter sebessége 200 MB/s-ra van korlátozva, függetlenül a gyártó által megadott értéktől.** Az a hatékonysági táblázat, amely a kapcsolati sebességet tartós értékre konvertálja, a PCIe-ből származik; egy USB NIC az *Ethernet* kapcsolati sebességét hirdeti, de az USB-busz és az illesztőprogram korlátozza. Egy USB 10GbE dongle korábban ~1063 MB/s „tartós” sebességet ért el – egy olyan értéket, amelyet soha nem teszteltek –, és az ebből eredő ütemezés a képkockák 6–18 %-át megrongálta, miközben továbbra is megfelelő cél-fps-t jelentett. Az USB-n keresztül csatlakoztatott hálózati kártyák sebessége jelenleg **200 MB/s**-re van korlátozva -ra (a korlátot a busz jelenti, így ez nem skálázódik a névleges értékkel; egy USB 1 GbE adapter ~80 MB/s sebességet ér el, és ez nem érinti). Az `wire_ceiling_source` a képességi rekordban ezt szóban is megfogalmazza, az `nic_is_usb` jelzi ezt. Mindkét esetben felülírható az `--wire-ceiling-mbps` beállítással.
- **A befogadási képesség a kiadási sávszélességet veszi figyelembe, nem pedig a teljes burst-ring ellentétet.** Egy egyidejű burstnak csak a *tranziens backlog*-ba kell beleférnie = `max(0, Σ per-cam arrival − host drain) × emit_window`, nem a teljes burst-ba. Gyors gazdagép / lassú kamera felépítésű hálózat esetén (például **PCIe**10G-s gazdagép + 4× 1 GbE-s kamerák: érkezés ≈ 320 MB/s, kiürítés ≈ 1063 MB/s) esetén a gazdagép gyorsabban üríti ki az adatokat, mint ahogy a kamerák feltöltik, a backlog ≈ 0, így a teljesfelbontású szimulációs kibocsátás**bejut**, annak ellenére, hogy a 25 MB-os adatcsomag meghaladja a 13,5 MB-os gyűrűt. Ha ugyanezt a négy kamerát egy**USB**10GbE adapter mögé helyezzük, a kiadási sebesség 200 MB/s lesz, nem 1063 — az érkezés meghaladja azt, és a veszteség nem alacsonyabb képkockasebességként, hanem sérült képkockákként jelenik meg. Egy 1 GbE gazdagépen a kamerák 31,25 MB/s-os DLThr alsó határértéke miatt az érkező adatmennyiség meghaladja a kiadási sebességet → a rendszer helyesen**blokkol**(**ez**a blokkolási osztály esetében csökkentse a ROI-t, vagy használjon ≥ 2-es binninget). A beengedés a**két** csatlakozási kapu egyike — a másik az alábbi összesített túljegyzési ellenőrzés.
- **A becsült fps egy konzervatív, soros lekérésre vonatkozó felső határ.**A gazdagép lekérési ciklusa jelenleg minden kamera pufferét**sorosan**(~egy-egy kibocsátási ablak kameránként), így a ciklust az `max(readout+emit, N × emit)` határolja, ahol a kameránkénti kibocsátás a kamera**hozzáférési kapcsolatára**van korlátozva (1 GbE ≈ 80 MB/s), nem pedig a gazdagép felfelé irányuló kapcsolatához. Egy 4 kamerás, teljes felbontású, 12 bites rendszer esetében ez**~2,8 fps**-t jelent, ami megegyezik a mért ~2,7–3,0 értékkel. Az fps szándékosan**expozíciótól független**, így gyenge megvilágítású jelenetekben a tényleges érték az expozíció hosszabbodásával kissé a felső határ alá csökkenhet. A soros adatlekérés jelenti a valódi fps-korlátozót; ennek párhuzamosítása az egyetlen adatküldési sebesség felé emelné a felső határt.
- **Az összesített túljegyzés a kapcsolat létrehozásának komoly akadálya.**A kameránkénti sávszélesség-allokáció alsó határa**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), így ha az alsó határ elérődik, az összesített igény (`per_cam × N`) meghaladhatja az**ütközésmentes vezetékes felső határt**(`sustained × sim_emit_factor`). Gyakorlati teljes felbontású felső határok 1 GbE-n:**6 kamera 1500 MTU-val, 9 kamera jumbo keretekkel**. Ez a felső határ kizárólag a vezeték és az alsó határ tulajdonsága —**független a keretmérettől**, így**a binning és a kisebb ROI NEM segít** (ezek a *keretenkénti* bájtokat csökkentik, nem pedig a GevSCPD által szabályozott *másodpercenkénti* bájtokat); az egyetlen megoldás a kamerák számának csökkentése, a végpontok közötti jumbo keretek használata vagy egy gyorsabb hálózati kártya. A tünet a GVSP-csomagvesztés lenne, nem pedig egy fokozatos fps-csökkenés, ezért az `analyze-array` nullára állítja az elérhető fps-értékeket, és kiírja az `**OVER-SUBSCRIBED**` kódot, míg az `array-connect` kód rögzített felbontás mellett **megtagadja a csatlakozást** (a walk-down egyébként a képkockákat alacsonyabb szintre sorolja, ami szintén nem szünteti meg ezt a blokktípust). Az `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` a kapcsolódási megtagadást hangos figyelmeztetéssé minősíti vissza a tesztelési munkákhoz — lásd [Környezeti változók](#environment-variables).

### A tömb állapota — melyik alrendszer veszít képkockákat

Egy csatlakoztatott tömb `GET /api/camera/array/<array_id>/capability` kódja egy élő
`health` blokkot hordoz, amelyet egy gördülő **10 másodperces** ablakban értékelnek újra. A keretveszteséget
két, egymással ellentétes javítást igénylő okra bontja, ahelyett, hogy egyetlen „hiányos”
arányt jelentené, amely egyik okot sem nevezi meg:

| Mező | Mit jelent | Melyik alrendszer |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (sorozatszámonként) | A keret **megérkezett, de szerkezetileg hibás volt**— GVSP csomagvesztés. |**Hálózat**: sávszélesség-keret, ütemezés, NIC RX gyűrű, MTU |
| `never_arrived_rate_pct` (sorozatszámonként) | A keret **egyáltalán nem érkezett meg**— a kamera nem indult el, vagy semmi sem hagyta el azt. |**Trigger / szinkronizálás**: M8 kábel, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Az egyes kamerák legrosszabb aránya. | — |
| `per_cam_rate_pct` | Kameránkénti összesített hiányzó arány (mindkét ok együtt). | — |
| `stable_for_seconds` | Mennyi ideig maradt minden kamera 0,01 % alatt. | — |

5 % felett a háttérrendszer egy `[array-health <id>] WARN` sort rögzít, amely megnevezi a változást — az
első túllépéskor, a súlyossági sáv változásakor, percenként egyszer, amíg fennáll, és egyszer, amikor
megszűnik. A hibás fele kameránként és
ok szerint az első találatkor az `[gvsp-corrupt <SN>]`-et írja ki, majd 60 másodpercenként egy összesítést. Minden értékelés továbbra is bekerül a háttérrendszer naplófájljába;
a számlálók minden puffer esetén továbbhaladnak, függetlenül attól, hogy mi kerül kinyomtatásra.

Ugyanez a rekord jelenti azt a számot, amelyhez az egész allokáció kapcsolódik:

| Mező | Mit jelent |
| --- | --- |
| `wire_ceiling_mbps` | A gazdagép érvényben lévő, fenntartott hálózati sávszélessége, MB/s. |
| `wire_ceiling_source` | Honnan származik ez az érték, szavakkal kifejezve — pl. `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` vagy `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, ha az `--wire-ceiling-mbps` (vagy a grafikus felhasználói felület **Vezeték-kapacitás** mezője) ezt állította be. |
| `nic_is_usb` | `true` USB Ethernet-adapter esetén — lásd a fenti 200 MB/s-os korlátot. |

**Értelmezés:** nem nulla `gvsp_corrupt_rate_pct` érték, miközben az `never_arrived_rate_pct` 0
azt jelenti, hogy a kiváltás és a kábelszinkronizálás tökéletes, és a veszteség 100%-a a hálózati
— csökkentse az `--wire-ceiling-mbps` értéket, majd csatlakoztassa újra. Az ellenkező mintázat inkább a
szinkronkábelre vagy a triggervezetékre utal.

> **Az `--target-fps` nem a sérült keretek szabályozója.** A GevSCPD ütemzés
> egyszer kerül beírásra a csatlakozáskor, így a triggerfrekvencia csökkentése a kitöltési arányt változtatja meg, nem pedig az
> egyidejű kibocsátási burst-sebességet. A mért 5-szeres igénycsökkentés nem hozott javulást;
> a vezetékes felső határ 240-ről 200 MB/s-ra történő csökkentése ugyanazon berendezésnél a 10,4 %-ról
> 0,00 %-ra csökkentette a sérült keretek arányát.

> **A folyamat közbeni automatikus méretcsökkentés a TRI032S firmware-en nem elérhető.** Egy futó tömb
> ezt önállóan nem tudja kijavítani; válassza le, majd csatlakoztassa újra, hogy a csatlakozási idő-választó
> az új felső határ értékével újratervezhessen.

### Tünet → megoldás

| Tünet (Tömbbeállítások / csatlakozás / `analyze_array_network`) | Ok | Megoldás |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | Az `ReceiveBufferLen` értéke 32-re áll vissza (általában illesztőprogram-frissítés után) | Állítsa be az `ReceiveBufferLen` értéket →256-ra, `PendingReceives`→64 értékre; nyissa meg újra a panelt (indítsa újra a háttérprogramot, ha az a régi gyűrűméretet tárolta a gyorsítótárban) |
| Az újraindítás/leállítás megakad; később `Invalid class` WMI-hibák, a hálózati kártya nem aktiválódik, az USB-meghajtók hiányoznak | Régi, 2016-os Realtek USB 10GbE illesztőprogram → BSOD `0x9F` → kényszerített kikapcsolások | Frissítse az adapter illesztőprogramját ≥ v10.67 (2026) verzióra, majd a fenti fogadógyűrű-beállítások újbóli alkalmazása |
| A csatlakozás sikeres, de natívnál alacsonyabb felbontást ad vissza | A Smart-Prep automatikusan kicsinyítette a keretet, hogy illeszkedjen a vezetékhez | A kapcsolat frissítése / fogadja el a zsugorítást / `--force-tier slip-emit-and-capture` |
| A tömb megfelelő cél-fps-t jelez, de annak csak egy töredékét szolgáltatja; `health.gvsp_corrupt_rate_pct` nem nulla, `never_arrived_rate_pct` 0 | A gazdagép által becsült vezeték-kapacitása túlbecsüli a ténylegesen fenntartható értéket (jellemző USB-Ethernet-adapterre, keskeny PCIe sávra, vagy megosztott hálózati struktúrában) | Csatlakozzon újra alacsonyabb `--wire-ceiling-mbps` értékkel, és ellenőrizze újra a működési állapotot. **Nem** `--target-fps` — A GevSCPD ütemezése a csatlakozáskor rögzül |
| Hiányzó kamerák a közzétett csoportokból; `health.never_arrived_rate_pct` nem nulla, `gvsp_corrupt_rate_pct` 0 | Trigger/szinkronizálási útvonal — a kamerák nem aktiválódnak, nem hálózati probléma | Ellenőrizze az M8 szinkronkábelet és az `--line` értéket; győződjön meg arról, hogy minden tag be van kapcsolva (`TriggerMode=On`) |
| Az `**OVER-SUBSCRIBED**` / `Wire budget` értéket túllépte az `analyze-array`, vagy csatlakozási elutasítás rögzített felbontással (`array over-subscribes the wire`) | A kameránkénti összesített igény (8 MB/s alsó határ × N kamera) meghaladja az ütközésmentes vezeték felső határát — 6 kamera teljes felbontásban 1 GbE-n @1500 MTU-nál, 9 jumbo kerettel | Kevesebb kamera, végpontok közötti jumbo keretek, vagy gyorsabb hálózati kártya. **A ROI/binning NEM segít** (a felső határ független a keretmérettől). Az `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` felülírja a tesztpadon (elfogadja a csomagvesztést) |

---

## `chloros-cli daq`

Spectral-érzékelő parancsok. Két osztály:
- **`pool-*`**— vékony HTTP kliensek, amelyek a háttérrendszer állandó poolján keresztül vezérlik az érzékelőt.**Ez a támogatott útvonal, és az egyetlen, amely a szállított CLI-ban megtalálható.** A háttérrendszer kezeli az adatátvitelt, így a grafikus felület, valamint az CLI és SDK szkriptek mind egy aktív kezelőt használnak, ahelyett, hogy a soros portért versengenének.
- **Minden egyéb**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — közvetlen hardverhozzáférés, a teljesség kedvéért az alábbiakban dokumentálva. Ezekhez az `daq` Python csomag szükséges, amely**nem szerepel egyetlen szállított artefaktumban sem**: a lefordított CLI kizárja azt (az `scripts/Build-CLI.ps1` beállítja az `--nofollow-import-to=daq`-et, a transportok pedig az `pyserial` / `bleak` / `zeroconf`-et is tartalmazza), és a PyPI SDK csomag sem tartalmazza. Ezek csak forráskód-lekérésből futtathatók, ezért inkább a MAPIR belső fejlesztési útvonalaként kezelje őket, mintsem elérhetőként.
- **`discover` / `list`** a kettő között helyezkednek el: forráskódból történő letöltés esetén közvetlen hardverparancsok, de a kiadott verzióban az `pool-discover`-re váltanak, és a háttérprogram végzi el a szkennelést. Így a szkennelés mindenhol működik – ami azért fontos, mert ez az egyetlen módja annak, hogy megismerjük egy DAQ-M BLE MAC-címét.

> **`chloros-cli daq --help`** (és az `-h` / `help`) felsorolja az `pool-*` alparancsokat — a súgót szándékosan a pool klienshez irányítják, így az a ténylegesen futó parancsokat tükrözi. Ha egy kiadott verzión közvetlen hardver-alparancsot hív meg, a program a hiányzó csomagot megnevező, kifejezett hibajelentéssel lép ki, és visszavezet az `pool-*` parancsra; semmi sem bukik meg észrevétlenül. (Az `discover` / `list` kivételt képeznek — ezek átirányítanak az `pool-discover`-re, és egyszerűen működnek.)
>
> **Minden, amire az ügyfélnek szüksége van, elérhető az `pool-*`-en keresztül** — csatlakozás, adatfolyam, kalibrált `.daq` fájlok rögzítése és érzékelőprofilok cseréje. A DAQ-t az Python weboldalról is vezérelheti az `chloros_sdk.connect_daq_sensor()` segítségével is vezérelhető, amely ugyanazt a közös útvonalat használja.

### A DAQ-érzékelő első csatlakozásának munkafolyamata

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### `pool-*` hivatkozás

| Alparancs | Cél |
| --- | --- |
| `daq pool-connect` (smart-detect) | Érzékelő megnyitása a háttérpoolban. |
| `daq pool-connect --port PORT` | DAQ-U egy adott soros porton. |
| `daq pool-connect --ble` | DAQ-M BLE-n keresztül, automatikus MAC-kereséssel. |
| `daq pool-connect --mac MAC` | DAQ-M BLE-n keresztül egy ismert MAC-címen (magában foglalja az `--ble` parancsot). |
| `daq pool-connect --eth-host HOST` | DAQ-E Etherneten keresztül egy ismert gazdagépen. |
| `daq pool-connect --eth` | DAQ-E Ethernet-en keresztül, a gazdagép automatikusan felderítve (mDNS + ARP tartalék; működik üres ARP-gyorsítótárból is a Windows és Linux oldalakon). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Integrációs ablak / AE állapot beállítása. |
| `daq pool-connect --no-stream` | Csatlakozás, de az adatátvitel még nem indul el (folytatás az `pool-stream --start` parancsával). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Cap-korrekciós profil. A háttérben az alapértelmezett érték `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Minden transzportot átvizsgál a csatlakoztatható érzékelők után, anélkül, hogy csatlakozna hozzájuk. **Így lehet megtalálni egy DAQ-M BLE MAC-címét.** Az `daq discover` / `daq list` parancsok a gyári kiadásokban automatikusan ide irányítanak. A poolban már nyitva lévő érzékelők nem szerepelnek a listában — a csatlakoztatott DAQ-M leállítja az adását —, ezért azokhoz használja az `pool-list` parancsot. |
| `daq pool-list` | Minden érzékelő megjelenítése a háttérpoolban. |
| `daq pool-disconnect --sensor-id ID [--all]` | Felszabadítás. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | A legfrissebb N spektrumkeret. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Folytatás / szüneteltetés. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | .daq felvétel indítása / leállítása. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | A cap-korrekciós profil cseréje futásidőben. |

### Közvetlen hardver-alparancsok (csak a forráskódból letölthető – a kiadott verziókban nem szerepelnek)

> A teljesség kedvéért szerepelnek a listában. Ezekhez szükség van az `daq` Python csomagra, valamint az `pyserial` / `bleak` / `zeroconf` csomagokat is igénylik, amelyek közül egyik sem található meg a lefordított CLI vagy a PyPI SDK verziókban — ezek kizárólag a MAPIR forráskód letöltéséből futtathatók. **Ha a Chloros kiadott verzióját használja, akkor inkább a fenti `pool-*` parancsokat használd**; ezek lefedik a csatlakozást, a streamelést, a felvételt és a kap kiválasztását.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Megnyit, csatlakozik és vezérli egy mentett Chloros projektet (egy mappát, amely tartalmazza az `cameras.json` + `sensors.json` + `project.json` fájlokat). Minden a háttérrendszeren keresztül történik, így a grafikus felület és az CLI azonos hardverállapotot eredményez.

### Alparancsok áttekintése

| Alparancs | Cél |
| --- | --- |
| `project open PATH` | A projekt eszközlistájának kinyomtatása (kamerák, rendszerek, érzékelők). |
| `project devices PATH [--reconnect]` | A felderítés listázása vagy újrafuttatása. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Minden mentett kamera / sorozat / érzékelő csatlakoztatása. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Egyetlen felvétel egy megnevezett kamerából vagy sorozatból. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | N képkockás sorozatfelvétel egy megnevezett kamerából vagy kameracsoportból (`-n/--count` alapértelmezett érték 5; `-i/--interval` képkockák közötti idő másodpercben, alapértelmezett érték 0). A kameracsoportos sorozatfelvételek eltávolítják az ismétlődő szinkronizált csoportokat (elavulás-ellenőrző), így egy részciklusú kameracsoport nem adhat vissza egy képkockából N másolatot; az iterációk eredményeit kinyomtatja. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Adatfolyam lemezre mentése háttérfeladat segítségével. `--poll-interval` = másodpercek az `/stats` lekérdezések között (alapértelmezett: 2,0). |
| `project sensor read PATH NAME [--json]` | Legfrissebb spektrumkép. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | .daq fájl rögzítése. |
| `project run PATH RECIPE.yaml` | YAML/JSON rögzítési receptet. Az `--dry-run` futtatás nélkül ellenőrzi az érvényességet. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Számítsa ki a tömb igazítását — lásd [az alábbi jelzőtáblázatot](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Az aktuális igazítási profil kinyomtatása. |
| `project align clear PATH NAME` | A gyorsítótárban tárolt profil törlése. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Egy szolga transzformációjának eltolása. |
| `project align export PATH NAME --to FILE` | A profil mentése a JSON fájlba. |
| `project align import PATH NAME --from FILE [--no-validate]` | Mentett profil betöltése. |

#### `project align calibrate` Opciók

| Jelző | Alapértelmezett | Leírás |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Igazítási módszer. **Ezek a megnevezések eltérnek az `lattice align-calibrate`** parancstól, amely az `orb` / `akaze` / `phase` rövidítéseket fogadja el; a két parancs ezen a jelzőn nem cserélhetők fel egymással. |
| `--model {translation, rigid, affine, homography}` | `affine` | A modell illesztése. |
| `--frames N` | `1` | Szinkronizált képkockák átlagolása átlaghoz. |
| `--reference SN` | a fő | Referencia kamera sorozatszáma; minden más elemet erre torzítanak. |
| `--max-features N` | `5000` | ORB jellemzőszám-korlát. |
| `--ratio-threshold F` | `0.75` | Lowe-féle arányteszt. |
| `--ransac-threshold-px F` | `3.0` | RANSAC belső pontok küszöbértéke. |
| `--min-matches N` | `15` | **Minőségi küszöb** — elutasítja a megoldást, ha az inlier-egyezések száma alacsonyabb ennél. |
| `--max-reproj-err-px F` | `4.0` | **Minőségi küszöb** — elutasítja a megoldást, ha az RMS-újraprojektálási hiba meghaladja ezt az értéket. |
| `--checkerboard RxC` | — | Az `--method checkerboard` táblájának geometriája, pl. `9x6`. |
| `--name PROFILE` | üres | A mentett JSON fájlba ágyazott profilnév. **Nem a tömb neve** — azaz a pozicionális `NAME`. |

A két minőségi ellenőrzés az oka annak, hogy egy kalibrálás sikeresen megoldódhat, de mégis
elutasíthatja a mentést: egy profil, amelyik bármelyikben is megbukik, minden
későbbi rögzítést csendben hibásan regisztrálna, ezért elutasításra kerül, ahelyett, hogy mentésre kerülne.

### Példák

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### Recept DSL

Az `project run RECIPE.yaml` elfogad egy YAML- vagy JSON fájlt, amely egy műveletsorozatot ír le:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Támogatott műveletek: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. Az `burst` művelet az `name` (kötelező), az `count` (alapértelmezett érték: 5), az `interval` (másodpercben, alapértelmezés szerint 0), az `output`, az `format` és az `settings` (ugyanaz a kameránkénti beállítási forma, mint az `apply`-nél); a tömbös sorozatfelvételek ugyanazt a frissen szinkronizált csoportos őrprogramot használják, mint az `project burst`.

Futtassa:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Környezeti változók

| Változó | Hatás |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Felülírja a háttérprogramot (URL) (alapértelmezett: `http://127.0.0.1:5000`) — **csak az `lattice`, `project` és az `daq pool-*` parancscsaládok támogatják.** Az alapvető parancsok (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) az `http://127.0.0.1:<port>`-et állítják be, és figyelmen kívül hagyják ezt a változót (az IPv4-literal megkerüli az Windows `localhost`→`::1` ~2 kérésenkénti büntetést), így mindig a helyi gépet célozzák meg. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` a tömb túlterhelés miatti csatlakozási elutasítást (összesített, kameránkénti igény &gt; ütközésbiztos vezeték-felső határ az `pin_resolution`-szel) hangos figyelmeztetéssel és folytatásra való engedélyezésre, elfogadva a GVSP csomagvesztést. Kizárólag tesztcélra — lásd [Tömb fps és burst modell](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Magát az CLI állítja be; utasítja a háttérrendszert a párhuzamos feldolgozás engedélyezésére. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` kihagyja a GVSP-visszaesési próbát (csak ICMP-eredmények esetén). **Ez kikapcsolja a jumbo-t, nem csupán elnémítja a naplóbejegyzéseket** — a kamera minden útvonalon legfeljebb 1500-ig válaszol a DF pingekre, így ez a próba az egyetlen, ami képes észlelni a jumbo-t. Kameránként és csatlakozásonként ~1 másodpercet takarít meg; költsége ~1,45-szeres vezetékes felső határhoz képest, ha a hálózat *képes* lett volna jumbo-csomagokat továbbítani. A SDK figyelmeztet, amikor beállítod. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | A GVSP-csomagméretet N bájtra rögzíti; a tesztelést teljesen kihagyja. A végleges beállítás helyett inkább a parancsonkénti beállítást (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) beállítást az állandó beállítás helyett: a rögzített méret nem alkalmazkodik az előtte lévő hálózathoz, és ha 9000-et rögzítünk egy olyan útvonalon, amely nem képes jumbo-csomagokat továbbítani, **minden** rögzítés időtúllépést eredményez az `SC_ERR_TIMEOUT -1011` hibával. |
| `TMPDIR` (Linux) | Felülírja a Nuitka onefile-kivonási könyvtárát. Az CLI automatikusan az `/mnt/ssd/tmp` parancsot használja, ha az meg van adva. |

---

## Kilépési kódok

| Kód | Jelentés |
| --- | --- |
| `0` | Siker. |
| `1` | Általános hiba (a legtöbb alparancs-hiba). |
| `2` | Argumentumhiba. |
| `130` | Ctrl+C billentyűkombinációval megszakítva. |

---

## Hibaelhárítási tippek

- **„Bejelentkezés szükséges”** → Futtassa egyszer az `chloros-cli login EMAIL PASSWORD` parancsot ezen a gépen.
- **„A háttérprogram nem érhető el”** → Indítsa el az Chloros asztali alkalmazást, vagy futtassa közvetlenül a háttérprogramot (`chloros-backend`), vagy távoli esetben ellenőrizze az `CHLOROS_BACKEND_URL` fájlt.
- **Az `lattice` parancsok hibaüzenettel buknak: „LATTICE kamera illesztőprogramok nem találhatók”** → Az Arena SDK futtatókörnyezet nincs telepítve; az CLI az `win32api` csomaggal érkezik a Windows oldalon, de a C futásidejű könyvtár a grafikus telepítő része.
- **Az Array connect / Array Settings ablakban a „FRAMES WILL DROP” vagy „Reduce ROI to enable”** üzenetet → A gazdagép hálózati kártyájának vételi gyűrűje túl kicsi (a hálózati kártya illesztőprogram frissítése után általában 32-re áll vissza). Lásd [A gazdagép hálózati kártyájának beállítása és hangolása](#host-nic-setup--tuning-lattice-arrays) — állítsa be az `ReceiveBufferLen=256`, `PendingReceives=64` értékre.
- **A gép újraindításkor/leállításkor lefagy, majd WMI `Invalid class` / a hálózati kártya nem engedélyezhető / hiányoznak az USB-meghajtók** → Elavult USB 10GbE adapter-illesztőprogram okozza az `DRIVER_POWER_STATE_FAILURE` hibát (BSOD `0x9F`). Frissítse az adapter illesztőprogramját — lásd [Host NIC beállítás és finomhangolás](#host-nic-setup--tuning-lattice-arrays) című részt.
- **Jetson swap figyelmeztetés** → Adjon hozzá fájlalapú swap-ot; az CLI oldalon megtalálhatók a pontos `fallocate` / `swapon` parancsok.
- **Hiányoznak a DAQ közvetlen parancsai** → Várható: a szállított `chloros-cli` szándékosan kizárja az `daq` csomagot, így csak az `pool-*` található meg (a PyPI SDK sem tartalmazza). Használja az `pool-*`-et, amely a háttérrendszeren keresztül ugyanazt az érzékelőt vezérli, vagy az Python webhelyről letölthető `chloros_sdk.connect_daq_sensor()`-et.

---

## Lásd még

- [Python SDK Referencia](sdk-reference.md) — minden CLI parancs programozási megfelelője.
- [DAQ érzékelő-útmutató](../daq/README.md) — érzékelő-specifikus bekötés + kalibrálás.
- Online dokumentáció: `https://mapir.gitbook.io/chloros/cli`
