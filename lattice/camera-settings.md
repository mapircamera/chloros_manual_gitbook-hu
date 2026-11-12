# Kamera beállítások

A **Kamerák** fül az Chloros LATTICE kamerákhoz tartozó élő vezérlőfelülete: egy fő képterület, amely minden csatlakoztatott kamerát élő csempeként jelenít meg, valamint egy oldalsáv, amely három oldal között vált – a **kameralista**, a**beállítási panel**(kameránkénti, tömbös vagy rögzítési beállítások – egyszerre egy), valamint az**Indexkalkulátor**. Ez az oldal a kameralista, a kameránkénti beállítási panel és a tömbös beállítási panel minden vezérlőjét ismerteti. A rögzítési módok, az exporttípus kiválasztása és az „All” rögzítési folyamat a kísérő oldalon, a [Rögzítési beállítások és módok](capture.md) alatt található.

A „Kamerák” fül az oldalsávon jelenik meg, miután a Chloros háttérrendszer készen áll. Az alábbi vezérlők mind az `127.0.0.1:5000`-en keresztül kommunikálnak a helyi háttérrendszerrel; a változtatások – ha másképp nem jelezzük – azonnal érvénybe lépnek az élő kameraképen.

## Az ezen az oldalon használt kameratípusok

A vezérlők a kiválasztott kamera típusától függően jelennek meg vagy rejtődnek el. A kézikönyvben az alábbi kifejezéseket használjuk:

| Kifejezés | Jelentés | Szűrőcsatornák |
| --- | --- | --- |
| **RGB-kamera** | LATTICE M3C FRGB szűrővel (a modell tartalmazza az `-FRGB` kódot) | Red / Green / Blue |
| **Bayer multispektrális** | LATTICE M3C FRGN, FOCN vagy FNGB szűrővel | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — egy keskenysávú szűrő, egy kalibrált sáv | Egy sáv |
| **Tömbtag** | Egy szinkronizált tömb részeként csatlakoztatott kamera (kombinált vagy különálló kijelző) | A szűrőjének megfelelően |

Az RGB-kamerák fotometrikus feldolgozást kapnak (fehér egyensúly, színprofilok, gamma); a multispektrális és monokróm kamerák a radiometrikus láncot kapják, és kihagyják a fotometrikus beállításokat. A tömbtagok a stream-szintű beállításokat (pixelformátum, felbontás, binning, trigger, képkockasebesség) a rendszerhez — ezek a sorok a kameránkénti ablakban írásvédetté válnak, és átkerülnek a rendszerbeállítások ablakába.

## A fő élőkép-terület

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->



