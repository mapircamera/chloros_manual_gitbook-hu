---
metaLinks: {}
---

# Bevezetés

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Az Chloros egy szoftveralkalmazás a [MAPIR](https://www.mapir.camera) által kifejlesztett szoftveralkalmazás, amely multispektrális képek feldolgozására, az MAPIR hardver élő vezérlésére, valamint érzékelőadatok rögzítésére szolgál. Az Chloros 1.2.0 verzió az MAPIR termékcsalád teljes skáláját támogatja:

* **Survey3 kamerák** — feldolgozza a RAW+JPG felvételeket kalibrált reflexiós és vegetációs index térképekké. Lásd: [Támogatott kamerák](supported-cameras.md).
* **LATTICE kamerák** — csatlakoztassa a GigE multispektrális kameramodulokat élőben, egyenként vagy szinkronizált többkamerás rendszerként: előnézet, felvétel és feldolgozás kalibrált sugárzási és reflexiós termékekké. Lásd a [LATTICE szakaszt](lattice/README.md).
* **DAQ fényérzékelők** — DAQ-U (USB), DAQ-M (Bluetooth) és DAQ-E (Ethernet) spektrális érzékelők: élő, kalibrált spektrumok, `.daq` felvételek és lefelé irányuló megvilágítás a reflexió feldolgozásához. Lásd a [DAQ szakaszt](daq/README.md).

{% hint style="success" %}
**Újdonságok az Chloros 1.2.0 verzióban**: élő LATTICE kamera- és sorozatvezérlés, DAQ fényérzékelő-integráció, rögzítési módok és felvevők, teljes LATTICE radiometrikus feldolgozási folyamat, projekt-automatizálás az CLI/SDK-ből, és még sok más. Tekintse meg az alábbi Újdonságok listát, és [Töltse le](download.md) a változásnaplót.
{% endhint %}

{% hint style="info" %}
**Az Chloros-et AI-asszisztenssel használja?** Ez a kézikönyv pont erre készült. Irányítsa asszisztensét a következőkre:

* `https://mapir.gitbook.io/chloros/llms.txt` — minden oldal géppel olvasható indexe.
* Bármely oldal nyers Markdown formátumban — illessze hozzá az `.md` kiterjesztést az URL kiterjesztéséhez (pl. `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* Az [CLI Referencia](reference/cli-reference.md) és [SDK hivatkozás](reference/sdk-reference.md) — teljes, pontos értékeket tartalmazó hivatkozási oldalak, amelyeket LLM-ek számára írtak.

Példa a promptra: *„Olvassa el az https://mapir.gitbook.io/chloros/reference/cli-reference.md, dokumentumot, majd írjon egy szkriptet, amely bejelentkezik, és feldolgozza a ~/flights/flight_001 mappát, hogy reflectance + NDVI GeoTIFF fájlokat hozzon létre.”*

Teljes útmutató: [Az Chloros használata AI-asszisztensekkel](ai-assistants.md).
{% endhint %}

***

## Újdonságok az Chloros 1.2.0 verzióban

* **Élő kameravezérlés — új „Kamerák” fül.** Csatlakoztassa a LATTICE kamerákat egyenként vagy szinkronizált többkamerás rendszerként (PTP időszinkronizálás, hardveres kiváltású felvétel), élő előnézeti átfedésekkel, sávonkénti hisztogramokkal, intelligens automatikus expozícióval, élő indexszámítóval és az alkalmazáson belüli kamera firmware-frissítésekkel.
* **Fényérzékelők — új „Fényérzékelők” fül.** Csatlakoztathat DAQ-U (USB), DAQ-M (Bluetooth) és DAQ-E (Ethernet) érzékelőket; tekintse meg az élő, kalibrált spektrumokat (W/m²/nm), rögzítse az `.daq` fájlokat a projektjébe, válasszon fényerő-korrekciós profilokat, és frissítse a DAQ-E firmware-jét a hálózaton keresztül.
* **Rögzítési módok és rögzítők.** Egyszeri / Folyamatos / Intervallumú rögzítés, valamint kizárólag nyers adatokat rögzítő „Leggyorsabb rögzítés” mód; projektenkénti kiválasztás, hogy a „Minden rögzítése” funkció mely kamerákból és exporttípusokból állítson elő adatokat; tömbös rögzítők monitoring minőségű indexvideókhoz és elemzési minőségű nyers adatcsomagokhoz, offline videó-összeállításokkal.
* **LATTICE feldolgozási folyamat.** Importálhatja a LATTICE rögzítési mappáit, és minden nyers képkockát kibontva debayered, előnézeti, float32 sugárzási (W/m²/sr/nm) és reflexiós termékekké alakíthat, termékenkénti kapcsolókkal. A reflexió származhat egy képkockán belüli kalibrációs célpontból vagy DAQ lefelé irányuló sugárzásból; az exportokra tömbös igazítás kerül alkalmazásra; a hiányzó gyári kalibrációt a kamera sorozatszáma alapján automatikusan letölti a rendszer.
* **A projektek megjegyzik a hardvert.** A csatlakoztatott kamerák és fényérzékelők a projekttel együtt kerülnek mentésre (`cameras.json` / `sensors.json`), és a projekt újbóli megnyitásakor a mentett beállításokkal csatlakoznak újra. Lásd [GUI: Projektek](projects.md).
* **Képmegjelenítő frissítések.** A kurzor pixel/index kiolvasása a fájlonkénti helyes reflexiós skálázással, réteghisztogramok, GSD-binning csúszka, „Per Trigger” / „Per Camera” rács módok, LATTICE terméknézetek, valamint index/LUT sandbox exportálás lemezre.
* **CLI és SDK, jelentősen kibővítve.** Új `lattice`, `daq pool-*`, `project` és `time-sync` parancscsaládok; új `process` opciók (`--input-level`, termékenkénti kapcsolók, `--reflectance-source`, tömb-igazítási jelzők); SDK intelligens csatlakozási kezelők (`connect_camera` / `connect_array` / `connect_daq_sensor`), amelyek automatikusan elindítják a háttérprogramot; `open_project()` automatizálás; az SDK wheel a telepítőkkel együtt kerül csomagolásra, és `chloros-sdk` néven kerül közzétételre a PyPI-n.
* **Egyértelmű hibaüzenetek.** Az a `chloros-cli process` futtatás, amely termékeket kért, de egyet sem írt ki, most egyértelmű hibaüzenettel jár és nem nulla kóddal lép ki; a sikeres futtatások jelentik, hogy hány képterméket írtak ki.
* **Új kimeneti elrendezés.** A termékek az `<project>/<camera>/<format>/<Product>_Images/` mappákba kerülnek, és megtartják a forrásfájl nevét – a mappanév, nem pedig a fájlnév-kiterjesztés azonosítja a terméket. Lásd [Kimeneti képformátumok](output-image-formats.md).
* **Több bemenet, terv és nyelv.** `.dng` bemeneti támogatás; mind a 38 felületi nyelv teljes mértékben kitöltve; tervenkénti hardverkorlátozások, ingyenes (bejelentkezés nélküli) használat esetén legfeljebb 4 kamera és 2 fényérzékelő.
* **Megbízhatóság.** A „Feldolgozás leállítása” funkció tisztán leáll, pontos futási összefoglalóval; a többkamerás projektek minden kamerát exportálnak, és a telepítő frissítései már nem jelentik ki a felhasználót.***

Az Chloros három felhasználói felületen érhető el:

## Chloros: Asztali GUI-alkalmazás

Önálló, külön ablak az összes funkcióval, beleértve az Élő kamerák és Fényérzékelők lapokat. _Csak Windows._

## [Chloros CLI: Parancssori felület](CLI.md)

Parancssori kötegelt feldolgozás, valamint az élő `lattice`, `daq pool-*`, `project` és `time-sync` parancsok. Tökéletes automatizáláshoz, szkripteléshez és headless üzemmódhoz. Elérhető az **Windows, Linux amd64 és Linux arm64 (NVIDIA Jetson)** verziókon. _A CLI eléréséhez fizetős Chloros+ szint szükséges._

## [Chloros API: Python SDK](api-python-sdk.md)

Programozási Python interfész automatizáláshoz és egyedi munkafolyamatokhoz: teljes feldolgozási folyamat, élő kamera-/tömb-munkamenetek, DAQ-érzékelő-munkamenetek és mentett projektek automatizálása. A desktop/CLI csomaggal együtt települ, és `pip install chloros-sdk` néven is elérhető. _Az API eléréséhez fizetős Chloros+ szint szükséges._

***

## Támogatott platformok

| Platform | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Igen | Igen | Igen |
| **Linux amd64 (x86_64)** | Nem | Igen | Igen |
| **Linux arm64 (NVIDIA Jetson)** | Nem | Igen | Igen |

Az Linux telepítési útmutatóját lásd az [Linux és Edge Computing](linux/linux-overview.md) szakaszban.

***

## Három lépésben az induláshoz

1. **Telepítés** — töltse le és futtassa a platformjához megfelelő telepítőt. Lásd: [Letöltés](download.md).
2. **Bejelentkezés (a grafikus felhasználói felület esetében opcionális)** — a grafikus felhasználói felület fiók nélkül is ingyenesen feldolgozza a képeket. A [Chloros+ bejelentkezés](chloros+-login.md) lehetővé teszi a párhuzamos feldolgozást, a GPU-gyorsítást, a magasabb eszközkorlátokat, valamint az CLI/SDK hozzáférést.
3. **Készítse el az első projektjét** — nyissa meg az Chloros alkalmazást, hozzon létre egy [Új projektet](projects.md), [adja hozzá a képeit](processing-images-gui/adding-files-to-a-project.md), majd [indítsa el a feldolgozást](processing-images-gui/starting-the-processing.md). Ha inkább élő hardvert szeretne vezérelni, nyissa meg a Kamerák vagy Fényérzékelők fület — lásd [GUI: Navigáció](navigation.md).***

## Chloros+

Bár az Chloros a legtöbb feladatra ingyenesen használható, előfordulhat, hogy többet szeretne. Ilyenkor az Chloros+ fizetős licenc előnyös lehet az Ön számára. Az Chloros+ licenccel olyan új funkciókat nyithat meg, mint például:

* **Többszálas feldolgozás**: jelentősen felgyorsíthatja a képfeldolgozást nagyobb projektek esetén azáltal, hogy a képeket párhuzamosan dolgozza fel a feldolgozási folyamatban.
* **GPU (CUDA) gyorsítás**: kihasználhatja a mai nagyobb GPU-memória-kapacitásokat a képfeldolgozási folyamat további felgyorsításához. A legjobb eredmények elérése érdekében 4 GB vagy annál nagyobb VRAM-ot ajánlunk.
* **Chloros+**[**CLI**](CLI.md)**Hozzáférés**: futtassa az Chloros+ parancsot a parancssorból a saját szoftverébe történő automatizálás és integrálás érdekében. Bármely fizetős csomagban elérhető; szerveroldalon érvényesítve.
* **Chloros+**[**API**](api-python-sdk.md)**Hozzáférés:** futtassa az Chloros+ parancsot az Python-ből a programozási vezérléshez, amely zökkenőmentes integrációt tesz lehetővé a kutatási folyamatokba, az adatelemzési munkafolyamatokba és az egyedi alkalmazásokba. Bármely fizetős csomagban elérhető; szerveroldalon érvényesül.
* **Magasabb hardveres korlátok**: egyszerre több kamera és fényérzékelő csatlakoztatható. Bejelentkezés nélkül a grafikus felhasználói felület (GUI) legfeljebb 4 kamerát és 2 DAQ fényérzékelőt csatlakoztat; a fizetős csomagok mindkét korlátot megemelik:

| Csomag | Kamerák | DAQ fényérzékelők |
| --- | --- | --- |
| Iron (ingyenes, bejelentkezés nélkül) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Több eszköz használata**: minden Chloros+ licenc 2 vagy több eszköz regisztrációját teszi lehetővé. A regisztrált eszközök kezeléséhez használja az MAPIR Cloud fiókját. Az Chloros+ licenc frissítésével további eszközök támogatását is hozzáadhatja.
* **Fejlett, textúraérzékeny debayer-módszer:** kiváló minőségű, élekkel számoló debayer, amely egy AI/ML zajszűrő modellel kombinálva szinte teljesen eltávolítja a debayerelésből származó zajt.
* **Egyéni multispektrális indexképletek:** adjon meg egyéni multispektrális indexeket az Chloros raszterkalkulátorokban, mind a feldolgozáshoz, mind a képnézegető sandboxhoz.
* **Linux és Edge Computing:** futtassa az Chloros szoftvert az Linux x86_64 és ARM64 platformokon, beleértve az NVIDIA Jetsont is, terepi és edge feldolgozás céljából. Lásd az [Linux áttekintést](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Árak és regisztráció</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->
