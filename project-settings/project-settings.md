# Projektbeállítások

A Projektbeállítások <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> oldalsáv az Chloros programban lehetővé teszi a képfeldolgozás, a kalibrációs célpontok felismerése, a multispektrális indexek kiszámítása és a projekt exportálási beállításainak konfigurálását. Ezek a beállítások a projekttel együtt kerülnek mentésre, és sablonként is elmenthetők, hogy több projektben is újra felhasználhatók legyenek.

## A Projektbeállítások eléréséhez

A Projektbeállítások eléréséhez:

1. Nyisson meg egy projektet az Chloros programban
2. Kattintson a **Projektbeállítások**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> fülre a bal oldali sávban
3. A beállítások panelen kategóriák szerint rendezve megjelennek az összes elérhető konfigurációs lehetőségek

***

## Célfelismerés

Ezek a beállítások szabályozzák, hogy az Chloros hogyan ismeri fel és dolgozza fel a képeken található kalibrációs célokat.

### Minimális kalibrációs minta terület (px)

* **Típus**: Szám
* **Tartomány**: 0–10 000 pixel
* **Alapértelmezett**: 25 pixel
* **Leírás**: Beállítja azt a minimális területet (pixelben), amely szükséges ahhoz, hogy egy észlelt régiót érvényes kalibrációs célpont mintának tekintsünk. A kisebb értékek kisebb célpontokat észlelnek, de növelhetik a téves pozitív eredmények számát. A nagyobb értékek nagyobb, egyértelműbb célterületeket igényelnek az észleléshez.
* **Mikor kell módosítani**:
  * Növelje, ha hamis észleléseket kap kis kép-artefaktumokon
  * Csökkentse, ha a kalibrációs célpontjai kicsinek tűnnek a képeken, és nem kerülnek észlelésre

### Minimális célpont-csoportosítás (0–100)

* **Típus**: Szám
* **Tartomány**: 0–100
* **Alapértelmezett**: 60
* **Leírás**: Szabályozza a hasonló színű régiók csoportosításának küszöbértékét a kalibrációs célpontok észlelésekor. A magasabb értékek több hasonló szín csoportosítását igénylik, ami konzervatívabb célpont-észlelést eredményez. Az alacsonyabb értékek több színváltozatot engedélyeznek egy célcsoporton belül.
* **Mikor kell módosítani**:
  * Növelje, ha a kalibrációs célpontok több felismerésre oszlanak
  * Csökkentse, ha a színváltozással rendelkező kalibrációs célpontok nem kerülnek teljes mértékben felismerésre

***

## Feldolgozás

Ezek a beállítások szabályozzák, hogy az Chloros hogyan dolgozza fel és kalibrálja a képeit.

### Vignett-korrekció

* **Típus**: Jelölőnégyzet
* **Alapértelmezett**: Engedélyezve (bejelölve)
* **Leírás**: Vignettkorrekciót alkalmaz a képek széleinél jelentkező lencse sötétedés kompenzálására. A vignettálás egy gyakori optikai jelenség, amelynek során a kép sarkai és szélei a lencse jellemzői miatt sötétebbnek tűnnek, mint a közepe.
* **Mikor kell letiltani**: Csak akkor tiltsa le, ha a fényképezőgép/objektív kombinációja már alkalmazott vignettkorrekciót, vagy ha a vignettálást utómunkálatok során kézzel szeretné korrigálni.

### Reflektancia-kalibrálás / fehéregyensúly

* **Típus**: Jelölőnégyzet
* **Alapértelmezett**: Engedélyezve (bejelölve)
* **Leírás**: Engedélyezi az automatikus reflektancia-kalibrálást a képeken észlelt kalibrációs célpontok felhasználásával. Ez normalizálja a reflektanciaértékeket az adatkészletben, és biztosítja a mérési eredmények konzisztenciáját a fényviszonyoktól függetlenül.
* **Mikor kell letiltani**: Csak akkor tiltsa le, ha nyers, nem kalibrált képeket szeretne feldolgozni, vagy ha más kalibrációs munkafolyamatot használ.

