# Képlapok

Az Chloros Image Viewer program Képlapok legördülő menüjével gyorsan válthat az ugyanazon kép különböző verziói között – az eredeti felvételektől a feldolgozott reflexiós kimenetekig és a számított indexképekig.

## Mik azok a képlapok?

Az Chloros programban a **rétegek** egy adott forrásképhez rendelkezésre álló különböző képkiadásokra utalnak. A képek feldolgozásakor az Chloros több változatot hoz létre:

* **Eredeti képek** (a fényképezőgép JPG és RAW fájljai)
* **Reflektancia-kalibrált** kimenetek (ha a reflektancia-kalibrálás engedélyezve volt)
* **Célképek** (ha a kép kalibrációs célokat tartalmaz)
* **Indexképek** (NDVI, NDRE, GNDVI stb., ha az indexek konfigurálva voltak)

A Képnézegető jobb felső sarkában található **Rétegválasztó legördülő menü** segítségével azonnal válthat ezek között a verziók között anélkül, hogy elhagyná a nézőt.

***

## Elérhető réteg típusok

### JPG

* A fényképezőgép eredeti JPG előnézeti képe
* Mindig elérhető minden képhez
* Feldolgozatlan, a fényképezőgép által rögzített formában
* A leggyorsabban betölthető és megjeleníthető

**Mikor érdemes megtekinteni:**

* Az eredeti felvétel gyors előnézete
* A kép kompozíciójának és keretezésének ellenőrzése
* A felvétel minőségének ellenőrzése a feldolgozás előtt

### RAW (eredeti)

* A fényképezőgép eredeti RAW érzékelő adatai
* Debayered, utólagos feldolgozás nélkül
* Magasabb bitmélység, mint a JPG (jellemzően 12 bites vagy 14 bites érzékelő adatok)

**Mikor érdemes megtekinteni:**

* Az eredeti érzékelő adatok minőségének ellenőrzése
* Érzékelő problémák vagy artefaktumok ellenőrzése
* A feldolgozás előtti és utáni eredmények összehasonlítása

### RAW (cél)

* Csak azoknál a képeknél jelenik meg, amelyek kalibrációs célpontokat tartalmaznak
* Megjeleníti az eredeti RAW képet a felismert célponttal
* A célpont felismerésének sikerességének ellenőrzésére szolgál

**Mikor érdemes megtekinteni:**

* A kalibrációs célpontok helyes felismerésének ellenőrzése
* A célpont képminőségének ellenőrzése
* Kalibrációs problémák elhárítása

{% hint style=&quot;info&quot; %}
**Célréteg**: Ez a réteg csak a kalibrációs célokat tartalmazó képek legördülő menüjében jelenik meg. A normál felvételeknél ez az opció nem elérhető.
{% endhint %}

### RAW (reflektancia)

* A kalibrált reflektancia kimeneti kép
* Vignettázás korrigálva (ha a feldolgozás során engedélyezve van)
* A céladatokkal kalibrált reflektancia (ha engedélyezve van)
* Többsávos TIFF az összes kamera csatornával
* A pixelértékek a reflektancia százalékát jelzik (ha százalékos módot használ)
* Készen áll a [Index/LUT Sandbox](index-lut-sandbox.md) segítségével történő manipulálásra

**Mikor érdemes megtekinteni:**

* Kalibrált eredmények ellenőrzése
* Kalibráció minőségének ellenőrzése
* Pixelértékek tudományos pontosságának ellenőrzése
* Összehasonlítás az eredetivel a kalibráció hatásának megtekintése érdekében

{% hint style=&quot;success&quot; %}
**Ajánlott**: Használjon RAW (Reflectance) réteget, ha pixelértékeket ellenőriz tudományos mérések és elemzések céljából.
{% endhint %}

### RAW (NDVI Index)... és hasonló

* Számított vegetációs index kép (ebben a példában NDVI)
* Az index neve attól függően változik, hogy melyik indexet konfigurálták a feldolgozás során
* Példák: RAW (NDVI Index), RAW (NDRE Index), RAW (GNDVI Index) stb.
* Az index számítási eredményeit bemutató egysávos szürkeárnyalatos kép
* A Projektbeállításokban konfigurált minden indexhez egy réteg jelenik meg

