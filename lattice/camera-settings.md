# Kamera beállítások

A **Kamerák**fül az Chloros LATTICE kamerákhoz tartozó élő vezérlőfelülete: egy fő képernyőterület, amely minden csatlakoztatott kamerát élő csempeként jelenít meg, valamint egy oldalsáv, amely három oldal között váltakozik — a**kameralista**, a**beállítási panel**(kameránkénti, tömbös vagy rögzítési beállítások – egyszerre egy) és az**Index Calculator** között. Ez az oldal a kameralista, a kameránkénti beállítási panel és a tömbös beállítási panel minden vezérlőjét ismerteti. A rögzítési módok, az exporttípus kiválasztása és az „All rögzítése” folyamat a kiegészítő oldalon, a [Rögzítési beállítások és módok](capture.md) alatt található.

A „Kamerák” fül az oldalsávon jelenik meg, miután az Chloros háttérrendszer készen áll. Az alábbi összes vezérlőelem az `127.0.0.1:5000`-en keresztül kommunikál a helyi háttérprogrammal; a módosítások – ha másképp nem jelezzük – azonnal érvénybe lépnek az élő kamerán.

## Az ezen az oldalon használt kameratípusok

A vezérlők a kiválasztott kamera típusától függően jelennek meg vagy rejtődnek el. A kézikönyvben az alábbi kifejezéseket használjuk:

| Kifejezés | Jelentés | Szűrőcsatornák |
| --- | --- | --- |
| **RGB kamera** | LATTICE M3C FRGB szűrővel (a modell tartalmazza az `-FRGB`-et) | Red / Green / Blue |
| **Bayer multispektrális** | LATTICE M3C FRGN, FOCN vagy FNGB szűrővel | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — egy keskenysávú szűrő, egy kalibrált sáv | Egy sáv |
| **Rendszerelem** | Szinkronizált rendszer részeként csatlakoztatott kamera (kombinált vagy különálló kijelző) | Szűrőnként |

Az RGB kamerák fotometrikus feldolgozáson mennek keresztül (fehér egyensúly, színprofilok, gamma); a multispektrális és monokróm kamerák esetében a radiometrikus lánc kerül alkalmazásra, a fotometrikus beállítások pedig kihagyásra kerülnek. A rendszer tagjai a stream-szintű beállításokat (pixelformátum, felbontás, binning, trigger, képkockasebesség) átadják a rendszernek — ezek a sorok a kameránkénti ablakban írásvédetté válnak, és átkerülnek a rendszerbeállítások ablakába.

## A fő élőkép-terület

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

Ha nincs csatlakoztatott kamera, az élőkép-területen egy **„Csatlakoztasson kamerát a kezdéshez”**felirat jelenik meg két gombbal:**Kamera csatlakoztatása**(zöld, megnyitja az egykamerás csatlakozási párbeszédablakot) és**Tömb csatlakoztatása** (kék, megnyitja a tömbcsatlakozási párbeszédablakot). Maguk a csatlakozási párbeszédablakok a [Kamerák csatlakoztatása](connecting.md) című dokumentumban, a tömbökkel kapcsolatos fogalmak (szinkronizálás, rétegek, sávszélesség) pedig a [Többkamerás rendszerek](arrays.md)című részben. Ha olyan mentett projektet nyit meg, amelyben kamerák vannak, a kezdőképernyőn egy forgó ikon jelenik meg „N mentett kamera újbóli megnyitása…” felirattal, miközben az Chloros visszaállítja az utolsó munkamenetből származó adatfolyamokat.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Felső sáv

| Vezérlő | Működése |
| --- | --- |
| **Nézetmód váltás**| Váltás a**rácsnézet**(minden csempe cellaként) és a**listanézet** (a kameratömbök teljes szélességben a tetején, alatta EGY aktív kamera) között. Eszköztippek: „Váltás rácsnézetre” / „Váltás listanézetre”. |
| **Rácszár**(lakat) | Alapértelmezés szerint**zárva** — a lapok a helyükön rögzítve. A reteszelés feloldásával a lapokat bármelyik helyre áthúzhatja (a hézagok megmaradnak). A rács automatikusan újra reteszelődik, amikor új kamera csatlakozik. Eszköztippek: „Rács reteszelésének feloldása (lapok áthúzásának engedélyezése)” / „Rács rögzítése (csempék rögzítése a helyükön)”. |
| **Feed Zoom** csúszka | Csempe mérete, 60 px-től a konténer teljes szélességéig. A cellák 4:3-as képarányt tartanak. 200 px-nél kisebb cellaszélesség esetén a név és az fps feliratok elrejtődnek, hogy a csempe tiszta maradjon. |

### Feed-csempék

