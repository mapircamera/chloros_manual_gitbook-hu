# Projektbeállítások

A Projektbeállítások <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> oldalsáv lehetővé teszi a képfeldolgozás, a kalibrációs célpontok felismerése, a multispektrális indexek kiszámítása és a projekt exportálási beállításainak konfigurálását. Ezek a beállítások a projekttel együtt kerülnek mentésre, és sablonként is elmenthetők, hogy több projektben is újra felhasználhatók legyenek.

## A projektbeállítások eléréséhez

A projektbeállítások eléréséhez:

1. Nyisson meg egy projektet az Chloros programban
2. Kattintson a **Projektbeállítások**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> fülre a bal oldali sávban.
3. A beállítások panelen minden elérhető konfigurációs opció kategóriák szerint rendezve megjelenik.

***

## Célpontok felismerése

Ezek a beállítások szabályozzák, hogy az Chloros hogyan ismeri fel és dolgozza fel a képeken található kalibrációs célpontokat.

### Minimális kalibrációs minta terület (px)

* **Típus**: Szám
* **Tartomány**: 0–10 000 pixel
* **Alapértelmezett**: 25 pixel
* **Leírás**: Beállítja a minimális területet (pixelben), amely szükséges ahhoz, hogy egy észlelt régió érvényes kalibrációs célminta legyen. A kisebb értékek kisebb célokat észlelnek, de növelhetik a téves pozitív eredmények számát. A nagyobb értékek nagyobb, tisztább célrégiókat igényelnek az észleléshez.
* **Mikor kell módosítani**:
  * Növelje, ha hamis észleléseket kap kis képalkotó eszközökön.
  * Csökkentse, ha a kalibrációs célpontok kicsinek tűnnek a képeken, és nem kerülnek észlelésre.

### Minimális célpont-csoportosítás (0-100)

* **Típus**: Szám
* **Tartomány**: 0 és 100 között
* **Alapértelmezett**: 60
* **Leírás**: A kalibrációs célok észlelésekor a hasonló színű régiók csoportosításának küszöbértékét szabályozza. Magasabb értékek esetén több hasonló színű régiót kell csoportosítani, ami konzervatívabb célészlelést eredményez. Alacsonyabb értékek esetén több színváltozatot engedélyez a célcsoporton belül.
* **Mikor kell beállítani**:
  * Növelje, ha a kalibrációs célok több észlelésre oszlanak.
  * Csökkentse, ha a színváltozatokkal rendelkező kalibrációs célok nem kerülnek teljes mértékben észlelésre.

***

## Feldolgozás

Ezek a beállítások szabályozzák, hogy az Chloros hogyan dolgozza fel és kalibrálja a képeket.

### Vignettázás korrekció

* **Típus**: Jelölőnégyzet
* **Alapértelmezett**: Engedélyezve (bejelölve)
* **Leírás**: Vignettázás korrekciót alkalmaz a képek szélén fellépő lencse elsötétülés kompenzálására. A vignettálás egy gyakori optikai jelenség, amelynek során a kép sarkai és szélei a lencse tulajdonságai miatt sötétebbnek tűnnek, mint a közepe.
* **Mikor kell letiltani**: Csak akkor tiltsa le, ha a fényképezőgép/lencse kombinációja már alkalmazta a vignettakorrekciót, vagy ha a vignettálást utólagosan, manuálisan szeretné korrigálni.

### Reflektancia kalibrálás / fehéregyensúly

* **Típus**: Jelölőnégyzet
* **Alapértelmezett**: Engedélyezve (bejelölve)
* **Leírás**: Engedélyezi az automatikus reflektancia kalibrálást a képeken észlelt kalibrációs célpontok felhasználásával. Ez normalizálja a reflektancia értékeket az adatkészletben, és biztosítja a konzisztens méréseket a fényviszonyoktól függetlenül.
* **Mikor kell kikapcsolni**: Csak akkor kapcsolja ki, ha nyers, nem kalibrált képeket szeretne feldolgozni, vagy ha más kalibrációs munkafolyamatot használ.

### Debayer-módszer

* **Típus**: Legördülő menü
* **Opciók**:
  * Kiváló minőség (gyorsabb) – Jelenleg ez az egyetlen elérhető opció
* **Alapértelmezett**: Kiváló minőség (gyorsabb)
* **Leírás**: Kiválasztja a demosaicing algoritmust, amelyet a nyers Bayer-mintázatú érzékelőadatok teljes színes képekké történő konvertálásához használnak. A „Kiváló minőség (gyorsabb)” módszer optimális egyensúlyt biztosít a feldolgozási sebesség és a képminőség között.
* **Megjegyzés**: Az Chloros jövőbeli verzióiban további debayer módszerek is hozzáadódhatnak.