**Lehetséges indexnevek:**

* RAW (NDVI Index)
* RAW (NDRE Index)
* RAW (GNDVI Index)
* RAW (OSAVI index)
* RAW (EVI index)
* RAW (SAVI index)
* És még sok más... (lásd [Multispektrális index képletek](../project-settings/multispectral-index-formulas.md))

**Mikor érdemes megtekinteni:**

* Az index számítási eredményeinek vizsgálata
* Az indexértékek tartományainak ellenőrzése
* Az érdeklődésre számot tartó területek azonosítása
* Az indexképek ellenőrzése a GIS-ben vagy elemzésben való felhasználás előtt

***

## A rétegválasztó használata

### A legördülő menü megnyitása

1. Nyissa meg a képet teljes képernyős módban (kattintson bármelyik miniatűrre a Képmegjelenítőben)
2. Keresse meg a **réteg legördülő menüt** a néző jobb felső sarkában
3. A legördülő menü a jelenleg kiválasztott réteget mutatja (pl. „JPG”)
4. Kattintson a legördülő menüre az összes elérhető réteg megtekintéséhez

### Rétegek közötti váltás

1. Kattintson a réteg legördülő menüre a lista megnyitásához
2. A jelenlegi képhez elérhető összes réteg megjelenik
3. Kattintson bármelyik réteg nevére az adott verzióra való váltáshoz
4. A kép azonnal frissül, és megjeleníti a kiválasztott réteget.

**Gyors váltás:**

* A legördülő menü megjegyzi az utolsó választását.
* A következő képhez való navigáláskor az Chloros megpróbálja megjeleníteni ugyanazt a réteg típust.
* Ha az a réteg nem létezik a következő képen, akkor alapértelmezés szerint JPG lesz.

### Rétegek elérhetősége

Nem minden réteg elérhető minden képhez:

**Mindig elérhető:**

* ✅ JPG (minden képnek van JPG előnézete)

**Feltételesen elérhető:**

* ⚠️ RAW (eredeti) – Csak akkor, ha a kép RAW vagy RAW+JPG módban lett rögzítve
* ⚠️ RAW (cél) – Csak akkor, ha a kép észlelt kalibrációs célokat tartalmaz
* ⚠️ RAW (Reflektancia) – Csak reflektancia kalibrálás engedélyezése után
* ⚠️ RAW (\[Index] Index) – Csak indexek konfigurálása után

***

## Réteg állandóság

### Képek közötti navigálás

Ha egy másik képre navigál (nyílgombokkal vagy a miniatűrökre kattintva):

**A rétegbeállítások megmaradnak:**

* Ha „RAW (Reflektancia)” nézetet használ, a következő kép „RAW (Reflektancia)” nézetet jelenít meg (ha elérhető)
* Ha „RAW (NDVI Index)” nézetet használ, a következő kép „RAW (NDVI Index)” nézetet jelenít meg (ha elérhető)
* Ha ugyanaz a réteg nem létezik, akkor alapértelmezés szerint JPG

**Példa a munkafolyamatra:**

1. Nyissa meg az 1. képet, váltson át a RAW (NDVI Index) rétegre.
2. Nyomja meg a → gombot a 2. kép megtekintéséhez.
3. A 2. kép automatikusan megjeleníti a RAW (NDVI Index) réteget.
4. Folytassa a navigálást – az összes kép NDVI réteget jelenít meg
5. Nagyon hatékony a sok kép indexeredményeinek áttekintéséhez

***

## Gyakori munkafolyamatok

### 1. munkafolyamat: Előtte/utána összehasonlítás

**Cél**: Az eredeti és a kalibrált kép összehasonlítása

1. Nyissa meg a feldolgozott képet a Képmegjelenítőben
2. Válassza ki a **RAW (Eredeti)** lehetőséget a legördülő menüből
3. Vegye figyelembe a vignettálást és a nem kalibrált értékeket
4. Váltson át a **RAW (Reflektancia)** lehetőségre a legördülő menüből
5. Összehasonlítás – a vignettálás eltávolítva, az értékek kalibrálva

### 2. munkafolyamat: Index áttekintése

