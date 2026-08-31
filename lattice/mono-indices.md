# Monokróm kamerák és vegetációs indexek

## Egy kamera = egy sáv

Az **M3M**kamera a Bayer**M3C**monokróm változata: egy IMX265 monokróm érzékelő, amely egy keskenysávú interferenciaszűrő mögött helyezkedik el. A modellnév jelöli a sávot — `M3M-<lens>-F<wavelength>`, pl. `M3M-L87-F685` (az Chloros-ben `LATT-M3M-L87-F685` néven jelenik meg). Az érzékelő**egyetlen szürkeárnyalatos sávot** biztosít, Bayer-mozaik nélkül: nincs mit demozikálni, nincs csatornák közötti átvitel, amit szét kellene választani, és nincs fehéregyensúly, amit be kellene állítani.

A monokróm rendszer tervezése előtt érdemes tudni a következőket:

* **A sugárzás és a visszaverődés sávonként teljes mértékben definiált.**Ezek sávonkénti radiometrikus térképek, így egy M3M kamera kalibrált float32 sugárzási értéket (W/m²/sr/nm) és uint16 visszaverődési értéket (`32768` = ρ 1,0) ad, pontosan úgy, mint egy M3C sáv. A monokróm képkockák**azonos** érzékelő-válaszmátrixot hordoznak – nincs szükség 3×3-as keverésmentesítésre, és azt nem is alkalmazzák.
* **Egyetlen monokróm kamera nem képes növényzetindexet előállítani.** Az NDVI, NDRE és társaikhoz legalább két sávra van szükség. A monokróm hardverből származó indexek kiszámításához több M3M kamerát kell kombinálni – lásd alább.
* Az M3M kamerák **Mono12** adatfolyamot továbbítanak (12 bit, 2 bájt/pixel a vezetéken), ami fontos az [adatmátrix sávszélesség-tervezés](arrays.md#bandwidth-the-rules-of-thumb) szempontjából.

## Mit hagy ki az Chloros a monó esetén – és hogyan jelzi ezt

A színes feldolgozási lépések egyszerűen nem vonatkoznak az egysávos érzékelőkre. Az Chloros **egy soros üzenettel hagyja ki őket**, ahelyett, hogy hibát jelezzen, és ugyanabban a munkamenetben bármely M3C (Bayer) kamera esetében továbbra is normál módon futtatja őket:

| Szakasz | Mono (M3M) viselkedés | M3C viselkedés |
| --- | --- | --- |
| Demosaic / debayer | Kihagyva — az `debayered` exportált kép egycsatornás szürkeárnyalatos kép. | 3-csatornás demosaic. |
| Fehér egyensúly (`lattice white-balance`) | Egy soros üzenettel kihagyva. | Normálisan fut. |
| Színprofil (`lattice color-profile`) | Egy soros üzenettel kihagyva. | Normálisan fut. |
| Telítettség/kontraszt (`lattice color`) | Egy soros üzenettel kihagyva. | Normálisan fut. |
| Spektrális keresztbeszélés szétválasztása | Identitás (nincs 3×3-as mátrix). | Kameránkénti 3×3-as mátrix alkalmazva. |
| Sugárzás / reflexió | **Fut** — sávonként, teljesen kalibrálva. | Sávonként fut. |

A grafikus felhasználói felület ugyanazt a szűrést alkalmazza: monokróm kamera esetén a kameránkénti beállítási ablak elrejti az RGB-hez tartozó sorokat (fehér egyensúly, gamma, színprofil, telítettség, kontraszt, csatornafelosztás), és az élő hisztogramot egyetlen **MONO** görbére rögzíti. A halmaz egészében a megkülönböztető a modell karakterláncban szereplő `M3M` token, amely a GUI/SDK-ben `is_mono` néven jelenik meg.

## Az indexekhez legalább 2 sáv szükséges: igazítás → halmozás → indexelés

A monokróm indexelési munkafolyamat mindig ugyanaz a három lépés:

1. **Igazítás** — több M3M kamerát kell különböző hullámhosszokra irányítani (pl. egy F650 „Red” és egy F850 „NIR”) különböző hullámhosszokra, csatlakoztassa őket [többkamerás rendszerként](arrays.md), majd hagyja, hogy az Chloros kiszámítsa a kamerák közötti ko-regisztrációs torzítást.
2. **Stack** — az igazított képkockák egy többsávos képpé válnak (minden kamera egy-egy megnevezett sávot ad hozzá).
3. **Index** — egy indexképletet értékelünk ki a stack sávjain, opcionálisan LUT-on keresztül renderelve azt.

A grafikus felhasználói felületen ez az egész lánc a **Kombinált kamerák**tömb megjelenítési mód: az élő kompozíció már igazítva van, és a tömb Indexkalkulátora (lent) határozza meg a rendereléshez használt képletet. A rögzített exportok az**Igazított** rögzítési opcióval ugyanarra az igazításra torzíthatók.

## Az Index Calculator

Az Index Calculator állítja össze az élő nézet és a kameránkénti index-exportok által használt indexkifejezést. Ez egy közös felület, amely a Cameras fül oldalsávjának két helyéről nyitható meg:

* **Kameránként**— Élő előnézet →**Index** fogaskerék (csak RGN/OCN/NGB Bayer kamerák; egy önálló monokróm kamera nem rendelkezik index-vezérléssel, mivel egy sávból nem lehet indexet létrehozni).
* **Tömbönként**— tömbbeállítások → Élő előnézet →**Index**fogaskerék. Ez a monokróm útvonal: a sávlista**az összes tagkamerára** kiterjed, így egy monokróm pár itt mindkét sávjával hozzájárul.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

A vezérlői, fentről lefelé:

* **Sávjelölők** („Sávok — kattintson a kifejezéshez való hozzáadáshoz”) — egy gomb minden elérhető sávhoz, a szín neve + hullámhossz nm-ben feltüntetve (az ismétlődő színnevek egyértelműsítve vannak, pl. „Color 850”). Kattintással a sávjelző beilleszthető a kurzor helyére. Azok a kamerák sávjai, amelyek nem képesek sávonkénti sugárzási intenzitást előállítani (RGB/FRGB), kiszűrésre kerülnek.
* **Műveleti és függvénygombok** — `+ - * / ( ) ^ ,` és `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Kifejezés szövegmező** — szabadon beírható képlet; a helyőrző a klasszikus NDVI formátumot, azaz `(NIR - Red) / (NIR + Red)`-et jeleníti meg. A felette található, csak olvasható, tokenizált előnézet a sávjelölőket, számokat és jelzőket ismeretlen tokenekként jeleníti meg.
* **Érvényességi sáv**— szürke „Üres — index nem kerül alkalmazásra”; zöld „Érvényes kifejezés”; piros a konkrét elemzési hibával (ismeretlen sáv, több kamera által rögzített, egyértelműtlen sáv, hiányzó zárójel, …); vagy sárga, ha a kifejezés érvényes, de**állandó** (pl. `X/X`, vagy egy NDVI nevező, amelyet `−`-ként írtak be `+` helyett) — egy konstans a teljes képkockát egyetlen színre képezi le.
* Külön sárga figyelmeztetés jelenik meg, ha az alkalmazott kifejezés helyes, de az **élő képkocka egyenletes** (lapos vagy telített jelenet) — a hisztogram összeomlását a rendszer automatikusan észleli.
* **LUT alkalmazása**(alapértelmezés szerint be van kapcsolva; ki = szürkeárnyalatos kiterjesztés),**Szint**2/3/5/7-stop (alapértelmezés szerint 7-stop), valamint a**Min / Max**beállítások a színátmeneti sáv két oldalán. A Min alapértelmezett értéke**0,2**— ez a színátmenetet a növényzet szempontjából releváns tartományra nagyítja, míg az ennél alacsonyabb értékek szürkeárnyalatként jelennek meg; állítsa a Min értéket −1-re a teljes index-tartományhoz (a**Reset** gomb visszaállítja a −1…+1 tartományt). A Max alapértelmezett értéke 1.
* Az indexeloszlás **élő hisztogramja** — négyzetgyök-skálázott oszlopok, borostyánszínű p2/p98 percentilis vonalak, egy fehér mediánvonal, valamint a tartományon kívüli értékeket jelző kijelzések („◀ N% &lt; lo” / „hi &lt; N% ▶”), amelyek 1 % felett borostyánszínűvé válnak, jelezve, hogy szélesíteni kell a Min/Max ablakot.
* Az **Alkalmaz**gomb az expressziót az élő adatfolyamra alkalmazza; a LUT-beállítások az Alkalmaz gomb megnyomása nélkül is azonnal hatályba lépnek. Az expressziók szándékosan**csak a munkamenetre vonatkoznak** — a munkamenetek között nem maradnak meg.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## Az CLI útvonal

Ugyanaz az igazítás → verem → index lánc, végpontok között szkriptelhető:

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` leképezi az előre beállított szimbólumokat a verem sávneveire. Két szabály segít elkerülni a futtatás sikertelenségét:

* **A szimbólumoknál a kis- és nagybetűk közötti különbséget figyelembe kell venni**, és pontosan meg kell egyezniük a preset csatornaneveivel — a presetek kisbetűket használnak (az NDVI-ek az `red`-ek,`nir`; ellenőrizze az `--list-presets`-et). Az `--channel red=Red_660` működik; az `--channel RED=660` hibaüzenettel bukik: „`channel_map missing entries`”.
* A sávoldalnak meg kell neveznie egy sávot az igazított veremben (az `lattice align-info --profile align.json` felsorolja őket). Az offline mód 0-tól kezdődő sávindexeket is elfogad, pl. `--channel red=0 --channel nir=1`.

Az `lattice index` egy mentett, igazított többsávos TIFF fájlra is teljesen offline módban fut:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Index-előbeállítások

Az `lattice index --preset` (és a Kép fül [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) menüpontja, amely ugyanazt a motort használja) a következő **22 előbeállítást** tartalmazza:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Futtassa az `chloros-cli lattice index --list-presets` parancsot az egyes előre beállított értékek képleteinek és csatornaszimbólumainak megtekintéséhez, az `--list-gradients` parancsot pedig a rendelkezésre álló színátmenetek megtekintéséhez. Az egyéni képletekhez az `--formula EXPR` parancsot kell használni, ugyanazzal a szintaxissal, mint az Index Calculator esetében. Figyelem: ez az előre beállított lista kifejezetten a LATTICE indexmotorhoz tartozik — az importált képek „Project Settings” feldolgozási legördülő menüje egy másik listát tartalmaz (lásd [Multispektrális indexképletek](../project-settings/multispectral-index-formulas.md)).

A teljes jelzőkészlet (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, az `--live` igazító torzítócsapjai, és még sok más) az [CLI Referencia § Index / Vegetációs matematika](../reference/cli-reference.md#index--vegetation-maths) című dokumentumban található; az SDK megfelelői az [SDK Referencia](../reference/sdk-reference.md) című dokumentumban találhatók.

## Indextermékek rögzítése egy mono-tömbből

Ha egy tömb csatlakoztatva van és egy indexkifejezés van alkalmazva, az `array-capture` (vagy a GUI **Capture All**) funkció elmenti a kameránkénti exportszinteket *és* az index-renderelést — az `--index`/`--no-index` az CLI-en kapcsolja be ezt a funkciót, és alapértelmezés szerint minden alkalmazható szintet rögzít. Egy monokamera hozzájárulása az egyes rögzítési csoportokhoz az egy sávja nyers/debayered (szürkeárnyalatos)/sugárzási/visszaverődési szinteken, valamint a megosztott kombinált index-kompozíció, amikor a tömb kombinált módban fut. Lásd [Többkamerás rendszerek § Rögzítés](arrays.md#capturing-monitoring-vs-analysis).