### Debayer-módszer

* **Típus**: Legördülő menü
* **Opciók**:
  * Standard (Gyors, Közepes minőség)
  * Textúraérzékeny (Lassú, Legmagasabb minőség) \[Chloros+]
* **Alapértelmezett**: Standard (Gyors, Közepes minőség)
* **Leírás**: Kiválasztja a nyers Bayer-mintázatú érzékelőadatok teljes színes képekké történő konvertálásához használt demosaicing algoritmust. A „Standard (Gyors, Közepes minőség)” módszer optimális egyensúlyt biztosít a feldolgozási sebesség és a képminőség között. A „Textúraérzékeny (Lassú, Legmagasabb minőség)” \[Chloros+] egy kiváló minőségű, élekkel számoló demosaicing algoritmust használ, amelyet egy AI/ML zajszűrő modellel kombinálnak, amely szinte az összes demosaicing zajt eltávolítja. A Texture Aware modell futtatásához GPU memória (VRAM) szükséges. A gyorsabb feldolgozás érdekében azt javasoljuk, hogy akkor használja, ha &gt;4 GB VRAM áll rendelkezésre.
* **Megjegyzés**: Az Chloros jövőbeli verzióiban további debayer módszerek kerülhetnek bevezetésre.

### Minimális újrakalibrálási intervallum

* **Típus**: Szám
* **Tartomány**: 0–3600 másodperc
* **Alapértelmezett**: 0 másodperc
* **Leírás**: Beállítja a kalibrációs célpontok használata közötti minimális időintervallumot (másodpercben) a kalibrációs célpontok használata között. Ha 0-ra van állítva, az Chloros minden észlelt kalibrációs célpontot felhasznál. Ha magasabb értékre van állítva, az Chloros csak azokat a kalibrációs célpontokat használja, amelyek között legalább ennyi másodperc telt el, csökkentve ezzel a feldolgozási időt azoknál az adatkészleteknél, ahol gyakran rögzülnek kalibrációs célpontok.
* **Mikor kell beállítani**:
  * Állítsa 0-ra a maximális kalibrációs pontosság érdekében, ha a fényviszonyok változnak
  * Növelje (pl. 60–300 másodpercre) a gyorsabb feldolgozás érdekében, ha a fényviszonyok állandóak és gyakoriak a kalibrációs célpont-képek

### Fényérzékelő időzóna-eltolódás

* **Típus**: Szám
* **Tartomány**: -12 és +12 óra között
* **Alapértelmezett**: 0 óra
* **Leírás**: Meghatározza az időzóna eltolást (órában kifejezve az UTC-től) a fényérzékelő adatainak időbélyegeihez. Ezt a PPK (utólagosan feldolgozott kinematikus) adatfájlok feldolgozásakor használják a képfelvételek és a GPS-adatok közötti helyes időszinkronizálás biztosítására.
* **Mikor kell beállítani**: Állítsa be a helyi időzóna eltérését, ha a PPK-adatok UTC helyett helyi időt használnak. Például:
  * Csendes-óceáni idő: -8 vagy -7 (a nyári időszámítástól függően)
  * Keleti idő: -5 vagy -4 (a nyári időszámítástól függően)
  * Közép-európai idő: +1 vagy +2 (a nyári időszámítástól függően)

### PPK-korrekciók alkalmazása