**Cél**: Az NDVI eredmények gyors áttekintése az adatkészletben

1. Nyissa meg az első feldolgozott képet
2. Válassza ki a legördülő menüből a **RAW (NDVI Index)** lehetőséget
3. A → nyíl gombbal lépjen a következő képre
4. Az NDVI réteg automatikusan megmarad
5. Folytassa az összes kép áttekintését, ellenőrizve az NDVI mintákat
6. Váltson át a **RAW (NDRE Index)**-re az összehasonlításhoz.

### 3. munkafolyamat: Célpontok ellenőrzése

**Cél**: Ellenőrizze, hogy az összes célpont kép helyesen lett-e felismerve.

1. Keresse meg a célpont képet.
2. Válassza ki a **RAW (Target)** lehetőséget a legördülő menüből.
3. Ellenőrizze, hogy a kalibrációs célpontok jól láthatóak és felismerhetők-e.
4. Keresse meg a következő célképet
5. Ismételje meg az ellenőrzést minden cél esetében

### 4. munkafolyamat: Pixelérték-ellenőrzés

**Cél**: Ellenőrizze a visszaverődési értékek tudományos pontosságát

1. Nyissa meg a feldolgozott képet
2. Válassza ki a **RAW (Reflectance)** réteget
3. Engedélyezze a **Pixel Percent** módot (gomb a jobb felső eszköztáron)
4. Vigye a kurzort a növényzet területére
5. Ellenőrizze, hogy a pixelértékek a várt tartományban vannak-e (30-70% az NIR esetében, 5-15% az Red esetében)
6. Ellenőrizze, hogy a talaj és a vízterületek értékei megfelelőek-e

***

## A pixelértékek rétegenkénti értelmezése

A különböző rétegek különböző pixelérték-tartományokat mutatnak:

### JPG réteg

* **Tartomány**: 0-255 (8 bites)
* **Jelentés**: Kijelzett értékek, gamma-korrigált
* **Használat**: Csak vizuális ellenőrzéshez, nem tudományos méréshez

### RAW (eredeti)

* **Tartomány**: 0-65535 (16 bites)
* **Jelentés**: Nyers szenzor digitális számok
* **Használat**: A szenzor teljesítményének ellenőrzése, nem kalibrált

### RAW (Reflektancia)

* **Tartomány**: 0-65 535 (16 bites TIFF) vagy 0,0-1,0 (32 bites százalék)
* **Jelentés**: Kalibrált százalékos visszaverődés
* **Használat**: Tudományos mérések és elemzések

**16 bites TIFF esetén:** Ossza el 65 535-tel a százalékos visszaverődés kiszámításához **32 bites százalék esetén:** Az értékek közvetlenül a százalékot jelzik (0,5 = 50% visszaverődés)

### RAW (indexképek)

* **Tartomány**: Indexenként változik (normálisan -1,0 és +1,0 között a normalizált indexek esetében)
* **Jelentés**: Index számítási eredmény
* **Példák**:
  * NDVI: -1 és +1 között (növényzet általában 0,4 és 0,9 között)
  * NDRE: -1 és +1 között (stresszérzékelés)
  * EVI: 0 és 1 között (továbbfejlesztett növényzet)

***

## Tippek és bevált gyakorlatok

### Hatékony rétegváltás

* **Billentyűparancsok ismerete**: Nincs billentyűparancs a rétegekhez, de a navigációs nyilak (←/→) minden rétegen működnek
* **Következetes munkafolyamatok**: Válasszon ki egy réteget (pl. NDVI), és tekintse át a teljes adatkészletet, mielőtt másikra váltana
* **Gyors összehasonlítás**: Váltson az Eredeti és a Reflektancia között a feldolgozás minőségének ellenőrzéséhez.

### Teljesítményre vonatkozó szempontok

* **A JPG a leggyorsabban töltődik be**: Használja a sok kép közötti gyors navigáláshoz.
* **A RAW rétegek lassabban töltődnek be**: Magasabb felbontás és bitmélység.
* **Index rétegek**: Hasonló sebességűek, mint a Reflektancia rétegek.
* **Az első betöltés a leglassabb**: Az azonos réteg későbbi megtekintései gyorsítótárba kerülnek, és gyorsabbak.

