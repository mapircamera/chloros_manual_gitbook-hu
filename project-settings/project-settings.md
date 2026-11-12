# Projektbeállítások

A Chloros-ban található „Projektbeállítások” <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

oldalsáv segítségével beállíthatja a képfeldolgozás, a kalibrációs célpontok felismerése, a multispektrális indexek kiszámítása, valamint a projekt exportálási beállításainak minden részletét. Ezek a beállítások a projekttel együtt kerülnek mentésre, és sablonként is elmenthetők, hogy több projektben is újra felhasználhassa őket.

## A projektbeállítások megnyitása

A projektbeállítások megnyitásához:

1. Nyisson meg egy projektet a Chloros ban
2. Kattintson a bal oldali oldalsávon a **Projektbeállítások** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

fülre
3. A beállítási panelen kategóriák szerint rendezve jelennek meg az összes elérhető konfigurációs lehetőség



<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**Azok a beállítások, amelyek más beállításoktól függenek, szürkén jelennek meg.** Ha egy szülőkapcsoló miatt egy beállítás nem állítható be (például a *Fényvisszaverődés-kalibrálás / fehéregyensúly* jelölőnégyzet kikapcsolása esetén a *Fényvisszaverődés exportálása* nem lehetséges), akkor a függő vezérlőelem letiltásra kerül, és a segítő szövegben megjelenik annak a kapcsolónak a neve, amelyet módosítani kell.
{% endhint %}

***

## Kijelző

### Képminiatűr felbontása

* **Típus**: Legördülő menü
* **Lehetőségek**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Alapértelmezett**: Alapértelmezett (512 px)
* **Leírás**: Az a felbontás (a leghosszabb él hossza, pixelben kifejezve), amelyen a kép-rácsos miniatűrök megjelennek. A magasabb értékek nagyításkor élesebb képet adnak, de lassabban töltődnek be és több memóriát igényelnek. A teljes felbontás az eredeti kép méretét adja vissza.
* **Megjegyzés**: Csak megjelenítésre szolgál — ez soha nem befolyásolja a feldolgozást vagy az exportált fájlokat.

***

## Célpont-felismerés

Ezek a beállítások szabályozzák, hogy a Chloros hogyan ismeri fel és dolgozza fel a képeken található kalibrációs célpontokat. Mindkettő csak akkor aktív, ha a **Fényvisszaverődés-kalibráció / fehéregyensúly** be van kapcsolva (ellenkező esetben szürkén jelennek meg, mert a célpont-felismerést teljesen kihagyják).

### Minimális kalibrációs mintafelület (px)

* **Típus**: Szám
* **Tartomány**: 0–10 000 pixel
* **Alapértelmezett**: 25 pixel
* **Leírás**: Beállítja azt a minimális területet (pixelben), amely szükséges ahhoz, hogy egy felismert régiót érvényes kalibrációs célminta-területnek tekintsünk. A kisebb értékek kisebb célokat is felismernek, de növelhetik a téves pozitív eredmények számát. A nagyobb értékek nagyobb, egyértelműbb célterületeket igényelnek a felismeréshez.
* **Mikor érdemes módosítani**:
  * Növelje az értéket, ha kis képzavarok miatt téves észlelések jelentkeznek
  * Csökkentse az értéket, ha a kalibrációs célpontok kicsinek tűnnek a képeken, és nem kerülnek észlelésre

### Minimális célpont-csoportosítás (0–100)

* **Típus**: Szám
* **Tartomány**: 0–100
* **Alapértelmezett**: 60
* **Leírás**: Szabályozza a hasonló színű területek csoportosításának küszöbértékét a kalibrációs célpontok észlelése során. A magasabb értékeknél több hasonló színű területet kell egy csoportba rendezni, ami konzervatívabb célpont-észlelést eredményez. Az alacsonyabb értékek több színváltozást engednek meg egy célpontcsoporton belül.
* **Mikor kell módosítani**:
  * Növelje, ha a kalibrációs célpontok több észlelésre oszlanak szét
  * Csökkentse, ha a színváltozással rendelkező kalibrációs célpontok nem kerülnek teljes mértékben észlelésre

***

## Feldolgozás

Ezek a beállítások szabályozzák, hogy a Chloros hogyan dolgozza fel és kalibrálja a képeit.

### Vignett-korrekció

* **Típus**: Jelölőnégyzet
* **Alapértelmezett**: Engedélyezve (bejelölve)
* **Leírás**: Vignett-korrekciót alkalmaz a képek szélein jelentkező lencse-sötétedés kompenzálására. A vignettálás egy gyakori optikai jelenség, amelynek során a kép sarkai és szélei a lencse jellemzői miatt sötétebbnek tűnnek, mint a közepe.
* **Mellékhatás**: Ez a kapcsoló azt is meghatározza, hogy a futtatás melyik *kalibrálatlan tartalékterméket* írja ki (lásd alább).

### Reflektancia-kalibrálás / fehéregyensúly