### Minimális újrakalibrálási intervallum

* **Típus**: Szám
* **Tartomány**: 0–3600 másodperc
* **Alapértelmezett**: 0 másodperc
* **Leírás**: Beállítja a kalibrációs célok használata közötti minimális időintervallumot (másodpercben). Ha 0-ra van állítva, az Chloros minden észlelt kalibrációs célt felhasznál. Ha magasabb értékre van állítva, az Chloros csak azokat a kalibrációs célokat használja, amelyek között legalább ennyi másodperc telt el, csökkentve ezzel a feldolgozási időt a gyakori kalibrációs célok rögzítésével rendelkező adatkészletek esetében.
* **Mikor kell beállítani**:
  * Állítsa 0-ra a maximális kalibrációs pontosság érdekében, ha a fényviszonyok változnak.
  * Növelje (pl. 60-300 másodpercre) a gyorsabb feldolgozás érdekében, ha a fényviszonyok állandóak és gyakori kalibrációs célképek állnak rendelkezésre.

### Fényérzékelő időzóna eltolása

* **Típus**: Szám
* **Tartomány**: -12 és +12 óra között
* **Alapértelmezett**: 0 óra
* **Leírás**: Meghatározza az időzóna eltolását (órákban az UTC-től) a fényérzékelő adatainak időbélyegeihez. Ezt a PPK (utólag feldolgozott kinematikus) adatfájlok feldolgozásakor használják, hogy biztosítsák a képek rögzítése és a GPS-adatok közötti pontos időszinkronizálást.
* **Mikor kell beállítani**: Állítsa be a helyi időzóna eltolását, ha a PPK-adatok UTC helyett helyi időt használnak. Például:
  * Csendes-óceáni idő: -8 vagy -7 (a nyári időszámítástól függően)
  * Keleti idő: -5 vagy -4 (a nyári időszámítástól függően)
  * Közép-európai idő: +1 vagy +2 (a nyári időszámítástól függően)

### PPK-korrekciók alkalmazása

* **Típus**: Jelölőnégyzet
* **Alapértelmezett**: Letiltva (jelölőnégyzet nincs bejelölve)
* **Leírás**: Engedélyezi a GPS-t (GNSS) tartalmazó MAPIR DAQ-felvevők utólagos kinematikus (PPK) korrekcióinak használatát. Ha engedélyezve van, az Chloros minden .daq naplófájlt felhasznál, amely expozíciós pin adatokat tartalmaz a projektkönyvtárában, és pontos földrajzi helymeghatározási korrekciókat alkalmaz a képeire.
* **Követelmény**: A projektkönyvtárában jelen kell lennie egy .daq naplófájlnak, amely expozíciós pin bejegyzéseket tartalmaz.
* **Mikor engedélyezze**: Javasoljuk, hogy mindig engedélyezze a PPK korrekciót, ha a .daq naplófájlban vannak expozíciós visszacsatolási bejegyzések.

### 1. expozíciós pin

* **Típus**: Legördülő menü
* **Láthatóság**: Csak akkor látható, ha a „PPK-korrekciók alkalmazása” engedélyezve van, ÉS a 1. pinhez expozíciós adatok állnak rendelkezésre.
* **Opciók**:
  * A projektben észlelt kameramodell nevek
  * „Ne használd” – Ez az expozíciós pin figyelmen kívül hagyható
* **Alapértelmezett**: A projekt konfigurációja alapján automatikusan kiválasztva
* **Leírás**: Egy adott kamerát rendel az 1. expozíciós csaphoz a PPK időszinkronizáláshoz. Az expozíciós csap rögzíti a kamera zárjának pontos időzítését, ami elengedhetetlen a pontos PPK földrajzi helymeghatározáshoz.
* **Automatikus kiválasztás**:
  * Egy kamera + egy csap: A kamera automatikus kiválasztása
  * Egy kamera + két csap: Az 1. csap automatikusan a kamerához van rendelve
  * Több kamera: Kézi kiválasztás szükséges

### 2. expozíciós pin

* **Típus**: Legördülő menüből választható
* **Láthatóság**: Csak akkor látható, ha a „PPK-korrekciók alkalmazása” be van kapcsolva, ÉS a 2. pinhez expozíciós adatok állnak rendelkezésre
* **Opciók**:
  * A projektben észlelt kameramodellek nevei
  * „Ne használd” – Ne vedd figyelembe ezt az expozíciós csapot
