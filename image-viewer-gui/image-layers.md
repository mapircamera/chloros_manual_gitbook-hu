# Képrétegek

A Képmegjelenítő jobb felső sarkában található **réteg legördülő menü** segítségével a megtekintett kép minden változata között válthat – a forrásfelvételtől kezdve az egyes feldolgozott termékeken át egészen a kiszámított indexképekig –, anélkül, hogy el kellene hagynia a megjelenítőt.

## Mik azok a kép rétegek?

Az Chloros programban egy „réteg” egy forrásképhez rendelt **termékfájl**. Az importálás során a forrásfájlok kerülnek beolvasásra; a feldolgozás során a futtatás során létrehozott minden termékhez egy-egy réteg kerül hozzáadásra. Az exportált fájlok megtartják a forrásfájl nevét – a terméket a**mappa** azonosítja, a réteg neve pedig az Chloros által annak a mappának adott címke.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## A réteg lista

### Mindig jelen van

| Réteg | Mi ez? |
| --- | --- |
| **JPG**(vagy**PNG**/**TIFF**) | A rögzítéssel együtt beérkezett alapfájl. Az Survey3 minden `.RAW` mellé importál egy `.JPG` fájlt; A LATTICE-felvételek egy PNG vagy TIFF megjelenítési előnézetet hoznak. A ténylegesen importált tartalom szerint címkézve |
| **RAW (Eredeti)** | A forrás nyers képkocka, a megjelenítéshez debayerezve, korrekciók nélkül. Az importálás pillanatától elérhető — nincs szükség feldolgozásra |

Az olyan LATTICE-felvétel, amelynek alapfájlja **éppen** a nyers képkocka, nem rendelkezik külön alapbejegyzéssel: ezt már az `RAW (Original)` fedi le.

### Survey3 feldolgozási eredmények

| Réteg | Írás helye | Létezik, amikor |
| --- | --- | --- |
| **RAW (Cél)** | — | A képkockát úgy azonosították, hogy kalibrációs célt tartalmaz |
| **RAW (Fényvisszaverődés)** | `Reflectance_Calibrated_Images/` | A fényvisszaverődés-kalibrálás sikeresen lezajlott ezen a képkockán |
| **Vignettkorrekcióval**| `Vignette_Corrected_Images/` | A képkockát nem lehetett reflektancia-kalibrálni**és** a *vignettkorrekció* be volt kapcsolva |
| **Érzékelőválasz**| `Sensor_Response_Images/` | A képkockát nem lehetett reflektancia-kalibrálni**és** a *vignett-korrekció* ki volt kapcsolva |
| **Fehér egyensúly** | `White_Balanced_Images/` | Fehér egyensúlyú termék került mentésre |

{% hint style="info" %}
**A vignettakorrekció és az érzékelő válasz egymást kizáró lehetőségek, soha nem fordulhatnak elő egyszerre.** Minden futtatás során pontosan egy kalibrálatlan tartalék termék létezik minden kameramodellhez, és a *Vignettakorrekció* kapcsoló választja ki, melyik legyen az. Lásd a [Projektbeállításokat](../project-settings/project-settings.md).
{% endhint %}

### LATTICE szintek

A LATTICE egyetlen feldolgozási lépésben rögzíti a fan out-ot ezekbe a szintekbe. Hogy melyek léteznek, az a Projektbeállításokban található, termékenkénti exportkapcsolóktól, valamint a kamerára vonatkozó beállításoktól függ.

| Réteg | Írás helye | Vonatkozik |
| --- | --- | --- |
| **RAW (Debayered)** | `Debayered_Images/` | RGB és multispektrális |
| **RAW (Előnézet)** | `Preview_Images/` | Multispektrális (hamis színű kiterjesztés) |
| **Fehér egyensúly** | `Preview_Images/` | RGB főkamerák — az RGB előnézetet ezen a néven regisztrálták, így egybeesik az azonos nevű Survey3 réteggel |
| **RAW (sugárzás)** | `Radiance_Images/` | Kizárólag multispektrális |
| **RAW (reflektancia)** | `Reflectance_Calibrated_Images/` | Csak multispektrális, és csak akkor, ha egy megfelelő `.daq` lefelé irányuló felvétel vagy egy minőségbiztosításon átesett, a képkockán belüli célpont fedi le a képkockát |

Az RGB mesterkamerák nem rendelkeznek sávonkénti radiometriával, ezért a sugárzás és a visszaverődés értékeit **nem alkalmazható**ként hagyják ki — a napló ezt jelzi, ahelyett, hogy csendben hibaüzenet nélkül továbbhaladna.

### Index-, LUT- és sandbox-rétegek

| Rétegminta | Példa | Honnan származik |
| --- | --- | --- |
| **RAW (`<INDEX>` Index)** | `RAW (NDVI Index)` | A Projektbeállításokban konfigurált indexenként egy, a feldolgozás során kiszámítva |
| **`<INDEX>` LUT** | `NDVI LUT` | Az index színképezett változata |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Egy-egy az [Index/LUT Sandbox](index-lut-sandbox.md) exportálási futtatásonként |

Ha ugyanazt az indexnevet többször is konfigurálják különböző beállításokkal, a második és az azt követő indexek nevébe számot illesztenek (`RAW (NDVI2 Index)`), így a rétegek megkülönböztethetők maradnak.

***

## A rétegválasztó használata

1. Nyissa meg a képet teljes képernyős módban a rácsban található miniatűrre kattintva
2. Kattintson a nézőprogram jobb felső sarkában található **réteg legördülő menüre**

3. Válasszon ki egy réteget — a kép azonnal frissül

A legördülő menüben először a **JPG, RAW (eredeti), RAW (cél), RAW (reflektancia)** jelenik meg, ebben a sorrendben, majd az összes többi a termékek regisztrációjának sorrendjében.

### Rétegbeállítások navigáláskor

A **←**/**→** gombok megnyomásával a következő képre léphet, és a rendszer megpróbálja ugyanazon a rétegen tartani:

1. **Először a pontos egyezés** — ha a következő képen van azonos nevű réteg, azt kapja meg. Ez biztosítja, hogy az egész sorozat végiglapozása során az `RAW (NDVI Index)` rétegen maradjon
2. **Ezután típus szerinti egyezés** — egy indexréteg bármely indexréteget keres, egy LUT bármely LUT-ot, a reflektancia a reflektanciát, a cél a célt, az eredeti az eredetit, az alap az alapot
3. **Ezt követően, kizárólag exportrétegek esetén** — a név megmarad akkor is, ha a réteg lista még nem frissült, mert a fájl már létezik a lemezen. Ez teszi lehetővé, hogy áttekintsd a termékeket, miközben a futtatás még írja őket
4. **Egyéb esetben** — az első elérhető réteg, ami általában az alapkép

A projektben található `.daq` és `.csv` sidecar fájlokat a nyílgombokkal történő navigálás kihagyja, így a képek közötti lépés során soha nem kerül sor fényérzékelős felvételre.

A nagyítás és a pásztázás a képek között is átvitelre kerül, ami egyszerűvé teszi ugyanazon mezőpozíció előtti és utáni összehasonlítását.

***

## A képpontértékek megértése rétegenként

A [Kijelölő értékek panel](opening-an-image-full-screen.md#cursor-values) a kurzor alatt lévő csatornánkénti valós értéket jeleníti meg abban a mértékegységben, amelyben az adott réteg tárolva van. Az oszlopok a rétegtől függően változnak:

| Réteg | Megjelenített mértékegység | Megjegyzések |
| --- | --- | --- |
| Alap (JPG / PNG / TIFF előnézet) | DN, 0–255 | Kijelzett értékek, RGB-en gamma-korrigálva. Csak vizuális ellenőrzésre |
| RAW (Eredeti) | DN | Nyers szenzoros digitális értékek. A hisztogram tengelye jelzi a mélységet: 255 (8 bites), 4095 (12 bites) vagy 65535 (16 bites) |
| RAW (Debayered) | DN | Lineáris, nincs kijelzőbeli kiterjesztés |
| RAW (Előnézet) / Fehér egyensúly | DN | Kijelzői érték — kiterjesztett vagy gamma-korrigált. Nem mérésre szolgál |
| RAW (Sugárzás) | **W/m²/sr/nm** | Float32 fizikai sugárzás. Nincs DN oszlop |
| RAW (reflektancia) | DN **és %** | A százalékos érték a fájl saját skálája alapján számítva — lásd alább |
| Index / LUT / sandbox exportok | Indexérték, vagy RGB komponensek | Egycsatornás indexfájl az indexértéket jelenti; egy színképezett LUT-fájl az Red/Green/Blue komponenseket jelenti |

### Fényvisszaverődés: a skála fájlonkénti

{% hint style="warning" %}
**A „65 535-tel osztás” csak az Survey3 esetében helyes.** A LATTICE fényvisszaverődési értékek más skálán vannak tárolva, és a két osztószám összekeverése a leggyakoribb módja annak, hogy a fényvisszaverődési értékek pontosan a felükre csökkenjenek.
{% endhint %}

| Forrás | A 1,0-s fényvisszaverődési értéknek megfelelő DN | Azonosító |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | Az `Chloros:PixelScale=32768` XMP-címke, amely minden LATTICE-reflektancia-exportba be van illesztve. A 2×-es tartalék azt jelenti, hogy az 1,0 feletti ρ értékek nem vágódnak le, hanem ábrázolhatók |
| **Survey3**|**65535** | Nincs Chloros XMP skála címke — az Survey3 kalibráció a ρ × dtype-max értéket írja be, és 1,0-nál levágja |

GIS és szkriptelés esetén: olvassa be az `Chloros:PixelScale` értéket a fájlból, és ossza el vele. Ha a címke hiányzik, a fájl Survey3-skálájú (65535). A néző, az index/LUT sandbox és az indexexport egyaránt ugyanígy határozza meg a skálát, így a kurzor helyén látható szám az az érték, amelyet az indexszámítás is használt.

Ezen skála feletti formátum-specifikus tárolás:

* **TIFF (32-bites, százalék)** a DN / 65535 értéket lebegőpontos számként tárolja
* **PNG (8 bites)**és**JPG (8 bites)** a DN × 255 / 65535 értéket tárolja
* Egy **8 bites forrásból származó rögzítés**8 bites TIFF**formátumú exportja** nem átméretezésre kerül, hanem 0–255 közé van korlátozva, és szándékosan nem tartalmaz skála-címkét. A panel ezeknél a fájloknál csak a DN-értéket jeleníti meg, százalékos oszlop nélkül

### Indexérték-tartományok

| Indexcsalád | Tipikus tartomány | Érték |
| --- | --- | --- |
| Normalizált különbség (NDVI, GNDVI, NDRE, ENDVI…) | −1 és +1 között | Az egészséges növényzet értéke általában 0,4–0,9; a csupasz talajé 0 közelében; a vízé negatív |
| Talajkorrigált (SAVI, OSAVI, MSAVI2…) | nagyjából −1 és +1,5 között | Hasonló érték, mint az NDVI esetében, de a talajháttér kiszűrésre került |
| Arány (GRVI, GCI, MSR, CIRE…) | felfelé korlátlan | Az arányok korlátlanul növekednek, ahogy a nevező sávja nullához közeledik |
| EVI / LAI | 0 és ~1, 0 és ~3,5 között | A felhők és más telített képpontok mindkettőt a tartományon kívülre tolják — először maszkolja ki őket |

Az egyes előre beállított értékek mögött álló pontos képleteket lásd a [Multispektrális indexképletek](../project-settings/multispectral-index-formulas.md) alatt.

***

## Gyakori munkafolyamatok

### Előtte–utána összehasonlítás

1. Válassza ki a **RAW (Eredeti)** réteget, és jegyezze meg a vignettálást és a kalibrálatlan értékeket
2. Váltson át a **RAW (Reflektancia)** rétegre
3. Hasonlítsa össze — a vignettálás eltűnt, az értékek kalibrálva vannak. A nagyítás és a pásztázás rögzítve marad, így ugyanazt a területet látja

### Egy index áttekintése a teljes sorozaton

1. Nyissa meg az első feldolgozott képet, és válassza ki az indexréteget
2. Nyomja meg többször a **→** gombot — az indexréteg követi Önt képről képre
3. Figyelje az oldalsávon látható hisztogramot közben: az olyan képkockát, amelynek eloszlása ugrásszerűen változik, érdemes alaposabban megvizsgálni

### A kalibrációs célpontok ellenőrzése

1. Válassza ki a **RAW (Cél)** elemet egy célképkockán
2. Győződjön meg arról, hogy a cél jól látható és felismerhető
3. Lépjen a következő célképkockára — a célréteg követi

### Ellenőrizze a reflexiós értékek pontosságát

1. Válassza ki a **RAW (Reflexió)** elemet
2. Olvassa le a **%** oszlopot a Cursor Values panelen – az adott fájlhoz már megfelelően van méretezve
3. Ellenőrizze a képben szereplő ismert anyagok alapján: az egészséges növényzet NIR értéke magas, a vörös értéke alacsony; a kalibrációs célpontnak a közzétett reflexiós értékéhez közel kell lennie

***

## Hibaelhárítás

### A várt réteg nem szerepel a legördülő menüben

**Lehetséges okok**

* A képet soha nem dolgozták fel — csak az alapréteg és az `RAW (Original)` létezik
* A termék exportálási kapcsolója nincs bejelölve a Projektbeállításokban
* A termék nem vonatkozik arra a kamerára (sugárzás és visszaverődés egy RGB mesterkamerán; bármely index egycsatornás M3M monokamerán)
* A reflexió kalibrálásához nem volt megfelelő alap — nincs `.daq` lefelé irányuló lefedettség, és nincs minőségbiztosításon átment célpont a képkockában — ezért a képkocka visszatért a „Vignette Corrected” vagy a „Sensor Response” értékre

**Mit kell tenni**

1. Ellenőrizze a futtatás naplóját: az Chloros jelzi, ha egy kért exporttermék elkészítése lehetetlen volt, és megadja az okát is
2. Ellenőrizze a [Project Settings](../project-settings/project-settings.md) menüpontban a termékenkénti exportkapcsolókat
3. Győződjön meg arról, hogy a termék mappa létezik a projekt kimeneti fákban
4. Végezze el újra a feldolgozást a termék engedélyezésével

### A réteg lista elavultnak tűnik

Az Chloros futás közben újra beolvassa a projekt termékmappáit, és a lemezen ténylegesen található adatok alapján kijavítja a hiányzó rétegregisztrációkat, így egy normálisan exportált réteg önmagában jelenik meg egy lekérdezés során. Ha elnavigál a képről, majd visszatér rá, az újbóli felbontást kényszerít.

### A reflexiós értékek a várt érték felének tűnnek

Szinte biztos, hogy a LATTICE fájlt 65535-tel osztja. Használja az `Chloros:PixelScale` (32768) parancsot, vagy olvassa el a **%** oszlopot, amelyben ez már alkalmazásra került.

### Az indexréteg létezik, de a kép üres

Az indexhez olyan sávokra van szükség, amelyekkel a réteg nem rendelkezik — például egy olyan index, amely egy harmadik csatornát olvas be, miközben egy egy- vagy kétcsatornás fájlra van alkalmazva. Váltson többsávos rétegre (reflektancia vagy debayered), vagy válasszon olyan indexet, amely megfelel a kamera szűrőjének.

***

## Következő lépések

* [**Kép megnyitása teljes képernyős módban**](opening-an-image-full-screen.md) — kurzorérték-leolvasás, hisztogram és GSD-vezérlés
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — interaktív indexmegjelenítés és export
* [**Multispektrális indexképletek**](../project-settings/multispectral-index-formulas.md) — az index-referencia
* [**A feldolgozás befejezése**](../processing-images-gui/finishing-the-processing.md) — a kimeneti mappafák, amelyekre ezek a rétegek hivatkoznak