* **Típus**: Jelölőnégyzet
* **Alapértelmezett**: Engedélyezve (bejelölve)
* **Leírás**: Engedélyezi a reflexiókalibrálást — a képkockán belül észlelt kalibrációs célpontok és/vagy a DAQ fényérzékelőjének lefelé irányuló sugárzási adatai alapján, a kamerától és a rendelkezésre álló adatoktól függően. Ez normalizálja a reflexióértékeket az adatkészletben, és biztosítja a mérési eredmények konzisztenciáját a fényviszonyoktól függetlenül.
* **Ha ki van kapcsolva**: A célpontok észlelését teljesen kihagyják, és**egyik kamera sem képes reflektancia-terméket előállítani** — sem a Survey3-célpont-vezérelt, sem a LATTICE DAQ-vezérelt rendszerek. A függő beállítások (*Reflektancia exportálása*, *Minimális újrakalibrálási intervallum* és a célpont-észlelési küszöbértékek) szürkén jelennek meg.

### Kalibrálatlan tartalék termékek: Érzékelőválasz exportálása / Vignettáláskorrigált exportálás

* **Típus**: Két jelölőnégyzet
* **Alapértelmezés**: Mindkettő engedélyezve (bejelölve)
* **Leírás**: Ha egy képkocka reflektancia-kalibrálása nem lehetséges (nem találtak kalibrációs célt, vagy a reflektancia-kalibrálás ki van kapcsolva), akkor azt *kalibrálatlan tartalék termékként* írja ki. **Minden futtatás során, minden kameramodell esetében** pontosan egy a két tartaléktermék közül létezik, amelyet a *Vignettkorrekció* kapcsoló választ ki:
  * Vignettakorrekció **be**→ `Vignette_Corrected_Images/` (az**Export vignettakorrigált** beállítás határozza meg)
  * Vignettakorrekció **ki**→ `Sensor_Response_Images/` (az**Export érzékelőválasz** beállítás határozza meg)
* Az éppen nem használt tartaléktermék szürkén jelenik meg. Az éppen használt termék jelölésének eltávolításával megakadályozható, hogy az adott fájl egyáltalán létrejöjjön.

### LATTICE exporttermékek

A LATTICE-felvételeket tartalmazó projektek esetében minden importált LATTICE-képkocka egyetlen feldolgozási lépésben szétágazik minden engedélyezett **és alkalmazható**termékre. Négy jelölőnégyzet szabályozza az elosztást (alapértelmezés szerint mind**be** van kapcsolva):

| Beállítás | Kimeneti mappa | Mit exportál |
| --- | --- | --- |
| **Debayered export** | `Debayered_Images/` | A lineáris debayered kép: a kalibrációs csomag sötét eltolásának eltávolítása után, majd élek figyelembevételével végzett demosaic, linearitás-korrekció, flat field vagy unmix nélkül — ugyanaz a fájl, amelyet a felvétel debayered szinten ment. Az érzékelő natív tájolásában marad (XMP-ben rögzítve).RGB- és multispektrális kamerákra vonatkozik. |
| **Előnézet exportálása** | `Preview_Images/` | A képernyőn megjelenő előnézet. RGB: fehér egyensúly (DAQ-megvilágítás, ha elérhető, egyébként szürke-világ) + gamma; multispektrális: hamis színű kiterjesztés. |
| **Sugárzás exportálása** | `Radiance_Images/` | Float32 spektrális sugárzás W/m²/sr/nm-ben. Csak multispektrális (M3C/M3M) – nem alkalmazható RGB-mesterfájlokra. Mindig 32 bites TIFF formátumban kerül írásra, függetlenül a *Kalibrált képformátum* beállítástól. |
| **Reflektancia exportálása**| `Reflectance_Calibrated_Images/` | Uint16 reflektancia, úgy skálázva, hogy**32768 = 1,0-s reflektancia** (XMP-ben `Chloros:PixelScale` néven szerepel). Csak multispektrális képek esetében kerül rögzítésre, ha a képkockát egy megfelelő `.daq` lefelé irányuló sugárzási rekord (vagy minőségbiztosításon átesett, a képkockán belüli célpont) fedi le. |

