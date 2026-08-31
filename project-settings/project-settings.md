# Projektbeállítások

Az Chloros szoftver „Projektbeállítások”<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

oldalsávján a képfeldolgozás, a kalibrációs célpontok felismerése, a multispektrális indexek kiszámítása, valamint a projekt exportálási beállításainak minden szempontját konfigurálhatja. Ezek a beállítások a projekttel együtt kerülnek mentésre, és sablonként is elmenthetők, hogy több projektben is újra felhasználhatók legyenek.

## A projektbeállítások eléréséhez

A projektbeállítások eléréséhez:

1. Nyisson meg egy projektet az Chloros programban
2. Kattintson a bal oldali oldalsávon a **Projektbeállítások**<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

fülre
3. A beállítási panelen kategóriák szerint rendezve jelennek meg az összes elérhető konfigurációs lehetőség

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Azok a beállítások, amelyek más beállításoktól függenek, szürkén jelennek meg.** Ha egy szülőkapcsoló miatt egy beállítás nem állítható be (például a *Fényvisszaverődés-kalibrálás / fehéregyensúly* jelölőnégyzet törlése miatt a *Fényvisszaverődés exportálása* nem lehetséges), a függő vezérlőelem letiltásra kerül, és a segítő szövegben megjelenik annak a kapcsolónak a neve, amelyet módosítani kell.
{% endhint %}

***

## Kijelző

### Képminiatűr felbontás

* **Típus**: Legördülő menü
* **Lehetőségek**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Alapértelmezett**: Alapértelmezett (512 px)
* **Leírás**: A kép-rácsos miniatűrök megjelenítésének felbontása (a leghosszabb él hossza, pixelben). A nagyobb értékek nagyításkor élesebb képet adnak, de lassabban töltődnek be és több memóriát igényelnek. A teljes felbontás az eredeti kép méretét adja vissza.
* **Megjegyzés**: Csak megjelenítésre szolgál — ez soha nem befolyásolja a feldolgozást vagy az exportált fájlokat.***

## Célpont-felismerés

Ezek a beállítások szabályozzák, hogy az Chloros hogyan ismeri fel és dolgozza fel a képeken található kalibrációs célpontokat. Mindkettő csak akkor aktív, ha a **Reflektancia-kalibrálás / fehéregyensúly** be van kapcsolva (ellenkező esetben szürkén jelennek meg, mert a célpont-felismerést teljesen kihagyják).

### Minimális kalibrációs mintafelület (px)

* **Típus**: Szám
* **Tartomány**: 0–10 000 pixel
* **Alapértelmezett**: 25 pixel
* **Leírás**: Beállítja azt a minimális területet (pixelben), amely ahhoz szükséges, hogy egy felismert régiót érvényes kalibrációs célminta-területnek tekintsünk. A kisebb értékek kisebb célokat is felismernek, de növelhetik a téves riasztások számát. A nagyobb értékek nagyobb, egyértelműbb célterületeket igényelnek a felismeréshez.
* **Mikor érdemes módosítani**:
  * Növelje az értéket, ha kis képi artefaktumoknál hamis észlelések jelentkeznek
  * Csökkentse az értéket, ha a kalibrációs célpontok kicsinek tűnnek a képein, és nem kerülnek észlelésre

### Minimális célpont-csoportosítás (0–100)

* **Típus**: Szám
* **Tartomány**: 0–100
* **Alapértelmezett**: 60
* **Leírás**: Szabályozza a hasonló színű régiók csoportosításának küszöbértékét a kalibrációs célpontok felismerése során. A magasabb értékeknél több hasonló színű területet kell csoportosítani, ami konzervatívabb célpontfelismerést eredményez. Az alacsonyabb értékeknél nagyobb színváltozás megengedett egy célcsoporton belül.
* **Mikor kell módosítani**:
  * Növelje az értéket, ha a kalibrációs célpontok több felismerésre oszlanak szét
  * Csökkentse az értéket, ha a színváltozással rendelkező kalibrációs célpontokat nem ismeri fel teljes mértékben a rendszer

***

## Feldolgozás

Ezek a beállítások szabályozzák, hogy az Chloros hogyan dolgozza fel és kalibrálja a képeit.

### Vignettkorrekció

* **Típus**: Jelölőnégyzet
* **Alapértelmezett**: Engedélyezve (bejelölve)
* **Leírás**: Vignettakorrekciót alkalmaz a képek szélein jelentkező lencse-sötétedés kompenzálására. A vignettálás egy gyakori optikai jelenség, amelynek során a kép sarkai és szélei a lencse tulajdonságai miatt sötétebbnek tűnnek, mint a közepe.
* **Mellékhatás**: Ez a kapcsoló azt is meghatározza, hogy a futtatás melyik *kalibrálatlan tartalékterméket* írja ki (lásd alább).

### Reflektancia-kalibrálás / fehéregyensúly

