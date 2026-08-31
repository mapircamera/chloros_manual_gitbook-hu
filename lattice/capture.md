# Felvételi beállítások és módok

A „Kamerák” lapon a felvételkészítést egy piros **Minden felvétel**gomb és egy**Felvételi beállítások** ablak vezérli, amely meghatározza, hogy a gomb milyen eredményt hoz: mely kamerák vesznek részt, milyen exporttípusokat ment az egyes kamerák, valamint hogy a zár egyszer, folyamatosan vagy időközönként sül-e el. Ez az oldal a teljes folyamatot ismerteti – a konfigurációt, magát a felvételt, a fájlok lemezre mentésének helyét, valamint azt, hogyan lehet őket később kalibrált termékekké feldolgozni. Maguk a kamera- és tömbvezérlők a [Kamera beállítások](camera-settings.md) oldalon találhatók.

{% hint style="info" %}
**A felvételekhez nyitott projekt szükséges.** Az „Összes felvétel” és a „Felvételi beállítások” fogaskerék gombok addig le vannak tiltva, amíg nincs nyitva egy projekt („Hozzon létre vagy nyisson meg egy projektet a felvételek mentéséhez”). Minden felvétel az `captures/` projektmappába kerül mentésre.
{% endhint %}

## A Felvételi beállítások ablak

Nyissa meg az oldalsáv kameralistáján található **„Minden felvétel” melletti fogaskerék**gombbal, vagy bármely kamera-specifikus beállítási ablak alján található**„Felvételi beállítások megnyitása…”** gombbal. A fejlécen a „Felvételi beállítások” felirat látható, mellette egy ← vissza gombbal.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

Az itt megadott beállítások – a bevont kamerák, a típusonkénti jelölőnégyzetek és a felvételi mód – **projektenként** kerülnek mentésre, és a projekt újbóli megnyitásakor visszaállnak.

### Felvételi módok

Három módgomb található az ablak tetején:

| Üzemmód | Működése | Albeállítások (alapértelmezett értékek) |
| --- | --- | --- |
| **Egyszeri** *(alapértelmezett)* | Egyetlen rögzítés minden kiválasztott kamerán. | — |
| **Folyamatos**| Egymást követő rögzítések egy leállási feltétel teljesüléséig. | Leállás**Rögzítések száma** alapján (alapértelmezett: 1) *vagy* **Rögzítési időtartam** alapján (alapértelmezett: 10 s; mértékegységek: másodperc / perc / óra / nap). |
| **Intervallum**(időfelvétel) | Időzített sorozatfelvételek. |**Felvételek / intervallum**(alapértelmezés: 1) ·**Minden**N egység után (alapértelmezés: 5 másodperc) ·**N egységig** (alapértelmezés: 1 perc). |

Folyamatos vagy Intervallum módban a „Minden felvétel” gomb futás közben **Leállítás (N)** gombbá válik, és számolja a beérkező felvételeket.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### Kamerák és exporttípusok kiválasztása

A panel súgószövege összefoglalja: válassza ki, hogy a „Minden felvétel” gomb mely kamerákból és milyen exporttípusokban készítsen felvételeket — alapértelmezés szerint minden be van kapcsolva, és a választások a projekthez kerülnek mentésre.

* A **Minden kijelölése / Semmi kijelölése** gombok egyszerre kapcsolják be vagy ki az összes kamera bevonási jelölőnégyzetét.
* **Tömeges exporttípus-kapcsolók**(két sor gomb):**Minden nyers / Minden debayered / Minden előnézet / Minden sugárzás / Minden visszaverődés / Minden index**. Mindegyik háromállapotú színkóddal rendelkezik: zöld ✓ = be van kapcsolva minden olyan kameránál, amely támogatja, sárga – = néhánynál be van kapcsolva, szürke = egyiknél sem. A kapcsoló akkor van letiltva, ha egyetlen csatlakoztatott kamera sem támogatja az adott típust. A „Fastest Capture” bekapcsolt állapotában mindegyik szürkére vált.
* **Kameránkénti sorok**: egy „Beleértve” jelölőnégyzet, valamint az adott kamerára alkalmazható exporttípusok kibővíthető (▸/▾) listája, egyedi jelölőnégyzetekkel. A sorban a bekapcsolt elemek számát jelzi, például „4/6”.

### Exporttípusok és azok támogatott kamerái