Ha nincs csatlakoztatott kamera, az élőkép-területen egy **„Csatlakoztasson kamerát a kezdéshez”**felirat jelenik meg két gombbal:**Kamera csatlakoztatása**(zöld, megnyitja az egykamerás csatlakozási párbeszédablakot) és**Rendszer csatlakoztatása** (kék, megnyitja a rendszer csatlakoztatási párbeszédablakot). Maguk a csatlakozási párbeszédablakok a [Kamerák csatlakoztatása](connecting.md című dokumentumban vannak leírva; a rendszerrel kapcsolatos fogalmak (szinkronizálás, rétegek, sávszélesség) a [Többkamerás rendszerek](arrays.md) című dokumentumban. Ha olyan mentett projektet nyit meg, amelyben kamerák vannak, a kezdőképernyőn egy forgó ikon jelenik meg a „N mentett kamera újbóli megnyitása…” felirattal, miközben a Chloros visszaállítja az utolsó munkamenetből származó adatfolyamokat.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Felső sáv

| Vezérlő | Működése |
| --- | --- |
| **Nézetmód váltás**| Váltás a**rácsnézet**(minden csempe cellaként) és a**lista nézet**(teljes szélességű sorok: minden egyes kombinált rendszer egy összetett sorban, minden**különálló** rendszer tagja saját sorban, saját adatfolyammal és kalibrációs célpont táblázattal, majd alul EGY önálló kamera — az aktív) között. Eszköztippek: „Váltás rácsnézetre” / „Váltás listanézetre”. |
| **Rácszár**(lakat) | Alapértelmezés szerint**zárva** — a lapok a helyükön rögzítve. A zár feloldásával a lapokat bármelyik helyre áthúzhatja (a hézagok megmaradnak). A rács automatikusan újra rögzül minden alkalommal, amikor új kamera csatlakozik. Eszköztippek: „Rács feloldása (csempék húzásának engedélyezése)” / „Rács rögzítése (csempék rögzítése a helyükön)”. |
| **Adatfolyam-nagyítás** csúszka | Csempe mérete, 60 px-től a konténer teljes szélességéig. A cellák 4:3-as képarányt tartanak. 200 px-nél kisebb cellaszélesség esetén a név és az fps feliratok elrejtődnek, hogy a csempe tiszta maradjon. |

### Feed-csempék

Minden kamera egy összetett élő csempét jelenít meg; egy kamera emellett három szürkeárnyalatos **csatornánkénti felosztású** csempét is megjeleníthet (lásd [Csatornafelosztások](#display-overlays-drawn-over-the-live-feed)), a kamerasorok pedig egy kombinált csempét jelenítenek meg. Az aktív csempe a kamera (vagy a kamerasor) színében egy kijelölőgyűrűt tartalmaz.

Ha az egérmutatót egy csempe fölé viszi, megjelenik egy **X** bezárás gomb:

* Ha egy **összetett** csempét bezárjuk, miközben a csatorna-felosztásai továbbra is láthatók, akkor csak az összetett csempe rejtőzik el.
* Ha **egy önálló kamera utolsó látható csempéjét** bezárjuk, akkor az adott kamera kapcsolata megszakad.
* **A kombinált kamerarendszer tagjainak felosztott csempéi soha nem szakítják meg** a kamera kapcsolatát — csupán elrejtik azokat.

A rács feloldása után bármelyik csempét bármelyik helyre húzhatja; az elrendezés a projekttel együtt mentésre kerül.

## Oldalsáv — kameralista

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

Az oldalsáv első oldala felsorolja az összes csatlakoztatott kamerát és kamerasorozatot:

* **Kamera csatlakoztatása**(zöld) /**Tömb csatlakoztatása** (kék, szkennelés közben „Észlelés...” feliratot jelenít meg). Mindkettő le van tiltva, amíg a csatlakozási párbeszédablak nyitva van.
* **Minden rögzítése** (piros) — a felsorolt összes kamerát rögzíti a Rögzítési beállításokban kiválasztott exporttípusokkal. Nyitott projekt szükséges hozzá. Teljes dokumentáció a [Rögzítési beállítások és módok](capture.md) alatt található.
* **Rögzítési beállítások fogaskerék** (az „Összes rögzítése” mellett) — megnyitja a [Rögzítési beállítások ablakot](capture.md#the-capture-settings-pane). Projekt hiányában vagy rögzítés közben le van tiltva.

### Kamerasorok

Minden kamerasor színkódolt szegélyt (a kamera egyéni színét), egy „CAM” feliratot — kék **M**(master) vagy zöld**S** (slave) szerepbetűvel a tömb tagjai esetében —, valamint a megjelenítési nevet. Az alapértelmezett név `LATTICE-MODEL (serial)`; átnevezhető a kameránkénti beállítások panelen. Sor gombok:

| Gomb | Hatás |
| --- | --- |
| **Szem**| Láthatóság váltása. A rejtett kamerák eltűnnek a rácsból, és**kizárásra kerülnek az „Összes rögzítése”** funkcióból. |
| **Fogaskerék** | Megnyitja a kameránkénti beállítások panelt (következő szakasz). |
| **Szünet / Lejátszás**|**Csak a képernyőn** befagyasztja az élő előnézetet — a háttérben a rögzítés továbbra is fut. A szüneteltetett kamerák nem tudnak rögzíteni. |
| **X** | Kapcsolat bontása. A felhasználói felület azonnal frissül (optimális esetben); a háttérkapcsolat bontása 10–30 másodpercig tarthat. |

### Tömb sorok

A tömb sorai a tömb színében megjelenő „ARRAY” jelölést, a tömb nevét (amely a tömb beállításaiban átnevezhető) és egy **DAQ · be/ki**gombot jelenítik meg —**be** állapotban, ha a tömb szintű fényérzékelő be van állítva *vagy* bármelyik tag rendelkezik kameránkénti érzékelővel; a segítő szöveg pontosan felsorolja, melyik érzékelő mit továbbít. A tagkamerák behúzva, saját sorokban szerepelnek alatta. Tömb sor gombok: **szem**(az ÖSSZES tagot egyszerre elrejti/megjeleníti),**fogaskerék**(tömbbeállítások ablak),**X**(az egész tömb leválasztása).

A tömb sorokban és a tömbbeállítások ablakban használt fényérzékelő (DLS) állapotnak négy állapota van:**ki**,**várakozás**(még nincs spektrum),**aktív**(az elmúlt 3 másodpercben érkezett spektrum), és**elavult** — 3 másodperc alatt nem érkezett friss spektrum, de az utolsó leolvasást *még mindig használja* (a DAQ-leolvasások soha nem járnak le a rögzítési útvonalon).

Amíg át nem rendezed, a lista először a sorozatokat, majd az önálló kamerákat jeleníti meg, a kamerák (és az egyes sorozatok tagjai) **kamera sorrendben**: M3C (Bayer) az M3M (mono) előtt; az M3C-k között: FRGB, FRGN, FOCN, majd FNGB; az M3M-ek szűrőhullámhossz szerint, a legalacsonyabbtól kezdve. A rács alapértelmezett elrendezése ugyanezt a sorrendet követi. Az oldalsávban az önálló kamerákat és a teljes sorozatcsoportokat egymás mellett húzva átrendezheti a listát; a sorozat elemei önállóan nem húzhatók.

## Kameránkénti beállítások panel

A kamera során található **fogaskerék** gombbal nyitható meg. A panel a kameralistára csúszik.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Fejléc**: a kamera**színmintája**(kattintással megnyitható a natív színválasztó — ezzel állítható be az oldalsáv szegélyének és a csempe kiválasztógyűrűjének színe), a**név**egy ceruzával jelölt**Átnevezés**gombbal (ha üres nevet ment, a rendszer visszaáll az alapértelmezett `MODEL (serial)` névre), valamint a**×** gomb a bezáráshoz.

### Élő hisztogram

A panel tetején egy élő luma-hisztogram látható, amelyet a ~8 Hz-es frekvenciával aJPEG

-előnézetből számítanak ki. Az átlagot Bayer-súlyozással számítják ki — (R+2G+B)/4 —, hogy illeszkedjen a kamera saját AE-méréséhez.

* **Narancssárga (Orange) szaggatott vonal**= az AE célérték.**Húzza vízszintesen az újracélzáshoz** — az elengedéskor egy parancs kerül elküldésre, a húzás pedig átváltja az AE célmódot Kézi üzemmódra.
* **Zöld (Green) folytonos vonal** = a tényleges átlagos fényerő (amit az AE jelenleg biztosít).
* **RGB gomb** (jobb felső sarok): a kamera szűrője szerint színezett, sávonkénti átfedő hisztogramok megjelenítését kapcsolja be/ki (pl. FRGN esetén: szürke NIR, zöld, piros). Mono (M3M) kameráknál a gomb felirata „MONO”, és le van tiltva — mono módban mindig az egysávos fényerő-hisztogram jelenik meg.
* Az X-tengely feliratai az aktuális képpontformátum szenzorának bitmélységét követik: 0..255, 0..1023, 0..4095 vagy 0..65535.

### Kameraadat-sorok

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->



| Sor | Viselkedés |
| --- | --- |
| **Modell** | Csak olvasható (pl. `LATT-M3C-L87-FRGN`). |
| **Radiometrikus kalibrálás**|**Green** **„Active”**jelölés, amelyen a kalibrációs szint, a hash, a kalibrálás dátuma és a sávlista látható, a kamera kalibrációs csomagjából betöltve (lásd [Gyári radiometrikus kalibrálás](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**RGB-kamerák esetén elrejtve** — ezek fotometrikus fehéregyensúly-kalibrálással rendelkeznek, nem sávonkénti sugárzási értékekkel. |
| **Kalibrációs jelentés**|**Letöltés** gomb — megnyitja a kamera sorozatszámonkénti NIST kalibrációs tanúsítványát PDF formátumban az operációs rendszer nézőjében. Ha a tanúsítvány még nincs a gyorsítótárban, a Chloros helyette egy tippet jelenít meg. |
| **Sorozatszám** | Csak olvasható. |
| **Firmware**| Megjeleníti az aktuális verziót, majd meghatározza az adott modellhez elérhető verziót (modellenként tárolva a gyorsítótárban — egy N kamerából álló rendszer egyszer ellenőrzi a szervert). Állapotok: „Ellenőrzés…” →**„Frissítés X-re”**gomb → „Frissítés folyamatban…” → „Frissítve A → B” / „Sikertelen: …” / „Kihagyva: …” / zöld**„Naprakész”**. A frissítés gomb eszköztippje: „Gyári visszaállítás + frissítés + UserSet1 újraprogramozása. ~2–3 perc; ne válassza le.” |
| **IP** | Csak olvasható. |
| **Hőmérséklet** | Csak olvasható, 3 másodpercenként frissül. 65 °C felett narancssárgára, 75 °C felett pedig pirosra vált, és megjelenik egy ⚠ jel. |
| **Kalibrációs cél** jelölőnégyzet | Engedélyezi az ArUco reflektancia-célpont felismerését a panelekhez tartozó NDVI-érvényesítési táblázattal az élő kép alatt (lista nézet) — lásd [A kalibrációs célpont táblázatot](#the-calibration-target-table). Csak munkamenetben elérhető — mindig kikapcsolt állapotban nyílik meg. |
| **Fényérzékelő** legördülő menü | Egy DAQ fényérzékelőt (DAQ-E/M/U, a Fényérzékelők fül listájából) rendel ehhez a kamerához a lefelé irányuló fény (DLS) megvilágítási korrekciója és a prediktív automatikus expozíció érdekében. A „A „Nincs” opció törli a hozzárendelést. Ha nincs csatlakoztatott érzékelő, a legördülő menü a következő üzenetet jeleníti meg: „(nincs csatlakoztatott érzékelő — nyissa meg a DAQ fület)”. A hozzárendelés a projekttel együtt kerül mentésre. |

#### A Kalibrációs célpont táblázat

A táblázat **kizárólag validációs célú** („csak validáció — nem módosítja a kalibrációt”). A fejlécében szerepel a kamera sorozatszáma, a szűrő, a célpont száma és a jelölő mérete pixelben, és panelonként egy sor található:

* **Panel**, majd sávonként egy oszlop a panel értékével — egy Bayer-kamera a három sávját név és hullámhossz szerint jeleníti meg (pl. FRGN: Red 660 nm / Green 550 nm / NIR 850 nm); egy **mono (M3M) kamera az egyetlen sávját** jeleníti meg (pl. `F750`), és nincs NDVI oszlopa.
* **Sat** — a panel telített képpontjainak aránya (borostyánsárga 1%-tól, piros 5%-tól).
* **NDVI Meas**,**NDVI Theo** (a panel referencia-reflektanciájának NDVI-értéke) és **Δ**— kizárólag Bayer multispektrális kamerák esetében. Az „NDVI Meas” után található kis jelölés jelzi, hogy a mért NDVI-értéket miből számították ki:**ρ**(reflektancia — fényérzékelő-leolvasás áll rendelkezésre),**L**(sugárzás — nincs fényérzékelő-érték) vagy**DN**(nyers kameraértékek). Csak a**ρ** NDVI összehasonlítható az NDVI Theo-val: L és DN esetén a Δ oszlop szürkén jelenik meg, és egy megjegyzés jelzi ezt („Nincs fényérzékelő-leolvasás: az NDVI a sugárzási értékből származik, ezért a Δ nem kalibrációs ellenőrzés” / „Az NDVI a nyers kamerás értékekből származik, ezért a Δ nem kalibrációs ellenőrzés”). ρ esetén a Δ 0,02 alatt zöld, 0,05 alatt sárga, egyéb esetben piros.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Expozíció és erősítés

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Az itt található összes numerikus beviteli mező gyorsított forgatógombot használ: érintés = ±1, 1,5 másodpercnél hosszabb lenyomva tartás = ±10, 3 másodpercnél hosszabb lenyomva tartás = ±100. Az érték a kamera felé akkor kerül elküldésre, amikor elengedi a gombot.

| Vezérlő | Tartomány / lehetőségek | Alapértelmezett | Vonatkozik | Működése |
| --- | --- | --- | --- | --- |
| **Expozíció (us)**| A kamera élő min./max. értéke | Automatikus | Minden | Expozíciós idő mikroszekundumokban,**Automatikus/Kézi**kapcsolóval. Automatikus = folyamatos, kamerán belüli automatikus expozíció. A kézi expozíció beállítása esetén az**Erősítés** automatikus marad, ha korábban is az volt, így a kamera teljes erősítési tartománya szabadon használható, amíg az erősítést nem szerkeszted vagy kézi üzemmódra nem állítod. |
| **Erősítés (dB)**| A kamera aktuális min./max. értéke (pl. legfeljebb 48 dB) | Kézi (ki) | Minden | Analóg/digitális erősítés saját**Auto/Manual** kapcsolóval. |
| **AE célfényerő**| 0–255 | 80,**Auto**mód | Minden (szerkeszthető, ha az AE vagy az automatikus erősítés be van kapcsolva) | Az AE által megcélzott fényerő.**Auto**(alapértelmezett) módban egy hisztogram-alapú háttérvezérlő maga választja ki a célt, az expozíciót az érzékelő maximális értékének 60–75 %-án tartva. Érték beírásával vagy a hisztogram narancssárga vonalának húzásával átvált**Kézi** módra. |
| **AE-simítás** | 0,5–40, lépésköz 0,1 | 8,0 | Minden | AE-csillapítás. Eszköztipp: „Alacsonyabb érték = az AE gyorsabban reagál (magas képkockasebességnél pulzálhat). Magasabb érték = simább / lassabb.” Az alapértelmezett értéktől jelentősen eltérő értékek az AE pulzálását okozhatják, és destabilizálhatják a streamet magas képkockasebesség mellett; a 8,0 a stabil alapértelmezett érték. |
| **AE érdeklődési terület**| Engedélyező jelölőnégyzet +**Célzás**gomb | Ki | Minden | Ha be van kapcsolva, az AE csak a zöld szaggatott vonallal jelölt területet méri a teljes képkocka helyett.**A célzás** aktiválja a kattintással történő elhelyezést az élő képen: egy kattintás a területet a képkocka 30%-ánál központosítja; kattintás és húzás egyéni téglalapot rajzol (minimum 5% × 5%). A Célzás egy elhelyezés után automatikusan kikapcsol. A területet a beállított forgatás/tükrözés szerint visszaalakítja a kamera natív koordinátarendszerébe, és a projekttel együtt elmenti. |
| **AE Tune Speed** | 0,1–5, 0,1-es lépésenként | 1,0 | Csak tömbtagok | Az automatikus AE-célpont milyen gyorsan követi a jelenet fényerejének változásait; 1,0× esetén 2,5 másodpercenként ellenőrzi újra. |
| **Fényes részek védelme** | Szigorú (1 %) / Normál (5 %) / Laza (15 %) | Szigorú | Azok a kamerák, amelyeknél ez a beállítás elérhető | A képkocka mekkora része válhat fehérré, mielőtt az AE elsötétíti a képet. |

{% hint style="info" %}
**Fényigény a Bayer multispektrális kamerákhoz (RGN / OCN / NGB):** a jelenetnek mindhárom csatornában elegendő fénynek kell lennie, különben a kalibrálás nem működik megfelelően — egyetlen érzékelő expozíciója lefedi mindhárom spektrumot. Használjon DAQ fényérzékelőt a fény méréséhez, vagy váltson teljes monó (M3M) üzemmódra, hogy minden sáv saját expozíciót kapjon. Ha egy felvétel megsérti ezt a feltételt, az Chloros észleli és figyelmezteti Önt (az unmix-clamp értesítés). Élő kép esetén egy csatorna-felosztott mozaik **„Alacsony megbízhatóságú sávadatok · N%”** feliratot jelenít meg, ha az unmix a sáv pixeljeinek 25%-ánál vagy annál nagyobb arányban korlátozta azt.
{% endhint %}

{% hint style="info" %}
**Hogyan osztja el az automatikus expozíció az expozíciós időt és az erősítést (1.3.0).** Ha az erősítés automatikus beállításon van, a Chloros 0 dB-en tartja az erősítést, amíg az expozíció el nem éri a megengedett leghosszabb értéket (az AE felső határát, egy tömb trigger periódusát vagy a DAQ expozíciós felső határát); csak ezután emelkedik az erősítés. Egy tömbben a trigger frekvencia az elsődleges: az automatikus expozíció kitölti a frekvencia által megadott periódust, majd emeli az erősítést, így a sötétedő jelenet már nem csökkenti az automatikus triggerfrekvenciát (az 1.2.0-s verzióban még csökkentette). Ha egy sorozat tagján **kézi**expozíciót állítunk be, amely hosszabb, mint a trigger periódus, a sorozat triggerje lelassul, hogy ehhez igazodjon; az automatikus frekvencia akkor áll vissza, ha minden olyan elem, amely lelassította, újra automatikus expozícióra vált. Ha az**Expozíció**és az**AE célfényerő** beállítások egyaránt automatikusak, és a DAQ fényérzékelő értéket jelez, az egyes kamerák expozíciója az automatikus expozíció stabilizálódása után változatlan marad, mindaddig, amíg a mért fényerő ±25 %-os tartományon belül marad; egy nagyobb, tartós fényváltozás, a beállítások újraindítása (például egy ROI-változás) vagy a saját expozíciós beállítások módosítása újraindítja a rendszert.
{% endhint %}

{% hint style="info" %}
**Az automatikus expozíció működése (1.3.0).**

* Kézi expozíció esetén az erősítés automatikus beállításon marad, amíg az erősítést nem módosítja.
* A kis fényes területek nem sötétítik el a jelenetet: a képkocka legfeljebb 1%-át kitevő tükröződést a rendszer figyelmen kívül hagyja, az ismert kalibrációs célpanelek esetében pedig a panel legfeljebb 0,5%-át.
* A DAQ expozíciós felső határa csak akkor érvényes, ha kalibrációs cél van a látómezőben; ha nincs látható panel, a kamera a jelenetnek megfelelően állítja be az expozíciót.
* Az expozíció állandó fényviszonyok mellett stabil marad, és jelentős változás esetén néhány másodpercen belül stabilizálódik. A túlvilágítás már nem korlátozza véglegesen az expozíciót, és az érdeklődési terület (ROI) megváltoztatása esetén a kamera újra beállítja az expozíciót.
* A fényerő-célérték nem lép ki a kamerahatókörén kívülre, így a visszatérő fény nem okoz tükröződést a képen.
* Az a kamera, amelynek automatikus expozíciója erős napfényben nem reagál, a fényérzékelő által mért, vagy – ha nincs csatlakoztatva – a kamera saját képei alapján mért fényszintet tartja fenn, és elengedi, amikor a fényviszonyok megváltoznak. Egy sötét vagy eltakart fényérzékelő ezt nem zavarja.
{% endhint %}

### Pixelformátum és felbontás

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**A tömb elemei** csak olvasható „Aktuális” (formátum + Szélesség x Magasság) és „Binning” sorokat jelenítnek meg a „A tömbbeállításokban állítható be” megjegyzéssel — egy elemnél végrehajtott adatfolyam-újraindítás megszakítaná a szinkronizálást, ezért ezeket a [tömbbeállítások panelen](#array-settings-pane) panelen kezelik.**Az önálló kamerák** a következőket kapják:

| Vezérlő | Választási lehetőségek | Működése |
| --- | --- | --- |
| **Pixelformátum** | 8-bit / 10-bit / 12-bit / 16-bit | Az érzékelő bitmélysége; a formátumcsalád az érzékelőé. Az alapértelmezett érték 12-bit, és a projekt elmenti a beállított értéket (beleértve a 8-bitet is), majd ehhez igazodva csatlakozik újra. |
| **Felbontás** | Teljes / Fele / Negyed | Az aktuális binninghez viszonyítva: Teljes = 2048/N × 1536/N N×N binning esetén. |
| **Binning** | 1x1 (nincs) / 2x2 / 4x4 | Hardveres N×N binning — a nagyobb értékek csökkentik a felbontást, de növelik az SNR-t és a képkockasebességet. A beállítás megváltoztatása újraindítja az adatfolyamot, és az összes ROI-t az új teljes látómezőre állítja vissza. |
| **Jelenlegi** | csak olvasható | A ténylegesen érvényes WxH és (x, y) eltolás. |

### Élő előnézet

Ebben a szakaszban minden **kizárólag a kijelzőre vonatkozik**— megváltoztatja az élő adásban látható képet, míg a mentett felvételek lineárisak és változatlanok maradnak — egy kivétellel: a**Vignette** radiometrikus, és az exportált fájlokat is érinti (lásd alább).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Vezérlő | Tartomány / lehetőségek | Alapértelmezett | Vonatkozik | Mit csinál |
| --- | --- | --- | --- | --- |
| **Renderelési felbontás** | 360p (leggyorsabb) / 480p / 720p / 1080p / Natív érzékelő felbontás (a leglassabb) | 720p | Minden | Az a magasság, amelyen a háttérrendszer futtatja a radiometrikus előnézeti láncot. Az alacsonyabb érték nagyobb képkockasebességet biztosít a látómező megváltoztatása nélkül. |
| **Index**| Engedélyezés jelölőnégyzet + fogaskerék | Ki | Csak Bayer multispektrális,**nem** kombinált sorozatú elemek | Élő növényzet-index előnézet. A fogaskerék megnyitja a megosztott [Indexkalkulátor](#index-calculator-pane) ablakot, amelybe előre betöltve vannak a kamera szűrőinek természetes sávjai (pl. `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). Az egyéni kifejezés és a LUT (be/ki, alapértelmezett szint 7-stop, alapértelmezett min. 0,2, alapértelmezett max. 1) minden előnézeti képkockán kiszámításra kerül. A kombinált tömb tagjai elrejtik ezt a sort — a tömb egy megosztott indexet tartalmaz. |
| **Fehér egyensúly** | Ki / Egyszer / Folyamatos + újrafelvétel gomb | Folyamatos | Csak RGB | Élő fehéregyensúly. A frissítés gomb újra rögzíti a fehéregyensúlyt az aktuális DLS-spektrumból (ki van kapcsolva, ha az üzemmód Ki). |
| **Gamma** | Be / Ki | Be | Csak RGB | Gamma (γ = 2,2 LUT) megjelenítése az élő előnézeten. A mentett felvételek lineárisak maradnak. |
| **Zajszűrés** | Jelölőnégyzet + erősség 0–100 | Ki / 50 | Minden (kameránként, még a tömbökön belül is) | Kétoldalas szűrő az élő előnézeten. Magasabb érték = simább, de lágyabb részletek. |
| **Élesség** | Jelölőnégyzet + erősség 0–100 | Ki / 30 | Minden | Élességmaszk az élő előnézetben, utoljára alkalmazva. Fokozhatja a zajt. Csak előnézetben. |
| **Vignette**| Jelölőnégyzet + erősség 0–100 | Ki / 0 | Minden | Kézi maradék vignettálás-eltávolítás (fényesíti a sarkokat), a sorozat Smart Vignette becslésére rétegezve.**Radiometrikus — hatással van az élő nézetre ÉS az exportokra**, ellentétben a Zajszűréssel/Élességgel. |
| **Színprofil** | Raw / Lineáris / Természetes / Fokozott / Egyéni hőmérséklet | Természetes | Csak RGB | Lásd alább. |
| **Színhőmérséklet** | 2000–10000 K, 100-as lépésenként | 5500 K | Csak RGB, Egyéni hőmérséklet profil | A fehér egyensúlyt egy rögzített korrelált színhőmérséklethez rögzíti (a DLS-bemenet figyelmen kívül marad). A legutóbb kiválasztott Kelvin-érték profilváltáskor is megmarad. |
| **Fényes részek** | −100–0, 1-es lépésenként | 0 (ki) | Csak RGB | Újdonság a 1.3.0-s verzióban. A kép legfényesebb, legfehérebb területeit sötétíti (0 = változatlan, −100 = legerősebb). Alkalmazható az élő előnézetre **és a kijelzőszintű (Előnézet) felvételekre**, amelyek rögzítik a beállítást (`Chloros:DisplayHighlights`); a sugárzási és visszaverődési értékek exportálására ez soha nem hat. Nem képes helyreállítani a túlexponált érzékelőadatokat — ehhez csökkentse az expozíciót. A zebra-réteg a sötétített képet követi. A Raw színprofil használata esetén letiltva („A Raw színprofil használata esetén nem alkalmazható.”). |
| **Telítettség** | 0–200 (100 = semleges) | 100 | Csak RGB | HSV-telítettség az élő előnézetben. |
| **Kontraszt** | 0–200 (100 = semleges) | 100 | Csak RGB | Lineáris kontraszt a közép-szürke környékén az élő előnézetben. |
| **Tükör H / Tükör V** | Jelölőnégyzetek | Ki | Minden | Az előnézet vízszintes / függőleges tükrözése. |
| **Forgatás**| 0° / 90° / 180° / 270° | 0° | Minden | Az előnézet elforgatása. A tájolás a háttér előnézeti lánc végén kerül alkalmazásra.**Mentett felvételek (1.3.0):**egy önálló kamera Előnézet, Sugárzás, Fényvisszaverődés és Index termékei a Tükör/Forgatás funkcióval forgathatók, míg**a Raw és a Debayered fájlok a kamera natív tájolását tartják meg**; minden felvételi fájl rögzíti a beállítást az XMP-jében, így a raw fájl későbbi feldolgozása során ugyanúgy alkalmazza azt.**A tömb tagfájljai natív állapotban maradnak** (a beállítás rögzítésre kerül, de nem kerül alkalmazásra), és a tömb összetett nézetei figyelmen kívül hagyják azt. |

**Színprofil-szemantika** (RGB-kamerák):

* **Raw** — teljes mértékben megkerüli a feldolgozási láncot.
* **Lineáris** — sötétjel + síkkép + fehéregyensúly; nincs színmátrix, nincs gamma.
* **Természetes** *(alapértelmezett)* — lineáris, plusz a mért színkorrekciós mátrix és egy jelenethez alkalmazkodó tónusgörbe.
* **Továbbfejlesztett**— Természetes, plusz élénkség és CLAHE helyi kontraszt. A többletköltség**csak az élő előnézetre** vonatkozik — a mentett felvételek mindig a teljes utómunkát kapják, függetlenül a profiltól.
* **Egyéni hőmérséklet** — Természetes, a fehéregyensúlyt a választott Kelvin-értékre rögzítve.

{% hint style="warning" %}
A „Természetes”, „Továbbfejlesztett” és „Egyéni hőmérséklet” beállítások esetén a panel egy megjegyzést jelenít meg: a képkockákat a saját jelenetüknek megfelelően világosítják, így a mentett *megjelenítési* képek nem összehasonlíthatók képkockáról képkockára. **A mérésekhez exportálja a sugárzási vagy visszaverődési értékeket.**
{% endhint %}

### Kijelző-átfedések (az élő kép fölé rajzolva)

Ezek kizárólag a felhasználói felületen jelennek meg — a videóra rajzolódnak, soha nem érintik az élő közvetítést vagy a rögzített felvételeket.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Átfedés | Vezérlők | Alapértelmezett | Működése |
| --- | --- | --- | --- |
| **Zebra** | Jelölőnégyzet + küszöbérték 200–255 | Ki / 250 | Magenta színű átlós csíkok a levágott képpontokon. |
| **Célkereszt** | Jelölőnégyzet | Ki | Képkocka-középpont jelölés. |
| **Rács** | Ki / 3 × 3 / 9 × 9 | Ki | Kompozíciós rács. |
| **Hisztogram** | Jelölőnégyzet + a képkocka szélességének 0,10–0,90-es aránya | Ki / 0,25 | A streamen megjelenő hisztogramcsík. |
| **Fókuszcsúcs** | Jelölőnégyzet + küszöbérték 20–200 + színminta | Ki / 80 / `#ff5722` | Sobel-élkiemelés a fókuszáláshoz. |
| **Csatornafelosztás** | „Felosztások megjelenítése (Red / Green / NIR)” / „Osztások elrejtése” gomb | Elrejtve | Három független, csatornánkénti szürkeárnyalatos mezőt ad hozzá a kompozit kép mellé (a gomb felirata a kamera szűrőcsatornáit követi). Minden osztott mező húzható, és a kamera keret színét veszi át. Monokróm kamerákon nem elérhető. A projekttel együtt mentésre kerül. |

### Pontmérő

* **Kattintás a mintavételhez**jelölőnégyzet: kattintson az élő képre egy pixel mintavételéhez (egy célkereszt jelöli meg), vagy kattintson és húzza a kurzort egy területre a pixelek átlagának kiszámításához. A**Törlés**gomb törli a mintát és a célkeresztet. Kizárja egymást az AE-ROI**Célzás** móddal.
* **Megjelenítés** legördülő menü:**Raw (bitmélység)**— natív digitális értékek az érzékelő bitmélységében (pl. 12-bit → 0..4095) — vagy**Kijelző (8-bit)** (alapértelmezett). Ha élő index aktív, a Kijelző helyette a kiszámított indexértéket (pl. NDVI) jeleníti meg.
* A kijelzőpanel felsorolja a képpontkoordinátákat, a képkocka méretét, a képpontformátumot, a bitmélységet, valamint egy csatornatáblázatot (Chan / Érték / %) a sávcímkékkel és hullámhosszakkal; a Bayer zöld párok átlagolva vannak; a régióminták „N px avg” feliratot jelenítik meg.

A pontmérő állapota csak a munkamenetre vonatkozik.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Prediktív automatikus expozíció (DLS-vezérelt)

Ez a szakasz csak akkor jelenik meg, ha **legalább egy DAQ fényérzékelő van csatlakoztatva** — a megoldónak élő, lefelé irányuló spektrumra van szüksége a működéshez.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Vezérlés | Tartomány | Alapértelmezett | Működése |
| --- | --- | --- | --- |
| **Engedélyezés** | Jelölőnégyzet | Be (önálló kamerák) | Egy zárt formájú megoldó a DLS-spektrumot és a kamera kalibrációs csomagjának skalárjait használja fel arra, hogy a legfényesebb sávot a telítettség közelébe állítsa, miközben a leghalványabb sávot az SNR-küszöb felett tartja — megoldásonként egyetlen expozíció írása, nincs stabilizáló hurok. Napenergiával működő időraffozáshoz tervezték, ahol minden felvételnek helyesen kell lennie exponálva. A háttérrendszer észrevétlenül visszatér a reaktív AE-re, ha a DLS-érték elavult vagy hiányzik, vagy a kalibrációs csomag nincs betöltve. |
| **Simaítás (α)** | 0,05–1,0, 0,05-ös lépésenként | 0,3 | Az egymást követő prediktív megoldások simítása (alacsonyabb érték = simább). |
| **Jelenet-reflektancia**|**ρ újrakalibrálása** gomb | — | Újra becsüli a megoldó által használt jelenet-reflektancia tényezőt. |

{% hint style="info" %}
**A tömbösszekapcsolás alapértelmezés szerint kikapcsolja a prediktív AE-t** — tömbök esetén a Chloros intelligens AE-je és a kamera oldali automatikus expozíció kezeli az expozíciót (telítettségvédelemmel), és a prediktív AE egyetlen jelenetrereflektancia-becslése vegyes jelenetek esetén nem megbízható. Itt kameránként újra bekapcsolhatja, ha kifejezetten DLS-vezérelt radiometrikus expozíciót szeretne.
{% endhint %}

**DAQ-vezérelt expozíciós felső határ és beeső fényre rögzített AE.** A fenti jelölőnégyzettől függetlenül, ha egy DAQ fényérzékelőt rendelnek egy RGB-kamerához, a Chloros — a mért abszolút lefelé irányuló besugárzás alapján — kiszámítja azt a maximális expozíció×erősítést, amelynél egy 100%-os fényvisszaverő felület a clipping alatt marad, és ezt **felső határ**az automatikus expozícióra. Az 1.3.0-s verziótól kezdve a felső határ csak akkor érvényes, ha egy kalibrációs célpont a kamera látómezejében van (az elmúlt 10 másodpercben a képkockáján belül látható); ha nincs cél a látómezőben, akkor a kamera szokásos automatikus expozíciója lép működésbe a saját fénycsúcs-védelmével, így a jelenet többi része nem marad szükségtelenül sötétebb. Amíg a felső határ aktív, a kamera**beeső fényre rögzített**: nyitott hurkú üzemmódban működik a beeső fény alapján mért expozícióval, 0 dB-es erősítéssel — az expozíció a mért fényt követi, nem a jelenet tartalmát. Mivel a felső határ csak lerövidítheti az expozíciót, önmagában nem okozhat túlvilágosodást. Az 1.3.0-s verziótól kezdve a felső határt a kamera saját expozíciós állandójával számítja ki a kalibrációs csomagjából (csak akkor használja a flottaátlagot, ha a csomag nem tartalmaz ilyet). A felső határ automatikusan kikapcsol — és a normál jelenet-AE újraindul —, ha nincs kalibrációs cél a látómezőben, hiányzik a DAQ-leolvasás, elavult (>30 s), sötét, vagy ha a képkocka ≥15 %-a a rögzített expozíció mellett túlvilágosodik (vagyis az érzékelő és a kamera eltérő megvilágítást érzékel). Nincs erre vonatkozó GUI-kapcsoló; ez a szokásos viselkedés, amikor egy RGB-kamerához DAQ van társítva.

### Adatgyűjtés és kiváltás

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

A tömb tagjai emellett olvasási joggal rendelkező **Szerep**(Master kék / Slave zöld),**Szinkronvonal**és**Társak** sorokat is megjelenítenek.

| Vezérlés | Választási lehetőségek | Alapértelmezett | Megjegyzések |
| --- | --- | --- | --- |
| **Trigger Mode** | Off / On | On | Tömbtagoknál letiltva (a tömb kezeli a triggerelést). |
| **Trigger forrás** | Szoftver / Line0 (M8) / Line1 / Line2 | Line0 | A Trigger mód kikapcsolt állapotában elrejtve; a tömb tagjainál letiltva. A Line0 az M8 optikailag izolált külső trigger bemenet. |
| **Képkockasebesség**| Automatikus / Kézi + érték | Automatikus |**Automatikus**: a kamera képkockasebesség-korlátozása ki van kapcsolva — az expozíció határozza meg az fps-t, és a mezőben a valós idejű tényleges sebesség jelenik meg.**Kézi**: a képkockasebességet egy csúszkával korlátozhatja (1-től a sávszélesség által korlátozott maximumig), amely az aktuális valós sebességből indul ki. A tömb tagjai egy csak olvasható „N fps (élő)” értéket látnak, amelyhez a „Beállítás a tömbbeállításokban” felirat tartozik. |

### Hálózat / Átvitel

| Sor | Viselkedés |
| --- | --- |
| **Csomagméret**| 1500 (Standard) / 9000 (Jumbo) — alapértelmezés szerint**Jumbo**. |
| **Átviteli sebesség** | Csak olvasható kapcsolati átviteli sebesség-korlát MB/s-ban. A háttérrendszer minden csatlakozáskor/leváláskor újraelosztja ezt az összes csatlakoztatott kamera között. |
| **Pufferkezelés** | Csak olvasható pufferkezelési mód. |

### Felvétel

A panel végén található egy **„Felvételi beállítások megnyitása…”** gomb, amely a [Felvételi beállítások panelre](capture.md#the-capture-settings-pane) ugrik (addig letiltva, amíg nincs megnyitva projekt — „Hozzon létre vagy nyisson meg egy projektet a rögzítések mentéséhez”). Ha a kamera el van rejtve vagy szüneteltetve, egy figyelmeztetés emlékezteti Önt, hogy a rögzítés előtt vegye elő a kamerát vagy folytassa a felvételt.

## Tömbbeállítások ablak

Nyissa meg az ARRAY soron található **fogaskerék**gombbal. Fejléc: a tömb neve, egy átnevezéshez szükséges ceruzával és a bezáráshoz szükséges**×** gombbal. Az alábbi, *csak kombinált* jelöléssel ellátott szakaszok csak a kombinált megjelenítési módban összekapcsolt tömbök esetén jelennek meg.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Szinkronizálás

Csak olvasható **Master**,**Slaves**és**Sync Line** sorok.

### Környezeti fényérzékelő

Kombinált és különálló tömbök esetén egyaránt megjelenik:

* **Fényérzékelő** legördülő menü — egy DAQ-t rendel a teljes tömbhöz. A kiválasztás azonnal hatályba lép, átterjed minden tagkamera saját Fényérzékelő legördülő menüjébe (kameránként továbbra is felülírható), és elkezdi a spektrumok továbbítását a tömb felé.
* Élő **Állapot** sor: Ki · „Várakozás az első spektrumra…” · „Aktív — a rendszer összes kamerája megvilágítás-korrigált” · „Az elmúlt 3 másodpercben nem érkezett új spektrum — továbbra is az utolsó leolvasott értéket használja (nincs elavult időtúllépés)…”.
* Megjegyzés az ablakban: „Tömb-szintű radiometrikus korrekció. A kameránkénti beállítások felülírják ezt.”

### Kalibrációs célpont

Saját csoportja közvetlenül a Környezeti fényérzékelő után található, mind a kombinált, mind az elkülönített tömbök esetében:

* **Engedélyezés** jelölőnégyzet — „MAPIR ArUco célpont észlelése és az NDVI-nek a panel reflexiójával kapcsolatos LUT-val való összehasonlítása”; ez vezérli a kombinált csempe célpont-átfedését és érvényesítési tábláit. **Különálló** sorozat esetén minden elem saját [Kalibrációs célpont tábláját](#the-calibration-target-table) (a listanézetben kamerasoronként egy-egy).

### Felvétel — egységes érzékelőbeállítások

Mind a kombinált, mind a különálló sorozatoknál megjelenik (a csoport fejlécén az áll: „Felvétel (kamerák között egységes)”). Ezek a beállítások egységesen vonatkoznak minden tagra (a tagonkénti módosítások megszakítanák a szinkronizálást). A szerkesztések előkészítésre kerülnek, és az **Alkalmaz**gombbal együtt kerülnek alkalmazásra — kivéve a**ROI be / ROI ki** kapcsolót, amely azonnal hatályba lép.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Vezérlő | Választási lehetőségek / tartomány | Működése |
| --- | --- | --- |
| **Pixelformátum** | 8-bites / 10-bites / 12-bites / 16-bites | Egységes bitmélység minden tag számára; minden kamera a saját érzékelőjéhez ezt a bitmélységet veszi alapul. |
| **Binning** | 1x1 / 2x2 / 4x4 | Hardveres binning — megőrzi a teljes látómezőt, miközben javítja a jel-zaj arányt és a képkockasebességet. A beállítás megváltoztatása az ROI mezőket az új teljes látómezőre állítja vissza. |
| **Felbontás** előbeállítás | Teljes / Fele / Negyed | A binninghez viszonyítva; az ROI mezőket központosított kivágással tölti ki. |
| **ROI kivágás (px)**|**ROI be / ROI ki**kapcsoló,**Kiválasztás**gomb, Szélesség / Magasság / X / Y mezők, teljes érzékelőtérkép | Érzékelő-kivágás — lásd [Az ROI-szerkesztőt](#the-roi-editor). A szélesség/magasság 16-os többszöröséhez igazodik (minimum 64); az eltolások 4-es többszöröséhez igazodnak. A „max WxH” segítség a felső határt jelzi, a**Visszaállítás** gomb pedig visszaállítja a mezőket a teljes látómezőre. Szerkesztés közben a narancssárga kivágási keret nyomon követi a térképen és a szenzortáblán végzett minden mezőmódosítást. |
| **Triggerfrekvencia**| Automatikus / Kézi kapcsoló + fps 0,5–10, lépésenként 0,5 |**Automatikus**(alapértelmezett): a háttérrendszer a felbontásból és a sávszélességből számítja ki a kiváltási gyakoriságot — a beviteli mező le van tiltva, és a kiszámított értéket jeleníti meg.**Kézi**: az**Alkalmaz** gomb megnyomásával rögzíti az értéket. Ha egy rögzített gyakoriságot az új képkockaméret nem tud támogatni, akkor a rendszer a méret által megengedett leggyorsabb gyakoriságra állítja át, és visszatér az eredeti értékre, amint a képkockaméret újra lehetővé teszi. |

Megjegyzés a panelen: „A formátum/felbontás megváltoztatása rövid ideig újraindítja az összes kamerát. A triggerfrekvencia azonnal hatályba lép.” Az **Alkalmazás / Mégse** gombok a panel alján találhatók.

#### Az ROI-szerkesztő

Újdonság a 1.3.0-s verzióban. Az ROI-kivágási sor és az alatta lévő térkép együtt működnek:

* **ROI be / ROI ki**—**azonnal**be- vagy kikapcsolja az érzékelő kivágását (a kamerák rövid ideig újraindulnak; a mátrixlapon a „Beállítások alkalmazása…” felirat jelenik meg).**Ki**beállítás esetén a szenzor teljes tartalma olvasható, és**a kivágás beállítása megmarad**— a mezők továbbra is ezt jelenítik meg;**Be** beállítás esetén a kivágás újra alkalmazásra kerül. Ha kivágás beállítása nélkül kapcsolja be az ROI-t, akkor egy középre igazított, fél méretű kivágás kerül alkalmazásra („Kivágás nem volt beállítva, ezért egy középre igazított, fél méretű kivágás került alkalmazásra. Állítsa be a mezők segítségével.”).
* **Kiválasztás**— megrajzolja a kivágást a térképen: húzza át rajta az egérrel, vagy kattintson két egymással szemben lévő sarokra (az Esc billentyűvel törölheti). A téglalap a W/H/X/Y mezőkbe kerül, a fenti lépésekhez igazodva; addig nem kerül alkalmazásra semmi, amíg meg nem nyomja az**Alkalmaz** gombot, így először a mező nyilakkal finomhangolhatja. A Választás gomb minden megnyomására egy téglalap jön létre.
* **A térkép**a kivágást az egész érzékelőn megjeleníti, így vissza is szélesítheti: „Teljes érzékelő, élő”, miközben a kamerák a teljes érzékelőt olvassák, vagy „Teljes képkockás pillanatkép {időpont}-tól. A kamerák csak a kivágott részt olvassák be.”, amíg kivágva vannak. Ha a sorozat eddig csak kivágott képet továbbított, a térkép a következő üzenetet jeleníti meg: „Még nincs teljes képkockás nézet: ez a sorozat eddig csak kivágott képet továbbított.” feliratot jelenít meg egy**Teljes képkocka megjelenítése** gombbal, amely kikapcsolja az ROI-t. A kivágás `W×H @ X,Y` címkéje a kereten kívül helyezkedik el (felette, vagy alatta, ha a keret érinti a felső szélét), mind a térképen, mind az élő csempén.
* Az **Alkalmaz**gomb véglegesíti a mezőket: egy kivágás bekapcsolja az ROI-t; a teljes képkocka (az**Alaphelyzetbe állítás** után) kikapcsolja azt, miközben a rendszer még emlékszik az előző kivágásra; egy kikapcsolt ROI, amelynek mezői még mindig a korábban tárolt kivágást mutatják, kikapcsolt állapotban marad, ha újabb módosítást (formátum, binning, képkockasebesség) alkalmaz.
* **Egy igazított sorozatnál minden tag kivágási ablakát az igazítás határozza meg**, így minden kamera a jelenet ugyanazon részét veszi fel. Egy olyan sorozat, amelyet még az igazítás előtt vágtak ki, először a teljes érzékelőre igazodik (a kivágás feloldódik, az igazítás megoldódik, majd a kivágás kameránkénti ablakokkal visszaáll).
* A kivágás megváltoztatása után az automatikus expozíció az új kerethez igazodik. Kivágás alatt egy elem expozíciója továbbra is a kioldási periódusig futhat.
* Az ROI be-/kikapcsolt állapota és a megjegyzett kivágás a projekttel együtt kerül mentésre.

### Igazítás (ko-regisztráció)

**Mind az összevont, mind az elkülönített tömbök** esetében megjelenik (az 1.2.0-s verzióban csak az összevont tömböknél volt látható); a csoportfejlécen az „Igazítás” felirat olvasható.

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Maradék** mező: „RMS x.xx px” — 1 px alatt zöld, 3 px alatt sárga, egyéb esetben piros, illetve ha bármelyik kamera meghibásodott; az első megoldás előtt „nincs profil”.
* Összefoglaló sor: „✓ Minden kamera igazítva (N)” / „⚠ p/N kamera igazítva —  <serial (filter)="">meghibásodott” / „Kivágás aktív — Újrakalibrálás az igazításhoz (a teljes érzékelőt használja)” / „Várakozás az expozíció stabilizálódására…” / „Beállítások alkalmazása…” (a kamerák újraindulnak) / „A rendszer nem továbbítja a képkockákat — csatlakoztassa újra a rendszert”.
* Kameránkénti táblázat: kamera (a sorozatszám utolsó 4 számjegye + szűrő), újraprojekciós hiba px-ben az egyezések számával („ref” a mesterkamerára), valamint az átfedés normalizált keresztkorrelációs pontszáma a 0,35-ös átmeneti küszöbértékhez viszonyítva.
* **Igazítás újrakalibrálása** gomb (az első profil előtt „Igazítás kalibrálása” felirat látható) — újra futtatja a ko-regisztrációt friss képkockákon.
* **„Kamerák automatikus expozíciója az igazításhoz”** jelölőnégyzet (alapértelmezés szerint bejelölve) — ideiglenesen felvilágosítja a sötét vagy lapos kamerákat (először az expozíciót, majd az erősítést), hogy legyen textúrájuk az illesztéshez, majd visszaállítja az automatikus expozíciót.

A Chloros mindkét megjelenítési módban automatikusan igazítja az összes tömböt a csatlakozás után, miután a tagok automatikus expozíciója stabilizálódott (legfeljebb 5 percig próbálkozik); a kombinált előnézet is igazodik megnyitáskor. Egy különálló sorozatnak nincs összetett csempéje, ezért ez a szakasz lekérdezi az igazítási állapotot, és maga futtatja az Újrakalibrálás funkciót. Végezzen újrakalibrálást, ha a fókusz vagy a jelenet mélysége megváltozott. Az igazítás **tervezés szerint csak a munkamenetre vonatkozik** — soha nem kerül mentésre profilba, mivel az adott pillanat jelenet-távolságától függ. A felvételek továbbra is pixel-regisztrálva exportálhatók (lásd [Igazított exportok](capture.md#per-array-controls)).

### Intelligens vignettálás

* **Korrekció engedélyezése**jelölőnégyzet — a kameránkénti vignettálási becslést alkalmazza a radiometrikus láncra (élő**és** exportok esetén).
* **Kalibrálás az aktuális nézetből**— először irányítsa a kamerasorozatot egy egyenletes felületre (lapos panel, fal vagy ég); minden kamerát külön-külön síkítanak, és az állapotjelentés „n/N kamera · −x,x %” síkításnövekedést jelez. A**Törlés** gomb eltávolítja a becslést.
* Állapot: „Kalibrálva”, „Nincs kalibrálva” vagy „Részben kalibrálva — futtassa újra”, ha csak egyes kamerák becslései érvényesek a jelenlegi képkivágásnál (például miután az Újrakalibrálás funkció elmozdította egy kamera kivágási ablakát) — futtassa újra a Kalibrálás funkciót.
* Kameraenkénti finomhangolás a [Live Preview](#live-preview) menüben található, kameránkénti **Vignette** csúszkával.

### Élő előnézet

* **Hisztogram**jelölőnégyzet + szélesség-csúszka (a képkocka 0,10–0,90-e) — az élő feed-en megjelenő hisztogramcsík egyszerre minden tag mozaikján (beleértve a csatornákra osztott mozaikokat is), valamint a teljes rendszerre beállított, kameránkénti [Hisztogram-átfedés](#display-overlays-drawn-over-the-live-feed) beállítása az egész tömb számára. Akkor jelenik meg bekapcsoltként, ha minden tagnál már be van kapcsolva. Egy**különálló** tömbön ez az egyetlen sor a csoportban.
* *Csak kombinált esetben:* **Index**: jelölőnégyzet + fogaskerék — megnyitja a megosztott [Index-kalkulátort](#index-calculator-pane), amelyen**az összes** tagkamera adataiból rajzolt sávok láthatók. Az alatta található kifejezés-előnézeti sor az aktuális kifejezést mutatja („Nincs beállított kifejezés — nyissa meg a kalkulátort, hogy létrehozzon egyet”), amely másodpercenként frissül.
* *Csak kombinált módban:* **Renderelési felbontás**legördülő menü (ugyanazok az előre beállított értékek, mint kameránként, alapértelmezés szerint 720p): az élőképi stream magassága**és** a mentett kompozit exportméret. Megjegyzés a panelen: „Előnézet + mentett kompozit méret. A kameránkénti képek mindig teljes felbontásban kerülnek exportálásra.”

### Megjelenítési rétegek *(csak kombinált módban)** **Engedélyezés** jelölőnégyzet (alapértelmezés szerint ki van kapcsolva — a fő kamera képe közvetlenül jelenik meg; be = réteges kompozíció).
* **Előtér** /**Háttér**legördülő menük: az egyes tagkamerák (név szerint) vagy**Index**. Ha az Előtér beállítása Index, a LUT Min/Max határain kívül eső pixelek a Háttér réteget jelenítik meg.

### Osztott nézet *(csak kombinált esetben)*

**„Tagkamerák megjelenítése”**— egy**Tagkamerák felosztása / elrejtése** gomb, amely az egyes tagkamerák saját élő felvételét külön rácskockákként jeleníti meg az összetett kép mellett. A rácskockák a tömb meglévő képkocka-pufferét olvassák be (nincs szükség külön kameracsatlakozásra). Csak rácsnézetben; tömbenként, a projekttel együtt mentve.

### Funkciók

5 másodpercenként frissülő, csak olvasható panel:

* **Szintcímke**: „Egyidejű rögzítés” (zöld) · „Egyidejű rögzítés (FTD-eltolt kibocsátás)” (zöld) · „Eltolt rögzítés (100 ms eltolás)” (sárga) · „Túl nagy konfiguráció” (piros).
* **Képkocka-állapot**: „x,xx % hiányos” — zöld 1 % alatt, sárga 5 % alatt, piros 5 % vagy annál több esetén.
* **Kapcsolati sor**: „NIC {mbps} Mbps – folyamatos {MB/s} MB/s”.

Ez a rendszer aktuális sávszélesség-kerete. Az alapul szolgáló fps- és hálózati modellről — valamint arról, hogy mit kell módosítani, ha a szint sárgára vagy pirosra vált — lásd a [Többkamerás rendszerek](arrays.md) és a [CLI-referencia](../reference/cli-reference.md) című dokumentumokat.

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Index Calculator panel

A harmadik oldalsáv-oldal, amelyet a kameránkénti Index-beállítás és a kombinált rendszer Index-beállítása oszt meg (egyszerre csak az egyik – a fejléc szövege „Index Calculator — <camera name="">” vagy „Index Calculator — <array name="">”). Ide kerül a sávlista (a kamera természetes szűrőtermészetes sávjait, vagy a rendszer összes tagjának összes sávját), az aktuális kifejezést és a LUT-konfigurációt (be/ki, szint – alapértelmezés szerint 7-stop, min – alapértelmezés szerint 0,2, max – alapértelmezés szerint 1), valamint egy élő index-hisztogramot. Az **Alkalmazás** gomb megnyomásával a kifejezés véglegesítésre kerül; a LUT-változások azonnal megjelennek az előnézetben.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Kameránkénti vs. tömb által kezelt beállítások

Gyors áttekintés arról, hogy mi hol található, ha egy kamera a tömb tagja:

| Tömb által kezelt (csak olvasható a kamera ablakban) | Továbbra is kameránkénti a tömbön belül |
| --- | --- |
| Pixelformátum, felbontás, binning | Automatikus expozíció (expozíció, erősítés, célérték, simítás, ROI) |
| Kioldási mód/forrás, képkockasebesség | Zajszűrés, élesség, vignettálás |
| | Tájolás (tükör/forgatás), kijelző-felületek (rendszerbeállítások → Az Élő előnézet egyszerre állítja be a hisztogramot minden tagon), pontmérő |
| | Index (külön kijelzős rendszerek), fényérzékelő-hozzárendelés |

Egyéb átfogó viselkedés:

* **Kombinált vagy külön megjelenítés**: a választás a mátrix csatlakoztatásakor történik: kombinált = egy igazított összetett csempe (az elemek csak Split View-on keresztül továbbítanak); külön = minden elem saját, szinkronizált csempét jelenít meg. Egy kamera soha nem jeleníti meg egyszerre az önálló képet és a mátrixcsempét.
* **Automatikus újracsatlakozás**: egy mentett projekt megnyitásakor a rendszer visszaállítja a kamerákat és a tömböket, és az adatfolyamok folytatását megelőzően minden mentett beállítást újraalkalmaz a háttérrendszerre.
* **Felvételi szűrés**: a rejtett vagy szüneteltetett kamerák nem kerülnek bele a „Capture All” funkcióba; egy tömb csak akkor blokkolódik teljesen, ha MINDEN tagja rejtett vagy szüneteltetett. Lásd [Felvételi beállítások és módok](capture.md).

## A beállítások megőrzése

A kamera fül állapota **a projekttel együtt** kerül mentésre, nem a böngészőben:

* Minden reaktív változás pillanatképet készít a kamerákról és a kameracsoportokról a projekt `cameras.json` fájljába (500 ms-os visszhangszűréssel). Ez magában foglalja a kamerák nevét és színeit, az expozíció/erősítés/AE beállításokat, a pixelformátumot/felbontást/binninget, a kioldási gyakoriságot, az előnézeti beállításokat (renderelési felbontás, zajszűrés, élesség, vignettálás, színprofil, kiemelt területek, telítettség/kontraszt), a tájolást, az átfedéseket, a csatornafelosztásokat, az indexkonfigurációt, a prediktív AE beállításokat, az AE ROI-t, tömbnevek, megjelenítési mód, tömbfelvételi beállítások (beleértve az ROI-kivágást, az ROI bekapcsoltságát, valamint a kikapcsolt állapotban megjegyzett kivágást), a tömb „Wire Budget” értékét és a rácsblokkot (feed-zoom, nézetmód, rácszár, kézi csempesorrend, rejtett kamerák, zárt csempék, aktív kamera).
* A fényérzékelő-hozzárendelések a projekt `sensors.json` fájljában kerülnek mentésre.
* A projekt újbóli megnyitásakor a rendszer újra csatlakoztatja a hardvert, és az összes beállítást újra alkalmazza.
* **Nincs megnyitott projekt = csak munkamenet**: projekt hiányában a Chloros bezárásakor semmi sem marad meg.
* Csak munkamenethez tartozik, projekttől függetlenül: szüneteltetési állapot, pontmérő-minták, a kameránkénti Kalibrációs cél jelölőnégyzet (alapértelmezés szerint mindig ki van kapcsolva), valamint a rács-igazítási profil (a tervezés szerint munkamenetenként újraszámítva).
* Egy kivétel: a **Felvételi beállítások** exportálási kiválasztásai és a felvételi mód projektenként megmaradnak a helyi alkalmazás-tárolóban, nem pedig az `cameras.json`-ben — lásd [Felvételi beállítások és módok](capture.md).</array></camera></serial>
