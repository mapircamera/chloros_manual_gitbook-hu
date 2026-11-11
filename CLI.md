# CLI: Parancssor

> **Teljes referencia:**A [CLI-referencia](reference/cli-reference.md)**minden alparancs minden jelzőjét** dokumentálja, és AI-asszisztensekhez van optimalizálva — illessze be az URL-jét az asszisztensébe, és kérjen egy működő parancsot: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Tipp az AI-eszközökhöz:** a kézikönyv bármely oldala nyers Markdown formátumban elérhető, ha az URL-hez hozzáfűzzük az `.md` szót (pl. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), az `https://mapir.gitbook.io/chloros/llms.txt` pedig az egész kézikönyvet indexeli az LLM-ek számára.

> **Szeretnéd, ha az asszisztensed maga futtatná a Chloros-t?** Az `chloros-mcp`, egy Python SDK-szel telepített MCP-kiszolgáló, együttműködik a Claude Desktop, a Claude Code, a Cursor és más MCP-kliensekkel: lásd [A Chloros használata AI-asszisztensekkel](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## Mi az a CLI
?

Az `chloros-cli` a Chloros
asztali alkalmazás által használt feldolgozó motor parancssori felülete. Ez egy vékony HTTP
-kliens a Chloros
-háttérrendszer felett (egy helyi szerver az `127.0.0.1:5000`-en) — a legtöbb parancs automatikusan elindítja a háttérrendszert, így egy szkriptnek csupán egy `chloros-cli process …`-hívásra van szüksége. A háttér ezután tovább fut a következő parancsra várva: az `chloros-cli logout` parancsra leáll (kivéve, ha az alkalmazás nyitva van, vagy futás vagy összeállítás van folyamatban), az `chloros-cli backend stop` parancsra, vagy önmagától 30 perc elteltével, ha nincs kliens, futás vagy csatlakoztatott eszköz. Az `chloros-cli backend status` parancs jelzi ezt.

**Windows
10/11 (x64)**és**Linux
(x86_64, valamint NVIDIA Jetson arm64 a JetPack 6-on)** rendszereken fut, bármely terminálon, GUI nélkül. Ellenőrizze a telepítést a következő paranccsal:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

A parancscsaládok áttekintése:

* **Feldolgozás és fiók** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 nyelv — lásd [Támogatott nyelvek](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (csak Linux
/Jetson)
* **Élő hardver** — `lattice` (LATTICE kamera vezérlés, több mint 45 alparancs), `daq pool-*` (DAQ fényérzékelők), `time-sync` (PTP)
* **Automatizálás** — `project` (Chloros
-projekt vezérlése headless módban, beleértve a YAML-rögzítési receptjeit is)

Érdemes tudni a következő globális beállításokat: `--port N` (háttérport, alapértelmezett: `5000`), `-v/--verbose`, `--restart` (a háttérprogram kényszerű újraindítása), `--backend-exe PATH`. A teljes listát lásd a [CLI
-referenciában](reference/cli-reference.md).

***

## Telepítés

A CLI
**minden platformon a Chloros
-telepítőben** található — nincs külön CLI
-letöltés. A telepítőt a [Letöltés](download.md) oldalról töltheti le.

### Windows


A telepítő a következő helyre helyezi a CLI
-t:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

és egyszer hozzáadja ezt a mappát a rendszer `PATH`-éhez (a frissítés eltávolítja a korábbi telepítők által hagyott duplikált bejegyzéseket is) — a telepítés után **nyisson meg egy új terminált**, hogy a rendszer felismerje a frissített `PATH`-et. A telepítő emellett elindító szkripteket (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) is elhelyez a telepítési gyökérkönyvtárban, valamint egy**Chloros
CLI
** Start-menü parancsikont is, amelyek mindegyike megnyit egy terminált, ahol az `chloros-cli` már használatra kész állapotban van.

### Linux


Telepítse az architektúrájához megfelelő `.deb` fájlt az `apt` paranccsal (ne az `dpkg -i` paranccsal, amely kihagyja a Python
SDK
ajánlott követelményeit):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

Ezzel az `chloros-cli`-t telepíti az `/usr/bin/chloros-cli`-re (már az `PATH`-en található), a háttérprogramot és a CLI
-programokat, amelyek az `/usr/lib/chloros/backend/` és az `/usr/lib/chloros/cli/` mappákba vannak kicsomagolva (az `/usr/lib/chloros/chloros-backend` egy indítóprogram, amely elindítja a háttérprogramot), az Arena SDK
futásidejét, amelyre a háttérprogramnak szüksége van a LATTICE kamerákhoz, valamint a rendszerhez tartozó Python
SDK
-t (`python3`). A részleteket lásd a [Linux
-telepítés](linux/linux-installation.md) című részben.

### Ellenőrzés

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Bejelentkezés és licencelés

A CLI
(és a Python
SDK
) eléréséhez **fizetős Chloros
+ csomag**szükséges – ez minden fizetős csomagban benne van, az ingyenes csomagban viszont nem. A korlátozást**szerveroldalon** a háttérprogram érvényesíti, nem a CLI
bináris fájl: a kijelentkezett állapotból érkező hívást `401 AUTH_REQUIRED` kóddal utasítja el, a bejelentkezett felhasználó által az ingyenes csomagból indított hívás pedig az `403 PLAN_UPGRADE_REQUIRED` hibakóddal, függetlenül attól, hogy az `chloros-cli`-ből, az SDK
-ből vagy egy saját fejlesztésű HTTP
-kliensből érkezik-e. Frissítsen a [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) oldalon.

**Gépenként egyszer** jelentkezzen be:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Különleges karaktereket tartalmazó jelszavak**(az `$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$`-et a shell eltorzítja (a CLI
ezt 401-es hibakóddal észleli és automatikusan újrapróbálja, de az egyszerű idézőjelek teljesen elkerülik a problémát).
{% endhint %}

A munkamenet az `~/.chloros/user_session.json`-ben kerül tárolásra, és a csomag türelmi ideje alatt (havi csomagok esetén 30 napig, éves csomagok esetén a lejáratig) offline módban is működik. Az `chloros-cli status` fizetett terv nélkül is működik, így az elutasítás oka mindig látható.

{% hint style="danger" %}
**Fej nélküli feladatot szeretne ütemezni? Először jelentkezzen be.**A háttérfolyamatot indító parancsok (`process`, `status`, `export-status`, …)**gyorsan nem jelez hibaüzenetet**, ha**nincs gyorsítótárazott munkamenet**– ehelyett interaktív `Email:` / `Password:` parancssorba vált át a stdin-en. Egy felügyelet nélküli cron-feladat vagy CI-lépés ezért**a bemenet várakozása közben lefagy**. Mielőtt bármit is ütemeznél, futtasd le egyszer az `chloros-cli login EMAIL 'PASSWORD'` parancsot a gépen.
{% endhint %}

***

## Az első feldolgozási futtatás

Irányítsa az `process` parancsot a rögzítések mappájára — a program automatikusan felismeri a Survey3
-at (`.raw` + `.jpg`), a LATTICE-t (`.tif`/`.tiff`), az `.dng`-et, vagy ezek kombinációját:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

A folyamatok élőben haladnak a pipeline szálak szerint (felismerés, elemzés, feldolgozás, exportálás), és egy sikeres futás azzal zárul, hogy jelentést ad arról, hány képproduktumot írt ki: `Image products written: N`, amelyet az `(plus M clamp masks)` követ, ha a futtatás során [clamp maszkokat](output-image-formats.md#clamp-masks) is írt (ezeket nem számítják terméknek).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Ha már fut egy Chloros
-háttérprogram (például a desktop alkalmazásé), az `process` azt használja, és abban nyitja meg a projektet. Az alkalmazás által megnyitott projekten futó folyamatot elutasítja: indítsa el az alkalmazásból, vagy zárja be azt a projektet. Ha egy korábbi futtatás még befejeződik, akkor legfeljebb 15 másodpercig vár rá, mielőtt elutasítaná. Az `--restart` új háttérprogramot indít. Linux
rendszeren a systemd szolgáltatás által futtatott háttérprogram soha nem indul újra: az `process` úgy használja, ahogy van, és ahol egyébként újraindítaná (`--restart`, vagy egy olyan import, amely nem talált semmit), az `sudo systemctl restart <unit>` üzenetet írja ki, majd kilép (`1`).

### Hová kerülnek a kimenetek

Az `process` egy **projektmappába** ír, nem a bemeneti mappádba:

* `-o` hiányában: a projekt az alapértelmezett projektmappában jön létre (amelyet a grafikus felülettel oszt meg; kezelése az `get-project-folder` / `set-project-folder` parancsokkal történik; ha nincs beállítva, akkor az `Documents/MAPIR/Chloros Projects` a felhasználói mappában), amelynek neve az `-n/--project-name` vagy – ha elhagyják – egy időbélyeg (`YYYYMMDD_HHMMSS`).
* Az `-o PATH` esetén: ez a mappa **éppen** a projektmappa. Ha már tartalmaz egy `project.json` fájlt, akkor a `_1`/`_2`… utótaggal ellátott testvérfájl jön létre a felülírás helyett. A relatív `PATH` elérési út az `chloros-cli` futtatásának helyét jelentő mappa viszonylatában kerül meghatározásra.

{% hint style="info" %}
**Relatív útvonalak (változás a 1.3.0-s verzióban).** Minden útvonal, amelyet a háttérprogram megnyit — `-o`/`--output` bármely parancs esetén (beleértve a rögzítési parancsok alapértelmezett `output` értékét is), `--input`, `--burst-dir`, `--profile`, `--save` és `--mask` — az a mappa alapján kerül feloldásra, amelyből az `chloros-cli` parancsot futtatja, így az `-o captures/` az `<current folder>/captures/` mappába ír. Az 1.2.0-s verzióban a háttérprogram a saját telepítési mappájában oldotta meg őket, ami „Hozzáférés megtagadva” hibaüzenettel végződött. Távoli háttérprogram (`CHLOROS_BACKEND_URL`) esetén az elérési útvonalakat a beírt formában küldik el, és azok a távoli gépen kerülnek feloldásra.
{% endhint %}

{% hint style="warning" %}
**Windows
: tartsa rövidre a projektmappát, hacsak a hosszú elérési útvonalak nincsenek bekapcsolva.** Ha a Windows
-ben a hosszú elérési útvonalak ki vannak kapcsolva (ez a Windows
alapértelmezett beállítása), egyetlen fájl elérési útvonala sem lehet hosszabb 260 karakternél, és a termékek legfeljebb körülbelül 130 karakterrel helyezkedhetnek el a projektmappa alatt. `process` ezt ellenőrzi az indítás előtt: a túl mély projektmappát **„A projektmappa túl mély ehhez a számítógéphez”** üzenettel elutasítja, megadja a leghosszabb megengedett projektmappát, valamint a hosszú útvonalak bekapcsolásának módját — semmi más nem kerül feldolgozásra. Használjon rövidebb `-o` (vagy `set-project-folder`) beállítást, vagy engedélyezze a Windows
hosszú elérési útjait (a telepítő **Windows
hosszú elérési útjainak engedélyezése** opciója, amely alapértelmezés szerint be van jelölve), majd indítsa újra a Chloros
programot.
{% endhint %}

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

A kamera mappa neve a LATTICE esetében `LATT-<sensor>-<lens>-F<filter>` (megegyezik a felvétel EXIF-adatával: `Model`), a Survey3 esetében pedig `<model>_<filter>` (pl. `Survey3N_RGN`) a Survey3
esetében. A formátum mappa a `--format` után a következő: `tiff16`, `tiff8`, `png8`, `jpg8` vagy `tiff32` az `TIFF (32-bit, Percent)` esetében.

Multispektrális M3C kamera esetén a formátummappában a termékmappák mellett megjelenhet egy `clamp_masks/` mappa is (például `tiff16/clamp_masks/Reflectance_Calibrated_Images/` és `tiff32/clamp_masks/Radiance_Images/`). Ez tartalmazza azoknak a sugárzási és visszaverődési termékeknek a képpontonkénti maszkjait, amelyekben a sávszétválasztás során a negatív értékeket nullára korlátozták — lásd [Korlátozó maszkok](output-image-formats.md#clamp-masks). Egyetlen termékmappa sem tartalmaz maszkot.

{% hint style="info" %}
**Minden exportált termék megtartja a FORRÁS fájl nevét.**Az `capture_..._raw.tif` sugárzási exportja továbbra is `capture_..._raw.tif` néven szerepel — csupán az `tiff32/Radiance_Images/` könyvtárban található.**A mappa azonosítja a terméket, nem a fájlnév**, ezért a könyvtárat keresse glob-kifejezéssel, ne pedig az `*radiance*` kiterjesztést.
{% endhint %}

### A ténylegesen használt opciók

| Opció | Alapértelmezett | Mit csinál |
| --- | --- | --- |
| `-o, --output PATH` | alapértelmezett projektmappa | A projektmappa helye (lásd fent). |
| `-n, --project-name NAME` | időbélyeg | Projektnév. |
| `--format FMT` | `TIFF (16-bit)` | Az alábbiak közül az egyik: `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` közül az egyik. A Chloros
nem nagyítja fel a képet: az összes 8 bites képen a mélyebb formátumot `TIFF (8-bit)` néven írja, és ezt egyszer jelzi. |
| `--indices NAME [NAME ...]` | nincs | Exportálandó vegetációs indexek (lásd [Vegetációs indexek](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = neurális debayer, lassabb, legmagasabb minőség (Chloros
+, CUDA GPU szükséges: ennek hiányában a futtatás az `standard`-et használja; elutasítva a LATTICE-felvételek esetében, amelyek mindig az `standard`-et kapják). |
| `--vignette / --no-vignette` | be | Vignettakorrekció. |
| `--reflectance / --no-reflectance` | be | Reflektancia-kalibrálás; a LATTICE esetében ez egyben a reflektancia-szorzat kapcsolója is. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | A feldolgozási folyamat belépési pontjának kényszerítése LATTICE TIFF-ek esetén. |

Minden egyéb beállítás – célpont-felismerés hangolása, PPK, expozíciós pontok, tömb-igazítási jelzők – tekintse meg az [CLI
-referencia `process` szakaszát](reference/cli-reference.md).

***

## Az exportálandó elemek kiválasztása (LATTICE termékek)

A LATTICE feldolgozás **egy lépésben**minden alkalmazható termékre kiterjed. A termékenkénti négy kapcsoló**alapértelmezés szerint mind be van kapcsolva**; az `--no-` űrlap használatával egyet kikapcsolhat:

| Kapcsoló | Termék |
| --- | --- |
| `--debayered` | Lineáris demosaic → `Debayered_Images/`: a kalibrációs csomag sötét eltolásának eltávolítása, majd élek figyelembevételével végzett demosaic, linearitás-korrekció, flat field vagy unmix nélkül — ugyanaz a fájl, amelyet a rögzítés az `debayered` szintjén ment. Az érzékelő natív tájolásában marad (XMP-ben rögzítve). |
| `--preview` | Megjelenítési előnézet (fehér egyensúly + gamma; hamis színű kiterjesztés multispektrális felvételekhez) → `Preview_Images/` |
| `--radiance` | float32 sugárzási intenzitás, W/m²/sr/nm → `Radiance_Images/` (mindig `tiff32/`) |
| `--reflectance` | uint16 fényvisszaverődés, Pix4D-kompatibilis → `Reflectance_Calibrated_Images/` |

Az RGB
-es főkamerák kizárólag debayered + előnézeti adatokat bocsátanak ki — a sávonkénti sugárzás/reflektancia szélessávú érzékelő esetében értelmetlen, ezért ezek a kapcsolók számukra hatástalanok. A Survey3
-as `.raw` figyelmen kívül hagyja a kapcsolókat, és a szabványos reflektancia/célútvonalat követi.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (alapértelmezett: `auto`) kiválasztja a reflexió-referenciát: `auto` a minőségbiztosításon átment, a képkockán belüli [kalibrációs céltárgyat](calibration-targets.md) tekinti abszolút referenciának, és cél hiányában a DAQ fényérzékelőjének lefelé irányuló fényeloszlására (ρ = π·L/E) értékre, ha nincs jelen célpont (a minőségbiztosítást jelzővel teljesítő célpont a mért pontszám tekintetében versenyez a DAQ-val); az `target` szigorú (nincs DAQ-helyettesítés); az `daq` a DAQ-nak ad elsőbbséget. Egységenkénti mért célpont-letapogatások az `--target-reflectance-dir`-szel adhatók meg.

{% hint style="info" %}
**Reflektancia-pixelek olvasása:**a ρ = 1,0-t jelentő DN**forrásonkénti** — a LATTICE-fájlok az XMP-be az `Chloros:PixelScale=32768` címkét illesztik be; a Survey3
-fájlok a 65535 értéket használják, és nem tartalmaznak `Chloros:PixelScale` címkét (viszont tartalmaznak más `Chloros:*` feldolgozási címkéket). Olvassa be a címkét, és osztja el azzal, ahelyett, hogy állandót feltételezne. A részletek és az egyetlen szándékos, nem skálázott szélsőséges eset a [CLI
-referenciában](reference/cli-reference.md) találhatók.
{% endhint %}

**A feldolgozás mindig az `raw`-től indul.** A származtatott termékek (debayered/radiance/reflectance exportok) soha nem kerülnek vissza a feldolgozási folyamatba — újbóli importálásuk és feldolgozásuk a kalibrációs számítások kétszeri alkalmazását jelentené, ezért az Chloros
kihagyja őket, és erről tájékoztatást is ad. Az `--input-level` egy szándékosan beépített menekülési útvonal arra az esetre, ha valóban szükség van egy belépési pont kényszerítésére.

***

## Ha egy futtatás sikertelen

Az 1.2.0-s verziótól kezdve az `process` hangosan jelzi a hibát, ahelyett, hogy látható eredmény nélkül „sikeresnek” tűnne:

* Az a futtatás, amely **termékeket kért, de egyet sem írt ki**— csak az `project.json` és az `calibration_data.json` — kiírja az `Processing finished but wrote no image products.` kódot, és**nem nulla értékkel lép ki**, így a szkriptek felismerhetik. A szokásos okok: a bemeneti mappát nem ismerte fel a rendszer rögzítésként (ellenőrizze az elrendezést és az `--input-level` fájlt), vagy az összes kért termék alkalmazhatatlan volt az adott kamerák esetében (pl. sugárzási/visszaverődési értékek kérése kizárólag RGB
-kameráktól).
* Egy **szándékosan csak metaadatokra korlátozott futtatás** (minden termék kikapcsolva, nincs `--indices`) továbbra is sikeresnek minősül — ebben az esetben az üres kép kimenet a helyes eredmény.
* Futtassa újra az `--verbose` beállítással, és ellenőrizze a háttérnaplóban az `[LATTICE-EXPORT]` / `[EXPORT-CHECK]` sorokat, amelyek kameránkénti kihagyásokat magyarázzák.
*Windows
rendszeren, ha a hosszú elérési utak ki vannak kapcsolva, a 260 karakteres elérési út-korlátot meghaladó mélységű projektmappát a feldolgozás megkezdése **előtt** elutasítja a rendszer, és a leghosszabb, a korlátba beleférő projektmappát használja (lásd [Hová kerülnek a kimenetek](#where-the-outputs-land)).
* A rögzítési parancsok (`lattice capture`, `lattice array-capture` és a többi rögzítési parancscsalád) felsorolják az összes olyan fájlt, amelyet nem tudtak írni, majd kilépnek (`1`); az ismételt rögzítések leállnak, ha a meghajtó megtelik.

Kilépési kódok: `0` siker · `1` általános hiba · `2` argumentumhiba · `130` a Ctrl+C billentyűkombinációval megszakítva.

***

## Vegetációs indexek

Futtassa az `--indices` parancsot egy vagy több előre beállított névvel; minden index a saját `<INDEX>_Index_Images/` mappájába kerül:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

A 22 előre beállított név, amelyet az `process --indices` elfogad:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

A projektbe felkerül egy olyan név, amely még nem szerepel a listában. Azokat a neveket, amelyek nem előre beállítottak, vagy amelyeket a kamerák sávjai nem tudnak létrehozni, a rendszer figyelmeztetéssel kihagyja, megnevezve azokat. Ha a CLI
nem tudja beolvasni a projekt aktuális indexlistáját, a listát változatlanul hagyja, és figyelmeztetést ad ki, ahelyett, hogy felülírná.

{% hint style="warning" %}
**Három indexlista létezik — ne keverje össze őket!**A GUI „Projektbeállítások” legördülő menüjében 27 képlet található (hozza hozzá az `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — ez az öt kizárólag a grafikus felületen használható, és**nem** érvényes az `--indices`-re). Az élő/offline `lattice index --preset` parancs saját, külön 22-es beállítási listát használ. A képletek és a sávszámítások leírása a [Multispektrális indexképletek](project-settings/multispectral-index-formulas.md) című dokumentumban található.
{% endhint %}

***

## DAQ fényérzékelők: Rövid áttekintés

Az `daq pool-*` család a háttérrendszer állandó poolján keresztül vezérli a MAPIR
DAQ spektrális érzékelőit (CLI-en keresztül a DAQ-U-t, BLE-n keresztül a DAQ-M-et, Etherneten keresztül a DAQ-E-t) — a grafikus felhasználói felület, a parancssori felület (CLI
) és az SDK
mind egy élő kezelőt használnak. **Az `pool-*` a szállított CLI
által támogatott DAQ-útvonal**; az egyéb `daq` alparancsok, amelyekre hivatkozhatnak, a MAPIR
belső, kizárólag forrásként szolgáló felületét jelentik, és egy kifejezett hibaüzenettel lépnek ki, amely az `pool-*`-re irányít.

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

`pool-record` `--duration` nélkül fut, amíg el nem éri az `pool-record --stop`-et; az alapértelmezett kimeneti könyvtár az `~/Documents/DAQ Live View/` **a háttérrendszer gépén**. A kondenzátor-korrekciós profilt a csatlakozáskor választják ki (`--cap-id`, háttérrendszer alapértelmezett: `sunshine_cosine`), és futás közben is cserélhető az `pool-set-cap` — a felső határprofilokról és az érzékelő kalibrált tartományáról a kézikönyv DAQ fejezetei szólnak.

{% hint style="warning" %}
**DAQ-E több hálózati kártyával rendelkező gazdagépen:** az indítás utáni első `pool-connect --eth` automatikus felismerés még működőképes érzékelő esetén is sikertelen lehet. Az `--eth-host <ip-or-hostname>` a megbízható megoldás — válassza ezt, ha a felismerés eredménytelen.
{% endhint %}

***

## LATTICE kamerák, PTP és projekt-automatizálás

Az `lattice` család (több mint 45 alparancs) a LATTICE kamerák használatát teljes körűen lefedi: felismerés, egyedi felvételek, állandó szinkronizált tömbök a grafikus felhasználói felület intelligens előkészítési csatlakozási folyamatával, élő böngésző-előnézet, igazítás, indexszámítás és gazdagép-hálózati kártya diagnosztika. Ízelítő:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Ehhez kapcsolódóan: az `chloros-cli time-sync` jelentést készít a PTP grandmasterről, amelyet a Chloros
-gazdagép futtat (a LATTICE kamerák és a DAQ-E érzékelők ehhez kapcsolódnak az eszközök közötti időbélyegek érdekében), az `chloros-cli project` pedig megnyit egy mentett Chloros
-projektet, és vezérli a kameráit, szenzortömbjeit és érzékelőit headless módban — beleértve a szkriptelt YAML-felvételi receptjeit is.

Ez a három család (`lattice`, `project`, `daq pool-*`) az egyetlenek, amelyek támogatják az `CHLOROS_BACKEND_URL` parancsot **távoli** háttérrendszer vezérléséhez; az alapvető parancsok mindig a helyi gépet célozzák meg.

A teljes útmutatók a kézikönyv LATTICE fejezeteiben találhatók; minden kapcsoló a [CLI
-referenciában](reference/cli-reference.md) szerepel.

***

## Hibaelhárítás: Az 5 leggyakoribb hiba

| Tünet | Megoldás |
| --- | --- |
| Az `Login required` vagy egy ütemezett feladat lefagy az `Email:` parancssorban | Futtassa egyszer az `chloros-cli login EMAIL 'PASSWORD'` parancsot ezen a gépen — a gyors hibaüzenet helyett a gyorsítótárban nem tárolt munkamenet nélküli parancsok interaktív módon futnak. |
| `backend unreachable` | Indítsa el a Chloros
asztali alkalmazást, vagy indítsa el saját maga a háttérprogramot: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` Windows
rendszeren (a telepítési mappa `backend` mappájában; az `PATH`-en nincs), `chloros-backend`-et Linux
on. Az `chloros-cli backend status` jelzi, hogy fut-e valamelyik; az `lattice`/`project`/`daq pool-*` soha ne indítson el egyet sem. Ha távoli háttérprogramra irányítja őket, ellenőrizze az `CHLOROS_BACKEND_URL` fájlt. |
| A tömbcsatlakozás blokkolva: `FRAMES WILL DROP` / `Reduce ROI to enable` | A gazdagép hálózati kártyájának vételi gyűrűje alapértelmezett értékekre állt vissza — ez a leggyakoribb oka annak, hogy egy korábban működő rendszer nem hajlandó csatlakozni, általában egy hálózati kártya-illesztőprogram frissítése után. Futtassa az `chloros-cli lattice network --fix` parancsot egy **feljogosított** terminálról (vagy állítsa be az `ReceiveBufferLen=256`, `PendingReceives=64` parancsokat); lásd a kézikönyv *Host NIC Setup &amp; Tuning* című részét. |
| Az `daq` alparancs kilép: „a teljes DAQ-csomag szükséges…” | Használja az `daq pool-*` parancsokat, amelyek a háttérrendszeren keresztül vezérlik ugyanazt az érzékelőt: csatlakozás, adatfolyam, rögzítés, csatornaválasztás, valamint DAQ-E esetén a hálózati beállítások, az újraindítás és a firmware-frissítések (vagy az `chloros_sdk.connect_daq_sensor()` parancs Python
ból). |
| A Jetson nagy mappák előtt swap-figyelmeztetést jelenít meg | Adjon hozzá fájlalapú swap-ot — a CLI
kijelzi a futtatandó pontos `fallocate`/`swapon` parancsokat. |

***

## Súgó

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Minden jelző, minden alparancs:** [CLI
-referencia](reference/cli-reference.md)
* **Python
-megfelelő:** [Python
SDK
](api-python-sdk.md) és az [SDK Reference](reference/sdk-reference.md)
* **Támogatás:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
