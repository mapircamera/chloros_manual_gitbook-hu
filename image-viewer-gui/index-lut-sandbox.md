# Index/LUT Sandbox

Az Index/LUT Sandbox egy interaktív munkaterület az Chloros Image Viewer programon belül, amely lehetővé teszi, hogy valós időben kísérletezzen a multispektrális indexek kiszámításával és a színek vizualizálásával. Ez a hatékony eszköz segít különböző indexek tesztelésében, az értéktartományok finomításában és publikálásra kész vizualizációk létrehozásában anélkül, hogy az egész adatkészletet újra kellene feldolgozni.

## Mi az Index/LUT Sandbox?

### Cél

A Sandbox a következőket biztosítja:

* **Valós idejű indexszámítás** – Bármely vegetációs index azonnali alkalmazása
* **Interaktív LUT-beállítás** – Színátmenetek és tartományok finomhangolása
* **Munkafolyamat-optimalizálás** – A legjobb beállítások meghatározása a kötegelt feldolgozás előtt

### Sandbox vs. projektfeldolgozás

**Index/LUT Sandbox (interaktív):**

* Egyszerre egy kép
* Azonnali visszajelzés
* Kísérleti és iteratív
* Nincs végleges változás a fájlokban
* Tökéletes a felfedezéshez és teszteléshez

**Projektfeldolgozás (tömeges):**

* Egész adatkészlet egyszerre
* Előre konfigurált beállítások
* Végleges kimeneti fájlok
* Időigényes
* Legjobb, ha a beállítások véglegesek

{% hint style=&quot;success&quot; %}
**Legjobb munkafolyamat**: Használja a Sandboxot a kísérletezéshez és az optimális index- és LUT-beállítások megtalálásához, majd alkalmazza ezeket a beállításokat a projekt feldolgozás során az egész adatkészletre.
{% endhint %}

***

## Munka az Index/LUT Sandboxszal

### Az előre kiszámított indexek megértése

Az Chloros-ben az indexek a projekt feldolgozása során alkalmazhatók. Az exportáláshoz alkalmazni kívánt index- és LUT-beállítások meghatározásához a legegyszerűbb a képnézegető sandbox használata.

A sandbox lehetővé teszi a következőket:

* **Új indexek és színátmenetek (LUT-ok) alkalmazása** az adatok vizualizálásához
* **A vizualizációs beállítások interaktív módosítása**
* **A már kiszámított indexképek megtekintése**
* **A pixelértékek ellenőrzése** minden nagyítási szinten

### A Sandbox megnyitása

Az Index/LUT Sandbox az **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> oldalsáv fülén keresztül érhető el:

1. Kattintson egy képre a fájlböngésző képrácsán, amely megnyílik a **Képmegjelenítő** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> fülön
2. Kattintson a **Képnézegető** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> fülre, hogy megnyissa a bal oldali kiugró oldalsávot, ha még nem nyitott meg

### Kép kiválasztása az index/LUT alkalmazásához

Az index kezeléséhez a Képmegjelenítő <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sandboxban:

1. **Nyissa meg a képet** a fő képrácsból, rákattintva
2. Megnyílik az **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> fül megnyílik
3. Kattintson a **Réteg legördülő menüre** (a néző jobb felső sarkában)
4. Válassza ki a réteget a legördülő menüből:
   * RAW (Reflektancia)

### Index alkalmazása egy képre

Miután a kép teljes képernyősre váltott, és a **Képmegjelenítő** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> lap oldalsávja megnyílik:

1. Jelölje be az Index jelölőnégyzetet az oldalsáv tetején.
2. Válassza ki a kamera szűrőjét a bal oldali legördülő menüből.
3. Válassza ki a kívánt indexképletet a jobb oldali legördülő menüből.
4. Húzza a szűrőcsatorna színes köröket az alábbi indexképletben szereplő helyekre.
5. Ha a képlet érvényes, a kép frissül és megjeleníti az indexértékeket.
6. Mozgassa az egérmutatót, hogy megtekintse az egérmutató helyén található értékeket.
7. Nagyítson rá az egyes pixelekre és a hozzájuk tartozó értékekre.

Minden indexnek megvan a maga értékhatára és jelentése:

#### NDVI példa