* **Alapértelmezett**: A projekt konfigurációja alapján automatikusan kiválasztva
* **Leírás**: Kettős kamera beállítás használata esetén egy adott kamerát rendel az expozíciós 2. csaphoz a PPK időszinkronizáláshoz.
* **Automatikus kiválasztás**:
  * Egy kamera + egy csap: A 2. csap automatikusan „Ne használd” beállításra kerül
  * Egy kamera + két pin: A 2. pin automatikusan „Ne használja” beállításra kerül.
  * Több kamera: Kézi kiválasztás szükséges.
* **Megjegyzés**: Ugyanaz a kamera nem rendelhető hozzá egyszerre a 1. és a 2. pinhez.

***

## Index

Ezekkel a beállításokkal konfigurálhatja a multispektrális indexeket az elemzéshez és a megjelenítéshez.

### Index hozzáadása

* **Típus**: Speciális index konfigurációs panel
* **Leírás**: Megnyit egy interaktív panelt, ahol kiválaszthatja és konfigurálhatja a multispektrális vegetációs indexeket (NDVI, NDRE, EVI stb.), amelyeket a képfeldolgozás során kell kiszámítani. Több indexet is hozzáadhat, mindegyikhez saját vizualizációs beállításokkal.
* **Elérhető indexek**: A rendszer több mint 30 előre definiált multispektrális indexet tartalmaz, többek között:
  * NDVI (normalizált növényzetindex)
  * NDRE (Normalizált különbség RedEdge)
  * EVI (Továbbfejlesztett vegetációs index)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * És még sok más (a teljes listaért lásd [Multispektrális index képletek](multispectral-index-formulas.md))
* **Jellemzők**:
  * Válasszon az előre definiált indexképletek közül
  * Konfigurálja a vizualizációs színátmenetek (LUT - Look-Up Tables)
  * Állítsa be az elemzés küszöbértékeit
  * Hozzon létre egyéni indexképleteket

### Egyéni képletek (Chloros+ funkció)

* **Típus**: Egyedi képletdefiníciók tömbje
* **Leírás**: Lehetővé teszi egyedi multispektrális indexképletek létrehozását és mentését sávmatematika segítségével. Az egyedi képletek a projektbeállításokkal együtt kerülnek mentésre, és ugyanúgy használhatók, mint a beépített indexek.
* **Létrehozás**:
  1. Az Index konfigurációs panelen keresse meg az egyéni képlet opciót.
  2. Határozza meg a képletet sávazonosítók segítségével (pl. NIR, Red, Green, Blue).
  3. Mentse el a képletet egy leíró névvel.
* **Képlet szintaxisa**: A szabványos matematikai műveletek támogatottak, beleértve:
  * Aritmetika: `+`, `-`, `*`, `/`
  * Számítások sorrendjét jelző zárójelek
  * Sávhivatkozások: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Exportálás

Ezek a beállítások szabályozzák az exportált feldolgozott képek formátumát és minőségét.

### Kalibrált képformátum

* **Típus**: Legördülő menü
* **Opciók**:
  * **TIFF (16 bites)** - Tömörítetlen 16 bites TIFF formátum
  * **TIFF (32 bites, százalék)** – 32 bites lebegőpontos TIFF, amelyben a visszaverődési értékek százalékban vannak megadva.
  * **PNG (8 bites)** - Tömörített 8 bites PNG formátum
  * **JPG (8 bites)** - Tömörített 8 bites JPEG formátum
* **Alapértelmezett**: TIFF (16 bites)
* **Leírás**: Kiválasztja a feldolgozott és kalibrált képek mentéséhez használt fájlformátumot.
* **Formátumajánlások**:
  * **TIFF (16 bites)**: Tudományos elemzésekhez és professzionális munkafolyamatokhoz ajánlott. Maximális adatminőséget biztosít tömörítési artefaktumok nélkül. Legalkalmasabb multispektrális elemzésekhez és további feldolgozáshoz GIS szoftverekben.
  * **TIFF (32 bites, százalékos)**: Legalkalmasabb olyan munkafolyamatokhoz, amelyek százalékos (0–100%) visszaverődési értékeket igényelnek. Maximális pontosságot biztosít radiometrikus mérésekhez.
  * **PNG (8 bites)**: Alkalmas webes megtekintéshez és általános megjelenítéshez. Kisebb fájlméret veszteségmentes tömörítéssel, de csökkentett dinamikatartománnyal.
  * **JPG (8 bites)**: A legkisebb fájlméret, leginkább előnézetekhez és webes megjelenítéshez alkalmas. Veszteséges tömörítést használ, amely nem alkalmas tudományos elemzésekhez.

***

## Projekt sablon mentése

Ez a funkció lehetővé teszi a jelenlegi projektbeállítások újrafelhasználható sablonként való mentését.

