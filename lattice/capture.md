# Felvételi beállítások és módok

A „Kamerák” fülön a felvételkészítést egy piros **Minden felvétel**gomb és egy**Felvételi beállítások** ablak vezérli, amely meghatározza, hogy a gomb milyen eredményt hoz: mely kamerák vesznek részt a felvételben, milyen exportformátumokat ment az egyes kamerák, valamint hogy a zár egyszer, folyamatosan vagy időközönként sül-e el. Ez az oldal a teljes folyamatot ismerteti – a konfigurációt, magát a felvételt, a fájlok lemezre mentésének helyét, valamint azt, hogyan lehet őket később kalibrált termékekké feldolgozni. Maguk a kamera- és tömbvezérlők a [Kamera beállítások](camera-settings.md) oldalon találhatók.

{% hint style="info" %}
**A felvételekhez nyitott projekt szükséges.** Az „Összes rögzítése” és a „Rögzítési beállítások” fogaskerék gombok addig nem használhatók, amíg nincs megnyitva egy projekt („Hozzon létre vagy nyisson meg egy projektet a rögzítések mentéséhez”). Minden rögzítést az `captures/` projektmappába ment a rendszer.
{% endhint %}

## A „Felvételi beállítások” ablak

Nyissa meg az oldalsáv kameralistáján található **„Minden felvétel” melletti fogaskerék**gombbal, vagy bármely kamera-specifikus beállítási ablak alján található**„Felvételi beállítások megnyitása…”** gombbal. A fejlécen a „Felvételi beállítások” felirat látható, mellette egy ← vissza gombbal.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Az itt megadott beállítások – a bevont kamerák, a típusonkénti jelölőnégyzetek és a rögzítési mód – **projektenként** kerülnek mentésre, és a projekt újbóli megnyitásakor visszaállnak.

### Rögzítési módok

Három módgomb található az ablak tetején:

| Mód | Működése | Albeállítások (alapértelmezett) |
| --- | --- | --- |
| **Egyszeri** *(alapértelmezett)* | Egyetlen rögzítés az összes kiválasztott kamerán. | — |
| **Folyamatos**| Egymást követő rögzítések egy leállási feltétel teljesüléséig. | Leállás**felvételek száma** alapján (alapértelmezett: 1) *vagy* **felvételi időtartam** alapján (alapértelmezett: 10 s; mértékegységek: másodperc / perc / óra / nap). |
| **Intervallum**(időfelvétel) | Időzített sorozatfelvételek. |**Felvételek / intervallum**(alapértelmezés: 1) ·**Minden**N egység után (alapértelmezés: 5 másodperc) ·**N** egységig (alapértelmezés: 1 perc). |