Hat exporttípus létezik: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Minden kamera sorában csak az adott kamerára vonatkozó típusok jelennek meg:

| Exporttípus | Tartalom | RGB (FRGB) | Bayer multispektrális (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayer-mozaik (mono: az egyetlen sáv) közvetlenül az érzékelőből | ✓ | ✓ | ✓ |
| **Debayered** | Lineáris demozájk (mono: 1-csatornás szürkeárnyalatos) | ✓ | ✓ | ✓ |
| **Előnézet** | Teljes megjelenítési lánc (fehér egyensúly + gamma a kamera profilja szerint; multispektrális: hamis színű kiterjesztés) | ✓ | ✓ | ✓ |
| **Sugárzás** | float32 W/m²/sr/nm a teljes radiometrikus láncon keresztül | — (nem elérhető) | ✓ | ✓ |
| **Fényvisszaverődés** | uint16 ρ (32768 = 1,0) | — (nem elérhető) | ✓ — csak akkor jelenik meg, ha a kamera rendelkezik DAQ fényérzékelővel (saját vagy a tömbből örökölt) | megegyezik a multispektrális értékkel |
| **Index** | Vegetációs index (LUT) megjelenítése | — | ✓ — a kamerán engedélyezett, nem üres indexkifejezést igényel, és nem elérhető kombinált sorozat tagjai számára (a sorozat egy közös indexszel rendelkezik) | — (egy indexhez ≥2 sáv szükséges; lásd [Mono kamerák és vegetációs indexek](mono-indices.md)) |

A sugárzás és a visszaverődés soha nem áll rendelkezésre az RGB kameráknál — a Bayer-mátrixonkénti sugárzás nem értelmezhető egy szélessávú fotometrikus érzékelő esetében.

### Leggyorsabb rögzítés

A **⚡ Leggyorsabb rögzítés — csak nyers**kapcsoló (bekapcsolt állapotban narancssárga) felülírja az összes exportálási beállítást**csak nyers** formátumra — plusz egy ingyenes kombinált indexű kompozíciót az array-ek esetében — így a képkocka a lehető leggyorsabban kerül mentésre: a sugárzás/reflektancia/megjelenítési számításokat a rögzítéskor teljesen kihagyják.

{% hint style="info" %}
**Egy `.daq` fájl továbbra is el lesz mentve.** Ha fényérzékelő van hozzárendelve, a „Leggyorsabb rögzítés” funkció továbbra is a DAQ lefelé irányuló mérési értékét írja a nyers képkockák mellé – így a sugárzási, visszaverődési és indexértékek később újrafeldolgozással mind létrehozhatók (lásd [Rögzítések újrafeldolgozása](#re-processing-captures-into-calibrated-products)). A Fastest Capture funkció nem befolyásolja a jelölőnégyzetekben megadott beállításokat: ha kikapcsolja, azok visszaállnak.
{% endhint %}

### Tömbönkénti vezérlők

Minden csatlakoztatott tömb saját csoportkártyát kap az ablaktáblában:

* **Beleértve jelölőnégyzet** (három állapotú a tagok között) és a tömb neve a megjelenítési módjával: „(összevont | külön)”.
* **Igazítás**jelölőnégyzet (alapértelmezés szerint**be**): a tagok exportálását a tömb igazítási profiljához igazítja, így az exportált adatok pixel-szinten regisztrálva vannak a kamerák között. A nyers adatok torzításmentesek maradnak, de a transzformációt a metaadataikban hordozzák. (Maga a profil az [array beállítások ablaktáblában](camera-settings.md#alignment-co-registration-combined-only) kerül kiszámításra.)
* Az elemkamerák sorait a kártyán belül helyezkednek el.

A tömbkártya két felvevőt is tartalmaz. Gondolj rájuk úgy, mint **megfigyelés és elemzés**:

| Felvevő | Fokozat | Mit rögzít |
| --- | --- | --- |
| **● Indexvideó rögzítése / ■ Felvétel leállítása** *(csak kombinált tömböknél)* | **Megfigyelés** | Az élő, kombinált indexű kompozíciót 10 fps sebességgel rögzíti videóként — 8 bites, előnézeti felbontás, beépített LUT. Nyitott projekthez és élő streamelt nézethez van szükség. A felvétel közben megjeleníti a képkockákat és az eltelt időt. |
| **⦿ Nyers sorozatfelvétel / ■ Nyers sorozatfelvétel leállítása** *(bármely sorozat)* | **Elemzés**| Nyers Bayer-képkockák az élő rögzítési sebességgel (feldolgozás nélkül), valamint képkockánkénti manifeszt és `.daq`-értékek, `captures/bursts/` formátumban. A sorozatfelvétel után megjelenik a**Videó készítése** gomb: ez offline módon újra feldolgozza a sorozatot kalibrált videóvá — kombinált index és/vagy kameránkénti sugárzás / visszaverődés / index — valamint opcionális TIFF-fájlokká. A kombinált index létrehozása automatikusan elindul, amikor leállítja a sorozatfelvételt. |##

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

Az „Összes rögzítése” folyamat

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

Nyomja meg a **„Összes rögzítése”** gombot az oldalsáv kameralistájában:

1. Minden bevont, látható, szüneteltetés nélküli kamera a kiválasztott exporttípusokkal rögzít. **A kameracsoportok egy szinkronizált indításként működnek** (egyetlen, az összes tagra kiterjedő szinkronizált csoport — lásd [Többkamerás csoportok](arrays.md)); az önálló kamerák egyenként rögzítenek.
2. A rejtett (szem) vagy szüneteltetett kamerákat a rendszer kihagyja. Egy tömb csak akkor blokkolódik teljesen, ha *minden* tagja rejtett vagy szüneteltetett állapotban van.
3. Ha fényérzékelő van hozzárendelve, a hozzá tartozó DAQ lefelé irányuló sugárzásmérési értéket `.daq` fájlként mentik a képanyag mellé – még a kizárólag nyers adatokat tartalmazó felvételek esetében is –, így a radiometrikus termékek később is bármikor kiszámíthatók.
4. A gomb élőben mutatja az előrehaladást — „Rögzítés… kész/összesen” —, és folyamatos/intervallum módban **Stop (N)**-re vált. Minden rögzítési elemnek 300 másodperces időkorlátja van.
5. Amikor a felvételi sorozat befejeződik, egy eredményüzenet jelzi:**„N fájl mentve”**vagy**„N fájl mentve, F sikertelen”**, valamint „(S elrejtve/szüneteltetve/kihagyva)”, ha kamerákat hagytak ki.

## A rögzítések tárolási helye

A rögzítéseket a megnyitott projektben az `<project>/captures/` alatt menti a rendszer. Minden exporttípus a **saját almappájába** kerül, így egy többszintű rögzítés során a típusok soha nem keverednek:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* Az `<ts>` a felvétel időbélyege, az `<serial>` pedig a kamera sorozatszáma. Az önálló rögzítések neve `capture_<ts>_SN<serial>_<level>`; az egy szinkronizált indítóból származó tömbös rögzítések neve `sync_<ts>_SN<serial>_<level>`, és **a csoport összes kamerájára közös időbélyeggel rendelkeznek** (a szint utótagot elhagyják, ha egy kamera csak egy szintet ment).
* **Egy tudnivaló eltérés:** a kijelzőszintet egy `preview/` nevű mappában tárolják, míg a fájlok neve megőrzi az `_display` részt — a mappa és az utótag csak ennél a szintnél különbözik.
* Az ismeretlen szintek a saját nevüket viselő mappába kerülnek; ha nem lehet almappát létrehozni, a fájl a felvételek gyökérkönyvtárába kerül, ahelyett, hogy elveszne.
* A rögzített TIFF-fájlok alapértelmezés szerint veszteségmentesen vannak tömörítve (DEFLATE), és a teljes kalibrációs és feldolgozási metaadatokat **a fájl XMP-jében** tárolják — a rögzítések önleíróak, az `.daq` nevű fájlon kívül nincsenek kiegészítő fájljaik.

Ez ugyanaz az elrendezés, amelyet az `chloros-cli lattice capture` / `array-capture` fájlok az `-o` könyvtárba írnak — erről bővebben a [CLI Referencia § Hogyan néz ki egy rögzítési mappa](../reference/cli-reference.md#what-a-captures-folder-looks-like) című részben dokumentálva.

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## A rögzítések újrafeldolgozása kalibrált termékekké

A rögzített nyers képkockák és a mentett `.daq` fájlok mindazt jelentik, amire a feldolgozási folyamatnak szüksége van — ezért használható a Fastest Capture biztonságosan valódi munkákhoz is.

* **GUI**: vegye fel a felvételek mappáját egy projektbe ([Fájlok hozzáadása egy projekthez](../processing-images-gui/adding-files-to-a-project.md)), majd dolgozza fel a szokásos módon.
* **CLI**: irányítsa az `process` fájlt a**felvételek gyökérkönyvtárára**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

`process` általában csak a megadott mappát importálja, de ha az a mappa nem tartalmaz képeket, viszont almappákkal rendelkezik, akkor automatikusan lefelé halad – így az almappák szintjei és a gyökérkönyvtárban található `.daq` fájlok egy lépésben kerülnek beolvasásra. Minden rögzítés **egyetlen képként** kerül importálásra, a többi szint pedig megtekinthető módként csatolva, nem pedig szintenként egy-egy képként.

A szint alkönyvtárának közvetlen megnevezése (pl. `…/captures/raw/`) szintén működik, de a gyökér `.daq` fájlokat hátrahagyja – ezeket is másolja át, amikor radiometrikus terméket származtat újra az `raw/`-ből, ellenkező esetben az időbélyeg-egyezésnek nincs mihez viszonyítania.

{% hint style="warning" %}
**A feldolgozás mindig az `raw`-től indul.**Minden egyes felvételen belül a nyers képkocka a feldolgozási folyamat forrása; az `debayered`, `radiance`, `reflectance` és `preview` fájlok megtekinthető módokként jelennek meg, de soha nem kerülnek vissza a feldolgozási folyamatba — egy származtatott termék újrafeldolgozása újra alkalmazná a képpontokba már beégetett vignettázást, színezést és sugárzási számításokat, ezért az Chloros elutasítja a feldolgozást, ahelyett, hogy kétszer dolgozná fel. Az `index/` és az `composite/` rendereléseket egyáltalán nem dolgozza fel a rendszer (ezek kimenetek, nem rögzítések). A „captures” mappa**nyersfájlok importálása nélkül** is normálisan menthető és jeleníthető meg, de az `process` kihagyja, és ezt jelzi is; az `--input-level {raw,debayered,processed}` egy szándékosan beépített menekülési út, amely kényszeríti a belépési pontot. A pontos kihagyási üzenetekért lásd az [CLI Referenciát](../reference/cli-reference.md#what-a-captures-folder-looks-like).
{% endhint %}

Még két viselkedés, amit érdemes tudni az újrafeldolgozás szkriptelése során:

* Egy olyan `chloros-cli process` futtatás, amely termékeket kért, de **nem írt ki képtermékeket, hangosan meghiúsul és nem nulla értékkel lép ki** — soha nem kaphat csendes, üres futtatást. A sikeres futtatások jelentik a termékek számát. (Egy szándékosan csak metaadatokra korlátozott futtatás is sikernek számít.)
* Az újraimportált, feldolgozott exportok soha nem foglalják el a rögzítés nyers adatait tartalmazó helyet — az eredeti nyers adat mindig a feldolgozási folyamat forrása marad.

## CLI megfelelői

Az ezen az oldalon található összes funkció vezető nélkül is vezérelhető. A GUI-felvételi módok közvetlenül az `chloros-cli lattice array-capture`-hez rendelődnek:

| GUI | CLI |
| --- | --- |
| Egyszeri | `chloros-cli lattice array-capture` |
| Folyamatos | `array-capture --continuous [--count N] [--duration S]` |
| Intervallum | `array-capture --interval S [--duration S]` |
| Leggyorsabb rögzítés | `array-capture --fastest` |
| Igazítás jelölőnégyzet | `--aligned / --no-aligned` |
| Exporttípusú jelölőnégyzetek | `--processing LEVEL` vagy `--levels L1,L2,…` (alapértelmezett: `all`) |
| Videó rögzítése index alapján | `chloros-cli lattice array-record` |
| Nyers sorozatfelvétel rögzítése / Videó összeállítása | `chloros-cli lattice array-burst` / `array-build-video` |

A teljes jelzőtáblák, az intelligens AE-vel történő rögzítési opció (`--smart`) és a folyamatos sebességű modell a [CLI Hivatkozás § Rögzítési módok, felvevők és offline újrafeldolgozás](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess) című fejezetben találhatók.