```
Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

A teljes indexképlet dokumentációját lásd: [Multispectral Index Formulas](../project-settings/multispectral-index-formulas.md).

***

## LUT-okkal (Look-Up Tables) való munka

### Mi az a LUT?

A **Look-Up Table (LUT)** a numerikus indexértékeket színekhez rendeli a vizualizáláshoz:

* **Bemenet**: Index pixelérték (pl. NDVI 0,65)
* **Kimenet**: RGB szín (pl. élénkzöld)
* **Cél**: A minták könnyebb láthatósága és értelmezhetősége

**Szürkeárnyalatos vs. színes LUT:**

* Szürkeárnyalatos: Tudományos és semleges, a nyers adatokat mutatja
* Színes LUT: Intuitív és hatásos, kiemeli a mintákat és a különbségeket

{% hint style=&quot;success&quot; %}
**Vizualizációs teljesítmény**: A színes LUT alkalmazása a szürkeárnyalatos indexképre jelentősen megkönnyíti a minták, anomáliák és érdekes területek egy pillanat alatt történő azonosítását.
{% endhint %}

### LUT alkalmazása indexképre

Ha már rendelkezik indexképpel, amelyen látható

1. Kattintson a <img src="../.gitbook/assets/image.png" alt="" data-size="line"> „+LUT hozzáadása” gombra
2. Válassza ki a színátmenetet
3. Állítsa be a vágás minimális/maximális végpontjait
4. Állítsa be a vágási módot
5. Jelölje be az Index jelölőnégyzetet az **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> fül oldalsávjában az Index jelölőnégyzetet a LUT alkalmazásához.

### Színátmenet kiválasztása

**Színátmenet kiválasztása:**

1. A LUT panelen keresse meg a **színes színátmenet sávot**.
2. Vigye az egérmutatót a sáv fölé a rendelkezésre álló színátmenet-beállítások megtekintéséhez.
3. Válassza ki a kívánt színátmenetet.
4. Az Index jelölőnégyzet bejelölésekor a kép **azonnal frissül** az új színekkel.

{% hint style=&quot;success&quot; %}
**Bevált gyakorlat**: Az NDVI-hez hasonló vegetációs indexek esetében az Red-Sárga-Green színátmenet a leginkább intuitív, mert összhangban van a természetes színasszociációkkal (zöld=egészséges, sárga=mérsékelt, piros=stresszes).
{% endhint %}

### Színosztályok beállítása

A **Classes control** (Osztályok vezérlő) határozza meg, hogy hány diszkrét színlépcső jelenik meg a gradiensben:

**Osztályszám opciók:**

* **2-5 osztály**: Nagyon tág kategóriák, jól elkülöníthető zónák
* **6-10 osztály**: Kiegyensúlyozott, osztályozáshoz alkalmas
* **11-20 osztály**: Sima színátmenetek, folytonos megjelenés
* **20+ osztály**: Szinte folytonos, maximális simaság

**Beállítás módja:**

1. A LUT panelen keresse meg a **színminták négyzeteit a színátmenet sáv alatt**
2. A + gombbal növelje az osztályok számát.
3. Kattintson duplán egy színmintára, hogy eltávolítsa az osztályok számát.
4. A gradiens **valós időben** frissül a képen.

**Hatása a megjelenítésre:**

* **Kevesebb osztály** (3-5): Különálló zónákat hoz létre, egyszerűsíti a besorolást, könnyebb megkülönböztetni a kategóriákat.
* **Közepes osztályok** (6-10): Kiegyensúlyozott megközelítés, a legtöbb alkalmazáshoz megfelelő
* **Több osztály** (15-20): Sima átmenetek, részletes variáció, fotószerű megjelenés

**Mikor használható:**

* **Kevés osztály (3-5)**: Prezentációs diák, osztályozási térképek, egyszerű jelentések
* **Közepes osztályok (6-10)**: Általános elemzés, kiegyensúlyozott részletek, szabványos jelentések
* **Sok osztály (15-20)**: Tudományos elemzés, részletes vizsgálat, publikálásra alkalmas eredmények

### Értéktartományok finomhangolása

Az **értéktartomány-vezérlők** határozzák meg, hogy az indexértékek a színátmenetben mely színekhez tartoznak:

**Tartományvezérlők a LUT panelen:**

* **Minimális érték**: A színskála alsó határa
* **Maximális érték**: A színskála felső határa
* **Közbenső értékek**: Automatikusan elosztva a min és max között (az osztályok száma alapján)

#### Min/Max értékek beállítása

**Az érték tartományok beállítása:**

1. A LUT panelen keresse meg a **Minimális érték** és **Maximális érték** beviteli mezőket
2. Kattintson a **Minimális érték** mezőre.
3. Írja be a kívánt minimális értéket (pl. `0.2`).
4. Nyomja meg az **Enter** billentyűt, vagy kattintson a mezőn kívülre.
5. Ismételje meg a **Maximális érték** mezővel (pl. `0.9`).
6. A megjelenítés **azonnal frissül**.

{% hint style=&quot;info&quot; %}
**Automatikus méretezés**: Amikor először alkalmazza a LUT-ot, az Chloros automatikusan beállítja a minimális/maximális értéket a kép tényleges adatsorozatára. Ezután szűkítheti ezt a tartományt, hogy az érdeklődésre számot tartó értékek tartományára összpontosítson.
{% endhint %}

**Példa NDVI tartománybeállításokra:**

* **Teljes tartomány**: `-1.0` és `1.0` között (az összes lehetséges érték megjelenítése)
* **Növényzetre fókuszált**: `0.2` és `0.9` között (kizárva a csupasz talaj és a víz)
* **Csak egészséges növényzet**: `0.5`-tól `0.9`-ig (csak az erőteljes növényeket emeli ki)
* **Stresszérzékelés**: `0.2`-tól `0.5`-ig (kiemelje a problémás területeket)
* **Egyéni tartomány**: A megfigyelt pixelértékek alapján állítsa be

**Miért érdemes beállítani a tartományokat?**

* **Növelje a kontrasztot** a kívánt területen
* **Kizárja az irreleváns értékeket** (pl. víztestek, csupasz talaj)
* **Szabványosítsa a megjelenítést** több kép vagy dátum között
* **Kiemelje a finom különbségeket** egy szűk értéktartományon belül

### A tartományon kívüli értékek kivágása

Ha a pixelértékek a megadott minimális/maximális tartományon kívül esnek, a **kivágási módok** segítségével szabályozhatja azok megjelenítését.

#### **Elérhető kivágási módok:**

#### 1. Minimum és maximum

* **A minimum alatt** lévő pixelek → a színátmenet **első színével** (pl. piros) jelennek meg
* **A maximális érték feletti** pixelek → a színátmenet **utolsó színével** (pl. zöld) jelennek meg
* **Használati eset**: A szélsőségek kiemelése, a teljes adatsor telített színekkel a határokon
* **Példa**: Az 0,2 alatti NDVI értékek mind pirosak, a 0,9 feletti értékek mind zöldek

#### 2. Átlátszó háttér

* A **tartományon kívüli** pixelek **teljesen átlátszóvá** válnak
* Csak a **tartományon belüli** pixelek mutatnak színátmenetet
* **Használati eset**: GIS-átfedés, meghatározott értéktartományok elkülönítése, csak a érdekes területek kiemelése
* **Példa**: Csak az NDVI 0,4–0,7 értéket színesen jeleníti meg, minden más átlátszó

{% hint style=&quot;warning&quot; %}
**Átlátszóság korlátozása**: Az átlátszó pixelek a nézőben háttérszínként jelennek meg. Feldolgozás közbeni exportáláskor az átlátszóság megmarad PNG formátumban, de JPG formátumban nem.
{% endhint %}

#### 3. Index háttér

* A **tartományon kívüli** pixelek **szürkeárnyalatúak** (nyers indexértékeket mutatnak)
* A **tartományon belüli** pixelek **színátmenetet** mutatnak
* **Használati eset**: Finom kiemelés, a kontextus megőrzése a figyelemre méltó területek kiemelése mellett
* **Példa**: Színes kiemeléssel hangsúlyozza a növényzetet (NDVI 0,3-0,5), míg az egészséges területeket szürkével jeleníti meg

#### 4. Eredeti háttér

* A **tartományon kívüli** pixelek **eredeti multispektrális képet** jelenítenek meg
* A **tartományon belüli** pixelek **színátmenetet** mutatnak
* **Használati eset**: A leginkább intuitív – ötvözi a természetes kép kontextusát az analitikus színréteggel
* **Példa**: A tényleges mező/termés megjelenése színkódolt stresszes területekkel átfedve

### A megfelelő kivágási mód kiválasztása

| Kivágási mód              | Legalkalmasabb                                   | Vizualizációs stílus          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum és maximum**    | Teljes adatmegjelenítés, tudományos elemzés     | Minden pixel színes           |
| **Átlátszó háttér** | GIS-rétegek, meghatározott tartományok elkülönítése    | Szín a tartományon belül, üresen kívül |
| **Index háttér**       | Finom hangsúlyozás, az adatok kontextusának megőrzése  | Szín a tartományon belül, szürke kívül  |
| **Eredeti háttér**    | Jelentések, prezentációk, intuitív elemzés | Szín a tartományon belül, fotó kívül |

### Egyéni LUT színek létrehozása

A vizualizáció teljes ellenőrzése érdekében **egyéni színátmenetek** hozhatók létre az egyes színállások szerkesztésével.

**Egyéni színátmenet létrehozása:**

1. A LUT panelen keresse meg a **színátmenet-előnézeti sávot**.
2. Keresse meg a **színminták négyzeteit** a színátmenet alatt.
3. **Kattintson egy színállásra** a kiválasztásához.
4. Megnyílik a **színválasztó**.
5. Válasszon új színt a következők segítségével:
   * **Színkerék**: Vizuális színválasztás.
   * **RGB/HSV csúszkák**: Pontos színvezérlés.
   * **Hex kód beírása**: Pontos színmeghatározás (pl. `#FF0000` a piroshoz).