Folyamatos vagy Intervallum módban a „Minden felvétel” gomb futás közben **Leállítás (N)** gombbá válik, és számolja a beérkező felvételeket. Egy teljes meghajtó leállítja a futást (lásd [Az „Összes rögzítés” folyamat](#the-capture-all-flow)).**A tömbökön történő folyamatos rögzítés a háttérben fut (új a 1.3.0-ban).** Ha a listában szereplő összes kamera egy tömbhöz tartozik (az önálló kamerák el vannak rejtve vagy kizárva), a „Folyamatos” üzemmód már nem küld minden rögzítés után egy-egy kérést: a Chloros háttérrendszer maga futtatja a rögzítési ciklust, egy szinkronizált csoportot írva, miközben kiválasztja a következőt, és a gomb lekérdezi a számlálót. Ha a listában önálló kamerák is szerepelnek, a Chloros a korábbiakhoz hasonlóan passzonként rögzít.

{% hint style="warning" %}
**Gyors rögzítés Windows rendszeren: a Chloros háttérprogramot kizárva kell tartani a Microsoft Defender vizsgálatából.** A nagy sebességű rögzítés során írt minden fájl valós idejű vizsgálata a MAPIR tesztjei szerint a rögzítési sebesség körülbelül 30 %-át veszi igénybe (másodpercenként több száz fájl egy kis ROI esetén). A Windows-telepítő felajánlja a **„Chloros-háttérprogram kizárása a Microsoft Defender vizsgálatából (sokkal gyorsabb kamerarögzítés; ajánlott)”** opciót, amely alapértelmezés szerint be van jelölve, és az eltávolításkor eltávolítja a kizárást. Ha ezt a jelölőnégyzetet törölte, futtassa újra a telepítőt bejelölt állapotban, vagy adjon hozzá egy Defender-folyamatkizárást az `chloros-backend.exe` fájlra (a Chloros telepítőmappa `backend` mappájában). Amikor a Defender egy vagy több CPU-magot használ egy tömb folyamatos rögzítése közben, a Chloros-háttérprogram ezt egyszer rögzíti a naplójában, ugyanezekkel a két megoldással.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Kamerák és exporttípusok kiválasztása

A panel súgószövege összefoglalja: válassza ki, mely kamerák és exporttípusok jelenjenek meg az „Összes rögzítése” funkcióval – alapértelmezés szerint minden be van kapcsolva, és a választások a projekthez kerülnek mentésre.

* **Minden kijelölése / Semmi kijelölése** gombok egyszerre kapcsolják be vagy ki az összes kamera beállítási jelölőnégyzetét.
* **Tömeges exporttípus-kapcsolók**(két sor gomb):**Minden Raw / Minden Debayered / Minden Előnézet / Minden Sugárzás / Minden Fényvisszaverődés / Minden Index**. Mindegyik háromállapotú színkóddal rendelkezik: zöld ✓ = be van kapcsolva minden olyan kameránál, amely támogatja, sárga – = néhánynál be van kapcsolva, szürke = egyiknél sem. Egy kapcsoló akkor van letiltva, ha egyetlen csatlakoztatott kamera sem támogatja az adott típust. A „Fastest Capture” bekapcsolt állapotában mindegyik szürkére vált.
* **Kameránkénti sorok**: egy „Beleértés” jelölőnégyzet, valamint az adott kamera alkalmazható exporttípusainak kibontható (▸/▾) listája, egyedi jelölőnégyzetekkel. A sorban a bekapcsolt elemek számát jelzi, például „4/6”.

### Exporttípusok és azok támogatottsága a kameráknál

Hat exporttípus létezik: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Az egyes kamerák sorában csak az adott kamerára vonatkozó típusok jelennek meg:

| Exporttípus | Tartalom | RGB (FRGB) | Bayer multispektrális (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Bayer-mozaik (mono: az egyetlen sáv) közvetlenül az érzékelőből | ✓ | ✓ | ✓ |
| **Debayered** | Lineáris demosaik: a kalibrációs csomag sötételtolása (DSNU) eltávolítva a nyers adatból, majd élek figyelembevételével végzett demosaik — lineáris korrekció, síkkép-korrekció vagy színszétválasztás nélkül. 3-csatornás RGB TIFF-ként mentve (mono: 1-csatornás szürkeárnyalatos) | ✓ | ✓ | ✓ |
| **Előnézet** | Teljes megjelenítési lánc (fehér egyensúly + gamma a kamera profilja szerint, valamint az RGB-kamera „Highlights” beállítása; multispektrális: hamis színű kiterjesztés) | ✓ | ✓ | ✓ |
| **Sugárzás** | float32 W/m²/sr/nm a teljes radiometrikus láncon keresztül | — (nem elérhető) | ✓ | ✓ |
| **Fényvisszaverődés** | uint16 ρ (32768 = 1,0) | — (nem elérhető) | ✓ — csak akkor jelenik meg, ha a kamera rendelkezik DAQ fényérzékelővel (saját vagy a sorozatból örökölt) | ugyanaz, mint a multispektrális |
| **Index** | Vegetációs index (LUT) megjelenítés | — | ✓ — a kamerán engedélyezett, nem üres indexkifejezést igényel, és kombinált sorozat tagjai számára nem elérhető (a sorozat egy közös indexszel rendelkezik) | — (az indexhez ≥2 sáv szükséges; lásd [Mono kamerák és vegetációs indexek](mono-indices.md)) |

A sugárzás és a visszaverődés soha nem elérhető az RGB-kameráknál — a Bayer-mátrixonkénti sugárzás szélessávú fotometrikus érzékelő esetén nem értelmezhető.

### Leggyorsabb rögzítés

A **⚡ Leggyorsabb rögzítés — csak nyers**kapcsoló (bekapcsolt állapotban narancssárga) felülírja az összes exportálási beállítást**csak nyers** formátumra — plusz egy ingyenes kombinált indexű kompozíciót a tömbökhöz — így a képkocka a lehető leggyorsabban kerül mentésre: a sugárzás/reflektancia/megjelenítési számításokat a rögzítéskor teljesen kihagyják.

{% hint style="info" %}
**Egy `.daq` fájl továbbra is elmentésre kerül.** Ha fényérzékelő van hozzárendelve, a „Leggyorsabb rögzítés” továbbra is írja a DAQ lefelé irányuló mérési értékét (a rögzítések `daq/` mappájába) — így a sugárzási, visszaverődési és indextermékek később újrafeldolgozással mind létrehozhatók (lásd [Felvételek újrafeldolgozása](#re-processing-captures-into-calibrated-products)). A Fastest Capture funkció nem írja felül a jelölőnégyzetekben megadott beállításokat: ha kikapcsolja, azok visszaállnak.
{% endhint %}

### Tömbönkénti vezérlők

Minden csatlakoztatott tömb saját csoportkártyát kap az ablaktáblában:

* **Beleértés** jelölőnégyzet (háromállapotú az elemek között) és a tömb neve a megjelenítési módjával: „(összevont | külön)”.
* **Igazítás**jelölőnégyzet (alapértelmezés szerint**be**): a tagfájlokat a tömb igazítási profiljához igazítja, így az exportált fájlok pixel-szinten regisztrálva vannak a kamerák között. A nyers adatok torzításmentesek maradnak, de a transzformációt a metaadatokban hordozzák. (Maga a profil az [array beállítások ablakban](camera-settings.md#alignment-co-registration) kerül kiszámításra.) Az array tagfájljait az egyes kamerák natív tájolásában menti el a rendszer, a Tükör/Forgatás beállításokat pedig a fájlba rögzíti, nem pedig alkalmazza.
* Az elemkamerák sorait a kártya belsejében helyezik el.

A tömbkártya két felvevőt is tartalmaz. Gondoljon rájuk úgy, mint **figyelés kontra elemzés**:

| Felvevő | Osztály | Mit rögzít |
| --- | --- | --- |
| **● Indexvideó rögzítése / ■ Felvétel leállítása** *(csak kombinált tömbök esetén)* | **Megfigyelés** | Az élő, kombinált indexkompozíciót 10 fps sebességgel videóként rögzíti — 8 bites, előnézeti felbontás, beépített LUT. Nyitott projektre és élő streaming nézetre van szükség. A felvétel közben megjeleníti a képkockákat és az eltelt időt. |
| **⦿ Nyers sorozatfelvétel / ■ Nyers sorozatfelvétel leállítása** *(bármely sorozat)* | **Elemzés**| Nyers Bayer-képkockák az élő rögzítési sebességgel (feldolgozás nélkül), valamint képkockánkénti manifeszt és `.daq`-értékek, `captures/bursts/` formátumban. A sorozatfelvétel után megjelenik a**Videó készítése** gomb: ez offline módon újrafeldolgozza a sorozatot kalibrált videóvá — kombinált index és/vagy kameránkénti sugárzás / visszaverődés / index — valamint opcionális TIFF-fájlokká. A kombinált indexű videó készítése automatikusan elindul, amikor leállítja a sorozatfelvételt. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## Az „Összes rögzítése” munkafolyamat

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->



Nyomja meg az **Összes rögzítése** gombot az oldalsáv kameralistájában:

1. Minden felvett, látható, szüneteltetés nélküli kamera a kiválasztott exporttípusokkal rögzít. **A kameracsoportok egy szinkronizált indításként működnek** (egyetlen, az összes tagra kiterjedő szinkronizált csoport — lásd [Többkamerás csoportok](arrays.md)); az önálló kamerák egyenként rögzítenek.
2. A rejtett (szem) vagy szüneteltetett kamerákat a rendszer kihagyja. Egy tömb csak akkor blokkolódik teljesen, ha *minden* tagja rejtett vagy szüneteltetett állapotban van.
3. Ha fényérzékelő van hozzárendelve, a hozzá tartozó DAQ lefelé irányuló mérési értéket `.daq` fájlként menti a felvételek `daq/` mappájába — még a kizárólag nyers adatokat tartalmazó felvételek esetében is —, így a radiometrikus termékek később bármikor kiszámíthatók.
4. A gomb élőben jelzi az előrehaladást — „Rögzítés… kész/összesen” —, és folyamatos/intervallum módban **Stop (N)**-re vált. Minden rögzítési elemnek 300 másodperces időkorlátja van.
5. Amikor a futtatás befejeződik, egy eredményüzenet jelzi:**„N fájl mentve”**vagy**„N mentve, F sikertelen”**, valamint „(S elrejtve/szüneteltetve/kihagyva)”, ha kamerákat hagytak ki. Ha a meghajtó megtelik, a futtatás leáll, és a következő üzenet jelenik meg:**„Leállítva: a meghajtó megtelt. N mentve, F sikertelen”** — minden olyan fájlt, amelyet nem sikerült írni, sikertelennek számítanak.

## A rögzítések mentési helye

A rögzítéseket a megnyitott projekt alatt az `<project>/captures/` mappába menti a rendszer. Minden exporttípus a **saját almappájába** kerül, így egy többszintű rögzítés során a típusok soha nem keverednek:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* Az `<ts>` a rögzítés időbélyege, az `<serial>` pedig a kamera sorozatszáma. Az önálló rögzítések neve `capture_<ts>_SN<serial>_<level>`; az egy szinkronizált kioldóból származó tömbös rögzítések neve `sync_<ts>_SN<serial>_<level>`, és **a csoport összes kamerájára közös időbélyeggel rendelkeznek** (a szint utótagot elhagyják, ha egy kamera csak egy szintet ment).
* **Egy tudnivaló eltérés:** a megjelenítési szint az `preview/` nevű mappában kerül tárolásra, míg a fájlok neve továbbra is `_display` marad — a mappa és az utótag csak ezen a szinten tér el egymástól.
* Az ismeretlen szintek a saját nevüket viselő mappába kerülnek; ha nem lehet almappát létrehozni, a fájl nem vész el, hanem a felvételek gyökérmappájába kerül.
* A TIFF-felvételek alapértelmezés szerint veszteségmentesen vannak tömörítve (DEFLATE), és a teljes kalibrációs és feldolgozási metaadatokat **a fájl XMP-jében** — a felvételek önleíróak, az `.daq` olvasási fájlon (és annak `.daq.cameras.json` fájlján) kívül nincsenek kiegészítő fájljaik.
* **Clamp-maszkok (újdonság az 1.3.0-ban).**Ahol a 3×3-as spektrális szétválasztásnak legalább egy képponton 0-ra kellett korlátoznia egy sávot, a sugárzási, visszaverődési és indextermékek képpontonkénti maszkot kapnak, `clamp_masks/<product folder>/<product name>_clampmask.tif` (0. bit = a termék R-csatornájában lévő sáv, 1. bit = G, 2. bit = B); a termék `Chloros:UnmixClampMask` XMP-címkéje erre mutat. A maszkok a termékmappák**mellett** találhatók, soha nem azok belsejében, így egy olyan termékmappa, mint például az `reflectance/`, változatlan formában átadható a fotogrammetriai szoftvernek. Az Chloros saját importálója kihagyja a maszkokat.
* **Tájolás (1.3.0).** Egy önálló kamera Preview, Radiance, Reflectance és Index termékei a Mirror/Rotation beállítás szerint forognak; a Raw és a Debayered fájlok úgy maradnak, ahogyan az érzékelő kiolvasta őket. A tömb tagfájljai mind natív formátumban maradnak. Minden fájl rögzíti a beállítást az XMP-jében, így az újrafeldolgozás is ugyanúgy alkalmazza azt.
* **Melyik kamerához tartozik egy `.daq` fájl (1.3.0).** Egy felvétel `.daq` azonosítóját a feldolgozás során kizárólag azokhoz a kamerákhoz tartozó képekhez rendeli hozzá, amelyeket az `.daq.cameras.json` azonosítója megnevez. Egy önálló kamera, amelynek nincs saját fényérzékelője, ezért már nem veszi át egy másik kamera mérési adatait — rendeljen hozzá egy érzékelőt, vagy rögzítsen egyet a Fényérzékelők fülön. Egy fényérzékelő-bejegyzés (és bármely `.daq`, amelyben nincs ilyen fájl, beleértve a régebbi rögzítéseket is) továbbra is minden kamerát kiszolgál.

Ez ugyanaz az elrendezés, amelyet az `chloros-cli lattice capture` / `array-capture` fájlok írnak az `-o` könyvtárba — dokumentálva a [CLI-referencia § Hogyan néz ki egy rögzítési mappa](../reference/cli-reference.md#what-a-captures-folder-looks-like) című részben dokumentálva.

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## A rögzítések újrafeldolgozása kalibrált termékekké

A rögzített nyers képkockák és a mentett `.daq` fájl minden, amire a feldolgozási folyamatnak szüksége van — ezért is használható a Fastest Capture biztonságosan valódi munkához.

* **GUI**: vegye fel a rögzítések mappáját egy projektbe ([Fájlok hozzáadása egy projekthez](../processing-images-gui/adding-files-to-a-project.md)), majd dolgozza fel a szokásos módon.
* **CLI**: irányítsa az `process` fájlt a **rögzítések gyökérmappájára**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

Az `process` általában csak a megadott mappát importálja, de ha az a mappa nem tartalmaz képeket, viszont almappákkal rendelkezik, akkor automatikusan lefelé halad – így az almappák szintjei és az `daq/` mappa egy lépésben kerülnek beolvasásra. Minden rögzítés **egyetlen képként** történik, a többi szint pedig megtekinthető módként csatolódik hozzá, nem pedig szintenként egy-egy kép formájában.

A szint almappájának közvetlen megnevezése (pl. `…/captures/raw/`) szintén működik, de az `daq/` mappát hátrahagyja — másolja át mellé, amikor-radiometrikus terméket származtat az `raw/`-ből, különben az időbélyeg-egyeztetésnek nincs mihez viszonyítania.

{% hint style="warning" %}
**A feldolgozás mindig az `raw`-től indul.** Minden felvételen belül a nyers képkocka a feldolgozási folyamat forrása; az `debayered`, `radiance`, `reflectance`, és az `preview` megjeleníthető módokként jelennek meg, de soha nem kerülnek vissza a feldolgozási folyamatba — egy származtatott termék újrafeldolgozása újra alkalmazná a pixelekbe már beépített vignettázást, színeket és sugárzási számításokat, ezért az Chloros inkább elutasítja, mintsem kétszer feldolgozza. Az `index/` és az `composite/` rendereléseket egyáltalán nem dolgozza fel a rendszer (ezek kimenetek, nem rögzítések). A „captures” mappa **nyers adatok importálása nélkül** is normálisan menthető és jeleníthető meg, de az `process` kihagyja, és erről értesítést is ad; az `--input-level {raw,debayered,processed}` egy szándékosan beépített menekülési út, amely kényszeríti a belépési pontot. A pontos kihagyási üzeneteket lásd a [CLI-referenciában](../reference/cli-reference.md#what-a-captures-folder-looks-like).
{% endhint %}

Még két viselkedés, amit érdemes tudni az újrafeldolgozás szkriptelése során:

* Egy olyan `chloros-cli process` futtatás, amely termékeket kért, de **nem írt ki képtermékeket, hangosan meghiúsul és nem nulla kóddal lép ki** — soha nem kaphat csendes, üres futtatást. A sikeres futtatások jelentik a termékek számát. (Egy szándékosan csak metaadatokra korlátozott futtatás is sikernek számít.)
* Az újraimportált, feldolgozott exportok soha nem foglalják el a rögzítés nyers adatsorát — az eredeti nyers adat mindig a feldolgozási folyamat forrása marad.

## CLI-megfelelők

Az ezen az oldalon található összes művelet parancssorból is végrehajtható. A grafikus felület (GUI) rögzítési módjai közvetlenül az `chloros-cli lattice array-capture` parancshoz rendelődnek:

| GUI | CLI |
| --- | --- |
| Egyszeri | `chloros-cli lattice array-capture` |
| Folyamatos | `array-capture --continuous [--count N] [--duration S]` |
| Intervallum | `array-capture --interval S [--duration S]` |
| Leggyorsabb rögzítés | `array-capture --fastest` |
| Igazítás jelölőnégyzet | `--aligned / --no-aligned` |
| Exporttípusú jelölőnégyzetek | `--processing LEVEL` vagy `--levels L1,L2,…` (alapértelmezett: `all`) |
| Indexvideó rögzítése | `chloros-cli lattice array-record` |
| Nyers sorozatfelvétel rögzítése / Videó összeállítása | `chloros-cli lattice array-burst` / `array-build-video` |

A teljes jelzőtáblák, az intelligens AE-vel történő rögzítés opció (`--smart`) és a folyamatos sebességű modell leírása a [CLI-referencia § Rögzítési módok, rögzítők és offline újrafeldolgozás](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess) című részben található.
