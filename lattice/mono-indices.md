# Monokróm kamerák és vegetációs indexek

## Egy kamera = egy sáv

Az **M3M**kamera a Bayer**M3C** monokróm változata: egy IMX265 monokróm érzékelő, amely egy keskenysávú interferenciaszűrő mögött helyezkedik el. A modellnév jelöli a sávot — `M3M-<lens>-F<wavelength>`, pl. `M3M-L87-F685` (a Chloros

-ban `LATT-M3M-L87-F685` néven jelenik meg). Az érzékelő **egyetlen szürkeárnyalatos sávot** biztosít, Bayer-mozaik nélkül: nincs mit demozikálni, nincs csatornák közötti átvitel, amit szét kellene választani, és nincs fehér egyensúly, amit be kellene állítani.

A monokróm rendszer tervezése előtt érdemes tudni a következőket:

* **A sugárzás és a visszaverődés sávonként teljes mértékben definiált.**Ezek sávonkénti radiometrikus térképek, így egy M3M kamera kalibrált float32 sugárzási értéket (W/m²/sr/nm) és uint16 reflexiós értéket (`32768` = ρ 1,0) ad, pontosan úgy, mint egy M3C sáv. A monokróm képkockák**azonos** érzékelő-válaszmátrixot hordoznak – nincs szükség 3×3-as szétválasztásra, és azt nem is alkalmazzák.
* **Egyetlen monokróm kamera nem képes növényzetindexet előállítani.** Az NDVI

, NDRE