* **Típus**: Jelölőnégyzet
* **Alapértelmezett**: Letiltva (nem jelölve)
* **Leírás**: Engedélyezi a GPS-t (GNSS) tartalmazó MAPIR DAQ-rögzítők utólagos kinematikai (PPK) korrekcióinak használatát. Ha engedélyezve van, az Chloros felhasználja a projektkönyvtárban található, expozíciós pin adatokat tartalmazó .daq naplófájlokat, és pontos földrajzi helymeghatározási korrekciókat alkalmaz a képeire.
* **Követelmény**: A projektkönyvtárban jelen kell lennie egy expozíciós pin bejegyzéseket tartalmazó .daq naplófájlnak
* **Mikor kell engedélyezni**: Javasolt mindig engedélyezni a PPK-korrekciót, ha a .daq naplófájlban vannak expozíciós visszacsatolási bejegyzések.

### 1. expozíciós pin

* **Típus**: Legördülő menü
* **Láthatóság**: Csak akkor látható, ha a „PPK-korrekciók alkalmazása” engedélyezve van, ÉS az 1. pinhez expozíciós adatok állnak rendelkezésre
* **Opciók**:
  * A projektben észlelt kameramodellek nevei
  * „Ne használja” – Ez az expozíciós pin figyelmen kívül hagyásra kerül
* **Alapértelmezett**: A projekt konfigurációja alapján automatikusan kiválasztásra kerül
* **Leírás**: Egy adott kamerát rendel az 1. expozíciós csatlakozóhoz a PPK időszinkronizáláshoz. Az expozíciós csatlakozó rögzíti a kamera zárjának pontos kioldási időpontját, ami elengedhetetlen a pontos PPK-helymeghatározáshoz.
* **Automatikus kiválasztás viselkedése**:
  * Egy kamera + egy csatlakozó: Automatikusan kiválasztja a kamerát
  * Egy kamera + két csatlakozó: Az 1. csatlakozó automatikusan a kamerához van rendelve
  * Több kamera: Kézi kiválasztás szükséges

### 2. expozíciós pin

* **Típus**: Legördülő menüből választható
* **Láthatóság**: Csak akkor látható, ha a „PPK-korrekciók alkalmazása” be van kapcsolva, ÉS a 2. pinhez expozíciós adatok állnak rendelkezésre
* **Opciók**:
  * A projektben észlelt kameramodellek nevei
  * „Ne használja” – Ez az expozíciós pin figyelmen kívül marad
* **Alapértelmezett**: A projekt konfigurációja alapján automatikusan kiválasztva
* **Leírás**: Kétkamerás felállítás használata esetén egy adott kamerát rendel az Expozíciós 2. csatlakozóhoz a PPK időszinkronizáláshoz.
* **Automatikus kiválasztás viselkedése**:
  * Egy kamera + egy csatlakozó: A 2. csatlakozó automatikusan „Ne használja” értékre van állítva
  * Egy kamera + két csatlakozó: A 2. csatlakozó automatikusan „Ne használja” értékre áll
  * Több kamera: Kézi kiválasztás szükséges
* **Megjegyzés**: Ugyanaz a kamera nem rendelhető hozzá egyszerre az 1. és a 2. csatlakozóhoz.***

## Index

Ezek a beállítások lehetővé teszik a multispektrális indexek konfigurálását elemzés és vizualizálás céljából.

### Index hozzáadása

* **Típus**: Speciális indexkonfigurációs panel
* **Leírás**: Megnyit egy interaktív panelt, ahol kiválaszthatja és konfigurálhatja a multispektrális vegetációs indexeket (NDVI, NDRE, EVI stb.), amelyeket a képfeldolgozás során kiszámít. Több indexet is hozzáadhat, mindegyikhez saját megjelenítési beállításokkal.
* **Elérhető indexek**: A rendszer több mint 30 előre definiált multispektrális indexet tartalmaz, többek között:
  * NDVI (Normalizált növényzetindex)
  * NDRE (Normalizált különbség RedEdge)
  * EVI (Továbbfejlesztett növényzetindex)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * És még sok más (a teljes listát lásd a [Multispektrális indexképletek](multispectral-index-formulas.md) alatt)