6. Kattintson a színválasztó **mellett, hogy az új színt alkalmazza**
7. A színátmenet **azonnal frissül** a képen

**Színstopok hozzáadása vagy eltávolítása:**

* **Stop hozzáadása**: Kattintson a + ikonra, hogy új színmintát adjon hozzá a végéhez
* **Stop eltávolítása**: Kattintson duplán a színnégyzetre, hogy eltávolítsa a színmintát

**Testreszabási stratégiák:**

* **Gradiens megfordítása**: Fordítsa meg a színek sorrendjét, hogy megfordítsa a jelentést (pl. zöld=alacsony, piros=magas)
* **Márka színei**: Illessze a jelentések színeit a szervezet színeinek palettájához
* **Színvakok számára is megfelelő**: Használjon narancssárga-kék vagy lila-sárga kombinációkat
* **Nyomtatás optimalizálása**: Válasszon olyan színeket, amelyek színes és szürkeárnyalatos nyomtatásban is jól mutatnak
* **Több küszöbérték**: Használjon különböző színeket meghatározott értékküszöbértékeknél a besoroláshoz

{% hint style=&quot;info&quot; %}
**Egyéni színátmenetek mentése**: Az egyéni színátmenetek menthetők és újra felhasználhatók. Kattintson a LUT panelen található mentés ikonra, hogy egyéni színsémáit elmentse későbbi felhasználás céljából.
{% endhint %}