* **Típus**: Jelölőnégyzet
* **Alapértelmezés**: Engedélyezve (bejelölve)
* **Leírás**: Engedélyezi a reflexiós kalibrálást — a képkockán belül észlelt kalibrációs célpontok és/vagy a DAQ fényérzékelőjének lefelé irányuló fényadatok alapján, a kamerától és a rendelkezésre álló adatoktól függően. Ez normalizálja a reflexiós értékeket az adatkészletben, és biztosítja a következetes méréseket a fényviszonyoktól függetlenül.
* **Ha ki van kapcsolva**: A célpont-felismerés teljesen kihagyásra kerül, és**egyik kamera sem képes reflektancia-értéket előállítani** — mind az Survey3 célpont-vezérelt, mind a LATTICE DAQ-vezérelt rendszerek esetében. A függő beállítások (*Reflektancia exportálása*, *Minimális újrakalibrálási intervallum* és a célpont-felismerési küszöbértékek) szürkén jelennek meg.

### Kalibrálatlan tartalék termékek: Érzékelőválasz exportálása / Vignettás korrekcióval ellátott export

* **Típus**: Két jelölőnégyzet
* **Alapértelmezett beállítások**: Mindkettő engedélyezve (bejelölve)
* **Leírás**: Ha egy képkocka reflektancia-kalibrálása nem lehetséges (nem találtak kalibrációs célt, vagy a reflektancia-kalibrálás ki van kapcsolva), akkor azt *nem kalibrált tartalék termékként* írja ki. **Minden futtatásonként és minden kameramodellnél pontosan egy a két tartaléktermék közül létezik**, amelyet a *Vignettakorrekció* kapcsoló választ ki:
  * Vignettakorrekció **be**→ `Vignette_Corrected_Images/` (az**Export vignettakorrekcióval** szabályozva)
  * Vignettakorrekció **ki**→ `Sensor_Response_Images/` (az**Export sensor response** beállítás határozza meg)
* Az éppen nem használt tartalék termék szürkén jelenik meg. Az aktív termék jelölésének eltávolításával megakadályozható a fájl mentése.

### LATTICE exporttermékek

A LATTICE-felvételeket tartalmazó projektek esetében minden importált LATTICE-képkocka egyetlen feldolgozási lépésben szétoszlik az összes engedélyezett **és alkalmazható**termékre. A szétosztást négy jelölőnégyzet vezérli (alapértelmezés szerint mind**be** van kapcsolva):

| Beállítás | Kimeneti mappa | Mit exportál |
| --- | --- | --- |
| **Debayered export** | `Debayered_Images/` | A lineáris, debayered kép. Az RGB és a multispektrális kamerákra vonatkozik. |
| **Előnézet exportálása** | `Preview_Images/` | A képernyőn megjelenő előnézet. RGB = fehéregyensúly (DAQ-fényforrás, ha elérhető, egyébként szürke világ) + gamma; multispektrális = hamis színű kiterjesztés. |
| **Sugárzási intenzitás export** | `Radiance_Images/` | Float32 spektrális sugárzási intenzitás W/m²/sr/nm-ben. Csak multispektrális (M3C/M3M) esetén — nem alkalmazható az RGB mestermintákra. Mindig 32 bites TIFF formátumban kerül írásra, függetlenül a *Kalibrált képformátum* beállítástól. |
| **Exportált reflexió**| `Reflectance_Calibrated_Images/` | Uint16 reflexió, úgy skálázva, hogy**32768 = 1,0-es reflexió** (XMP-ként `Chloros:PixelScale` jelöléssel). Csak multispektrális esetben, akkor íródik, ha egy megfelelő `.daq` lefelé irányuló rekord (vagy egy minőségbiztosításon átesett, a képkockában található célpont) lefedi a képkockát. |

* Az RGB főkamerák debayered + előnézeti adatokat bocsátanak ki; a sugárzás/reflektancia értékeket ezek esetében kihagyják, mivel nem alkalmazhatók.
* A debayered/előnézeti adatok bitmélysége a *Kalibrált képformátum* beállítást követi; a sugárzás mindig float32.
* Az Survey3 feldolgozást ezek a négy kapcsoló nem befolyásolja.

Ugyanezek a négy kapcsolók headless módban is megtalálhatók `chloros-cli process --debayered / --preview / --radiance / --reflectance` néven, valamint az SDK megfelelő paramétereiként. Ezek váltották fel a régi `--radiometric-output` jelzőt, amely már nem létezik.

{% hint style="warning" %}
**Minden alkalmazható termék kikapcsolása a futtatás meghiúsulásához vezet.** Az 1.2.0-s verziótól kezdve egy olyan feldolgozási futtatás, amely termékeket kért, de nem írt ki képterméket, hibaüzenetet jelez, és az CLI nem nulla értékkel lép ki, ahelyett, hogy csendben sikeresnek jelentené. A napló megnevezi azt a terméket, amelyet nem tudott létrehozni, és megadja az okát. A szándékosan kizárólag metaadatokra korlátozott futtatás (nincs kérés) továbbra is sikeresnek minősül.
{% endhint %}