* **Típus**: Szövegbeviteli mező + Mentés gomb
* **Leírás**: Adjon meg egy leíró nevet a beállítási sablonhoz, majd kattintson a mentés ikonra. A sablon az aktuális projektbeállításokat (célfelismerés, feldolgozási opciók, indexek és exportformátum) tárolja, hogy azokat a jövőbeni projektekben könnyen újra felhasználhassa.
* **Használati esetek**:
  * Sablonok létrehozása különböző kamerarendszerekhez (RGB, multispektrális, NIR)
  * Standard konfigurációk mentése meghatározott növénytípusokhoz vagy elemzési munkafolyamatokhoz
  * Egységes beállítások megosztása a csapat tagjai között
* **Használata**:
  1. Konfigurálja az összes kívánt projektbeállítást
  2. Adjon meg egy sablonnevet (pl. „RedEdge Survey3 NDVI Standard”)
  3. Kattintson a mentés ikonra
  4. A sablon mostantól betölthető új projektek létrehozásakor

***

## Projektmappa mentése

Ez a beállítás határozza meg, hogy az új projektek alapértelmezés szerint hová kerülnek mentésre.

* **Típus**: Könyvtár elérési útjának megjelenítése + Szerkesztés gomb
* **Alapértelmezett**: `C:\Users\[Username]\Chloros Projects`
* **Leírás**: Megjeleníti az aktuális alapértelmezett könyvtárat, ahol az új Chloros projektek létrehozásra kerülnek. Kattintson a szerkesztés ikonra, hogy másik könyvtárat válasszon.
* **Mikor kell módosítani**:
  * Csapatmunka esetén állítsa be hálózati meghajtóra.
  * Nagy adatállományok esetén válasszon nagyobb tárhellyel rendelkező meghajtót.
  * Rendezze a projekteket év, ügyfél vagy projekt típus szerint különböző mappákba.
* **Megjegyzés**: A beállítás módosítása csak az ÚJ projekteket érinti. A meglévő projektek az eredeti helyükön maradnak.

***

## Beállítások megőrzése

Az összes projektbeállítás automatikusan elmentésre kerül a projektfájllal együtt (`.mapir` projektformátum). Amikor újra megnyit egy projektet, az összes beállítás pontosan úgy áll vissza, ahogyan hagyta.

### Beállítások hierarchiája

A beállítások a következő sorrendben kerülnek alkalmazásra:

1. **Rendszer alapértelmezett beállításai** – A Chloros által meghatározott beépített alapértelmezett beállítások
2. **Sablonbeállítások** – Ha sablont tölt be a projekt létrehozásakor
3. **Mentett projektbeállítások** – A projektfájllal együtt mentett beállítások
4. **Kézi módosítások** – A jelenlegi munkamenet során végzett bármilyen módosítás

### Beállítások és képfeldolgozás

A legtöbb beállításmódosítás (különösen a Feldolgozás és Export kategóriákban) a képek újrafeldolgozását indítja el, hogy azok tükrözzék az új beállításokat. Egyes beállítások azonban „csak exportálásra” vonatkoznak, és nem igényelnek azonnali újrafeldolgozást:

* Projekt sablon mentése
* Munkakönyvtár
* Kalibrált képformátum (exportáláskor alkalmazandó)

***

## Bevált gyakorlatok

1. **Kezdje az alapértelmezett beállításokkal**: Az alapértelmezett beállítások a legtöbb MAPIR kamerarendszer és tipikus munkafolyamat esetében jól működnek.
2. **Sablonok létrehozása**: Miután optimalizálta a beállításokat egy adott munkafolyamathoz vagy kamerához, mentse őket sablonként, hogy biztosítsa a projektek közötti konzisztenciát.
3. **Tesztelés a teljes feldolgozás előtt**: Új beállítások kipróbálásakor tesztelje azokat egy kis képkészleten, mielőtt feldolgozná a teljes adatkészletet.
4. **Dokumentálja a beállításokat**: Használjon leíró sablonneveket, amelyek jelzik a kamerarendszert, a feldolgozás típusát és a tervezett felhasználást (pl. „Survey3\_RGB\_NDVI\_Agriculture”).
5. **Exportformátum kiválasztása**: Válassza ki az exportformátumot a végső felhasználás alapján:
   * Tudományos elemzés → TIFF (16 bites vagy 32 bites)
   * GIS-feldolgozás → TIFF (16 bites)
   * Gyors megjelenítés → PNG (8 bites)
   * Webes megosztás → JPG (8 bites)

***

Az Chloros multispektrális indexeiről további információkat a [Multispektrális indexek képletei](multispectral-index-formulas.md) oldalon talál.
