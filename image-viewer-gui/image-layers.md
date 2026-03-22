# Képrétegek

Az Chloros Képmegjelenítő „Képrétegek” legördülő menüjével gyorsan válthat az azonos kép különböző verziói között – az eredeti felvételektől a feldolgozott reflektancia-kimenetekig és a számított indexképekig.

## Mik azok a kép rétegek?

Az Chloros programban a **rétegek** egy adott forrásképhez rendelkezésre álló különböző kép kimeneteket jelentenek. A képek feldolgozásakor az Chloros több változatot hoz létre:

* **Eredeti képek** (a fényképezőgép JPG és RAW fájljai)
* **Reflektancia-kalibrált** kimenetek (ha a reflektancia-kalibrálás engedélyezve volt)
* **Célképek** (ha a kép kalibrációs célokat tartalmaz)
* **Indexképek** (NDVI, NDRE, GNDVI stb., ha indexek voltak konfigurálva)

A Képmegjelenítő jobb felső sarkában található **Rétegválasztó legördülő menü** segítségével azonnal válthat ezek között a verziók között anélkül, hogy elhagyná a megjelenítőt.***

## Elérhető rétegtípusok

### JPG

* A fényképezőgépéről származó eredeti JPG előnézeti kép
* Minden kép esetében mindig elérhető
* Feldolgozatlan, a fényképezőgép által rögzített állapotban
* A leggyorsabban betöltődik és jelenik meg

**Mikor érdemes megtekinteni:**

* Az eredeti felvétel gyors előnézete
* A kép kompozíciójának és keretezésének ellenőrzése
* A felvétel minőségének ellenőrzése a feldolgozás előtt

### RAW (Eredeti)

* A fényképezőgép eredeti RAW szenzordatai
* Debayered, utómunkálatok nélkül
* Magasabb bitmélység, mint a JPG-nél (jellemzően 12-bites vagy 14-bites szenzordata)

**Mikor érdemes megtekinteni:**

* Az eredeti érzékelőadatok minőségének ellenőrzése
* Érzékelőproblémák vagy artefaktok ellenőrzése
* A feldolgozás előtti és utáni eredmények összehasonlítása

### RAW (Cél)

* Csak azoknál a képeknél jelenik meg, amelyeknél kalibrációs célokat azonosítottak
* Megjeleníti az eredeti RAW képet az észlelt céllal
* A cél észlelésének sikerességének ellenőrzésére szolgál

**Mikor érdemes megtekinteni:**

* A kalibrációs célok helyes észlelésének megerősítése
* A célkép minőségének ellenőrzése
* Hibaelhárítás kalibrációs problémák esetén

{% hint style="info" %}
**Célréteg**: Ez a réteg csak a kalibrációs célokat tartalmazó képek legördülő menüjében jelenik meg. A szokásos felvételeknél ez az opció nem elérhető.
{% endhint %}

### RAW (reflektancia)

* A kalibrált reflektancia kimeneti kép
* Vignettáláskorrekcióval (ha a feldolgozás során engedélyezve van)
* A céladatok felhasználásával kalibrált reflektancia (ha engedélyezve van)
* Többsávos TIFF az összes kameracsatornával
* A pixelértékek a reflektancia százalékát jelzik (százalékos mód használata esetén)
* Készen áll a [Index/LUT Sandbox](index-lut-sandbox.md) segítségével történő manipulálásra

**Mikor érdemes megtekinteni:**

* A kalibrált eredmények ellenőrzése
* A kalibrálás minőségének ellenőrzése
* A pixelértékek tudományos pontosságának ellenőrzése
* Összehasonlítás az eredetivel a kalibrálás hatásainak megtekintése érdekében

{% hint style="success" %}
**Ajánlott**: Használja a RAW (reflektancia) réteget a pixelértékek tudományos mérések és elemzések céljából történő ellenőrzésekor.
{% endhint %}

### RAW (NDVI Index)... és hasonló

* Számított vegetációs index kép (ebben a példában NDVI)
* Az index neve attól függően változik, hogy melyik indexet konfigurálták a feldolgozás során
* Példák: RAW (NDVI Index), RAW (NDRE Index), RAW (GNDVI Index) stb.
* Az indexszámítás eredményeit bemutató, egycsatornás szürkeárnyalatos kép
* A Projektbeállításokban konfigurált minden indexhez egy réteg jelenik meg