### Reflektancia-forrás (projektbeállítás, az CLI/SDK segítségével állítható be)

A projekt azt is tárolja, hogy a LATTICE reflektancia-termék melyik **reflektancia-referenciát** használja. A beállítások panelen nincs erre szolgáló vezérlőelem; az érték az `Processing → "Target reflectance source"` néven van tárolva a projektkonfigurációban, és az `chloros-cli process --reflectance-source {auto,target,daq}` vagy az SDK `reflectance_source` paraméterrel állítható be:

* **`auto`** (alapértelmezett): a minőségbiztosítási ellenőrzésen átment, a képkockán belüli kalibrációs célpont lesz az abszolút referencia; célpont hiányában vagy a minőségbiztosítási ellenőrzés sikertelensége esetén a rendszer a DAQ lefelé irányuló osztási értékére (ρ = πL/E) értékre.
* **`target`**: szigorúan célpont-vezérelt reflexió — nincs DAQ-helyettesítés.
* **`daq`**: a DAQ-adatok alapján meghatározott reflektancia; a képkockán belüli célpontokat nem használják referenciaként.

A tárolt érték nagy- és kisbetűk, és néhány eltérő írásmód is elfogadott aliasként: `target`, `target_image`, `empirical` és `empirical_line` mind **cél**-t jelentenek; az `daq`, `dls`, `light_sensor` és `sensor` mindegyike**daq**-ot jelentenek. Minden egyéb – beleértve a hiányzó kulcsot is –**auto**-ra értelmeződik.

Az egységenkénti **mért** célérték-beolvasásokat a cél egység sorozatszáma/QR-kódja alapján keresik meg, például `<serial>.csv` formában, három helyen: az `--target-reflectance-dir`-szel megadott könyvtárban (amelyet `Processing → "Target reflectance dir"` néven tárolnak), a projekt saját `target_reflectance/` mappájában, valamint az `CHLOROS_TARGET_REFLECTANCE_DIR` környezeti változóban megadott elérési útvonalon. Ha az adott egységhez nem létezik mért szkennelés, akkor helyette a célmodell névleges, közzétett görbéjét használja a rendszer.

### Demoszaik-eltávolítási módszer

* **Típus**: Legördülő menü
* **Beállítások**:
  * Standard (Gyors, közepes minőség)
  * Textúra-érzékeny (Lassú, legmagasabb minőség) \[Chloros+]
* **Alapértelmezett**: Standard (Gyors, Közepes minőség)
* **Leírás**: Kiválasztja azt a demosaicing algoritmust, amelyet a nyers Bayer-mintázatú érzékelőadatok teljes színes képekké történő átalakításához használnak. A „Standard (Gyors, Közepes minőség)” módszer optimális egyensúlyt biztosít a feldolgozási sebesség és a képminőség között. A „Textúraérzékeny (lassú, legmagasabb minőség)” \[Chloros+] módszer egy kiváló minőségű, élekkel számoló demosaicing algoritmust alkalmaz, amelyet egy AI/ML zajszűrő modellel kombinálnak, így szinte az összes demosaicing zajt eltávolítják. A Texture Aware modell futtatásához GPU-memória (VRAM) szükséges. A gyorsabb feldolgozás érdekében azt javasoljuk, hogy akkor használja, ha &gt;4 GB VRAM áll rendelkezésre.
* **Ha a sor egyáltalán legördülő menü**: a két opciót tartalmazó legördülő menü csak akkor jelenik meg, ha**mindkét**feltétel teljesül — be van jelentkezve egy megfelelő Chloros+ előfizetéssel,**és** a projekt nem tartalmaz LATTICE-felvételeket. Ellenkező esetben a sor egyszerű szövegként jelenik meg, amelyen az `Standard (Fast, Medium Quality)` felirat olvasható, és nincs mit kiválasztani.
* **LATTICE-megjegyzés**: Nincs LATTICE-re betanított Texture Aware modell, és a feldolgozási folyamat a tárolt értéktől függetlenül a standard demosaic-ot kényszeríti a LATTICE-képkockákra. Ha hozzáad egy LATTICE-mappát egy olyan projekthez, amelyben már be volt jelölve a Texture Aware, az „Chloros” beállítás visszaáll „Standard”-ra, ahelyett, hogy az elavult értéket („`project.json`”) hagyná meg.

### Minimális újrakalibrálási időköz

* **Típus**: Szám
* **Tartomány**: 0–3 600 másodperc
* **Alapértelmezett**: 0 másodperc
* **Leírás**: Beállítja a kalibrációs célpontok használata közötti minimális időközöt (másodpercben). 0-ra állítva az az Chloros minden észlelt kalibrációs célt felhasznál. Magasabb értékre állítva az Chloros csak azokat a kalibrációs célokat használja, amelyek között legalább ennyi másodperc telt el, ezzel csökkentve a feldolgozási időt olyan adatsorok esetén, ahol gyakran rögzülnek kalibrációs célok.
* **Mikor érdemes módosítani**:
  * Állítsa 0-ra a maximális kalibrációs pontosság érdekében, ha a fényviszonyok változnak
  * Növelje az értéket (pl. 60–300 másodpercre) a gyorsabb feldolgozás érdekében, ha a fényviszonyok állandóak, és gyakran készülnek kalibrációs célpont-képek