és hasonló indexekhez legalább két sávra van szükség. Az indexek monokróm hardverről történő kiszámításához több M3M kamerát kell kombinálni – lásd alább.
* Az M3M kamerák **Mono12** adatfolyamot továbbítanak (12-bites: 2 bájt/pixel a vezetéken, vagy 1,5 bájt/pixel, ha a sorozat [tömörített 12-bites átvitelt](arrays.md#packed-12-bit-transport) használ), ami fontos a [sorozat sávszélesség-tervezése](arrays.md#bandwidth-how-many-cameras-fit).

## Mit hagy ki aChloros

a monó esetén — és hogyan jelzi ezt

A színes feldolgozási lépések egyszerűen nem alkalmazhatók egycsatornás érzékelőre. A Chloros

**egy soros üzenettel hagyja ki őket**, ahelyett, hogy hibát jelezzen, és ugyanabban a munkamenetben bármely M3C (Bayer) kameránál továbbra is normál módon futtatja őket:

| Szakasz | Mono (M3M) viselkedés | M3C viselkedés |
| --- | --- | --- |
| Demosaic / debayer | Kihagyva — az `debayered` exportszint egycsatornás szürkeárnyalatos kép. | 3-csatornás demosaic. |
| Fehéregyensúly (`lattice white-balance`) | Egy soros üzenettel kihagyva. | Normálisan fut. |
| Színprofil (`lattice color-profile`) | Egy soros üzenettel kihagyva. | Normálisan fut. |
| Telítettség/kontraszt (`lattice color`) | Egy soros üzenettel kihagyva. | Normálisan fut. |
| Spektrális átvitel-szétválasztás | Identitás (nincs 3×3-as mátrix). | Kameránkénti 3×3-as mátrix alkalmazva. |
| Sugárzás / visszaverődés | **Futtatva** — sávonként, teljesen kalibrálva. | Sávonként fut. |

A grafikus felhasználói felület ugyanazt a szűrést alkalmazza: monokróm kamera esetén a kameránkénti beállítások panele elrejti a kizárólag RGB

-re vonatkozó sorokat (fehér egyensúly, gamma, színprofil, telítettség, kontraszt, csatornafelosztás), és az élő hisztogram egyetlen **MONO** görbére van rögzítve. A halmaz egészében a megkülönböztető jel a modell karakterláncban szereplő `M3M` token, amely a grafikus felhasználói felületen (GUI) és az SDK

-ben `is_mono` néven jelenik meg.

## Az indexekhez legalább 2 sáv szükséges: igazítás → halmozás → indexelés

A mono index munkafolyamat mindig ugyanaz a három lépés:

1. **Igazítás** — irányítson több M3M kamerát különböző hullámhosszokra (pl. egy F650 „Red

” és egy F850 „NIR

”), csatlakoztassa őket [többkamerás rendszerként](arrays.md) formájában, majd hagyja, hogy az Chloros

kiszámítsa a kamerák közötti ko-regisztrációs torzítást.
2. **Összeállítás** — az igazított képkockák egy többsávos képpé válnak (minden kamera egy-egy megnevezett sávot ad hozzá).
3. **Index** — kiértékel egy indexképletet a halmaz sávjain, opcionálisan LUT-on keresztül renderelve azt.

A grafikus felhasználói felületen ez az egész lánc a **Kombinált kamerák**tömb megjelenítési mód: az élő kompozíció már igazítva van, és a tömb Indexkalkulátora (lent) határozza meg a rendereléshez használt képletet. A rögzített exportok az**Aligned** rögzítési opcióval ugyanarra az igazításra torzíthatók.

## Az Index Calculator

Az Index Calculator állítja össze az élő nézet és a kameránkénti index-exportok által használt indexkifejezést. Ez egy közös felület, amely a Cameras fül oldalsávjának két helyéről nyitható meg:

* **Kameránként**— Élő előnézet →**Index** fogaskerék (csak RGN

/OCN

/NGB

Bayer kamerák esetén; egy önálló monokróm kamera nem rendelkezik indexvezérléssel, mivel egy sávból nem lehet indexet létrehozni).
* **Tömbönként**— tömbbeállítások → Élő előnézet →**Index**fogaskerék. Ez a monokróm útvonal: a sávlista**az összes tagkamerára** kiterjed, így egy monokróm pár itt mindkét sávjával hozzájárul.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Vezérlői, fentről lefelé:

* **Sávchipek** („Sávok — kattintson a kifejezéshez való hozzáadáshoz”) — minden elérhető sávhoz egy gomb tartozik, amelyen a szín neve és a hullámhossz nm-ben szerepel (az azonos nevű színek megkülönböztetése például „Color 850” formában történik). A gombra kattintva a sávjelölő a kurzor helyére kerül. Azok a kamerák sávjai, amelyek nem képesek sávonkénti sugárzási értéket előállítani (RGB

/FRGB), kiszűrésre kerülnek.
* **Operátor- és függvénygombok** — `+ - * / ( ) ^ ,` és `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Kifejezés szövegmező** — szabadon beírható képlet; a helyőrző a klasszikus NDVI

-formát, az `(NIR - Red) / (NIR + Red)`-et jeleníti meg. A felette található, csak olvasható, tokenekre bontott előnézet a sávjelölőket, számokat és jelzőket ismeretlen tokenekként ábrázolja.
* **Érvényességi sáv**— szürke „Üres — index nem kerül alkalmazásra”; zöld „Érvényes kifejezés”; piros a konkrét elemzési hibával (ismeretlen sáv, több kamera által rögzített, egyértelműtlen sáv, hiányzó zárójel, …); vagy sárga, ha a kifejezés érvényes, de**állandó** (pl. `X/X`, vagy egy NDVI

-nevező, amelyet `−`-ként írtak be `+` helyett) — egy konstans a teljes képkockát egyetlen színre képezi le.
* Külön sárga figyelmeztetés jelenik meg, ha az alkalmazott kifejezés rendben van, de az **élő képkocka egyenletes** (lapos vagy telített jelenet) — a hisztogram összeomlását a rendszer automatikusan észleli.
* **LUT alkalmazása**(alapértelmezés szerint be van kapcsolva; ki = szürkeárnyalatos kiterjesztés),**Szint**2/3/5/7-stop (alapértelmezés szerint 7-stop), valamint a**Min / Max**beállítások a színátmeneti sáv két oldalán. A Min alapértelmezett értéke**0,2**— ez a színátmenetet a növényzet szempontjából releváns tartományra nagyítja, míg az alatti értékek szürkeárnyalatként jelennek meg; állítsa a Min értéket −1-re a teljes index-tartományhoz (a**Reset** gomb visszaállítja a −1…+1 tartományt). A Max alapértelmezett értéke 1.
* Az indexeloszlás **élő hisztogramja** — négyzetgyök-skálázott oszlopok, borostyánszínű p2/p98 percentilis vonalak, fehér mediánvonal, valamint a tartományon kívüli értékeket jelző kijelzések („◀ N% &lt; lo” / „hi &lt; N% ▶”), amelyek 1 % felett borostyánszínűvé válnak, jelezve, hogy szélesíteni kell a Min/Max ablakot.
* Az **Alkalmaz** gomb az expressziót az élő adatfolyamra alkalmazza; a LUT-beállítások az Alkalmaz gomb megnyomása nélkül is azonnal hatályba lépnek. Nyitott projekt esetén az expresszió és a LUT-beállítások a projekttel együtt mentésre kerülnek, és a projekt újbóli megnyitásakor visszaállnak; projekt hiányában csak a munkamenet idejére maradnak meg.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->



## Az CLI

-útvonal

Ugyanaz az igazítás → halmozás → indexelés lánc, végpontok között szkriptelhető:

```bash
chloros-cli lattice align-calibrate --serials SN_RED,SN_NIR --save align.json
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

Az `align-calibrate` egy kapcsolt tömböt használ, amely tárolja a kameráit, vagy maga nyitja meg őket az adott futtatáshoz. Az `index --live` mindig maga nyitja meg a kamerákat (egy [ideiglenes tömb](../reference/cli-reference.md#multi-camera-transient-sessions)), és nincs `--serials`-e: a hálózaton lévő összes kamerát megnyitja. Tehát engedje fel az összes kamerát, amelyet a Chloros

korábban tartott – a GUI, az `cam-connect`, az `array-connect` (`chloros-cli lattice array-disconnect --all`): ha egy lefoglalt kamerát másodszor is megnyit, az leállíthatja a tulajdonos streamjét, vagy összeomlaszthatja a háttérrendszert. Az a kamera, amelyik nem szállít képkockát, meghiúsítja a futást, és nem íródik sem stack, sem index.

Az `--channel` egy preset szimbólumait a stack sávneveihez rendeli hozzá. Két szabály segít elkerülni a futtatás meghiúsulását:

* **A szimbólumoknál a kis- és nagybetűk közötti különbséget figyelembe kell venni**, és pontosan meg kell egyezniük az előre beállított csatornák neveivel — az előre beállított csatornák kisbetűket használnak (az NDVI

-k: `red`, `nir`; lásd `--list-presets`). Az `--channel red=Red_660` működik; az `--channel RED=660` hibaüzenettel bukik: „`channel_map missing entries`”.
* A sávoldalon meg kell nevezni egy sávot az igazított sorozatból (az `lattice align-info --profile align.json` felsorolja őket). Az offline mód 0-tól kezdődő sávindexeket is elfogad, pl. `--channel red=0 --channel nir=1`.

Az `lattice index` offline módban is fut egy mentett, igazított többsávos TIFF

-fájlra. Minden űrlap a háttérrendszeren fut, ezért annak futnia kell, és be kell jelentkezni egy fizetős Chloros

+ szintre; egy relatív `--input` vagy `--output` parancs a parancs futtatásának helye szerinti mappára vonatkozik:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Index-előbeállítások

Az `lattice index --preset` (és a Kép fül [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) menüpontja, amely ugyanazt a motort használja) a következő **22 előbeállítást** tartalmazza:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Futtassa az `chloros-cli lattice index --list-presets` parancsot az egyes előre beállított értékek képleteinek és csatornaszimbólumainak megtekintéséhez, az `--list-gradients` parancsot pedig a rendelkezésre álló színátmenetek megtekintéséhez. Az egyéni képletekhez az `--formula EXPR` parancsot kell használni, az Index Calculator-ral megegyező szintaxissal. Figyelem: ez az előre beállított lista kifejezetten a LATTICE indexmotorhoz tartozik — az importált képeknél a Projektbeállítások feldolgozás legördülő menüje egy másik listát tartalmaz (lásd [Multispektrális indexképletek](../project-settings/multispectral-index-formulas.md)).

A teljes jelzőkészlet (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, az `--live` igazítási torzítócsapjai és egyebek) a [CLI

-referencia § Index / Vegetációs matematika](../reference/cli-reference.md#index--vegetation-maths) című részben található; az SDK

-megfelelők az [SDK

-referenciában](../reference/sdk-reference.md) találhatók.

## Indextermékek rögzítése egy monokróm tömbből

Ha egy tömb csatlakoztatva van és egy indexkifejezés van alkalmazva, az `array-capture` (vagy a GUI **Capture All**) elmenti a kameránkénti exportszinteket *és* az index-renderelést — az `--index`/`--no-index` kapcsolja be ezt a funkciót a CLI

-en, és alapértelmezés szerint minden alkalmazható szintet rögzít. Egy monokamerának az egyes rögzítési csoportokhoz való hozzájárulása az egy sávja nyers/debayered (szürkeárnyalatos)/sugárzási/visszaverődési szinteken, valamint a megosztott kombinált indexű kompozit, amikor a rendszer kombinált módban működik. Lásd [Többkamerás rendszerek § Rögzítés](arrays.md#capturing-monitoring-vs-analysis).