***

## Interaktív munkafolyamat

### Valós idejű frissítések

A sandboxban végzett összes LUT-beállítás **azonnal és interaktív módon** frissíti a képet:

* **Réteg váltás** → A kép azonnal megváltozik
* **Gradiens kiválasztása** → A színek azonnal frissülnek
* **Értéktartomány beállítása** → A kontraszt valós időben változik
* **Osztályok módosítása** → A gradiens simasága azonnal frissül
* **Kivágás módosítása** → A háttér megjelenítése azonnal megváltozik
* **Színek szerkesztése** → Az egyéni gradiens azonnal alkalmazásra kerül

**Nincs szükség „Alkalmazás” gombra** – minden változás élőben és interaktív módon történik!

{% hint style=&quot;success&quot; %}
**Élő visszajelzés**: Az azonnali vizuális visszajelzés lehetővé teszi, hogy gyorsan kipróbáljon különböző beállításokat, amíg meg nem találja az elemzési igényeinek leginkább megfelelő vizualizációt.
{% endhint %}

### Iteratív finomítási munkafolyamat

**Tipikus LUT-optimalizálási munkafolyamat:**

1. **Válassza ki az indexréteget** (pl. RAW (Reflectance))
2. **Alkalmazzon indexet** – Válassza ki a kamera szűrőt és az indexképletet, húzza a színes köröket a megfelelő helyre az indexképletben
3. **LUT gradiens alkalmazása** – Kezdje az Red-Yellow-Green előre beállított értékkel
4. **Pixelértékek ellenőrzése** – Mozgassa a kurzort, jegyezze fel az értékek tartományait
5. **Minimális/maximális értékek beállítása** – Szűkítse a növényzetre való fókuszáláshoz (pl. 0,2-től 0,9-ig)
6. **Vágás kiválasztása** – Próbálja ki az „Eredeti háttér” lehetőséget a kontextushoz
7. **Színek finomítása** – Szükség esetén testreszabhatja a gradienst a kívánt hangsúlyozáshoz
8. **Beállítások véglegesítése** – Dokumentálja a beállításokat, és másolja át a Projektbeállításokba az exportáláshoz

