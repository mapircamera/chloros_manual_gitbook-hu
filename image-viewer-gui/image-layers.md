# Képrétegek

A Képmegjelenítő jobb felső sarkában található **réteg legördülő menü** segítségével a megtekintett kép minden változata között válthat – a forrásfelvételtől kezdve az egyes feldolgozott termékeken át egészen a kiszámított indexképekig –, anélkül, hogy el kellene hagynia a megjelenítőt.

## Mik azok a képrétegek?

A „réteg” a Chloros

ban egy forrásképhez rendelt **termékfájl**. Az importálás során megkapja a forrásfájlokat; a feldolgozás során a futtatás minden egyes termékéhez egy réteg kerül hozzáadásra. Az exportált fájlok megtartják a forrásfájl nevét – a terméket a**mappa** azonosítja, a réteg neve pedig az Chloros

által annak a mappának adott címke.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## A réteg lista

### Mindig jelen van

| Réteg | Mi ez? |
| --- | --- |
| **JPG**(vagy**PNG

**/**TIFF

**) | A rögzítéssel beérkezett alapfájl. A Survey3

minden `.RAW` mellé egy `.JPG` fájlt importál; a LATTICE-rögzítések PNG

- vagy TIFF

-megjelenítési előnézetet hoznak. A ténylegesen importált tartalom szerint címkézve |
| **RAW (Eredeti)** | A forrás nyers képkocka, megjelenítésre debayerezve, korrekciók nélkül. Az importálás pillanatától elérhető — nincs szükség feldolgozásra |

Egy olyan LATTICE-felvétel, amelynek alapfájlja **éppen** a nyers képkocka, nem rendelkezik külön alapbejegyzéssel: ezt már az `RAW (Original)` fedi le.

### A Survey3

-as feldolgozási termékek

| Réteg | Írás helye | Létezik, ha |
| --- | --- | --- |
| **RAW (Cél)** | — | A képkockát úgy azonosították, hogy kalibrációs célt tartalmaz |
| **RAW (Fényvisszaverődés)** | `Reflectance_Calibrated_Images/` | A fényvisszaverődés-kalibrálás sikeresen lezajlott ezen a képkockán |
| **Vignettakorrekcióval**| `Vignette_Corrected_Images/` | A képkocka reflektancia-kalibrálása nem volt lehetséges**és** a *vignettakorrekció* be volt kapcsolva |
| **Érzékelőválasz**| `Sensor_Response_Images/` | A képkockát nem lehetett reflektancia-kalibrálni**és** a *vignett-korrekció* ki volt kapcsolva |
| **Fehér egyensúly** | `White_Balanced_Images/` | Fehér egyensúlyú termék került létrehozásra |

{% hint style="info" %}
**A vignettakorrekció és az érzékelő válasz egymást kizáró lehetőségek, soha nem fordulhatnak elő egyszerre.** Minden futtatás során pontosan egy kalibrálatlan tartalék termék létezik minden kameramodellhez, és a *Vignettakorrekció* kapcsoló választja ki, melyik legyen az. Lásd a [Projektbeállításokat](../project-settings/project-settings.md).
{% endhint %}

### LATTICE szintek

A LATTICE egyetlen feldolgozási lépésben rögzíti ezeket a szinteket. Hogy melyek léteznek, az a Projektbeállításokban található, termékenkénti exportkapcsolóktól, valamint attól függ, hogy mi vonatkozik az adott kamerára.

| Réteg | Írási hely | Alkalmazás |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB

és multispektrális |
| **RAW (Előnézet)** | `Preview_Images/` | Multispektrális (hamis színű kiterjesztés) |
| **Fehér egyensúly** | `Preview_Images/` | RGB

-mesterkamerák — az RGB

-előnézetet ezen a néven regisztrálják, így egybeesik az azonos nevű Survey3

-réteggel |
| **RAW (sugárzás)** | `Radiance_Images/` | Csak multispektrális |
| **RAW (reflektancia)** | `Reflectance_Calibrated_Images/` | Csak multispektrális, és csak akkor, ha egy megfelelő `.daq` lefelé irányuló felvétel vagy egy minőségbiztosításon átesett, a képkockán belüli célpont fedi le a képkockát |

Az RGB

-mesterkameráknak nincs sávonkénti radiometriájuk, ezért a sugárzási és visszaverődési értékeket **nem alkalmazható**ként hagyják ki — a napló ezt jelzi, ahelyett, hogy csendben hibaüzenetet adna.

{% hint style="info" %}
**A nyers és a debayered rétegek igazodnak a termékekhez (újdonság a 1.3.0-s verzióban).**A LATTICE nyers és debayered fájlok az érzékelő natív tájolásában kerülnek mentésre, és az XMP-ben rögzítik a hozzájuk tartozó tájolást; a többi termék már az adott tájolással mentésre kerül. A néző és a képrács a megjelenítéshez a bejegyzés alapján**RAW (Eredeti)**és**RAW (Debayered)** módba vált, így a képkocka már nem fordul el, amikor a termékréteg és a nyers réteg között váltunk. Ez csak a megjelenítésre vonatkozik — a fájlokat soha nem írják át. Az orientációs bejegyzés nélküli fájlok (amelyeket az 1.3.0 verzió előtt rögzítettek, vagy más szoftver írt) a tárolásuk szerinti állapotban jelennek meg. Lásd [Kimeneti képformátumok](../output-image-formats.md#orientation).
{% endhint %}

A korlátozó maszkok (`*_clampmask.tif` az `clamp_masks/` mappában) nem rétegek: az Chloros

importáláskor kihagyja őket. Lásd: [Kimeneti képformátumok](../output-image-formats.md#clamp-masks).

### Index-, LUT- és sandbox-rétegek

| Rétegminta | Példa | Honnan származik |
| --- | --- | --- |
| **RAW (`<INDEX>` Index)** | `RAW (NDVI Index)` | A Projektbeállításokban konfigurált indexenként egy, a feldolgozás során kiszámítva |
| **`<INDEX>` LUT** | `NDVI LUT` | Az index színre leképezett változata |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Egy-egy az [Index/LUT Sandbox](index-lut-sandbox.md) exportálási futtatásonként |

Ha ugyanazt az indexnevet többször is beállítják különböző beállításokkal, a második és az azt követő indexek nevébe számot illesztenek (`RAW (NDVI2 Index)`), így a rétegek megkülönböztethetők maradnak.

***

## A rétegválasztó használata

1. Nyissa meg a képet teljes képernyős módban a rácsban található miniatűrre kattintva
2. Kattintson a nézőprogram jobb felső sarkában található **réteg legördülő menüre**

3. Válasszon ki egy réteget — a kép azonnal frissül

A legördülő menüben először a **JPG, RAW (Eredeti), RAW (Cél), RAW (Fényvisszaverődés)** jelenik meg, ebben a sorrendben, majd az összes többi a termékek regisztrációjának sorrendjében.

### Rétegbeállítások navigáláskor

A **←**/**→** gombok megnyomásával a következő képre léphet, és a rendszer megpróbálja ugyanazon a rétegen tartani:

1. **Először a pontos egyezés** — ha a következő képen van azonos nevű réteg, azt kapja meg. Ez biztosítja, hogy az egész sorozat végiglapozása közben az `RAW (NDVI Index)` rétegen maradjon
2. **Ezután típus szerinti egyezés** — egy indexréteg bármely indexréteget keres, egy LUT bármely LUT-ot, a reflektancia a reflektanciát, a cél a célt, az eredeti az eredetit, az alap az alapot
3. **Ezt követően, kizárólag exportrétegek esetén** — a név megmarad akkor is, ha a réteg lista még nem frissült, mert a fájl már létezik a lemezen. Ez teszi lehetővé a termékek áttekintését, miközben a futtatás még írja őket
4. **Egyéb esetben** — az első elérhető réteg, ami általában az alapkép

A projektben található `.daq` és `.csv` sidecar fájlokat a nyílgombokkal történő navigálás kihagyja, így a képek közötti lépés során soha nem kerül sor fényérzékelő-felvételre.

A nagyítás és a pásztázás a képek között is átvihető, ami egyszerűvé teszi ugyanazon mezőpozíció előtti és utáni összehasonlítását.

***

## A képpontértékek megértése rétegenként

A [Kurzorértékek panel](opening-an-image-full-screen.md#cursor-values) a kurzor alatt lévő csatornánkénti valós értéket jeleníti meg abban a mértékegységben, amelyben az adott réteg tárolva van. Oszlopai a rétegtől függően változnak:

| Réteg | Megjelenített mértékegység | Megjegyzések |
| --- | --- | --- |
| Alap (JPG / PNG

/ TIFF

előnézet) | DN, 0–255 | Megjelenítési értékek, RGB

-re gamma-korrigálva. Csak vizuális ellenőrzésre |
| RAW (Eredeti) | DN | Nyers érzékelői digitális értékek. A hisztogram tengelye jelzi a mélységet: 255 (8 bites), 4095 (12 bites) vagy 65535 (16 bites) |
| RAW (Debayered) | DN | Lineáris, nincs képernyőn történő kiterjesztés |
| RAW (Előnézet) / Fehér egyensúly | DN | Kijelzőn megjelenő érték — kiterjesztve vagy gamma-korrigálva. Nem mérésre szolgál |
| RAW (Sugárzás) | **W/m²/sr/nm** | Float32 fizikai sugárzás. Nincs DN oszlop |
| RAW (Fényvisszaverődés) | DN **és %** | A fájl saját skálája alapján számított százalék — lásd alább |
| Index / LUT / sandbox exportok | Indexérték, vagy RGB

-komponensek | Az egycsatornás indexfájl az indexértéket jelenti; a színleképezett LUT-fájl a Red

/Green

/Blue

komponenseket jelenti |

### Reflektancia: a skála fájlonkénti

{% hint style="warning" %}
**A „65 535-tel osztás” csak a Survey3

esetében helyes.** A LATTICE reflektancia más skálán van tárolva, és a két osztószám keverése a leggyakoribb módja annak, hogy a reflektanciaértékek pontosan a felük legyenek a valós értéknek.
{% endhint %}

| Forrás | A 1,0-s fényvisszaverődési értéknek megfelelő DN | Azonosító |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | Az `Chloros:PixelScale=32768` XMP-címke, amelyet minden LATTICE-reflektancia-exportba beágyaznak. A kétszeres tartalék azt jelenti, hogy az 1,0 feletti ρ értékek ábrázolhatók, nem pedig levágódnak |
| **Survey3**| **65535** | Nincs Chloros

XMP skála címke — a Survey3

kalibráció a ρ × dtype-max értéket írja, és 1,0-nál levágja |

GIS és szkripteléshez: olvassa be az `Chloros:PixelScale` értéket a fájlból, és ossza el vele. Ha a címke hiányzik, a fájl Survey3

-skálájú (65535). A nézőprogram, az index/LUT sandbox és az index-export egyaránt ugyanígy határozza meg a skálát, így a kurzor helyén látható szám az, amelyet az index-számítás is használt.

Ezen skála feletti formátum-specifikus tárolás:

* **A TIFF** (32-bites, százalékos) a DN / 65535 értéket lebegőpontos számként tárolja
* **A PNG**(8-bites) és **a JPG** (8-bites) a DN × 255 / 65535 értéket tárolja
* Egy **8-bites felvételből származó 8-bites DN-érték**, illetve az 1.3.0-ás verzió előtt írt reflektanciafájl 0–255-re van korlátozva, és nem tartalmaz skála-címkét. A panel ezeknél a fájloknál csak a DN-értéket jeleníti meg, százalékos oszlop nélkül

### Indexérték-tartományok

| Indexcsalád | Jellemző tartomány | Érték |
| --- | --- | --- |
| Normalizált különbség (NDVI

, GNDVI

, NDRE

, ENDVI…) | −1 és +1 között | Egészséges növényzet esetén általában 0,4–0,9; csupasz talaj esetén 0 közelében; víz esetén negatív |
| Talajkorrigált (SAVI

, OSAVI

, MSAVI2…) | nagyjából −1 és +1,5 között | Hasonló értékek, mint az NDVI

-nél, de a talajháttér kiszűrésre került |
| Arány (GRVI

, GCI

, MSR, CIRE…) | felfelé korlátlan | Az arányok korlátlanul növekednek, ahogy a nevező sáv értéke nullához közeledik |
| EVI

/ LAI

| 0 és ~1, illetve 0 és ~3,5 között | A felhők és más telített képpontok mindkettőt a tartományon kívülre tolják — ezeket először maszkolja ki |

Az egyes előre beállított értékek mögött álló pontos képleteket lásd a [Multispektrális indexképletek](../project-settings/multispectral-index-formulas.md) című részben.

***

## Gyakori munkafolyamatok

### Előtte–utána összehasonlítás

1. Válassza ki a **RAW (Eredeti)** lehetőséget, és jegyezze meg a vignettálást és a kalibrálatlan értékeket
2. Váltson át **RAW (Reflectance)** módra
3. Hasonlítsa össze — a vignettálás eltűnt, az értékek kalibrálva vannak. A nagyítás és a pásztázás rögzítve marad, így ugyanazt a területet látja

### Egy index áttekintése az egész sorozaton

1. Nyissa meg az első feldolgozott képet, és válassza ki az indexréteget
2. Nyomja meg többször a **→** gombot — az indexréteg követi Önt képről képre
3. Figyelje az oldalsávon található hisztogramot közben: az olyan képkockát, amelynek eloszlása ugrásszerűen változik, érdemes alaposabban megvizsgálni

### A kalibrációs célpontok ellenőrzése

1. Válassza ki a **RAW (Cél)** elemet egy célképkockán
2. Győződjön meg arról, hogy a cél jól látható és felismerhető
3. Lépjen a következő célképkockára — a célréteg követi

### Ellenőrizze a reflexiós értékek pontosságát

1. Válassza ki a **RAW (Reflexió)** elemet
2. Olvassa le a **%** oszlopot a Cursor Values panelen – az adott fájlhoz már megfelelően van méretezve
3. Ellenőrizze a képben található ismert anyagok alapján: az egészséges növényzet magas NIR

-értékkel és alacsony vörösértékkel rendelkezik; a kalibrációs célértéknek közel kell lennie a közzétett reflexiós értékéhez

***

## Hibaelhárítás

### A várt réteg nem szerepel a legördülő menüben

**Lehetséges okok**

* A képet soha nem dolgozták fel — csak az alapréteg és az `RAW (Original)` létezik
* A termék exportálási kapcsolója nincs bejelölve a Projektbeállításokban
* A termék nem alkalmazható az adott kamerára (sugárzás és reflexió egy RGB

-mesterkép esetén; bármely index egycsatornás M3M monokameránál)
* A reflexiókalibrálásnak nem volt mire támaszkodnia — nincs `.daq` lefelé irányuló lefedettség és nincs minőségbiztosításon átment képkockán belüli célpont — így a képkocka visszatért a Vignette Corrected vagy a Sensor Response beállításra

**Mit kell tenni?**

1. Ellenőrizze a futtatás naplóját: a Chloros

jelzi, ha egy kért exporttermék elkészítése lehetetlen volt, és megadja az okát is
2. Ellenőrizze a [Projektbeállítások](../project-settings/project-settings.md) menüpontban a termékenkénti exportkapcsolókat
3. Győződjön meg arról, hogy a termékmappa létezik a projekt kimeneti fa szerkezetében
4. Végezze el újra a feldolgozást a termék engedélyezésével

### A réteg lista elavultnak tűnik

A Chloros

a futás közben újra beolvassa a projekt termékmappáit, és a lemezen ténylegesen található adatok alapján kijavítja a hiányzó rétegregisztrációkat, így egy exportálását befejező réteg általában önmagától megjelenik egy lekérdezés során. Ha elnavigál a képről, majd visszatér rá, az újbóli felbontást kényszerít.

### A reflexiós értékek a várt érték felének tűnnek

Szinte biztos, hogy egy LATTICE fájlt oszt 65535-tel. Használja az `Chloros:PixelScale` (32768) parancsot, vagy olvassa el a **%** oszlopot, amelyben ez már alkalmazásra került.

### Az indexréteg létezik, de a kép üres

Az indexhez olyan sávokra van szükség, amelyekkel a réteg nem rendelkezik — például egy olyan index, amely egy harmadik csatornát olvas ki, miközben egy egy- vagy kétcsatornás fájlra van alkalmazva. Váltson többsávos rétegre (reflektancia vagy debayered), vagy válasszon olyan indexet, amely illeszkedik a kamera szűrőjéhez.

***

## Következő lépések

* [**Kép megnyitása teljes képernyős módban**](opening-an-image-full-screen.md) — kurzorérték-kijelzés, hisztogram és GSD-vezérlés
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — interaktív index-megjelenítés és export
* [**Multispektrális indexképletek**](../project-settings/multispectral-index-formulas.md) — az index-referencia
* [**A feldolgozás befejezése**](../processing-images-gui/finishing-the-processing.md) — a kimeneti mappafák, amelyekre ezek a rétegek mutatnak