**Lehetséges indexnevek:**

* RAW (NDVI Index)
* RAW (NDRE Index)
* RAW (GNDVI Index)
* RAW (OSAVI Index)
* RAW (EVI Index)
* RAW (SAVI Index)
* És még sok más... (lásd [Multispektrális indexképletek](../project-settings/multispectral-index-formulas.md))

**Mikor érdemes megtekinteni:**

* Az indexszámítás eredményeinek vizsgálata
* Az indexérték-tartományok ellenőrzése
* A érdeklődésre számot tartó területek azonosítása
* Az indexképek ellenőrzése GIS-ben vagy elemzésben való felhasználás előtt

***

## A rétegválasztó használata

### A legördülő menü megnyitása

1. Nyisson meg egy képet teljes képernyős módban (kattintson bármelyik miniatűrre a Képmegjelenítőben)
2. Keresse meg a **réteg legördülő menüt** a néző jobb felső sarkában
3. A legördülő menü az aktuálisan kiválasztott réteget mutatja (pl. „JPG”)
4. Kattintson a legördülő menüre az összes elérhető réteg megtekintéséhez

### Rétegek közötti váltás

1. Kattintson a réteg legördülő menüre a lista megnyitásához
2. Megjelennek az aktuális képhez elérhető összes rétegek
3. Kattintson bármelyik réteg nevére az adott verzióra való váltáshoz
4. A kép azonnal frissül, és a kiválasztott réteget jeleníti meg

**Gyors váltás:**

* A legördülő menü megjegyzi az utolsó kiválasztást
* A következő képre lépéskor az Chloros megpróbálja ugyanazt a rétegtípust megjeleníteni
* Ha az a réteg nem létezik a következő képen, akkor alapértelmezésként a JPG jelenik meg

### Rétegek elérhetősége

Nem minden réteg érhető el minden képhez:

**Mindig elérhető:*** ✅ JPG (minden képhez tartozik JPG-előnézet)

**Feltételesen elérhető:**

* ⚠️ RAW (Eredeti) – Csak akkor, ha a képet RAW vagy RAW+JPG módban rögzítették
* ⚠️ RAW (Cél) – Csak akkor, ha a kép felismert kalibrációs célpontokat tartalmaz
* ⚠️ RAW (Reflektancia) – Csak reflektancia-kalibrációval történő feldolgozás után
* ⚠️ RAW (\[Index] Index) – Csak indexek konfigurálásával történő feldolgozás után

***

## Rétegek megőrzése

### Képek közötti navigálás

Ha egy másik képre vált (a nyílgombokkal vagy a miniatűrökre kattintva):**A rétegbeállítások megmaradnak:**

* Ha „RAW (Reflectance)” nézetet tekint meg, a következő kép „RAW (Reflectance)” nézetet jelenít meg (ha elérhető)
* Ha „RAW (NDVI Index)” nézetet tekint meg, a következő kép „RAW (NDVI Index)” nézetet jelenít meg (ha elérhető)
* Ha ugyanaz a réteg nem létezik, alapértelmezésként JPG jelenik meg

**Példa a munkafolyamatra:**

1. Nyissa meg az 1. képet, váltson át a RAW (NDVI Index) rétegre
2. Nyomja meg a → gombot a 2. kép megtekintéséhez
3. A 2. kép automatikusan a RAW (NDVI Index) réteget jeleníti meg
4. Folytassa a navigálást – az összes kép az NDVI réteget jeleníti meg
5. Nagyon hatékony az indexeredmények áttekintéséhez számos kép esetében

***

## Gyakori munkafolyamatok

### 1. munkafolyamat: Előtte/utána összehasonlítás

**Cél**: Az eredeti és a kalibrált kép összehasonlítása

1. Nyissa meg a feldolgozott képet a Képmegjelenítőben
2. Válassza ki a **RAW (Eredeti)** lehetőséget a legördülő menüből
3. Jegyezze meg a vignettálást és a nem kalibrált értékeket
4. Váltson át a **RAW (Reflektancia)** lehetőségre a legördülő menüből
5. Hasonlítsa össze – a vignettálás eltűnt, az értékek kalibrálva vannak