### Minőségellenőrzés

* **Mindig ellenőrizze a RAW (eredeti) fájlt**: Ellenőrizze a forrásadatok minőségét, mielőtt megbízna a feldolgozott kimenetekben.
* **Rétegek összehasonlítása**: Használja a rétegváltást a feldolgozás helyességének ellenőrzéséhez.
* **Index tartományok ellenőrzése**: Használja a Pixel Percent módot az index rétegekkel, hogy ellenőrizze, hogy az értékek ésszerűek-e.

***

## Hibaelhárítás

### Réteg nem elérhető

**Probléma**: A várt réteg nem jelenik meg a legördülő menüben.

**Lehetséges okok:**

* A kép nem lett feldolgozva (csak JPG és RAW (eredeti) formátumok állnak rendelkezésre).
* A feldolgozás során a reflektancia kalibrálás le volt tiltva.
* A projektbeállításokban nem lett konfigurálva a konkrét index.
* A kép csak célkép (a célokhoz nem lettek generálva indexek).

**Megoldások:**

1. Ellenőrizze, hogy a kép feldolgozásra került-e (ellenőrizze a feldolgozott fájlok kimeneti mappáját).
2. Ellenőrizze a projektbeállításokat, hogy az indexek konfigurálva lettek-e.
3. Feldolgozza újra a kívánt indexek engedélyezésével.

### Rossz réteg jelenik meg

**Probléma**: A kép váratlan rétegben nyílik meg.

**Ok**: Az előző kép rétegbeállításai átkerültek, de az a réteg nem létezik a jelenlegi képen.

**Megoldás:** Az Chloros automatikusan JPG-re vált, ha a preferált réteg nem elérhető – ez normális viselkedés.

### A kalibrációs célpontok nem láthatók

**Probléma:** A RAW (célpont) réteg nem jeleníti meg a célpontok észlelését.

**Lehetséges okok:**

* A célpontok feldolgozás közben nem lettek észlelve.
* A kép valójában nem tartalmaz célpontokat.
* A célpont-felismerési beállítások túl szigorúak

**Megoldások:**

1. Ellenőrizze a hibakeresési naplóban, hogy vannak-e „Célpont megtalálva” üzenetek.
2. Ellenőrizze, hogy a kép valóban tartalmaz-e látható kalibrációs célpontokat.
3. Állítsa be a célpont-felismerési beállításokat a Projektbeállításokban.
4. Lásd: [Célképek kiválasztása](../processing-images-gui/choosing-target-images.md)

***

## Kapcsolódó funkciók

### Képmegjelenítő eszközök

Bármely réteg megtekintésekor használhatja a következőket:

* **Zoom vezérlők**: Nagyítás a részletek megtekintéséhez
* **Panoráma**: Kattintson és húzza az egérrel a nagyított kép mozgatásához
* **Pixelérték-ellenőrzés**: Az értékek megtekintése a kurzor helyén
* **Navigációs nyilak**: Képek közötti mozgás a réteg megtartása mellett
* **Pixel százalékos mód**: Váltás DN és százalékos kijelzés között

A képnézegető teljes dokumentációját lásd a [Kép teljes képernyős megnyitása](opening-an-image-full-screen.md) című részben.

### Index/LUT Sandbox

Interaktív index tesztelés és vizualizálás:

* **Valós idejű index számítás**: Különböző index képletek tesztelése
* **LUT szín leképezés**: Színátmenetek alkalmazása szürkeárnyalatos indexekre
* **Vizualizációk exportálása**: Színes index képek mentése

A részleteket lásd az [Index/LUT Sandbox](index-lut-sandbox.md) című dokumentumban.

***

## Következő lépések

Most, hogy már ismeri a képrétegeket:

* [**Kép teljes képernyős megnyitása**](opening-an-image-full-screen.md) – Teljes Image Viewer útmutató
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – Interaktív index vizualizáció
* [**Multispektrális index képletek**](../project-settings/multispectral-index-formulas.md) – Elérhető indexek referencia
* [**A feldolgozás befejezése**](../processing-images-gui/finishing-the-processing.md) – A feldolgozott kimenetek megértése