### Fényérzékelő időzóna-eltolódás

* **Típus**: Szám
* **Tartomány**: -12 és +12 óra között
* **Alapértelmezett**: 0 óra
* **Leírás**: Meghatározza a fényérzékelő-adatok időbélyegeinek időzóna-eltolását (órában kifejezve az UTC-hez képest), amelyet a fényérzékelő-naplók és a képfelvételek időpontjainak összehangolásához használnak. Az újabb `.daq` felvételek saját időzóna-adatokat tartalmaznak, ezért ez főként a helyi idő szerint rögzített régebbi naplók esetében szükséges.

### PPK-korrekciók alkalmazása

* **Típus**: Jelölőnégyzet
* **Alapértelmezett**: Letiltva (bejelölés nélkül)
* **Leírás**: Engedélyezi a GPS-t (GNSS) tartalmazó MAPIR DAQ-felvevőkből származó utólagosan feldolgozott kinematikai (PPK) korrekciók használatát. Ha engedélyezve van, az Chloros a projektkönyvtárban található, expozíciós pin-adatokat tartalmazó .daq naplófájlokat használja, és pontos földrajzi helymeghatározási korrekciókat alkalmaz a képeire.
* **Követelmény**: A projektkönyvtárban jelen kell lennie egy expozíciós pin-bejegyzéseket tartalmazó .daq naplófájlnak
* **Mikor kell engedélyezni**: Javasolt mindig engedélyezni a PPK-korrekciót, ha a .daq naplófájlban expozíciós visszacsatolási bejegyzések találhatók.

### 1. expozíciós pin

* **Típus**: Legördülő menü
* **Láthatóság**: Csak akkor látható, ha a „PPK-korrekciók alkalmazása” engedélyezve van, ÉS az 1. pinhez expozíciós adatok állnak rendelkezésre
* **Beállítások**:
  * A projektben felismert kameramodellek nevei
  * „Ne használd” – Ez az expozíciós csatlakozó figyelmen kívül hagyásra kerül
* **Alapértelmezett**: A projekt konfigurációja alapján automatikusan kiválasztásra kerül
* **Leírás**: A PPK időszinkronizáláshoz egy adott kamerát rendel az 1. expozíciós csatlakozóhoz. Az expozíciós pin rögzíti a kamera zárjának pontos kioldási időpontját, ami elengedhetetlen a pontos PPK-alapú helymeghatározáshoz.
* **Automatikus kiválasztás viselkedése**:
  * Egy kamera + egy csatlakozó: A kamera automatikusan kiválasztásra kerül
  * Egy kamera + két csatlakozó: Az 1. csatlakozó automatikusan a kamerához rendelődik
  * Több kamera: Kézi kiválasztás szükséges

### 2. expozíciós csatlakozó

* **Típus**: Legördülő menüből választható
* **Láthatóság**: Csak akkor látható, ha a „PPK-korrekciók alkalmazása” be van kapcsolva, ÉS az expozíciós adatok elérhetők a 2. pinhez
* **Opciók**:
  * A projektben felismert kameramodellek nevei
  * „Ne használja” – Ezt az expozíciós pint figyelmen kívül hagyja
* **Alapértelmezett**: A projekt konfigurációja alapján automatikusan kiválasztva
* **Leírás**: Kétkamerás felállítás használata esetén egy adott kamerát rendel a 2. expozíciós csatlakozóhoz a PPK időszinkronizáláshoz.
* **Automatikus kiválasztás viselkedése**:
  * Egy kamera + egy csatlakozó: A 2. csatlakozó automatikusan „Ne használd” értékre áll
  * Egy kamera + két csatlakozó: A 2. csatlakozó automatikusan „Ne használd” értékre áll
  * Több kamera: Kézi kiválasztás szükséges
* **Megjegyzés**: Ugyanaz a kamera nem rendelhető hozzá egyszerre az 1. és a 2. csatlakozóhoz.***

## DAQ fényérzékelő

Ez a szakasz a Projektbeállítások menüpontban jelenik meg, és felsorolja a projekt összes DAQ lefelé irányuló fényfájlját — `.daq` felvételek és DAQ-M `.csv` lefelé irányuló napfény-naplók. A „Fényérzékelők” fülön készített felvételek automatikusan hozzáadódnak a megnyitott projekthez.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->

Minden sor tartalmazza a fájlt, az érzékelő modelljét, valamint az adott fájlra ténylegesen érvényes diffúzor-sapka korrekciót. A sorok felett egy projekt-szintű vezérlőelem található:

### Sapka felülírás (minden fájl)

* **Típus**: Legördülő menü
* **Beállítások**: `Auto`, valamint a projektben szereplő érzékelőtípusokra érvényes sapka-korrekciós profilok
* **Alapértelmezett**: Automatikus
* **Tárolás**: `Processing → "DAQ cap id"` (alapértelmezett: `auto`)
* **Leírás**: Az `Auto` az egyes fájlokban rögzített napfény-korrekciót használja (ha nincs rögzítve, akkor a Sunshine-korrekciót veszi alapul — az összes MAPIR adatgyűjtő a Sunshine-korrektorral együtt kerül forgalomba). Egy adott korrekciós érték kiválasztása felülírja a projektben található**minden** lefelé irányuló fájlt: a nyers felvételeket ezzel korrigálja, a már korrekciós értékkel rendelkező felvételeket pedig újra referenciálja (a rögzített korrekciót visszavonja, és a kiválasztottat alkalmazza).
* **Fontos**: A kiválasztott korrekciós értéknek meg kell egyeznie azzal a korrekciós értékkel, amelyet a felvétel során fizikailag felszereltek. Sem az érzékelő, sem a szoftver nem képes felismerni a fizikai fedelet – az eltérő fedél-azonosító a spektrumok helytelen korrekciójához vezet.

Szándékosan **egy** projekt-szintű vezérlőelem van, ahelyett, hogy fájlonként lennének legördülő menük: a beállítás a projekt minden lefelé irányuló forrására kiterjed.***

## Mátrix-igazítás

Ez a szakasz **csak** akkor jelenik meg, ha a projektben legalább egy kép tartalmazza azt a modulok közötti igazítási transzformációt, amelyet a LATTICE-tömbök a rögzítéskor jelölnek meg (XMP `Chloros:Alignment*` címkék). Megmutatja, hány kép tartalmaz igazítási címkéket, melyik kamera a referencia (`REF` jelölés), valamint kameránkénti táblázatot a képek számáról.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Tömbigazítás alkalmazása

* **Típus**: Jelölőnégyzet
* **Alapértelmezett**: Engedélyezve (bejelölve)
* **Tárolás**: `Processing → "Array alignment"`
* **Leírás**: Minden feldolgozott terméket (debayered / preview / radiance / reflectance / index) a felvételkészítéskor bejegyzett transzformáció segítségével a rendszer közös referencia-geometriájába illeszti. Ki = exportálás az egyes érzékelők natív geometriájában.

### Kivágás a közös átfedésre

* **Típus**: Jelölőnégyzet (csak akkor aktív, ha a *Tömb igazítás alkalmazása* be van kapcsolva)
* **Alapértelmezés**: Engedélyezve (bejelölve)
* **Tárolás**: `Processing → "Array alignment crop"`
* **Leírás**: Az igazított exportokat a kameramodulok által közösen használt területre vágja, így minden sáv azonos méretű lesz. Ki állapotban a teljes érzékelőterület megmarad (a forráson kívül fekete kitöltéssel).

### Újramintavétel

* **Típus**: Legördülő menü (csak akkor aktív, ha az *Tömb igazítás alkalmazása* be van kapcsolva)
* **Beállítások**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Alapértelmezett**: Bilineáris
* **Tárolás**: `Processing → "Array alignment interpolation"`
* **Leírás**: Az igazítási torzítás által használt interpoláció. A „Legközelebbi” beállítás megőrzi a forrás pontos értékeit (nincs pixelek közötti keveredés) a szigorú radiometrikus elemzéshez; a „Bilineáris” a legalkalmasabb térképezéshez és vizuális használatra.