### 2. munkafolyamat: Index áttekintése

**Cél**: Az NDVI eredmények gyors áttekintése az adatkészleten belül

1. Nyissa meg az első feldolgozott képet
2. Válassza ki a legördülő menüből a **RAW (NDVI Index)** lehetőséget
3. A → nyílgombbal lépjen a következő képre
4. Az NDVI réteg automatikusan megmarad
5. Folytassa az összes kép áttekintését, ellenőrizve az NDVI mintákat
6. Váltson át a **RAW (NDRE Index)**-re az összehasonlításhoz

### 3. munkafolyamat: Célpont-ellenőrzés

**Cél**: Ellenőrizze, hogy az összes célképet helyesen észlelte-e a rendszer

1. Keresse meg a célképet
2. Válassza ki a legördülő menüből a **RAW (Target)** elemet
3. Ellenőrizze, hogy a kalibrációs célpontok jól láthatóak és észlelhetők-e
4. Keresse meg a következő célképet
5. Ismételje meg az ellenőrzést az összes cél esetében

### 4. munkafolyamat: Pixelérték-ellenőrzés

**Cél**: Ellenőrizze a visszaverődési értékek tudományos pontosságát

1. Nyissa meg a feldolgozott képet
2. Válassza ki a **RAW (Reflectance)** réteget
3. Kapcsolja be a **Pixel Percent** módot (gomb a jobb felső eszköztáron)
4. Vigye a kurzort a növényzet területeire
5. Ellenőrizze, hogy a pixelértékek a várt tartományban vannak-e (30–70% az NIR esetében, 5–15% az Red esetében)
6. Ellenőrizze a talaj- és vízterületek megfelelő értékeit

***

## A pixelértékek megértése rétegenként

A különböző rétegek eltérő pixelérték-tartományokat mutatnak:

### JPG réteg

* **Tartomány**: 0–255 (8 bites)
* **Jelentés**: Megjelenítési értékek, gamma-korrigált
* **Használat**: Csak vizuális ellenőrzésre, nem tudományos mérésre

### RAW (Eredeti)

* **Tartomány**: 0–65535 (16 bites)
* **Jelentés**: Nyers érzékelői digitális értékek
* **Használat**: Az érzékelő teljesítményének ellenőrzése, nem kalibrált

### RAW (reflektancia)

* **Tartomány**: 0–65 535 (16 bites TIFF) vagy 0,0–1,0 (32 bites százalék)
* **Jelentés**: Kalibrált százalékos reflektancia
* **Használat**: Tudományos mérések és elemzések**16 bites TIFF esetén:**Oszd el 65 535-tel a százalékos reflektancia kiszámításához**32 bites százalékos értékek esetén:** Az értékek közvetlenül a százalékot jelzik (0,5 = 50% reflektancia)

### RAW (indexképek)

* **Tartomány**: Indexenként változik (normalizált indexek esetén általában -1,0 és +1,0 között)
* **Jelentés**: Indexszámítás eredménye
* **Példák**:
  * NDVI: -1 és +1 között (növényzet esetén általában 0,4 és 0,9 között)
  * NDRE: -1 és +1 között (stresszfelismerés)
  * EVI: 0 és 1 között (fokozott növényzet)

***

## Tippek és bevált gyakorlatok

### Hatékony rétegváltás

* **Billentyűparancsok**: Nincs billentyűparancs a rétegekhez, de a navigációs nyilak (←/→) minden rétegen működnek
* **Következetes munkafolyamatok**: Válasszon ki egy réteget (pl. NDVI), és tekintse át a teljes adatsort, mielőtt átváltana egy másikra
* **Gyors összehasonlítás**: Váltson az Eredeti és a Reflektancia rétegek között a feldolgozás minőségének ellenőrzéséhez

### Teljesítménybeli szempontok

* **A JPG betöltődik a leggyorsabban**: Használja a sok kép közötti gyors navigáláshoz
* **A RAW rétegek lassabban töltődnek be**: Magasabb felbontás és bitmélység
* **Index rétegek**: Hasonló sebesség, mint a Reflektancia rétegeknél
* **Az első betöltés a leglassabb**: Ugyanazon réteg későbbi megtekintései gyorsítótárba kerülnek, és gyorsabbak