* **Funkciók**:
  * Válasszon az előre definiált indexképletek közül
  * Konfigurálja a vizualizációs színátmeneteket (LUT – Look-Up Tables)
  * Állítsa be az elemzés küszöbértékeit
  * Hozzon létre egyéni indexképleteket

### Egyéni képletek (Chloros+ funkció)

* **Típus**: Egyéni képletdefiníciók tömbje
* **Leírás**: Lehetővé teszi egyéni multispektrális indexképletek létrehozását és mentését sávszámítások segítségével. Az egyéni képletek a projektbeállításokkal együtt kerülnek mentésre, és ugyanúgy használhatók, mint a beépített indexek.
* **Hogyan hozhatja létre**:
  1. Az Index konfigurációs panelen keresse meg az egyéni képlet opciót
  2. Határozza meg a képletet sávazonosítók használatával (pl. NIR, Red, Green, Blue)
  3. Mentse el a képletet egy leíró névvel
* **Képlet szintaxisa**: A szokásos matematikai műveletek támogatottak, beleértve:
  * Aritmetika: `+`, `-`, `*`, `/`
  * Zárójelek a műveletek sorrendjéhez
  * Sávhivatkozások: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Export

Ezek a beállítások szabályozzák az exportált, feldolgozott képek formátumát és minőségét.

### Kalibrált képformátum

* **Típus**: Legördülő menü
* **Opciók**:
  * **TIFF (16-bit)** - Tömörítetlen 16-bites TIFF formátum
  * **TIFF (32-bites, százalék)** - 32-bites lebegőpontos TIFF, a reflexiós értékek százalékban
  * **PNG (8 bites)** - Tömörített 8 bites PNG formátum
  * **JPG (8 bites)** - Tömörített 8 bites JPEG formátum
* **Alapértelmezett**: TIFF (16 bites)
* **Leírás**: Kiválasztja a feldolgozott és kalibrált képek mentéséhez használt fájlformátumot.
* **Formátumajánlások**:
  * **TIFF (16 bites)**: Tudományos elemzésekhez és professzionális munkafolyamatokhoz ajánlott. Megőrzi a maximális adatminőséget, tömörítési torzulások nélkül. Legalkalmasabb multispektrális elemzésekhez és további feldolgozáshoz GIS szoftverekben.
  * **TIFF (32 bites, százalékos)**: Legalkalmasabb olyan munkafolyamatokhoz, amelyeknél a visszaverődési értékeket százalékban (0–100%) kell megadni. Maximális pontosságot biztosít radiometrikus mérésekhez.
  * **PNG (8 bites)**: Webes megtekintéshez és általános vizualizáláshoz alkalmas. Kisebb fájlméret veszteségmentes tömörítéssel, de csökkentett dinamikatartománnyal.
  * **JPG (8 bites)**: A legkisebb fájlméret, kizárólag előnézetekhez és webes megjelenítéshez alkalmas. Veszteséges tömörítést használ, amely nem alkalmas tudományos elemzésekhez.***

## Projekt sablon mentése

Ez a funkció lehetővé teszi az aktuális projektbeállítások újrafelhasználható sablonként való mentését.

* **Típus**: Szövegbeviteli mező + Mentés gomb
* **Leírás**: Adjon meg egy leíró nevet a beállítási sablonjának, majd kattintson a mentés ikonra. A sablon elmenti az összes aktuális projektbeállítást (célfelismerés, feldolgozási opciók, indexek és exportformátum) a jövőbeli projektekben való egyszerű újrafelhasználás érdekében.
* **Alkalmazási példák**:
  * Sablonok létrehozása különböző kamerarendszerekhez (RGB, multispektrális, NIR)
  * Szabványos konfigurációk mentése meghatározott növénytípusokhoz vagy elemzési munkafolyamatokhoz
  * Egységes beállítások megosztása a csapat tagjai között