Ugyanezek a három beállítások „headless” formában is elérhetők: `chloros-cli process --array-alignment`, `--array-alignment-crop` és `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Index

Ezek a beállítások lehetővé teszik a multispektrális indexek konfigurálását az elemzés és a megjelenítés céljából.

### Index hozzáadása

* **Típus**: Speciális indexkonfigurációs panel
* **Leírás**: Megnyit egy interaktív panelt, ahol kiválaszthatja és konfigurálhatja a képfeldolgozás során kiszámítandó multispektrális vegetációs indexeket (NDVI, NDRE, EVI stb.). Több indexet is hozzáadhat, mindegyikhez saját megjelenítési beállításokkal.
* **Elérhető indexek**: A grafikus felület legördülő menüje**27** előre definiált multispektrális indexképletet tartalmaz (a teljes listát lásd a [Multispektrális indexképletek](multispectral-index-formulas.md) a teljes listaért, beleértve azokat a neveket is, amelyeket az CLI/SDK `--indices` opció is elfogad).
* **Funkciók**:
  * Válasszon az előre definiált indexképletek közül
  * Húzza a kamera szűrőcsatornáit az indexképlet sávhelyeire
  * Konfigurálja a vizualizációs színátmeneteket (LUT – Look-Up Tables)
  * Állítsa be a küszöbértékeket és a levágási módokat
  * Hozzon létre egyéni indexképleteket
* **Megjegyzés**: Az egycsatornás LATTICE M3M monokamerák esetében az indexek nem kerülnek kiszámításra — a többsávos indexek egy sávon nincsenek definiálva. Az Survey3 és a LATTICE M3C esetében ez nem érintett.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->

Minden hozzáadott index a képletét matematikai kifejezésként jeleníti meg, sávhelyenként egy-egy színes körrel: piros = Red, zöld = Green, kék = Blue, narancssárga = Orange, cián = Cyan, lila = NIR, magenta = RE. Húzzon egy kört a képlet feletti sorból egy helyre a hozzárendeléshez; kattintson duplán egy hozzárendelt helyre a törléshez. Az index csak akkor számolja ki az értéket, ha a képlet által használt minden helyhez tartozik csatorna.

### Egyéni képletek (Chloros+ funkció)

* **Típus**: Egyéni képletdefiníciók tömbje
* **Elérhetőség**: Megfelelő Chloros+ előfizetéssel való bejelentkezés szükséges.
* **Leírás**: Lehetővé teszi egyedi multispektrális indexképletek létrehozását és mentését sávszámítás segítségével. Az egyedi képletek a projektbeállításokkal együtt kerülnek mentésre, és ugyanúgy használhatók, mint a beépített indexek.
* **Létrehozás**:
  1. Az Index konfigurációs panelen nyissa meg az egyedi képlet-számológépet
  2. Írja be a képletet a **sávhely-szimbólumok** használatával, ne a sávnevekkel
  3. Mentse el a képletet egy leíró névvel — ezután megjelenik a képlet legördülő menü alján, és a kamera csatornaköröit pontosan úgy húzhatja a helyekre, mint egy beépített előre beállított értéknél
* **Képlet szintaxisa**:
  * Sávhelyek: `x`, `y`, `z`, `a`, `b`, `c` — hat pozíció, amelyeket húzással rendelhet a valódi csatornákhoz
  * Műveletek: `+`, `-`, `*`, `/`, `^` és `()` csoportosításhoz
  * Funkciók: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Miért szimbólumok, és nem sávnevek**: az `(y-x)/(y+x)` formában írt képlet bármelyik kamerán működik, mert a húzásés elhúzás alapján történő leképezés határozza meg, hogy az `y` egy RGN szűrő 850 nm-es NIR változata-e, vagy egy OCN szűrő 808 nm-es NIR-e egy OCN szűrőben. A beépített előre beállított értékek ugyanúgy vannak tárolva — lásd [Multispektrális indexképletek](multispectral-index-formulas.md) című részt az összes 27 szimbólum pontos formájához.
* **Hol használhatók**: az egyéni képletek a projektbeállításokkal együtt kerülnek mentésre, és felhasználhatók az [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) és a feldolgozás során is használhatók. Ezeket**nem** fogadja el az CLI/SDK `--indices` névlista, amely kizárólag a 22 beépített előre beállított név kiterjesztésére szolgál.***

## Exportálás

Ezek a beállítások szabályozzák az exportált, feldolgozott képek formátumát és minőségét.

### Kalibrált képformátum

* **Típus**: Legördülő menü
* **Beállítások**:
  * **TIFF (16-bit)** – Tömörítetlen 16-bites TIFF formátum
  * **TIFF (32 bites, százalék)** – 32 bites lebegőpontos TIFF, ahol a fényvisszaverődési értékek százalékban vannak megadva
  * **PNG (8 bites)** – Tömörített 8 bites PNG formátum
  * **JPG (8 bites)** – Tömörített 8 bites JPEG formátum
* **Alapértelmezett**: TIFF (16 bites)
* **Leírás**: Kiválasztja a feldolgozott és kalibrált képek mentéséhez használt fájlformátumot. Az exportált fájlok az egyes kamerák mappáján belül, formátumonkénti almappákba kerülnek (`tiff16`, `tiff32`, `png8`, `jpg8`), termékenként egy `<Product>_Images/` mappával. Az exportált fájlok megtartják a forrásfájl nevét – a terméket a mappa, nem pedig a fájlnévkiterjesztés azonosítja.
* **Formátumajánlások**:
  * **TIFF (16 bites)**: Tudományos elemzésekhez és professzionális munkafolyamatokhoz ajánlott. Tömörítési torzulások nélkül megőrzi a maximális adatminőséget. Legalkalmasabb multispektrális elemzésekhez és a GIS-szoftverekben történő további feldolgozáshoz.
  * **TIFF (32-bites, százalékos)**: Legalkalmasabb olyan munkafolyamatokhoz, amelyeknél a reflexiós értékeket százalékban (0–100%) kell megadni. Maximális pontosságot biztosít radiometrikus mérésekhez.
  * **PNG (8 bites)**: Webes megtekintéshez és általános vizualizációhoz alkalmas. Kisebb fájlméret veszteségmentes tömörítéssel, de csökkentett dinamikatartománnyal.
  * **JPG (8 bites)**: A legkisebb fájlméret, kizárólag előnézetekhez és webes megjelenítéshez ideális. Veszteséges tömörítést használ, amely nem alkalmas tudományos elemzésekhez.
* **Megjegyzés**: A LATTICE sugárzási értékeket e beállítástól függetlenül mindig 32-bites lebegőpontos TIFF formátumban exportálja a rendszer.***

## Projekt-sablon mentése

Ez a funkció lehetővé teszi az aktuális projektbeállítások újrafelhasználható sablonként való mentését.

* **Típus**: Szövegbeviteli mező + Mentés gomb
* **Leírás**: Adjon meg egy leíró nevet a beállítási sablonjának, majd kattintson a mentés ikonra. A sablon az összes aktuális projektbeállítást (célfelismerés, feldolgozási beállítások, indexek és exportformátum) a jövőbeli projektekben való egyszerű újrafelhasználás céljából. A sablonok a projekt mentési mappáján belüli `Project Templates/` mappában kerülnek tárolásra, és a főmenüből is kiválaszthatók vagy exportálhatók (*Sablon kiválasztása* / *Sablon mentése* / *Sablon exportálása*).
* **Alkalmazási példák**:
  * Sablonok létrehozása különböző kamerarendszerekhez (RGB, multispektrális, NIR)
  * Szabványos konfigurációk mentése meghatározott növénytípusokhoz vagy elemzési munkafolyamatokhoz
  * Egységes beállítások megosztása a csapat tagjai között
* **Használata**:
  1. Állítsa be az összes kívánt projektbeállítást
  2. Adjon meg egy sablonnevet (pl. „RedEdge Survey3 NDVI Standard”)
  3. Kattintson a mentés ikonra
  4. A sablon mostantól betölthető új projektek létrehozásakor

***

## Projektmappa mentése

Ez a beállítás határozza meg, hogy az új projekteket alapértelmezés szerint hova menti a rendszer.

* **Típus**: Könyvtárútvonal megjelenítése + Szerkesztés gomb
* **Alapértelmezett (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Alapértelmezett (Linux)**: `~/Chloros Projects`
* **Leírás**: Megjeleníti az aktuális alapértelmezett könyvtárat, ahová az új Chloros projektek létrehozásra kerülnek. Kattintson a szerkesztés ikonra egy másik könyvtár kiválasztásához. A felülírás egyetlen szövegsorban kerül tárolásra az `~/.chloros/working_directory.txt` fájlban, azaz az `C:\Users\<Username>\.chloros\working_directory.txt` fájlban. Ha ez a fájl hiányzik, vagy olyan elérési utat nevez meg, amely már nem létezik, az Chloros a fenti alapértelmezett beállításra tér vissza. Az CLI ugyanazt a fájlt olvassa és írja, így az `chloros-cli` és a grafikus felület mindig egyetért abban, hogy hol találhatók a projektek.
* **A projektsablonok** ebben a könyvtárban található `Project Templates/` almappában találhatók.
* **Mikor érdemes módosítani**:
  * Állítsa be hálózati meghajtóra a csapatmunkához
  * Váltson nagyobb tárhelyű meghajtóra nagy adathalmazok esetén
  * Rendezze a projekteket év, ügyfél vagy projekttípus szerint különböző mappákba
* **Megjegyzés**: A beállítás módosítása csak az ÚJ projekteket érinti. A meglévő projektek az eredeti helyükön maradnak.***

## A beállítások megőrzése

Egy Chloros projekt egy **mappa**. Az összes projektbeállítás az abban található `project.json` mappába kerül mentésre; a csatlakoztatott hardverek adatai pedig az `cameras.json` és az `sensors.json` mappákban kerülnek elmentésre, így a projekt újbóli megnyitásakor a kamerák és a fényérzékelők is újra csatlakoznak. A projekt újbóli megnyitásakor az összes beállítás pontosan úgy áll vissza, ahogyan hagyta őket. A mentett projekteket a `chloros-cli project` vagy az SDK `open_project` fájljával vezető nélkül is lehet irányítani.

### A beállítások hierarchiája

A beállítások a következő sorrendben kerülnek alkalmazásra:

1. **Rendszer alapértelmezések** – Az Chloros által meghatározott beépített alapértelmezések
2. **Sablonbeállítások** – Ha sablont tölt be a projekt létrehozásakor
3. **Mentett projektbeállítások** – A projektfájllal együtt mentett beállítások
4. **Kézi módosítások** – Az aktuális munkamenet során végzett bármilyen változtatás

### Beállítások és képfeldolgozás

A feldolgozási beállítások a feldolgozási futtatás indításakor kerülnek beolvasásra. A beállítások módosítása nem érinti visszamenőlegesen a lemezen már meglévő eredményeket – az új beállítások alkalmazásához futtassa újra a feldolgozást. Néhány beállítás egyáltalán nem befolyásolja a feldolgozást:

* Képminiatűr felbontás (csak megjelenítésre szolgál)
* Projekt sablon mentése
* Projektmappa mentése

***

## Konfigurációs kulcsok áttekintése

Az automatizáláshoz (CLI `--config`, SDK `configure`, vagy az `project.json` közvetlen olvasása esetén) a következő kulcsok találhatók az `Project Settings` alatt:

| Kulcsútvonal | Típus | Alapértelmezett |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | 0–10000 közötti szám | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | 0–100 közötti szám | `60` |
| `Processing → Vignette correction` | logikai érték | `true` |
| `Processing → Reflectance calibration / white balance` | logikai érték | `true` |
| `Processing → Export sensor response` | bool | `true` |
| `Processing → Export vignette corrected` | bool | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Standard |
| `Processing → Minimum recalibration interval` | szám 0–3600 | `0` |
| `Processing → Light sensor timezone offset` | -12..12 közötti szám | `0` |
| `Processing → Apply PPK corrections` | logikai érték | `false` |
| `Processing → DAQ cap id` | korlátprofil-azonosító vagy `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | indexkonfigurációk listája | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Az `Array alignment` kulcsok akkor kerülnek beírásra, amikor az Array Alignment szakasz először renderel, vagy egy automatizálási hívás beállítja őket. Amíg ezek hiányoznak, a feldolgozási folyamat a fentiekben látható értékeket használja (`true`, `true`, bilineáris), így egy projekt, amelyben ezek hiányoznak, pontosan ugyanúgy viselkedik, mint egy olyan, amelyben szerepelnek.