* Az RGB-es főkamerák debayered + előnézeti adatokat bocsátanak ki; a sugárzás/reflektancia értékeket ezek esetében kihagyják, mivel nem alkalmazhatók.
* A debayered/előnézeti adatok bitmélysége a *Kalibrált képformátum* beállításnak felel meg; a sugárzás értéke mindig float32.
* A Survey3-feldolgozást ezek a négy kapcsoló nem befolyásolják.
* A multispektrális M3C kamerák sugárzási és visszaverődési termékei szintén írhatnak pixelenkénti **clamp maszkot** a termékmappák melletti `clamp_masks/` mappába, rögzítve azokat a helyeket, ahol a sávszétválasztás során a negatív értékeket nullára vágta. Lásd [Kimeneti képformátumok](../output-image-formats.md#clamp-masks).

Ugyanezek a négy kapcsolók headless módban is elérhetők `chloros-cli process --debayered / --preview / --radiance / --reflectance` néven. Ezek váltották fel a régi CLI-es `--radiometric-output` jelzőt, amely már nem létezik.

{% hint style="warning" %}
**Minden alkalmazható termék kikapcsolása a futtatás meghiúsulását eredményezi.** Az 1.2.0-s verziótól kezdve egy olyan feldolgozási futtatás, amely termékeket kért, de nem írt ki képterméket, hibaüzenetet jelez, és az CLI nem nulla értékkel lép ki, ahelyett, hogy csendben jelentette volna a sikeres futtatást. A napló megnevezi azt a terméket, amelyet nem tudott kiírni, és megadja az okát. Egy szándékosan kizárólag metaadatokra korlátozott futtatás (semmi nem lett lekérve) továbbra is sikeresnek minősül.
{% endhint %}

### Reflektancia-forrás (projektbeállítás, az CLI-n keresztül állítható be)

A projekt azt is tárolja, hogy a LATTICE reflektancia-termék melyik **reflektancia-referenciát** használja. A beállítások panelen nincs erre szolgáló vezérlőelem; az érték `Processing → "Target reflectance source"` néven van tárolva a projektkonfigurációban, és az `chloros-cli process --reflectance-source {auto,target,daq}` (az SDK `configure()`-jéhez nincs ehhez tartozó kulcsszó):

* **`auto`** (alapértelmezett): a minőségbiztosítást (QA) sikeresen teljesítő, a képkockán belüli kalibrációs célpont lesz az abszolút referencia; ha nincs célpont, vagy a minőségbiztosítás sikertelen, akkor a rendszer a DAQ lefelé irányuló sugárzás osztására (ρ = πL/E) tér vissza. Az a célpont, amely jelzéssel (leesett panel vagy meghibásodott sáv) felel meg a minőségbiztosításnak, nem kerül automatikusan előnyben részesítésre: a célpont, a „célpont+DAQ” és a „csak DAQ” láncokat a kihagyott paneleken mért pontszámuk sorrendjében próbálják ki.
* **`target`**: szigorúan célpont-vezérelt reflexió — nincs DAQ-helyettesítés.
* **`daq`**: DAQ-vezérelt reflexió; a képkockán belüli célpontokat nem használják referenciaként.

A tárolt érték nagy- és kisbetűktől függetlenül kerül összehasonlításra, és néhány írásmód is elfogadott aliasként: `target`, `target_image`, `empirical` és `empirical_line` mindegyike **célpontot**jelent; az `daq`, `dls`, `light_sensor` és `sensor` mindegyike**daq***-ot jelent*-t jelentenek. Minden egyéb – beleértve a hiányzó kulcsot is – **auto**-ra értelmeződik.

Az egységenkénti **mért** célérték-letapogatások a cél egység sorozatszámának/QR-kódjának alapján, például `<serial>.csv` formában, három helyen kerülnek lekérdezésre: az `--target-reflectance-dir`-szel megadott könyvtárban (amelyet `Processing → "Target reflectance dir"` néven tárolnak), a projekt saját `target_reflectance/` mappájában, valamint az `CHLOROS_TARGET_REFLECTANCE_DIR` környezeti változóban megadott elérési útvonalon. Ha az adott egységhez nem létezik mért szkennelés, akkor helyette a célmodell névleges, közzétett görbéjét használja a rendszer.

### Debayer-módszer

* **Típus**: Legördülő menüből választható
* **Beállítások**:
  * Standard (Gyors, közepes minőség)
  * Textúraérzékeny (Lassú, legmagasabb minőség) \[Chloros+]
* **Alapértelmezett**: Standard (Gyors, közepes minőség)
* **Leírás**: Kiválasztja azt a demosaicing algoritmust, amelyet a nyers Bayer-mintázatú érzékelőadatok teljes színes képekké történő átalakításához használnak. A „Standard (Gyors, közepes minőség)” módszer optimális egyensúlyt biztosít a feldolgozási sebesség és a képminőség között. A „Texture Aware (Lassú, legmagasabb minőség)” \[Chloros+] egy kiváló minőségű, élek figyelembevételével működő demosaicing algoritmust használ, amelyet egy AI/ML zajszűrő modellel kombinálnak, így szinte az összes demosaicing zajt eltávolítja. A Textúra-érzékeny modell futtatásához GPU-memória (VRAM) szükséges. A gyorsabb feldolgozás érdekében azt javasoljuk, hogy akkor használja, ha >4 GB VRAM áll rendelkezésre.
* **Ha a sor egyáltalán legördülő menü**: a két lehetőséget tartalmazó legördülő menü csak akkor jelenik meg, ha**mindkettő** igaz – be van jelentkezve egy megfelelő Chloros+ előfizetéssel, **és** a projekt nem tartalmaz LATTICE-felvételeket. Ellenkező esetben a sor egyszerű szövegként jelenik meg (`Standard (Fast, Medium Quality)`), és nincs mit kiválasztani.
* **LATTICE-megjegyzés**: Nincs LATTICE-re betanított Texture Aware modell, és a feldolgozási folyamat a tárolt értéktől függetlenül a standard demosaic-ot kényszeríti a LATTICE-képkockákra. Ha hozzáad egy LATTICE-mappát egy olyan projekthez, amelyben már be volt jelölve a Texture Aware, a Chloros visszaállítja a beállítást Standardra, ahelyett, hogy elavult értéket hagyna az `project.json` fájlban.

### Minimális újrakalibrálási intervallum

* **Típus**: Szám
* **Tartomány**: 0–3 600 másodperc
* **Alapértelmezett**: 0 másodperc
* **Leírás**: Beállítja a kalibrációs célpontok használata közötti minimális időintervallumot (másodpercben). Ha 0-ra van állítva, a Chloros minden észlelt kalibrációs célpontot felhasznál. Ha nagyobb értékre van állítva, a Chloros csak azokat a kalibrációs célpontokat használja, amelyek között legalább ennyi másodperc telt el, így csökkentve a feldolgozási időt azoknál az adatkészleteknél, ahol gyakran rögzítenek kalibrációs célpontokat.
* **Mikor érdemes módosítani**:
  * Állítsa 0-ra a maximális kalibrációs pontosság érdekében, ha a fényviszonyok változnak
  * Növelje az értéket (pl. 60–300 másodpercre) a gyorsabb feldolgozás érdekében, ha a fényviszonyok állandóak, és gyakran készülnek kalibrációs célpont-képek

### Fényérzékelő időzóna-eltolódás

* **Típus**: Szám
* **Tartomány**: -12 és +12 óra között
* **Alapértelmezett**: 0 óra
* **Leírás**: Meghatározza a fényérzékelő-adatok időbélyegeinek időzóna-eltolását (órában kifejezve az UTC-hez képest), amelyet a fényérzékelő-naplók és a képfelvételek időpontjainak összehangolásához használnak. Az újabb `.daq` felvételek saját időzóna-adatot tartalmaznak, ezért ez főként a helyi idő szerint rögzített régebbi naplók esetében szükséges.

### PPK-korrekciók alkalmazása

* **Típus**: Jelölőnégyzet
* **Alapértelmezett**: Letiltva (bejelölés nélkül)
* **Leírás**: Engedélyezi a GPS-t (GNSS) tartalmazó MAPIR DAQ-rögzítőkből származó utólagosan feldolgozott kinematikai (PPK) korrekciók használatát. Ha engedélyezve van, a Chloros felhasználja a projektkönyvtárban található, expozíciós pin-adatokat tartalmazó .daq naplófájlokat, és pontos földrajzi helymeghatározási korrekciókat alkalmaz a képeire.
* **Követelmény**: A projektkönyvtárban jelen kell lennie egy expozíciós pont bejegyzéseket tartalmazó .daq naplófájlnak
* **Mikor érdemes engedélyezni**: Javasolt mindig engedélyezni a PPK-korrekciót, ha a .daq naplófájlban vannak expozíciós visszacsatolási bejegyzések.

### 1. expozíciós pin

* **Típus**: Legördülő menü
* **Láthatóság**: Csak akkor látható, ha a „PPK-korrekciók alkalmazása” be van kapcsolva, ÉS az 1. expozíciós pont számára elérhetőek az expozíciós adatok
* **Beállítások**:
  * A projektben felismert kameramodellek nevei
  * „Ne használja” – Ez az expozíciós pont figyelmen kívül hagyandó
* **Alapértelmezett**: A projektkonfiguráció alapján automatikusan kiválasztva
* **Leírás**: Egy adott kamerát rendel az 1. expozíciós csaphoz a PPK időszinkronizáláshoz. Az expozíciós csap rögzíti a kamera zárjának pontos kioldási időpontját, ami elengedhetetlen a pontos PPK-alapú helymeghatározáshoz.
* **Automatikus kiválasztás viselkedése**:
  * Egy kamera + egy csatlakozó: A rendszer automatikusan kiválasztja a kamerát
  * Egy kamera + két csatlakozó: Az 1. csatlakozó automatikusan a kamerához van rendelve
  * Több kamera: Kézi kiválasztás szükséges

### 2. expozíciós pin

* **Típus**: Legördülő menüből történő kiválasztás
* **Láthatóság**: Csak akkor látható, ha a „PPK-korrekciók alkalmazása” be van kapcsolva, ÉS a 2. pinhez expozíciós adatok állnak rendelkezésre
* **Opciók**:
  * A projektben felismert kameramodellek nevei
  * „Ne használja” – A 2. expozíciós csatlakozót figyelmen kívül hagyja
* **Alapértelmezett**: A projekt konfigurációja alapján automatikusan kiválasztásra kerül
* **Leírás**: Kétkamerás felállítás használata esetén egy adott kamerát rendel a 2. expozíciós csatlakozóhoz a PPK-időszinkronizáláshoz.
* **Automatikus kiválasztás viselkedése**:
  * Egy kamera + egy csatlakozó: A 2. csatlakozó automatikusan „Ne használja” értékre van állítva
  * Egy kamera + két csatlakozó: A 2. csatlakozó automatikusan „Ne használja” értékre van állítva
  * Több kamera: Kézi kiválasztás szükséges
* **Megjegyzés**: Ugyanaz a kamera nem rendelhető hozzá egyszerre az 1. és a 2. csatlakozóhoz.

***

## DAQ fényérzékelő

Ez a szakasz a Projektbeállítások menüpontban jelenik meg, és felsorolja a projektben található összes DAQ lefelé irányuló fájlt — az `.daq` felvételeket és a DAQ-M `.csv` lefelé irányuló napfény-naplók. A „Fényérzékelők” fülön készített felvételek automatikusan hozzáadódnak a megnyitott projekthez.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->



Minden sor tartalmazza a fájlt, az érzékelő modelljét és az adott fájlra ténylegesen érvényes diffúzor-sapka korrekciót. A sorok felett egy, a teljes projektre kiterjedő vezérlőelem található:

### Sapka felülírása (minden fájl)

* **Típus**: Legördülő menü
* **Beállítások**: `Auto`, valamint a projektben szereplő érzékelőtípusokra érvényes fedélkorrekciós profilok
* **Alapértelmezett**: Automatikus
* **Mentés neve**: `Processing → "DAQ cap id"` (alapértelmezett: `auto`)
* **Leírás**: Az `Auto` az egyes fájlokban rögzített fedelet használja (ha nincs rögzítve semmi, akkor a Sunshine fedelet veszi alapul, de a DAQ-U, DAQ-M és DAQ-E készülékek fedél nélkül kerülnek forgalomba, és az Chloros nem tudja megállapítani, hogy mi van felszerelve, ezért a feldolgozás figyelmeztet a feltételezett korlátra: válassza ki a ténylegesen felszerelt korlátot). Egy konkrét korlát kiválasztása felülírja a projektben található **minden** lefelé irányuló fájlt, kivéve a DAQ-E-S-ét, amelynek diffúzora a kalibrációjának része; egy kizárólag DAQ-E-S-ből álló projektben nem jelenik meg legördülő menü: a nyers felvételeket ezzel korrigálják, azok a felvételek pedig, amelyek már tartalmaznak fedelet, újra referenciálódnak (a rögzített korrekció visszavonásra kerül, és a kiválasztott kerül alkalmazásra).
* **Fontos**: A kiválasztott fedélnek meg kell egyeznie azzal a fedéllel, amelyet a felvétel során fizikailag felszereltek. Sem az érzékelő, sem a szoftver nem képes felismerni a fizikai fedelet — egy nem megfelelő fedél-azonosító helytelenül korrigálja a spektrumokat.

Szándékosan **csak egy**, a teljes projektre kiterjedő vezérlőelem van a fájlonkénti legördülő menük helyett: a beállítás a projekt minden lefelé irányuló forrására kiterjed.

***

## Tömb-igazítás

Ez a szakasz **csak** akkor jelenik meg, ha a projektben legalább egy kép tartalmazza azt a modulok közötti igazítási transzformációt, amelyet a LATTICE-tömbök a rögzítéskor jelölnek meg (XMP `Chloros:Alignment*` címkék). Megmutatja, hány kép tartalmaz igazítási címkéket, melyik kamera a referencia (`REF` jelölés), valamint kameránkénti táblázatot a képek számáról.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Tömbigazítás alkalmazása

* **Típus**: Jelölőnégyzet
* **Alapértelmezett**: Engedélyezve (bejelölve)
* **Tárolás**: `Processing → "Array alignment"`
* **Leírás**: Minden feldolgozott terméket (debayered / előnézet / sugárzás / visszaverődés / index) a rögzítéskor rögzített transzformáció segítségével a mátrix közös referenciageometriájába illeszt. Ki = exportálás az érzékelőnkénti natív geometriában.

### Kivágás a közös átfedésre

* **Típus**: Jelölőnégyzet (csak akkor aktív, ha az *Tömb igazítás alkalmazása* be van kapcsolva)
* **Alapértelmezett**: Engedélyezve (bejelölve)
* **Tárolás**: `Processing → "Array alignment crop"`
* **Leírás**: Az igazított exportokat a kameramodulok által közösen használt területre vágja, így minden sáv azonos méretű lesz. A kikapcsolt állapotban a teljes érzékelőterület megmarad (a forráson kívül fekete kitöltéssel).

### Újramintavétel

* **Típus**: Legördülő menü (csak akkor aktív, ha az *Tömb igazítás alkalmazása* be van kapcsolva)
* **Beállítások**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Alapértelmezett**: Bilineáris
* **Tárolás**: `Processing → "Array alignment interpolation"`
* **Leírás**: Az igazítási torzítás által használt interpoláció. A „Legközelebbi” opció megőrzi a forrás pontos értékeit (nincs pixelek közötti keveredés) a szigorú radiometrikus elemzéshez; a „Bilineáris” a legalkalmasabb térképezéshez és vizuális felhasználáshoz.

Ugyanezek a három beállítások „headless” formában is elérhetők: `chloros-cli process --array-alignment`, `--array-alignment-crop` és `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Index

Ezek a beállítások lehetővé teszik a multispektrális indexek konfigurálását elemzés és vizualizálás céljából.

### Index hozzáadása

* **Típus**: Speciális indexkonfigurációs panel
* **Leírás**: Megnyit egy interaktív panelt, ahol kiválaszthatja és konfigurálhatja a képfeldolgozás során kiszámítandó multispektrális vegetációs indexeket (NDVI, NDRE, EVI stb.). Több indexet is hozzáadhat, mindegyikhez saját megjelenítési beállításokkal.
* **Elérhető indexek**: A grafikus felület legördülő menüje**27** előre definiált multispektrális indexképletet tartalmaz (a teljes listát, beleértve azokat a neveket is, amelyeket az CLI/SDK `--indices` opció).
* **Funkciók**:
  * Válasszon az előre definiált indexképletek közül
  * Húzza a kamera szűrőcsatornáit a képlet sávhelyeire
  * Konfigurálja a megjelenítési színátmeneteket (LUT – Look-Up Tables)
  * Állítsa be a küszöbértékeket és a levágási módokat
  * Hozzon létre egyéni indexképleteket
* **Megjegyzés**: Az egycsatornás LATTICE M3M monokamerák esetében az indexek nem kerülnek kiszámításra — a többsávos indexek egy csatornán nem definiáltak. A Survey3 és a LATTICE M3C esetében ez nem érintett.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->



Minden hozzáadott index a képletét matematikai kifejezésként jeleníti meg, sávhelyenként egy színes körrel: piros = Red, zöld = Green, kék = Blue, narancs = Orange, cián = Cyan, lila = NIR, magenta = RE. Húzzon egy kört a képlet feletti sorból egy helyre a hozzárendeléshez; kattintson duplán egy hozzárendelt helyre annak törléséhez. Az index csak akkor számolódik ki, ha a képlet által használt minden helyhez tartozik csatorna.

### Egyéni képletek (Chloros+ funkció)

* **Típus**: Egyéni képletdefiníciók tömbje
* **Elérhetőség**: Megfelelő Chloros+ előfizetéssel való bejelentkezés szükséges.
* **Leírás**: Lehetővé teszi egyéni multispektrális indexképletek létrehozását és mentését sávszámítás segítségével. Az egyéni képleteket a projektbeállításokkal együtt menti a rendszer, és ugyanúgy használhatók, mint a beépített indexek.
* **Létrehozás**:
  1. Az Index konfigurációs panelen nyissa meg az egyéni képlet-kalkulátort
  2. Írja be a képletet a **sáv-slot szimbólumok** használatával, ne a sávnevekkel
  3. Mentse el a képletet egy leíró névvel — ezután megjelenik a képlet legördülő menü alján, és a kamera csatornaköröket pontosan úgy húzhatja a helyükre, mint egy beépített előre beállított érték esetén
* **Képlet szintaxisa**:
  * Sávhelyek: `x`, `y`, `z`, `a`, `b`, `c` — hat pozíció, amelyeket húzással rendelhet a valódi csatornákhoz
  * Operátorok: `+`, `-`, `*`, `/`, `^` és `()` csoportosításhoz
  * Funkciók: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Miért szimbólumok, és nem zenekarnevek?**: az `(y-x)/(y+x)` formában megadott képlet bármelyik kamerán működik, mivel a drag-and-drop leképezés határozza meg, hogy az `y` egy RGN szűrő 850 nm-es NIR-je-e, vagy egy OCN szűrő 808 nm-es NIR-je-e. A beépített előre beállított értékek is ugyanúgy vannak tárolva — az összes 27 szimbólum pontos formáját lásd a [Multispektrális indexképletek](multispectral-index-formulas.md) oldalon.
* **Hol használhatók**: az egyéni képleteket a projektbeállításokkal együtt menti a rendszer, és felhasználhatók az [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) környezetben, valamint a feldolgozás során is. Az CLI/SDK `--indices` névlista **nem** fogadja el őket, mivel az csak a 22 beépített előre beállított név kiterjesztését tartalmazza.

***

## Exportálás

Ezek a beállítások szabályozzák az exportált, feldolgozott képek formátumát és minőségét.

### Kalibrált képformátum

* **Típus**: Legördülő menü
* **Beállítások**:
  * **TIFF (16-bit)** – Tömörítetlen 16-bites TIFF formátum
  * **TIFF (8 bites)** – Tömörítetlen 8 bites TIFF formátum
  * **TIFF (32 bites, százalék)** – 32 bites lebegőpontos TIFF, százalékos fényvisszaverődési értékekkel
  * **PNG (8 bites)** – Tömörített 8 bites PNG formátum
  * **JPG (8-bit)** – Tömörített 8-bites JPEG formátum
* **Alapértelmezett**: TIFF (16-bit); egy teljes egészében 8-bites projekt esetén a TIFF (8-bit) jelenik meg, és csak a 8-bites formátumok közül lehet választani
* **Leírás**: Kiválasztja a feldolgozott és kalibrált képek mentéséhez használt fájlformátumot. Az exportált fájlok az egyes kamerák mappáin belül, formátumonkénti almappákba kerülnek (`tiff16`, `tiff32`, `png8`, `jpg8`), termékenként egy `<Product>_Images/` mappával. Az exportált fájlok megtartják a forrásfájl nevét — a terméket a mappa, nem pedig a fájlnév kiterjesztése azonosítja.
* **Formátumajánlások**:
  * **TIFF (16 bites)**: Tudományos elemzésekhez és professzionális munkafolyamatokhoz ajánlott. Tömörítési torzulások nélkül megőrzi a maximális adatminőséget. Legalkalmasabb multispektrális elemzésekhez és a GIS-szoftverekben történő további feldolgozáshoz.
  * **TIFF (32-bites, százalékos)**: Legalkalmasabb olyan munkafolyamatokhoz, amelyeknél a reflexiós értékeket százalékban (0–100%) kell megadni. Maximális pontosságot biztosít a radiometrikus mérésekhez.
  * **PNG (8 bites)**: Alkalmas webes megtekintésre és általános vizualizációra. Kisebb fájlméret veszteségmentes tömörítéssel, de csökkentett dinamikatartománnyal.
  * **JPG (8 bites)**: A legkisebb fájlméret, kizárólag előnézetekhez és webes megjelenítéshez ideális. Veszteséges tömörítést használ, ami nem alkalmas tudományos elemzésekhez.
* **Megjegyzés**: A LATTICE sugárzási értékeit e beállítástól függetlenül mindig 32 bites lebegőpontos TIFF formátumban exportálja a rendszer.

***

## Projekt sablon mentése

Ez a funkció lehetővé teszi az aktuális projektbeállítások újrafelhasználható sablonként való mentését.

* **Típus**: Szövegbeviteli mező + Mentés gomb
* **Leírás**: Adjon meg egy leíró nevet a beállítási sablonhoz, majd kattintson a mentés ikonra. A sablon az összes aktuális projektbeállítást (célfelismerés, feldolgozási opciók, indexek, és exportformátumot) a jövőbeli projektekben való egyszerű újrafelhasználás érdekében. A sablonok a projekt mentési mappáján belüli `Project Templates/` mappában kerülnek tárolásra, és a főmenüből is kiválaszthatók vagy exportálhatók (*Sablon kiválasztása* / *Sablon mentése* / *Sablon exportálása*).
* **Alkalmazási példák**:
  * Sablonok létrehozása különböző kamerarendszerekhez (RGB, multispektrális, NIR)
  * Szabványos konfigurációk mentése meghatározott növénytípusokhoz vagy elemzési munkafolyamatokhoz
  * Egységes beállítások megosztása a csapat tagjai között
* **Használati útmutató**:
  1. Állítsa be az összes kívánt projektbeállítást
  2. Adjon meg egy sablonnevet (pl. „RedEdge Survey3 NDVI Standard”)
  3. Kattintson a mentés ikonra
  4. A sablon mostantól betölthető új projektek létrehozásakor

***

## Projektmappa mentése

Ez a beállítás határozza meg, hogy az új projektek alapértelmezés szerint hova kerüljenek mentésre.

* **Típus**: Könyvtárútvonal megjelenítése + Szerkesztés gomb
* **Alapértelmezett (Windows)**: `Documents\MAPIR\Chloros Projects` a Dokumentumok mappában — általában `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`, vagy ahová a OneDrive vagy a vándorló profil áthelyezte a Dokumentumokat
* **Alapértelmezett (Linux)**: `~/Documents/MAPIR/Chloros Projects`
* **Leírás**: Megjeleníti az aktuális alapértelmezett könyvtárat, ahol az új Chloros-projektek létrehozásra kerülnek. Kattintson a szerkesztés ikonra egy másik könyvtár kiválasztásához. A felülírás egyetlen szövegsor formájában kerül tárolásra az `~/.chloros/working_directory.txt` fájlban — Windows rendszeren ez az `C:\Users\<Username>\.chloros\working_directory.txt`. Ha ez a fájl hiányzik, vagy olyan elérési utat nevez meg, amely már nem létezik, a Chloros a fenti alapértelmezett értékre tér vissza. A CLI ugyanazt a fájlt olvassa és írja, így az `chloros-cli` és a grafikus felület mindig egyetért abban, hogy hol találhatók a projektek.
* **A projektsablonok** ebben a könyvtárban található `Project Templates/` almappában találhatók.
* **Mikor érdemes módosítani**:
  * Állítsa be hálózati meghajtóra a csapatmunkához
  * Váltson nagyobb tárhelyű meghajtóra nagy adathalmazok esetén
  * Rendezze a projekteket év, ügyfél vagy projekttípus szerint különböző mappákba
* **Megjegyzés**: A beállítás módosítása csak az ÚJ projekteket érinti. A meglévő projektek az eredeti helyükön maradnak.
* **Windows-útvonal hossza**: amennyiben a Windows hosszú útvonalai nincsenek bekapcsolva (a telepítő **Windows hosszú útvonalak engedélyezése** opciója), a feldolgozás előtt elutasításra kerül minden olyan projektmappa, amelynek neve meghaladja a 130 karaktert, mivel az így létrejövő fájlok túllépnék a Windows 260 karakteres útvonalhossz-korlátját. Lásd: [A feldolgozás elindítása](../processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

***

## A beállítások megőrzése

Egy Chloros-projekt egy **mappa**. Az összes projektbeállítás az abban található `project.json` fájlban kerül mentésre; a csatlakoztatott hardverek adatait pedig az `cameras.json` és az `sensors.json` mappákban tárolja, így a projekt újbóli megnyitásakor a kamerák és a fényérzékelők is újra csatlakoznak. A projekt újbóli megnyitásakor az összes beállítás pontosan úgy áll vissza, ahogyan hagyta őket. A mentett projekteket a `chloros-cli project` fájl vagy az SDK `open_project` fájlja segítségével vezető nélkül is vezérelhetők.

### A beállítások hierarchiája

A beállítások a következő sorrendben kerülnek alkalmazásra:

1. **Rendszer alapértelmezések** – A Chloros által meghatározott beépített alapértelmezések 2. **Sablonbeállítások** – Ha sablont tölt be a projekt létrehozásakor
3. **Mentett projektbeállítások** – A projektfájllal együtt mentett beállítások
4. **Kézi módosítások** – Az aktuális munkamenet során végzett bármilyen változtatás

### Beállítások és képfeldolgozás

A feldolgozási beállításokat a feldolgozás elindításakor olvassa be a rendszer. A beállítások módosítása nem érinti visszamenőlegesen a lemezen már meglévő eredményeket – a új beállítások alkalmazásához futtassa újra a feldolgozást. Néhány beállítás egyáltalán nem befolyásolja a feldolgozást:

* Képminiatűr felbontása (csak megjelenítésre szolgál)
* Projektsablon mentése
* Projektmappa mentése

***

## Konfigurációs kulcsok áttekintése

Automatizáláshoz (CLI: `--config`, SDK: `configure`, vagy az `project.json` fájl közvetlen olvasásához) az alábbiak az `Project Settings` fájlban található pontos kulcsok:

| Kulcs elérési útja | Típus | Alapértelmezett |
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
| `Processing → Minimum recalibration interval` | szám 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | szám -12..12 | `0` |
| `Processing → Apply PPK corrections` | bool | `false` |
| `Processing → DAQ cap id` | korlátprofil-azonosító vagy `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | indexkonfigurációk listája | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (8-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

Az `Array alignment` kulcsok akkor kerülnek beírásra, amikor az Array Alignment szakasz először renderel, vagy egy automatizálási hívás beállítja őket. Amíg ezek hiányoznak, a feldolgozási folyamat a fentiekben látható értékeket használja (`true`, `true`, bilineáris), így egy projekt.json ezek nélkül pontosan ugyanúgy viselkedik, mint egy ilyenekkel rendelkező.

### Az `project.json`-ben tárolt kulcsok, amelyekhez nincs vezérlőelem a beállítási panelen

Ezek ugyanazon az `Project Settings` fa alatt találhatók, és a feldolgozás során beolvasásra kerülnek, de az oldalsávban nem találsz hozzájuk tartozó widgetet:

| Kulcsútvonal | Típus | Alapértelmezett | Beállítja |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Felülírja a LATTICE bemeneti TIFF-fájlok értelmezését; az `auto` az egyes fájlok `Chloros:ProcessingLevel` XMP-címkéjéből és a csatornák számából következtet. A Survey3-as `.raw` felvételek esetében figyelmen kívül hagyja. Szándékosan nem GUI-beállítás — az „auto” érték minden normál esetben helyes. |
| `Processing → Target reflectance dir` | elérési út karakterlánc | `""` | `chloros-cli process --target-reflectance-dir`, vagy a projekt cél-API-ja |
| `Processing → Target reflectance config` | a kamera sorozatszámával indexelt szótár | `{}` | Képkockán belüli célpont regisztrálása (mód: `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | elérési út karakterlánc | `""` | SDK `process_folder(daq_log_path=…)`. Egy `.daq` felvételre vagy azok mappájára mutat |
| `Target Detection → Minimum calibration target squares` | szám | `4` | Régi alapértelmezés; nincs vezérlőelem és nincs CLI-jelző |
| `UI → Grid thumbnail size` | szám | `160` | A képrajz saját miniatűr-nagyító csúszkája |

Két nézői beállítás **a legfelső szinten, az `project.json`-ben** van tárolva, teljesen az `Project Settings`-en kívül, mivel ezek inkább megjelenítési állapotok, mint feldolgozási beállítások:

| Kulcsút | Típus | Alapértelmezett | Beállítja |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | 1–256 közötti egész szám | `1` | A képfül GSD (px) vezérlője — lásd [Kép teljes képernyős megnyitása](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Bevált gyakorlatok

1. **Kezdje az alapértelmezett beállításokkal**: Az alapértelmezett beállítások a legtöbb MAPIR kamerarendszer és a tipikus munkafolyamatok esetében jól működnek.
2. **Készítsen sablonokat**: Miután optimalizálta a beállításokat egy adott munkafolyamat vagy kamera számára, mentse el őket sablonként, hogy biztosítsa a projektek közötti következetességet.
3. **Teszteljen a teljes feldolgozás előtt**: Ha új beállításokkal kísérletezik, tesztelje azokat a képek egy kis részhalmazán, mielőtt a teljes adatkészletet feldolgozná.
4. **Dokumentálja a beállításait**: Használjon olyan leíró sablonneveket, amelyek jelzik a kamerarendszert, a feldolgozás típusát és a tervezett felhasználást (pl. „Survey3\_RGB\_NDVI\_Agriculture”).
5. **Exportformátum kiválasztása**: Válassza ki az exportformátumot a végfelhasználás alapján:
   * Tudományos elemzés → TIFF (16 bites vagy 32 bites)
   * GIS-feldolgozás → TIFF (16 bites)
   * Gyors megjelenítés → PNG (8 bites)
   * Webes megosztás → JPG (8 bites)

***

A Chloros multispektrális indexeiről további információkat a [Multispektrális indexek képletei](multispectral-index-formulas.md) oldalon talál.