Minden kamera egy összetett élő csempét jelenít meg; egy kamera emellett három szürkeárnyalatos **csatornánkénti felosztású** csempét is megjeleníthet (lásd [Csatornafelosztások](#display-overlays-drawn-over-the-live-feed)), a tömbök pedig egy kombinált csempét jelenítnek meg. Az aktív csempe a kamera (vagy a tömb) színében egy kijelölőgyűrűt tartalmaz.

Ha az egérmutatót egy csempe fölé viszed, megjelenik egy **X** bezárás gomb:

* Ha egy **összetett** csempét bezársz, miközben a csatorna-felosztásai továbbra is láthatók, az összetett csempe csak elrejtődik.
* Egy **önálló kamera utolsó látható mozaikjának** bezárása leválasztja azt a kamerát.
* **A kombinált kamerarendszerhez tartozó felosztott mozaikok soha nem választják le** a kamerát — csupán elrejtik.

A rács rögzítésének feloldásával bármely mozaikot bármelyik helyre húzhat; az elrendezés a projekttel együtt mentésre kerül.

## Oldalsáv — kameralista

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

Az oldalsáv első oldala felsorolja az összes csatlakoztatott kamerát és rendszert:

* **Kamera csatlakoztatása**(zöld) /**Rendszer csatlakoztatása** (kék, szkennelés közben „Észlelés...” feliratot jelenít meg). Mindkettő le van tiltva, amíg egy csatlakozási párbeszédablak nyitva van.
* **Minden rögzítése** (piros) — a felsorolt összes kamerát rögzíti a Rögzítési beállításokban kiválasztott exporttípusokkal. Nyitott projekt szükséges. Teljes körűen dokumentálva a [Rögzítési beállítások és módok](capture.md) alatt.
* **Rögzítési beállítások fogaskerék** (az „Összes rögzítése” gomb mellett) — megnyitja a [Rögzítési beállítások panelt](capture.md#the-capture-settings-pane). Projekt hiányában vagy rögzítés közben letiltva van.

### Kamerasorok

Minden kamerasor színkódolt szegélyt (a kamera egyéni színét), egy „CAM” feliratot — amelyhez a rendszer tagjai esetében kék **M**(master) vagy zöld**S** (slave) szerepkörbetű társul —, valamint a megjelenítési nevet tartalmazza. Az alapértelmezett név `LATTICE-MODEL (serial)`; átnevezhető a kameránkénti beállítások panelen. Sor gombok:

| Gomb | Hatás |
| --- | --- |
| **Szem**| Láthatóság váltása. A rejtett kamerák eltűnnek a rácsból, és**kizáródnak az „Összes rögzítése” funkcióból**. |
| **Fogaskerék** | Megnyitja a kameránkénti beállítások panelt (következő szakasz). |
| **Szünet / Lejátszás**|**Csak a kijelző oldalon** befagyasztja az élő előnézetet — a háttérben a rögzítés továbbra is fut. A szüneteltetett kamerák nem tudnak rögzíteni. |
| **X** | Kapcsolat bontása. A felhasználói felület azonnal frissül (optimális esetben); a háttérben a kapcsolat bontása 10–30 másodpercig is eltarthat. |

### Tömb sorok

A tömb sorai a tömb színében egy „ARRAY” jelölést, a tömb nevét (amely a tömb beállításaiban átnevezhető) és egy **DAQ · be/ki**gombot jelenítenek meg —**be** állapotban, ha a tömb szintű fényérzékelő be van állítva *vagy* bármelyik tag rendelkezik kameránkénti érzékelővel; a tooltip pontosan felsorolja, melyik érzékelő mit továbbít. A tagkamerák behúzva, saját sorokban szerepelnek alatta. Tömb sor gombok: **szem**(az ÖSSZES tagot egyszerre elrejti/megjeleníti),**fogaskerék**(tömbbeállítások ablak),**X**(az egész tömb kapcsolatának megszüntetése).

A tömb sorokban és a tömb beállítások ablakában használt fényérzékelő (DLS) állapotnak négy állapota van:**ki**,**várakozás**(még nincs spektrum),**aktív**(az elmúlt 3 másodpercben érkezett spektrum), és**elavult** — 3 másodperc alatt nem érkezett friss spektrum, de az utolsó leolvasott értéket *még mindig használja* (a DAQ-leolvasások soha nem járnak le a rögzítési útvonalon).

Az oldalsávban az önálló kamerákat és a teljes rendszercsoportokat egymás mellett húzva átrendezheti a listát; a rendszer tagjai önállóan nem húzhatók.

## Kameránkénti beállítási panel

A kamera során található **fogaskerék** gombbal nyitható meg. A panel a kameralistára csúszik.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->

**Fejléc**: a kamera**színmintája**(kattintással megnyitható a natív színválasztó — beállítja az oldalsáv szegélyének és a csempe kiválasztógyűrűjének színét), a**név**egy ceruzával ellátott**Átnevezés**gombbal (ha üres nevet ment, a rendszer visszaáll az alapértelmezett `MODEL (serial)` névre), valamint**×** a bezáráshoz.

### Élő hisztogram

A panel tetején egy élő luma-hisztogram látható, amelyet az JPEG előnézetből ~8 Hz-es frekvenciával számítanak ki. Az átlagot Bayer-súlyozással számítják ki — (R+2G+B)/4 —, hogy illeszkedjen a kamera saját AE-méréséhez.

* **Orange szaggatott vonal**= az AE-célérték.**Húzza vízszintesen az újracélzáshoz** — az elengedéskor egy parancs kerül elküldésre, a húzás pedig átváltja az AE-célmódot Kézi üzemmódra.
* **Green folytonos vonal** = a tényleges átlagos fényerő (amit az AE jelenleg biztosít).
* **RGB gomb** (jobb felső sarok): a kamera szűrője szerint színezett, sávonkénti átfedő hisztogramok közötti váltás (pl. FRGN esetén: szürke NIR, zöld, piros). Mono (M3M) kamerák esetén a gomb felirata „MONO”, és le van tiltva — mono módban mindig az egysávos luma-hisztogram jelenik meg.
* Az X-tengely címkéi az aktuális képpontformátum érzékelőjének bitmélységét követik: 0..255, 0..1023, 0..4095 vagy 0..65535.

### Kameraadatok sorai

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

| Sor | Viselkedés |
| --- | --- |
| **Modell** | Csak olvasható (pl. `LATT-M3C-L87-FRGN`). |
| **Radiometrikus kalibrálás**| Green**„Aktív”**jelölés, amely alatt a kalibrációs szint, hash-értékét, a kalibrálás dátumát és a sávlistát, amely a kamera kalibrációs csomagjából lett betöltve (lásd [Gyári radiometrikus kalibrálás](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Rejtve az RGB kameráknál** — ezek fotometrikus fehéregyensúly-kalibrálással rendelkeznek, nem sávonkénti sugárzási kalibrációval rendelkeznek. |
| **Kalibrációs jelentés**|**Letöltés** gomb — megnyitja a kamera sorozatszámonkénti NIST kalibrációs tanúsítványát PDF formátumban az operációs rendszer nézőjében. Ha a tanúsítvány még nincs a gyorsítótárban, az Chloros helyette egy tippet jelenít meg. |
| **Sorozatszám** | Csak olvasható. |
| **Firmware**| Megjeleníti az aktuális verziót, majd megkeresi az adott modellhez elérhető verziót (modellenként tárolva a gyorsítótárban — egy N kamerából álló rendszer egyszer ellenőrzi a szervert). Állapotok: „Ellenőrzés…” →**„Frissítés X-re”**gomb → „Frissítés folyamatban…” → „Frissítve A → B” / „Hiba: …” / „Kihagyva: …” / zöld**„Naprakész”**. A frissítés gomb eszköztippje: „Gyári beállítások visszaállítása + frissítés + UserSet1 újraprogramozása. ~2–3 perc; ne válassza le a készüléket.” |
| **IP** | Csak olvasható. |
| **Hőmérséklet** | Csak olvasható, 3 másodpercenként frissül. 65 °C felett narancssárgára, ⚠ jelzéssel pedig pirosra vált jelenik meg ≥75 °C-on. |
| **Kalibrációs cél** jelölőnégyzet | Engedélyezi az ArUco fényvisszaverődési cél felismerését a panelekhez tartozó NDVI érvényesítési táblázattal az élő kép alatt (lista nézet). Csak munkamenetben elérhető — mindig kikapcsolt állapotban nyílik meg. |
| **Fényérzékelő** legördülő menü | Egy DAQ fényérzékelőt (DAQ-E/M/U, a „Fényérzékelők” fül listájából) rendel ehhez a kamerához a lefelé irányulófény (DLS) megvilágítási korrekciója és a prediktív automatikus expozíció érdekében. A „Nincs” opció törli a hozzárendelést. Ha nincs csatlakoztatott érzékelő, a legördülő menü a „(nincs csatlakoztatott érzékelő – nyissa meg a DAQ fület)” feliratot jeleníti meg. A hozzárendelés a projekttel együtt kerül mentésre. |

### Expozíció és erősítés

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Az itt található összes numerikus beviteli mezőben a hosszan tartott érintéssel gyorsítható forgógombok használhatók: érintés = ±1, 1,5 másodpercnél hosszabb tartás = ±10, 3 másodpercnél hosszabb tartás = ±100. Az érték a kamera felé akkor kerül elküldésre, amikor elengedi a gombot.

| Vezérlő | Tartomány / lehetőségek | Alapértelmezett | Vonatkozik | Működése |
| --- | --- | --- | --- | --- |
| **Expozíció (us)**| A kamera élő min./max. értékei | Automatikus | Minden | Expozíciós idő mikroszekundumokban,**Automatikus/Kézi** kapcsolóval. Automatikus = folyamatos, kamerán belüli automatikus expozíció. |
| **Erősítés (dB)**| A kamera aktuális min./max. értéke (pl. legfeljebb 48 dB) | Kézi (ki) | Minden | Analóg/digitális erősítés, saját**Automatikus/Kézi** kapcsolóval. |
| **AE célfényerő**| 0–255 | 80,**Auto**| Minden (szerkeszthető, ha az AE vagy az automatikus erősítés be van kapcsolva) | Az AE által megcélzott fényerő.**Auto**(alapértelmezett) üzemmódban egy hisztogram-alapú háttérvezérlő maga választja ki a célt, az expozíciót az érzékelő maximális értékének 60–75 %-án tartva. Érték beírásával vagy a hisztogram narancssárga vonalának húzásával**Kézi** módra vált. |
| **AE simítás** | 0,5–40, lépésköz 0,1 | 8,0 | Minden | AE csillapítás. Eszköztipp: „Alacsonyabb érték = az AE gyorsabban reagál (magas képkockasebesség mellett pulzálhat). Magasabb érték = simább / lassabb.” Az alapértelmezett értéktől jelentősen eltérő értékek az AE pulzálását okozhatják, és destabilizálhatják a streamelést magas képkockasebesség mellett; a 8,0 a stabil alapértelmezett érték. |
| **AE érdeklődési terület**| Engedélyező jelölőnégyzet +**Célzás**gomb | Ki | Minden | Ha be van kapcsolva, az AE csak a zöld szaggatott vonallal jelölt területet méri a teljes képkocka helyett. A**Célzás** gomb aktiválja a kattintással történő elhelyezést az élő képen: egy kattintás a területet a képkocka 30%-ánál központosítja; kattintás és húzás egy egyéni téglalapot (minimum 5 % × 5 %). A Célzás egy elhelyezés után automatikusan kikapcsol. A területet a beállított forgatás/tükörkép alapján visszaalakítja a kamera natív koordinátarendszerébe, és a projekttel együtt elmenti. |
| **AE Tune Speed** | 0,1–5, 0,1-es lépésközzel | 1,0 | Csak Array-tagok számára | Az automatikus AE-célpont milyen gyorsan követi a jelenet fényerejének változásait; 1,0× esetén 2,5 másodpercenként ellenőrzi újra. |
| **Fényes részek védelme** | Szigorú (1 %) / Normál (5 %) / Laza (15 %) | Szigorú | Azok a kamerák, amelyeknél ez a beállítás elérhető | A képkocka mekkora része válhat fehérré, mielőtt az AE elsötétíti a képet. |

{% hint style="info" %}
**Fényigény a Bayer multispektrális kamerákhoz (RGN / OCN / NGB):** a jelenetnek mindhárom csatornán elegendő fénynek kell lennie, különben a kalibrálás nem működik megfelelően — egyetlen érzékelő expozíciója lefedi mindhárom spektrumot. Használjon DAQ fényérzékelőt a fény méréséhez, vagy váltson teljesen monokróm (M3M) módot, hogy minden sáv saját expozíciót kapjon. Ha egy felvétel megsérti ezt a feltételt, az Chloros észleli és figyelmezteti Önt (az „unmix-clamp” értesítés).
{% endhint %}

### Pixelformátum és

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

felbontás**A tömb tagjai** írásvédett „Current” (formátum + WxH) és „Binning” sorokat jelenítnek meg a „Set in array settings” megjegyzéssel — egy tag stream-újraindítása megszakítaná a szinkronizálást, ezért ezeket a [tömbbeállítások panelen](#array-settings-pane) kezelik.**Önálló kamerák** esetében a következők érhetők el:

| Vezérlő | Választási lehetőségek | Működése |
| --- | --- | --- |
| **Pixelformátum** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Az érzékelő pixelformátuma (bitmélység). |
| **Felbontás** | Teljes / Fele / Negyed | Az aktuális binninghez viszonyítva: Teljes = 2048/N × 1536/N N×N binning esetén. |
| **Binning** | 1x1 (nincs) / 2x2 / 4x4 | Hardveres N×N binning — a nagyobb értékek alacsonyabb felbontást eredményeznek, de javítják a jel-zaj arányt (SNR) és a képkockasebességet. A beállítás megváltoztatása újraindítja a streamet, és az összes ROI-t az új teljes látómezőre állítja vissza. |
| **Aktuális** | csak olvasható | A tényleges WxH és (x, y) eltolás. |

### Élő előnézet

Ebben a szakaszban minden **csak a kijelzőre vonatkozik**— ez megváltoztatja, amit az élő közvetítésben lát, míg a mentett felvételek lineárisak és változatlanok maradnak — egy kivétellel: a**Vignette** radiometrikus, és az exportált fájlokat is érinti (lásd alább).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->

| Vezérlő | Tartomány / lehetőségek | Alapértelmezett | Vonatkozik | Mit csinál |
| --- | --- | --- | --- | --- |
| **Renderelési felbontás** | 360p (leggyorsabb) / 480p / 720p / 1080p / Natív érzékelő felbontás (leglassabb) | 720p | Minden | Az a magasság, amelyen a háttérrendszer futtatja a radiometrikus előnézeti láncot. Alacsonyabb értékkel a látómező megváltoztatása nélkül növelhető a képkockasebesség. |
| **Index**| Engedélyezés jelölőnégyzet + fogaskerék | Ki | Csak Bayer multispektrális,**nem** kombinált sorozatú tagok | Élő növényzet-index előnézet. A fogaskerék megnyitja a megosztott [Index Calculator](#index-calculator-panelt), amelybe előre betöltődtek a kamera szűrő-természetes sávjai (pl. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Az egyéni kifejezés és a LUT (be/ki, alapértelmezett szint 3, alapértelmezett min. 0,2, alapértelmezett max. 1) minden előnézeti képkockán kiszámításra kerül. A kombinált tömb tagjai elrejtik ezt a sort – a tömb egy megosztott indexet birtokol. |
| **Fehér egyensúly** | Ki / Egyszer / Folyamatos + újrafelvétel gomb | Folyamatos | Csak RGB | Élő fehér egyensúly. A frissítés gomb a fehér egyensúlyt az aktuális DLS-spektrumból újra rögzíti (ki van kapcsolva, ha az üzemmód Ki). |
| **Gamma** | Be / Ki | Be | Csak RGB | Gamma megjelenítése (γ = 2,2 LUT) az élő előnézetben. A mentett felvételek lineárisak maradnak. |
| **Zajszűrés** | Jelölőnégyzet + erősség 0–100 | Ki / 50 | Minden (kameránként, még a tömbökön belül is) | Kétoldalas szűrő az élő előnézetben. Magasabb érték = simább, de lágyabb részletek. |
| **Élesség** | Jelölőnégyzet + erősség 0–100 | Ki / 30 | Minden | Élességmaszk az élő előnézetben, utoljára alkalmazva. Fokozhatja a zajt. Csak előnézetben. |
| **Vignette**| Jelölőnégyzet + erősség 0–100 | Ki / 0 | Minden | Kézi maradék vignettálás-eltávolítás (fényesíti a sarkokat), a tömb Smart Vignette becslésére rétegezve.**Radiometrikus — hatással van az élő nézetre ÉS az exportált fájlokra**, ellentétben a Zajszűréssel/Élességgel. |
| **Színeprofil** | Raw / Lineáris / Természetes / Fokozott / Egyéni hőmérséklet | Természetes | Csak RGB | Lásd alább. |
| **Színhőmérséklet** | 2000–10000 K, 100-as lépésenként | 5500 K | Csak RGB, Egyéni hőmérséklet profil | A fehér egyensúlyt egy rögzített korrelált színhőmérséklethez rögzíti (a DLS-bemenet figyelmen kívül marad). A legutóbb kiválasztott Kelvin-érték profilváltáskor is megmarad. |
| **Telítettség** | 0–200 (100 = semleges) | 100 | Csak RGB | HSV-telítettség az élő előnézetben. |
| **Kontraszt** | 0–200 (100 = semleges) | 100 | Csak RGB | Lineáris kontraszt a közép-szürke környékén az élő előnézetben. |
| **Tükör H / Tükör V** | Jelölőnégyzetek | Ki | Minden | Az előnézet vízszintes / függőleges tükrözése. |
| **Forgatás**| 0° / 90° / 180° / 270° | 0° | Minden | Az előnézet elforgatása. A tájolás a háttér előnézeti lánc végén kerül alkalmazásra —**a mentett felvételek a kamera natív tájolásában maradnak**, a tömbösszetett nézetek pedig figyelmen kívül hagyják. |**Színprofil-szemantika** (RGB kamerák):

* **Raw** — a feldolgozási láncot teljesen megkerüli.
* **Lineáris** — sötétjel + síkkép + fehéregyensúly; nincs színmátrix, nincs gamma.
* **Természetes** *(alapértelmezett)* — lineáris plusz a mért színkorrekciós mátrix és egy jelenethez alkalmazkodó tónusgörbe.
* **Továbbfejlesztett**— Természetes, plusz élénkség és CLAHE helyi kontraszt. A felár**csak az élő előnézetre** vonatkozik — a mentett felvételek mindig teljes körű utómunkát kapnak, függetlenül a profiltól.
* **Egyéni hőmérséklet** — Természetes, a fehéregyensúlyt a választott Kelvin-értékre rögzítve.

{% hint style="warning" %}
Természetes, Továbbfejlesztett és Egyéni hőmérséklet beállítások esetén az ablak egy tónusra vonatkozó megjegyzést jelenít meg: a képkockákat a saját jelenetüknek megfelelően világosítják, így a mentett *megjelenítési* képek nem összehasonlíthatók képkockáról képkockára. **A mérésekhez exportálja a sugárzási vagy visszaverődési értékeket.**
{% endhint %}

### Kijelző-átfedések (az élő kép fölé rajzolva)

Ezek kizárólag a felhasználói felületen jelennek meg — a videóra vannak rajzolva, soha nem érintik az élő közvetítést vagy a rögzített képeket.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Átfedés | Vezérlők | Alapértelmezett | Működése |
| --- | --- | --- | --- |
| **Zebra** | Jelölőnégyzet + küszöbérték 200–255 | Ki / 250 | Magenta színű átlós csíkok a levágott képpontokon. |
| **Célkereszt** | Jelölőnégyzet | Ki | Képközépponti jelölés. |
| **Rács** | Ki / 3 × 3 / 9 × 9 | Ki | Kompozíciós rács. |
| **Hisztogram** | Jelölőnégyzet + szélesség 0,10–0,90 a képkocka szélességéhez viszonyítva | Ki / 0,25 | Élő hisztogramcsík. |
| **Fókuszcsúcs** | Jelölőnégyzet + küszöbérték 20–200 + színminta | Ki / 80 / `#ff5722` | Sobel-élkiemelés az élességállításhoz. |
| **Csatornafelosztás** | &quot;Osztások megjelenítése (Red / Green / NIR)&quot; / „Osztások elrejtése” gomb | Elrejtve | Három független, csatornánkénti szürkeárnyalatos lapkát ad hozzá a kompozit kép mellé (a gomb felirata a kamera szűrőcsatornáit követi). Minden osztott lapka húzható, és a kamera szegélyszínét veszi át. Monokróm kameráknál nem elérhető. A projekttel együtt mentésre kerül. |

### Pontmérő

* **Kattintás a mintavételhez**jelölőnégyzet: kattintson az élő képre egy pixel mintavételéhez (egy célkereszt jelöli meg), vagy kattintson és húzza a kézzel egy területet a pixelek átlagának kiszámításához. A**Törlés**gomb törli a mintát és a célkeresztet. Kizárja az AE-ROI**Célzás** módot.
* **Megjelenítés**legördülő menü:**Raw (bitmélység)**— natív digitális értékek az érzékelő bitmélységében (pl. 12-bit → 0..4095) — vagy**Kijelző (8-bit)** (alapértelmezett). Ha élő index aktív, a Kijelző helyette a kiszámított indexértéket (pl. NDVI) jeleníti meg.
* A kijelzőpanel felsorolja a képpontkoordinátákat, a képkocka méretét, a képpontformátumot, a bitmélységet, valamint egy csatornatáblázatot (Csatorna / Érték / %) sávcímkékkel és hullámhosszakkal; a Bayer-zöld párok átlagolva vannak; a régióminták „N px avg” feliratot jelenítik meg.

A pontmérő állapota csak a munkamenetre vonatkozik.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Prediktív automatikus expozíció (DLS-vezérelt)

Ez a szakasz csak akkor jelenik meg, ha **legalább egy DAQ fényérzékelő csatlakozik** — a megoldónak élő, lefelé irányuló spektrumra van szüksége a működéshez.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->

| Vezérlés | Tartomány | Alapértelmezett | Működése |
| --- | --- | --- | --- |
| **Engedélyezés** | Jelölőnégyzet | Be (önálló kamerák) | Egy zárt formájú megoldó a DLS-spektrumot és a kamera kalibrációs csomagjának skalárjait használja fel arra, hogy a legfényesebb sávot a telítettség közelébe állítsa, miközben a leghalványabb sávot az SNR-küszöb felett tartja — megoldásonként egyetlen expozíciós írás, nincs stabilizáló hurok. Napenergiával működő timelapse-felvételekhez tervezték, ahol minden felvételnek helyes expozícióval kell rendelkeznie. A háttérrendszer automatikusan visszatér a reaktív AE-re, ha a DLS-érték elavult vagy hiányzik, vagy a kalibrációs csomag nincs betöltve. |
| **Simaítás (α)** | 0,05–1,0, 0,05-ös lépésenként | 0,3 | Az egymást követő prediktív megoldások simítása (alacsonyabb érték = simább). |
| **Jelenet-visszaverődés**|**ρ újrakalibrálása** gomb | — | Újrabecsüli a megoldó által használt jelenet-visszaverődési tényezőt. |

{% hint style="info" %}
**A tömbösszekapcsolás alapértelmezés szerint kikapcsolja a prediktív AE-t** — tömbök esetén az Chloros „intelligens AE” és a kamera oldali automatikus expozíció kezeli az expozíciót (telítettségvédelemmel), és a prediktív AE egyetlen jelenet-reflektancia becslése vegyes jelenetek esetén nem biztonságos. Itt kameránként újra engedélyezheti, ha kifejezetten DLS-vezérelt radiometrikus expozíciót szeretne.
{% endhint %}

**DAQ-vezérelt expozíciós felső határ és beeső fényhez rögzített AE.**A fenti jelölőnégyzettől függetlenül, amikor egy DAQ fényérzékelőt rendelnek egy RGB kamerához, az Chloros – a mért abszolút lefelé irányuló besugárzás alapján – kiszámítja azt a maximális expozíciót × erősítést, amelynél egy 100%-os visszaverődésű felület a clipping alatt marad, és ezt**felső határként**az automatikus expozícióra. Amíg a felső határ aktív, a kamera**beeső fényhez rögzített**: nyitott hurkon működik a beeső fény alapján mért expozícióval, 0 dB-es erősítéssel — az expozíció a mért fényt követi, nem pedig a jelenet tartalmát. Mivel a felső határ csak rövidítheti az expozíciót, önmagában nem okozhat túlexponálást. A felső határ automatikusan kikapcsol — és a normál jelenet-AE folytatódik —, ha a DAQ-érték hiányzik, elavult (&gt;30 s), vagy sötét, vagy ha a képkocka ≥15%-a túlexponálódik a rögzített expozíció mellett (vagyis az érzékelő és a kamera eltérő megvilágítást érzékel). Nincs erre vonatkozó GUI-kapcsoló; ez a standard viselkedés, ha egy RGB kamera DAQ-hoz van kapcsolva.

### Az Acquisition &amp; Trigger

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

Array elemei emellett olvasásra korlátozott **Role**(Master kékben / Slave zöldben),**Sync Line**és**Peers** sorokat is megjelenítenek.

| Vezérlés | Választási lehetőségek | Alapértelmezett | Megjegyzések |
| --- | --- | --- | --- |
| **Trigger mód** | Ki / Be | Be | Tömbtagoknál letiltva (a tömb kezeli a triggert). |
| **Trigger forrás** | Szoftver / Line0 (M8) / Line1 / Line2 | Line0 | A Trigger mód kikapcsolt állapotában elrejtve; a tömb tagjai számára letiltva. A Line0 az M8 optikailag szigetelt külső trigger bemenet. |
| **Képkockasebesség**| Automatikus / Kézi + érték | Automatikus |**Automatikus**: a kamera képkockasebesség-korlátozása ki van kapcsolva — az expozíció határozza meg az fps-t, és a mezőben a valós idejű tényleges sebesség jelenik meg.**Kézi**: a képkockaszámot egy csúszkával korlátozhatja (1-től a sávszélesség által korlátozott maximumig), amely az aktuális tényleges frekvenciából indul ki. A tömb tagjai egy csak olvasható „N fps (élő)” értéket látnak, amely mellett a „Beállítva a tömbbeállításokban” felirat szerepel. |

### Hálózat / Átvitel

| Sor | Viselkedés |
| --- | --- |
| **Csomagméret**| 1500 (Standard) / 9000 (Jumbo) — alapértelmezés szerint**Jumbo**. |
| **Átviteli sebesség** | Csak olvasható kapcsolati átviteli sebesség-korlát MB/s-ban. A háttérrendszer minden csatlakozáskor/leváláskor újraelosztja ezt az összes csatlakoztatott kamera között. |
| **Pufferkezelés** | Csak olvasható pufferkezelési mód. |

### Felvétel

A panel egy **„Felvételi beállítások megnyitása…”** gombbal, amely a [Felvételi beállítások ablaktáblára](capture.md#the-capture-settings-pane) ugrik (a gomb addig letiltva van, amíg nincs megnyitva projekt — „Hozzon létre vagy nyisson meg egy projektet a felvételek mentéséhez”). Ha a kamera el van rejtve vagy szünetel, egy figyelmeztetés emlékezteti Önt, hogy a felvétel készítése előtt állítsa vissza a láthatóságot vagy folytassa a felvételt.

## Tömbbeállítások ablak

Az ARRAY soron található **fogaskerék**gombbal nyitható meg. Fejléc: a tömb neve, egy átnevezésre szolgáló ceruzával és a bezáráshoz szükséges**×** gombbal. Az alábbi, *csak kombinált* jelöléssel ellátott szakaszok csak a kombinált megjelenítési módban összekapcsolt tömbök esetén jelennek meg.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->

### Szinkronizálás

Csak olvasható **Master**,**Slaves**és**Sync Line** sorok.

### Környezeti fényérzékelő

Kombinált és különálló tömbök esetén egyaránt megjelenik:

* **Kalibrációs cél** jelölőnégyzet — „Az MAPIR ArUco célpont észlelése és az NDVI panelreflektancia-LUT-val való összehasonlítása”; vezérli a kombinált csempe célpont-átfedését és érvényesítési táblázatát.
* **Fényérzékelő** legördülő menü — egy DAQ-t rendel a teljes tömbhöz. A kiválasztás azonnal érvénybe lép, átterjed minden tagkamera saját Fényérzékelő legördülő menüjére (kameraenként továbbra is felülírható), és elkezdi a spektrumok továbbítását a mátrix felé.
* Élő **Állapot** sor: Ki · „Az első spektrumra vár…” · „Aktív — a rendszerben minden kamera megvilágítás-korrigált” · „Nincs új spektrum az elmúlt 3 másodpercben — továbbra is az utolsó leolvasást használja (nincs elavult időtúllépés)…”.
* Megjegyzés az ablaktáblában: „Rendszerre kiterjedő radiometrikus korrekció. Az egyes kamerák beállításai felülírják ezt.”

### Felvétel — egységes érzékelőbeállítások *(csak együttesen)*

Ezek a beállítások egységesen vonatkoznak minden tagra (a tagonkénti változtatások megszakítanák a szinkronizálást). A módosítások előkészítésre kerülnek, majd együttesen kerülnek alkalmazásra.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Vezérlő | Választási lehetőségek / tartomány | Működése |
| --- | --- | --- |
| **Pixelformátum** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Egységes érzékelőformátum minden tag számára. |
| **Binning** | 1x1 / 2x2 / 4x4 | Hardveres binning — megőrzi a teljes látómezőt, miközben javítja a jel-zaj arányt (SNR) és a képkockasebességet. A beállítás megváltoztatása az ROI mezőket az új teljes látómezőre állítja vissza. |
| **Felbontás** előbeállítás | Teljes / Fele / Negyed | A binninghez viszonyítva; a ROI mezőket központosított kivágással tölti ki. |
| **ROI kivágás (px)**| Szélesség / Magasság / X / Y numerikus mezők | Érzékelő-kivágás. A szélesség/magasság 16-os többszöröséhez igazodik (minimum 64); az eltolások 4-es szorzókhoz igazodnak. A „max WxH” jelzés a felső határt mutatja, a**Visszaállítás** gomb pedig visszatér a teljes látómezőhöz. Szerkesztés közben egy élő, narancssárga kivágási előnézeti keret jelenik meg a sorozatlapkán (beleértve az érzékelő teljes vázlatát is, ha a kivágást kifelé bővítik). |
| **Kiváltási gyakoriság**| Automatikus / Kézi kapcsoló + fps 0,5–10, lépésenként 0,5 |**Automatikus**(alapértelmezett): a háttérrendszer a felbontásból és a sávszélességből számítja ki a kiváltási gyakoriságot — a beviteli mező le van tiltva, és a kiszámított értéket jeleníti meg.**Kézi**: az**Alkalmaz** gomb megnyomásával rögzíti az értéket. |

Megjegyzés az ablakban: „A formátum/felbontás módosítása rövid ideig újraindítja az összes kamerát. A triggerfrekvencia azonnal hatályba lép.” Az **Alkalmazás / Mégse** gombok az ablak alján találhatók.

### Igazítás (ko-regisztráció) *(csak kombinált)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Maradék** mező: „RMS x,xx px” — 1 px alatt zöld, 3 px alatt sárga, egyéb esetben piros, illetve ha bármelyik kamera meghibásodott; „nincs profil” az első megoldás előtt.
* Összefoglaló sor: „✓ Minden kamera igazítva (N)” / „⚠ p/N kamera igazítva —  <serial (filter)="">meghibásodott” / „Kivágás aktív — Újrakalibrálás az igazításhoz (a teljes érzékelőt használja)” / „Várakozás az expozíció stabilizálódására…”.
* Kameránkénti táblázat: kamera (a sorozatszám utolsó 4 számjegye + szűrő), újraprojekciós hiba px-ben az egyezések számával („ref” a mesterkamerára), és az átfedés normalizált keresztkorrelációs pontszáma a 0,35-ös küszöbértékhez viszonyítva.
* **„Igazítás újrakalibrálása”** gomb (az első profil előtt „Igazítás kalibrálása” felirat látható) — újra futtatja a ko-regisztrációt friss képkockákon.
* **„Kamerák automatikus expozíciója az igazításhoz”** jelölőnégyzet (alapértelmezés szerint bejelölve) — ideiglenesen felvilágosítja a sötét vagy lapos kamerákat (először az expozíciót, majd az erősítést), hogy legyen textúrájuk az illesztéshez, majd visszaállítja az automatikus expozíciót.

A kombinált előnézet megnyitáskor automatikusan illeszkedik; ha a fókusz vagy a jelenet mélysége megváltozott, kalibrálja újra. Az illesztés **tervezés szerint csak a munkamenetre vonatkozik** — soha nem kerül elmentésre egy profilba sem, mivel a pillanatnyi jelenet távolságától függ. A felvételek továbbra is pixel-regisztrálva exportálhatók (lásd [Igazított exportok](capture.md#per-array-controls)).

### Intelligens vignettálás

* **Korrekció engedélyezése**jelölőnégyzet — a kameránkénti vignettálási becslést alkalmazza a radiometrikus láncra (élő**és** exportált képek esetén).
* **Kalibrálás az aktuális nézetből**— először irányítsa a kamerasorozatot egy egyenletes felületre (lapos panel, fal vagy ég); minden kamerát külön-külön kiegyenesítenek, és az állapotjelentés „n/N kamera · −x,x %” kiegyenesítési nyereséget jelez. A**Törlés** gomb eltávolítja a becslést.
* Kameránkénti finomhangolás a [Élő előnézet](#live-preview) ablakban található, kameránkénti **Vignetta** csúszkával.

### Élő előnézet *(csak kombinált)** **Index**: jelölőnégyzet + fogaskerék — megnyitja a megosztott [Index-kalkulátort](#index-calculator-pane), amelyen**az összes** tagkamera adatai alapján rajzolt sávok láthatók. Az alatta található kifejezés-előnézeti sor az aktuális kifejezést mutatja („Nincs beállított kifejezés — nyissa meg a kalkulátort, hogy létrehozzon egyet”), amely másodpercenként frissül.
* **Renderelési felbontás**legördülő menü (ugyanazok az előre beállított értékek, mint kameránként, alapértelmezés szerint 720p): az élő nézet stream magassága**és** a mentett kompozit exportméret. Megjegyzés a panelen: „Előnézet + mentett kompozit méret. A kameránkénti képek mindig teljes felbontásban kerülnek exportálásra.”

### Megjelenítési rétegek *(csak kombináltan)** **Engedélyezés** jelölőnégyzet (alapértelmezés szerint ki van kapcsolva — a fő kamera képe közvetlenül jelenik meg; be = réteges kompozíció).
* **Előtér**/**Háttér**legördülő menük: az egyes tagkamerák (név szerint) vagy**Index**. Ha az Előtér beállítása Index, a LUT Min/Max határain kívül eső képpontok a Háttér réteget jelenítik meg.

### Osztott nézet *(csak kombinált esetben)*

**„Tagkamerák megjelenítése”**— egy**Tagkamerák megosztása / elrejtése** gomb, amely az egyes tagok saját élő felvételeit külön rácskockákként jeleníti meg a kompozíció mellett. A rácselemek a tömb meglévő képkocka-pufferét olvassák be (nincs szükség külön kameracsatlakozásra). Csak rácsnézetben; tömbönként, a projekttel együtt mentve.

### Funkciók

5 másodpercenként frissülő, csak olvasható panel:

* **Szintcímke**: „Egyidejű rögzítés” (zöld) · „Egyidejű rögzítés (FTD-elhalasztott kibocsátás)” (zöld) · „Elhalasztott rögzítés (100 ms eltolódás)” (sárga) · „Túl nagy konfiguráció” (piros).
* **Képkocka állapota**: „x,xx % hiányos” — zöld 1 % alatt, sárga 5 % alatt, piros 5 % vagy annál több esetén.
* **Kapcsolati sor**: „NIC {mbps} Mbps – folyamatos {MB/s} MB/s”.

Ez a rendszerélő sávszélesség-kerete. Az alapul szolgáló fps- és hálózati modellről — valamint arról, hogy mit kell módosítani, ha a szint sárgára vagy pirosra vált — lásd a [Többkamerás rendszerek](arrays.md) és az [CLI Referencia](../reference/cli-reference.md) című dokumentumban.



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Index Calculator ablak

A harmadik oldalsáv, amelyet a kameránkénti Index-beállítás és a kombinált tömb Index-beállítása oszt meg (egyszerre csak az egyik – a fejlécen az „Index Calculator — <camera name="">” vagy „Index Calculator —<array name="">

”). Ez fogadja a sávlistát (a kamera szűrőjének természetes sávjait, vagy a rendszer összes tagjának összes sávját), az aktuális kifejezést és a LUT-konfigurációt (be/ki, szint – alapértelmezett 3, min – alapértelmezett 0,2, max – alapértelmezett 1), valamint egy élő index-hisztogramot. Az **Alkalmazás** gomb az expressziót véglegesíti; a LUT-változások élőben jelennek meg az előnézetben.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Kameránkénti vs. tömb által kezelt beállítások

Gyors áttekintés arról, hogy mi hol található, ha egy kamera a tömb tagja:

| Tömb által kezelt (csak olvasható a kamera ablakban) | Továbbra is kameránkénti a tömbön belül |
| --- | --- |
| Pixelméret, felbontás, binning | Automatikus expozíció (expozíció, erősítés, célérték, simítás, ROI) |
| Kioldási mód/forrás, képkockasebesség | Zajszűrés, élesség, vignettálás |
| | Tájolás (tükörkép/elforgatás), kijelző-felülfedések, pontmérő |
| | Index (külön kijelzős rendszerek), fényérzékelő-hozzárendelés |

Egyéb átfogó viselkedés:

* A **kombinált vagy különálló megjelenítés** a tömb csatlakoztatásakor választható: kombinált = egy igazított összetett csempe (a tagok csak Split View-n keresztül továbbítanak); különálló = minden tag a saját szinkronizált csempéjét jeleníti meg. Egy kamera soha nem jeleníti meg egyszerre az önálló felvételt és a tömbcsempét.
* **Automatikus újracsatlakozás**: egy mentett projekt megnyitásakor a rendszer visszaállítja a kamerákat és a mátrixokat, és az adatfolyamok folytatását megelőzően minden mentett beállítást újraalkalmaz a háttérrendszerben.
* **Felvétel-szűrés**: a rejtett vagy szüneteltetett kamerák nem kerülnek bele az „Összes felvétele” funkcióba; egy tömb csak akkor blokkolódik teljesen, ha MINDEN tagja rejtett vagy szüneteltetett. Lásd [Felvételi beállítások és módok](capture.md).

## A beállítások megőrzése

A „Kamera” fül állapota **a projekttel együtt** kerül mentésre, nem a böngészőben:

* Minden reaktív változás pillanatképet készít a kamerákról és a kameracsoportokról a projekt `cameras.json` fájljába (500 ms-os visszhangszűréssel). Ez magában foglalja a kamera neveket és színeket, az expozíció/erősítés/AE beállításokat, a pixelformátumot/felbontást/binninget, a kiváltási frekvenciát, az előnézeti beállításokat (renderelési felbontás, zajszűrés, élesség, vignettálás, színprofil, telítettség/kontraszt), tájolás, átfedések, csatornafelosztás, indexkonfiguráció, prediktív AE-beállítások, AE ROI, tömbnevek, megjelenítési mód, tömbfelvételi beállítások (beleértve az ROI kivágási pozícióját), valamint a rácsblokk (feed-zoom, nézetmód, rácszárlat, kézi csempesorrend, rejtett kamerák, zárt csempék, aktív kamera).
* A fényérzékelő-hozzárendelések a projekt `sensors.json` fájljában kerülnek mentésre.
* A projekt újbóli megnyitásakor a rendszer újra csatlakoztatja a hardvert, és az összes beállítást újra alkalmazza.
* **Nincs megnyitott projekt = csak munkamenet**: projekt hiányában az Chloros bezárásakor semmi sem marad meg.
* Csak munkamenet, projekttől függetlenül: szüneteltetett állapot, pontmérő-minták, a kameránkénti „Kalibrációs célpont” jelölőnégyzet (alapértelmezés szerint mindig ki van jelölve) és a mátrix-igazítási profil (a tervezés szerint munkamenetenként újraszámítva).
* Egy kivétel: a **Rögzítési beállítások** exportálási választásai és a rögzítési mód projektként maradnak meg a helyi alkalmazás-tárolóban, nem pedig az `cameras.json`-ben — lásd [Rögzítési beállítások és módok](capture.md).</array></camera></serial>