### Az `project.json`-ben tárolt kulcsok, amelyekhez nincs vezérlőelem a beállítási panelen

Ezek ugyanazon az `Project Settings`-fa alatt találhatók, és a feldolgozás során beolvasásra kerülnek, de az oldalsávban nem találsz hozzájuk tartozó widgetet:

| Kulcsútvonal | Típus | Alapértelmezett | Beállítja: |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Felülírja a LATTICE bemeneti TIFF-fájlok értelmezését; az `auto` az egyes fájlok `Chloros:ProcessingLevel` XMP-címkéjéből és a csatornák számából következtet. Az Survey3 `.raw` rögzítéseknél figyelmen kívül marad. Szándékosan nem GUI-beállítás — az „auto” érték minden normál esetben helyes. |
| `Processing → Target reflectance dir` | elérési út karakterlánc | `""` | `chloros-cli process --target-reflectance-dir`, vagy a projekt célja API |
| `Processing → Target reflectance config` | a kamera sorozatszáma alapján indexelt szótár | `{}` | Képkockán belüli célpont regisztrálása (mód: `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | elérési út karakterlánc | `""` | SDK `process_folder(daq_log_path=…)`. Egy `.daq` felvételre vagy azok egy mappájára |
| `Target Detection → Minimum calibration target squares` | szám | `4` | Régi alapértelmezés; nincs vezérlés és nincs CLI jelző |
| `UI → Grid thumbnail size` | szám | `160` | A képrács saját miniatűr nagyító csúszkája |

Két nézőbeállítás van **a legfelső szinten az `project.json`-ben**, teljesen az `Project Settings`-en kívül, mivel ezek inkább megjelenítési állapotok, mint feldolgozási beállítások:

| Kulcsút | Típus | Alapértelmezett | Beállítja |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | egész szám 1–256 | `1` | A Kép fül GSD (px) vezérlője — lásd [Kép teljes képernyős megnyitása](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Bevált gyakorlatok

1. **Kezdje az alapértelmezett beállításokkal**: Az alapértelmezett beállítások a legtöbb MAPIR kamerarendszer és a tipikus munkafolyamatok esetében jól működnek.
2. **Készítsen sablonokat**: Miután optimalizálta a beállításokat egy adott munkafolyamat vagy kamera számára, mentse el őket sablonként, hogy biztosítsa a projektek közötti egységességet.
3. **Tesztelje a beállításokat a teljes feldolgozás előtt**: Ha új beállításokkal kísérletezik, tesztelje azokat a képek egy kis részén, mielőtt a teljes adathalmazt feldolgozná.
4. **Dokumentálja a beállításait**: Használjon leíró sablonneveket, amelyek jelzik a kamerarendszert, a feldolgozás típusát és a tervezett felhasználást (pl. „Survey3\_RGB\_NDVI\_Agriculture”).
5. **Exportformátum kiválasztása**: Válassza ki az exportformátumot a végfelhasználás alapján:
   * Tudományos elemzés → TIFF (16-bites vagy 32-bites)
   * GIS-feldolgozás → TIFF (16 bites)
   * Gyors megjelenítés → PNG (8 bites)
   * Webes megosztás → JPG (8 bites)

***

Az Chloros multispektrális indexeiről további információkat az [Multispektrális indexképletek](multispectral-index-formulas.md) oldalon talál.