* **Használata**:
  1. Konfigurálja az összes kívánt projektbeállítást
  2. Adjon meg egy sablonnevet (pl. „RedEdge Survey3 NDVI Standard”)
  3. Kattintson a mentés ikonra
  4. A sablon mostantól betölthető új projektek létrehozásakor

***

## Projektmappa mentése

Ez a beállítás határozza meg, hogy az új projektek alapértelmezés szerint hova kerüljenek mentésre.

* **Típus**: Könyvtár elérési út megjelenítése + Szerkesztés gomb
* **Alapértelmezett (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Alapértelmezett (Linux)**: `~/.local/share/chloros/projects`
* **Leírás**: Megmutatja az aktuális alapértelmezett könyvtárat, ahol az új Chloros projektek létrehozásra kerülnek. Kattintson a szerkesztés ikonra egy másik könyvtár kiválasztásához.
* **Mikor érdemes megváltoztatni**:
  * Állítsa be hálózati meghajtóra a csapatmunkához
  * Váltson nagyobb tárhelyű meghajtóra nagy adathalmazok esetén
  * Rendezze a projekteket év, ügyfél vagy projekttípus szerint különböző mappákba
* **Megjegyzés**: A beállítás módosítása csak az ÚJ projekteket érinti. A meglévő projektek az eredeti helyükön maradnak.***

## Beállítások megőrzése

Minden projektbeállítás automatikusan elmentésre kerül a projektfájllal együtt (`.mapir` projektformátum). Amikor újra megnyit egy projektet, az összes beállítás pontosan úgy áll vissza, ahogyan hagyta.

### Beállítások hierarchiája

A beállítások a következő sorrendben kerülnek alkalmazásra:

1. **Rendszer alapértelmezések** - Az Chloros által meghatározott beépített alapértelmezések
2. **Sablonbeállítások** - Ha sablont tölt be projekt létrehozásakor
3. **Mentett projektbeállítások** – A projektfájllal együtt mentett beállítások
4. **Kézi módosítások** – Az aktuális munkamenet során végzett bármilyen változtatás

### Beállítások és képfeldolgozás

A legtöbb beállításváltoztatás (különösen a Feldolgozás és Exportálás kategóriákban) a képek újrafeldolgozását indítja el, hogy azok tükrözzék az új beállításokat. Néhány beállítás azonban „csak exportálásra vonatkozik”, és nem igényel azonnali újrafeldolgozást:

* Projekt sablon mentése
* Munkakönyvtár
* Kalibrált képformátum (exportáláskor érvényes)

***

## Bevált gyakorlatok

1. **Kezdje az alapértelmezett beállításokkal**: Az alapértelmezett beállítások a legtöbb MAPIR kamerarendszer és tipikus munkafolyamat esetében jól működnek.
2. **Készítsen sablonokat**: Miután optimalizálta a beállításokat egy adott munkafolyamat vagy kamera számára, mentse el őket sablonként, hogy biztosítsa a projektek közötti konzisztenciát.
3. **Teszteljen a teljes feldolgozás előtt**: Ha új beállításokkal kísérletezik, tesztelje azokat a képek egy kis részén, mielőtt a teljes adatkészletet feldolgozná.
4. **Dokumentálja a beállításait**: Használjon leíró sablonneveket, amelyek jelzik a kamerarendszert, a feldolgozás típusát és a tervezett felhasználást (pl. „Survey3\_RGB\_NDVI\_Agriculture”).
5. **Exportformátum kiválasztása**: Válassza ki az exportformátumot a végfelhasználás alapján:
   * Tudományos elemzés → TIFF (16-bites vagy 32-bites)
   * GIS-feldolgozás → TIFF (16-bites)
   * Gyors megjelenítés → PNG (8 bites)
   * Webes megosztás → JPG (8 bites)

***

Az Chloros multispektrális indexeiről további információkat az [Multispektrális indexképletek](multispectral-index-formulas.md) oldalon talál.