### Minőségellenőrzés

* **Mindig ellenőrizze a RAW (Eredeti) réteget**: Ellenőrizze a forrásadatok minőségét, mielőtt megbízna a feldolgozott eredményekben
* **Hasonlítsa össze a rétegeket**: Használja a rétegváltást, hogy ellenőrizze, a feldolgozás megfelelően működött-e
* **Ellenőrizze az index tartományokat**: Használja a Pixel Percent módot az index rétegekkel, hogy ellenőrizze, az értékek ésszerűek-e***

## Hibaelhárítás

### A réteg nem elérhető

**Probléma**: A várt réteg nem jelenik meg a legördülő menüben**Lehetséges okok:**

* A kép nem került feldolgozásra (csak JPG és RAW (eredeti) áll rendelkezésre)
* A feldolgozás során a reflektancia-kalibrálás le volt tiltva
* Az adott index nem volt beállítva a Projektbeállításokban
* A kép kizárólag célpontot tartalmaz (a célpontokhoz nem generálódnak indexek)

**Megoldások:**

1. Ellenőrizze, hogy a kép feldolgozásra került-e (ellenőrizze a kimeneti mappát a feldolgozott fájlok után)
2. Ellenőrizze a Projektbeállításokat, hogy az indexek be vannak-e állítva
3. Dolgozza fel újra a kívánt indexek engedélyezésével

### Rossz réteg jelenik meg

**Probléma**: A kép váratlan rétegben nyílik meg**Ok**: Az előző kép rétegbeállítása átkerült, de az a réteg nem létezik a jelenlegi képen**Megoldás**: Az Chloros automatikusan JPG-re vált, ha a preferált réteg nem elérhető – ez normális viselkedés

### A kalibrációs célpontok nem láthatók

**Probléma**: A RAW (Célpont) réteg nem jeleníti meg a célpont-felismerést**Lehetséges okok:**

* A feldolgozás során nem sikerült felismerni a célpontokat
* A kép valójában nem tartalmaz célpontokat
* A célpontfelismerési beállítások túl szigorúak

**Megoldások:**

1. Ellenőrizze a hibakeresési naplóban, hogy szerepel-e „Célpont megtalálva” üzenet
2. Ellenőrizze, hogy a kép valóban tartalmaz-e látható kalibrációs célpontokat
3. Állítsa be a célpontfelismerési beállításokat a Projektbeállításokban
4. Lásd [Célképek kiválasztása](../processing-images-gui/choosing-target-images.md)

***

## Kapcsolódó funkciók

### Képmegjelenítő eszközök

Bármely réteg megtekintésekor a következőket használhatja:

* **Nagyítási vezérlők**: Nagyítson a részletek megtekintéséhez
* **Panorámázás**: Kattintson és húzza az egérrel a nagyított kép mozgatásához
* **Pixelesszám-ellenőrzés**: Az értékek megtekintése a kurzor helyén
* **Navigációs nyilak**: Váltson a képek között a réteg megőrzése mellett
* **Pixelszázalék mód**: Váltson a DN és a százalékos megjelenítés között

A Képmegjelenítő teljes dokumentációját lásd a [Kép teljes képernyős megnyitása](opening-an-image-full-screen.md) című részben.

### Index/LUT Sandbox

Interaktív index teszteléshez és vizualizáláshoz:

* **Valós idejű index számítás**: Különböző index képletek tesztelése
* **LUT színleképezés**: Színátmenetek alkalmazása szürkeárnyalatos indexekre
* **Vizualizációk exportálása**: Színes index képek mentése

A részleteket lásd az [Index/LUT Sandbox](index-lut-sandbox.md) oldalon.

***

## Következő lépések

Most, hogy már ismeri a képrétegeket:

* [**Kép teljes képernyős megnyitása**](opening-an-image-full-screen.md) – Teljes Image Viewer útmutató
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – Interaktív index-megjelenítés
* [**Multispektrális indexképletek**](../project-settings/multispectral-index-formulas.md) – A rendelkezésre álló indexek jegyzéke
* [**A feldolgozás befejezése**](../processing-images-gui/finishing-the-processing.md) – A feldolgozott kimenetek megértése