### Pixelértékek ellenőrzése

A tényleges pixelértékek megértése elengedhetetlen a hatékony LUT-tartományok beállításához:

**Az értékek ellenőrzése:**

1. A pixelértékek akkor jelennek meg, ha a képnél az Index vagy az Index és a LUT **négyzetek be vannak jelölve**.
2. **Vigye a kurzort** a kép különböző területeire
3. **Figyelje meg a pixelértékeket**, amelyek a legenda ablakban jelennek meg, amikor az egérmutatót rájuk helyezi
4. Nagyítson rá, hogy megnézze az egyes pixeleket, amelyek lebegő értékkel vannak kiemelve
5. **Jegyzetelje le** a különböző jellemzők értékhatárait:
   * **Egészséges növényzet**: pl. NDVI 0,55–0,85
   * **Stresszes növényzet**: pl. NDVI 0,30–0,50
   * **Csupasz talaj**: pl. NDVI 0,05–0,25
   * **Víz** (ha van): pl. NDVI -0,05–0,10

**Pixelértékek használata a LUT tartományok beállításához:**

A pixelértékek ellenőrzése után állítsa be a LUT min/max értékeit ennek megfelelően:

**Példa:**

* **Megfigyelés**: Talajértékek = 0,05-0,25, Stresszes = 0,25-0,50, Egészséges = 0,50-0,85
* **Cél**: Csak a növények egészségi állapotának megjelenítése (a talaj kizárása)
* **LUT beállítások**: Min = `0.25`, Max = `0.85`
* **Kivágás**: „Eredeti háttér” a talaj természetes színének megjelenítéséhez
* **Eredmény**: A színátmenet csak a növényzetre vonatkozik, a talaj az eredeti képen látható

{% hint style=&quot;info&quot; %}
**Dinamikus tartomány**: A különböző növények, évszakok és növekedési szakaszok eltérő értéktartományokkal rendelkeznek. A LUT-tartományok beállítása előtt mindig ellenőrizze a konkrét adatkészlet pixelértékeit.
{% endhint %}

***

## Egyéni indexek (Chloros+)

### Egyéni indexképletek létrehozása

{% hint style=&quot;info&quot; %}
**Hol hozhatók létre**: Az egyéni indexek a feldolgozás előtt a **Projektbeállítások** menüpontban, valamint a Képmegjelenítő sandbox oldalsávján konfigurálhatók.
{% endhint %}

**Egyéni index létrehozása:**

1. **Nyissa meg a Projektbeállításokat** (a feldolgozás előtt) vagy a Képmegjelenítő sandbox oldalsávját.
2. Keresse meg az **Index képlet legördülő menüt**.
3. Keresse meg a **„Egyéni”** opciót (be kell jelentkeznie az Chloros+ licenccel).
4. **Határozza meg a képletet** a sávváltozók segítségével:
   * Sávnevek: `NIR`, `Red`, `Green`, `Blue`, `RedEdge` stb.
   * Operátorok: `+`, `-`, `*`, `/`, `^` (exponens)
   * Funkciók: `sqrt()`, `abs()` stb. (ha támogatott)
   * Zárójelek: `()` a műveletek sorrendjéhez
5. **Nevezze el az indexet** (pl. „MyIndex” vagy „CustomNDVI”)
6. **Mentse a konfigurációt**

**Példák egyéni képletekre:**

```
Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style=&quot;warning&quot; %}
**Képlet érvényesítése**: Győződjön meg arról, hogy a képlet a kamerájában elérhető sávokat használja. Például az RedEdge csak az RedEdge szűrővel rendelkező kamerákban érhető el.
{% endhint %}

***

## Következő lépések

Most, hogy megértette az Index/LUT Sandbox működését:

* **Alkalmazás a feldolgozásra**: Használja a [Projektbeállítások](../project-settings/project-settings.md) menüpontban található beállításokat
* **Tömeges feldolgozás**: Alkalmazza az optimalizált indexeket a teljes adatkészletekre
* **További információk**: Olvassa el a [Multispektrális index képletek](../project-settings/multispectral-index-formulas.md) című részt

Kapcsolódó dokumentáció:

* [**Képrétegek**](image-layers.md) – Rétegkezelés és megjelenítés
* [**Kép teljes képernyős megnyitása**](opening-an-image-full-screen.md) – Képnézegető alapjai
* [**Képek feldolgozása (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Teljes feldolgozási munkafolyamat
