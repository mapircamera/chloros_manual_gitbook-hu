# Index/LUT Sandbox

Az Index/LUT Sandbox az Chloros Image Viewer alkalmazáson belüli interaktív munkaterület, amely lehetővé teszi a multispektrális indexek kiszámításával és a színes vizualizációkkal való valós idejű kísérletezést. Ez a hatékony eszköz segít különböző indexek tesztelésében, az értéktartományok finomításában, valamint publikálásra kész vizualizációk létrehozásában anélkül, hogy az egész adatkészletet újra kellene feldolgozni.

## Mi az Index/LUT Sandbox?

### Cél

A Sandbox a következőket kínálja:

* **Valós idejű indexszámítás** – Bármely vegetációs index azonnali alkalmazása
* **Interaktív LUT-beállítás** – Színátmenetek és tartományok finomhangolása
* **Munkafolyamat-optimalizálás** – A legjobb beállítások meghatározása a kötegelt feldolgozás előtt

### Sandbox vs. projektfeldolgozás

**Index/LUT Sandbox (interaktív):**

* Egyszerre egy kép
* Azonnali visszajelzés
* Kísérleti és iteratív
* Nincs végleges változás a fájlokban
* Tökéletes felfedezéshez és teszteléshez

**Projektfeldolgozás (tömeges):**

* Egyszerre az egész adatkészlet
* Előre konfigurált beállítások
* Végleges kimeneti fájlok
* Időigényes
* A legjobb, ha a beállítások véglegesek

{% hint style="success" %}
**A legjobb munkafolyamat**: Használja a Sandboxot a kísérletezéshez és az optimális index- és LUT-beállítások megtalálásához, majd alkalmazza ezeket a beállításokat a Projektfeldolgozás során az egész adatkészletre.
{% endhint %}

***

## Az Index/LUT Sandbox használata

### Az előre kiszámított indexek megértése

Az Chloros-ben az indexek a projekt feldolgozása során alkalmazhatók. Az exportáláshoz alkalmazni kívánt index- és LUT-beállítások meghatározásához a legegyszerűbb a képnézegető sandbox használata.

A sandbox lehetővé teszi a következőket:

* **Új indexek és színátmenetek (LUT-ok) alkalmazása** az adatok vizualizálásához
* **A vizualizációs beállítások interaktív módosítása*** **A már kiszámított indexképek megtekintése*** **A pixelértékek ellenőrzése** minden nagyítási szinten

### A Sandbox megnyitása

Az Index/LUT Sandbox a **Képmegjelenítő** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> oldalsáv fülén keresztül érhető el:

1. Kattintson egy képre a fájlböngésző képrácsában, ez megnyitja a **Képmegjelenítő** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> fülön
2. Kattintson **az Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> fülre, hogy megnyissa a bal oldali felugró oldalsávot, ha az még nincs megnyitva

### Kép kiválasztása az Index/LUT alkalmazásához

Az Index/LUT Sandbox használatához <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> sandboxban:

1. **Nyisson meg egy képet** a fő képrácsból rákattintva
2. Ekkor megnyílik az **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> fül megnyílik
3. Kattintson a **Réteg legördülő menüre** (a néző jobb felső sarkában)
4. Válassza ki a réteget a legördülő menüből:
   * RAW (Reflektancia)

### Index alkalmazása egy képre

Miután a kép teljes képernyősre váltott, és a **Képmegjelenítő** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> fül oldalsávja megnyílt:

1. Jelölje be az Index négyzetet az oldalsáv tetején
2. Válassza ki a fényképezőgép szűrőjét a bal oldali legördülő menüből
3. Válassza ki a kívánt index képletet a jobb oldali legördülő menüből
4. Húzza a szűrőcsatorna színköröket az alábbi index képlet megfelelő helyeire
5. Amint a képlet érvényes, a kép frissül és megjeleníti az index értékeket
6. Mozgassa az egérmutatót, hogy megtekintse az értékeket a mutató helyén
7. Nagyítson rá, hogy megtekintse az egyes pixeleket és a hozzájuk tartozó értékeket

Minden indexnek megvan a maga értéktartománya és jelentése:

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

Az indexképletek teljes dokumentációját lásd a [Multispektrális indexképletek](../project-settings/multispectral-index-formulas.md) alatt.

***

## LUT-okkal (Look-Up Tables) való munka

### Mi az a LUT?

A **Look-Up Table (LUT)** a numerikus indexértékeket színekhez rendeli a vizualizáláshoz:

* **Bemenet**: Index pixelérték (pl. NDVI 0,65)
* **Kimenet**: RGB szín (pl. élénkzöld)
* **Cél**: A minták könnyebb láthatósága és értelmezése**Szürkeárnyalatos vs. színes LUT:**

* Szürkeárnyalatos: Tudományos és semleges, a nyers adatokat mutatja
* Színes LUT: Intuitív és hatásos, kiemeli a mintákat és a különbségeket

{% hint style="success" %}
**Vizualizációs erő**: Ha színes LUT-ot alkalmazunk egy szürkeárnyalatos indexképre, az jelentősen megkönnyíti a minták, anomáliák és érdekes területek azonnali azonosítását.
{% endhint %}

### LUT alkalmazása indexképre

Miután rendelkezünk egy indexképpel, amelyen látható

1. Kattintson a <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> „+LUT hozzáadása” gombra
2. Válassza ki a színátmenetet
3. Állítsa be a kivágás minimális/maximális végpontjait
4. Állítsa be a kivágási módot
5. Jelölje be az Index négyzetet az **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> fül oldalsávján, hogy alkalmazza a LUT-ot

### Színátmenet kiválasztása

**Átmenet kiválasztása:**

1. A LUT panelen keresse meg a**színes átmeneti sávot**

2. Vigye az egérmutatót a sáv fölé a rendelkezésre álló átmeneti beállítások megtekintéséhez
3. Válassza ki a kívánt átmenetet
4. A kép **azonnal frissül** az új színekkel, ha az Index jelölőnégyzet be van jelölve

{% hint style="success" %}
**Bevált gyakorlat**: Az olyan vegetációs indexek esetében, mint az NDVI, az Red-Sárga-Green színátmenet a legintuitívabb, mivel összhangban van a természetes színkapcsolatokkal (zöld=egészséges, sárga=közepes, piros=stresszes).
{% endhint %}

### Színosztályok beállítása

A **Classes vezérlő**határozza meg, hogy hány különálló színlépcső jelenik meg a színátmenetben:**Osztályszám opciók:*** **2–5 osztály**: Nagyon tág kategóriák, jól elkülönülő zónák
* **6–10 osztály**: Kiegyensúlyozott, jó osztályozáshoz
* **11–20 osztály**: Sima színátmenetek, folytonos megjelenés
* **20+ osztály**: Majdnem folytonos, maximális simaság**Beállítás módja:**

1. A LUT panelen keresse meg a**színminta négyzeteket a színátmenet sáv alatt**

2. Állítsa be az osztályok számát a + gombbal
3. Távolítsa el az osztályokat a színminta dupla kattintásával
4. A színátmenet **valós időben** frissül a képen**Hatása a vizualizációra:*** **Kevesebb osztály** (3-5): Különálló zónákat hoz létre, egyszerűsített osztályozás, könnyebb megkülönböztetni a kategóriákat
* **Közepes számú osztály** (6-10): Kiegyensúlyozott megközelítés, a legtöbb alkalmazáshoz megfelelő
* **Több osztály** (15-20): Sima átmenetek, részletes variáció, fotószerű megjelenés**Mikor használható:*** **Kevés osztály (3–5)**: Prezentációs diák, osztályozási térképek, egyszerű jelentések
* **Közepes számú osztály (6–10)**: Általános elemzés, kiegyensúlyozott részletesség, szabványos jelentések
* **Sok osztály (15–20)**: Tudományos elemzés, részletes vizsgálat, publikációs minőségű kimenetek

### Értéktartományok finomhangolása

Az **értéktartomány-vezérlők**határozzák meg, hogy a gradiensben mely indexértékek mely színekhez tartoznak:**Tartományvezérlők a LUT panelen:*** **Minimális érték**: A színskála alsó határa
* **Maximális érték**: A színskála felső határa
* **Közbenső értékek**: Automatikusan elosztva a min. és max. között (az osztályok száma alapján)

#### A minimális/maximális értékek beállítása

**Az értéktartományok beállítása:**

1. A LUT panelen keresse meg a**Minimális érték**és a**Maximális érték** beviteli mezőket
2. Kattintson a **Minimális érték** mezőre
3. Írja be a kívánt minimális értéket (pl. `0.2`)
4. Nyomja meg az **Enter** gombot, vagy kattintson a mezőn kívülre
5. Ismételje meg a műveletet a **Max. érték** mezőnél (pl. `0.9`)
6. A vizualizáció **azonnal frissül**{% hint style="info" %}**Automatikus méretezés**: Amikor először alkalmaz egy LUT-ot, az Chloros automatikusan beállítja a min/max értéket a kép tényleges adattartományára. Ezután szűkítheti ezt a tartományt, hogy az érdeklődésre számot tartó konkrét értéktartományokra összpontosítson.
{% endhint %}

**Példa az NDVI tartomány beállításaira:*** **Teljes tartomány**: `-1.0` – `1.0` (az összes lehetséges érték megjelenítése)
* **Növényzetre fókuszálva**: `0.2` és `0.9` között (a csupasz talaj és a víz kizárása)
* **Csak egészséges növényzet**: `0.5` és `0.9` között (csak az erőteljes növényeket emelje ki)
* **Stresszfelismerés**: `0.2` – `0.5` (a problémás területek kiemelése)
* **Egyéni tartomány**: A megfigyelt képpontértékek alapján állítsa be**Miért érdemes beállítani a tartományokat?*** **Növelje a kontrasztot** a vizsgált területen
* **A nem releváns értékek kizárása** (pl. víztestek, csupasz talaj)
* **A vizualizáció egységesítése** több kép vagy dátum között
* **A finom különbségek kiemelése** egy szűk értéktartományon belül

### A tartományon kívüli értékek kivágása

Ha a pixelértékek a meghatározott min/max tartományon kívül esnek, a **kivágási módok** segítségével szabályozhatja azok megjelenítését.

#### **Elérhető kivágási módok:**

#### 1. Minimum és Maximum

* **A minimum alatti**pixelek → a színátmenet**első színével** (pl. piros) jelennek meg
* **A maximum feletti**pixelek → a színátmenet**utolsó színével** (pl. zöld) jelennek meg
* **Alkalmazási eset**: A szélsőértékek kiemelése, a teljes adattartomány megjelenítése telített színekkel a határértékeken
* **Példa**: Az 0,2 alatti NDVI értékek mind pirosak, a 0,9 feletti értékek mind zöldek

#### 2. Átlátszó háttér

* A **tartományon kívüli**pixelek**teljesen átlátszóvá** válnak
* Csak a **tartományon belüli** pixelek mutatnak színátmenetet
* **Alkalmazási eset**: GIS-réteg, meghatározott értéktartományok elkülönítése, csak a releváns területek kiemelése
* **Példa**: Csak az NDVI 0,4–0,7 közötti értékeket mutassa színesen, minden mást átlátszóan

{% hint style="warning" %}
**Átlátszóság korlátozása**: Az átlátszó pixelek a nézőben a háttérszínként jelennek meg. Feldolgozás közbeni exportáláskor az átlátszóság megmarad az PNG formátumban, de a JPG-ben nem.
{% endhint %}

#### 3. Index háttér

* A **tartományon kívüli**pixelek**szürkeárnyalatos** módon jelennek meg (a nyers indexértékeket mutatják)
* A **tartományon belüli**pixelek**színátmenetet** mutatnak
* **Alkalmazási eset**: Finom kiemelés, a kontextus megőrzése a figyelemre méltó területek kiemelése mellett
* **Példa**: A stresszhatásnak kitett növényzet színes kiemelése (NDVI 0,3–0,5), míg az egészséges területek szürkén jelennek meg

#### 4. Eredeti háttér

* A **tartományon kívüli**pixelek a**eredeti multispektrális képet** jelenítik meg
* A **tartományon belüli**pixelek**színátmenetet** mutatnak
* **Alkalmazási eset**: A legintuitívabb – ötvözi a természetes kép kontextusát az analitikai színréteggel
* **Példa**: A tényleges mező/termés megjelenése színkódolt, kiemelt területekkel átfedve

### A megfelelő kivágási mód kiválasztása

| Kivágási mód              | Legalkalmasabb                                   | Megjelenítési stílus          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Minimum és Maximum**    | Teljes adatmegjelenítés, tudományos elemzés     | Minden pixel színezett           |
| **Átlátszó háttér** | GIS-rétegek, meghatározott tartományok elkülönítése    | Szín a tartományon, üres a tartományon kívül |
| **Index háttér**       | Finom kiemelés, az adatkontextus megőrzése  | Szín a tartományon, szürke a tartományon kívül  |
| **Eredeti háttér**    | Jelentések, prezentációk, intuitív elemzés | Szín a tartományon, fénykép a tartományon kívül |

### Egyéni LUT-színek létrehozása

A vizualizáció teljes körű vezérléséhez **egyéni színátmeneteket** hozhat létre az egyes színállomások szerkesztésével.**Egyéni színátmenet létrehozása:**

1. A LUT panelen keresse meg a**színátmenet-előnézeti sávot**

2. Keresse meg a színátmenet alatt található**színmintákat**

3.**Kattintson egy színállomásra** a kiválasztásához
4. Megnyílik a **színválasztó**

5. Válasszon új színt a következők segítségével:
   * **Színkerék**: Vizuális színválasztás
   * **RGB/HSV csúszkák**: Pontos színszabályozás
   * **Hex kód beírása**: Pontos színmeghatározás (pl. `#FF0000` a piroshoz)
6. Kattintson a színválasztón kívülre **az új szín alkalmazásához**

7. A színátmenet**azonnal frissül** a képen**Színállomások hozzáadása vagy eltávolítása:*** **Állomás hozzáadása**: Kattintson a + ikonra egy új színminta hozzáadásához a végére
* **Állomás eltávolítása**: Kattintson duplán a színnégyzetre a színminta eltávolításához**Testreszabási stratégiák:*** **Gradiens invertálása**: Fordítsa meg a színek sorrendjét a jelentés megfordításához (pl. zöld=alacsony, piros=magas)
* **Márka színei**: Illessze a jelentésekhez a szervezet színpalettáját
* **Színvakok számára is megfelelő**: Használjon narancs-kék vagy lila-sárga kombinációkat
* **Nyomtatási optimalizálás**: Válasszon olyan színeket, amelyek színes és szürkeárnyalatos nyomtatás esetén is jól mutatnak
* **Több küszöbérték**: Használjon különböző színeket meghatározott értékhatárokhoz a besoroláshoz

{% hint style="info" %}
**Egyéni színátmenetek mentése**: Az egyéni színátmenetek elmenthetők és újra felhasználhatók. Kattintson a mentés ikonra a LUT panelen, hogy elmentse az egyéni színsémákat későbbi felhasználásra.
{% endhint %}

***

## Interaktív munkafolyamat

### Valós idejű frissítések

A sandboxban végzett összes LUT-beállítás **azonnal és interaktív módon** frissíti a képet:

* **Réteg váltás** → A kép azonnal megváltozik
* **Gradiens kiválasztása** → A színek azonnal frissülnek
* **Értéktartomány beállítása** → A kontraszt valós időben változik
* **Osztályok módosítása** → A gradiens simasága azonnal frissül
* **Kivágás módosítása** → A háttér megjelenése azonnal megváltozik
* **Színek szerkesztése** → Az egyéni színátmenet azonnal alkalmazásra kerül**Nincs szükség „Alkalmazás” gombra** – minden változás élőben és interaktív módon történik!

{% hint style="success" %}
**Élő visszajelzés**: Az azonnali vizuális visszajelzés lehetővé teszi, hogy gyorsan kísérletezzen a különböző beállításokkal, amíg meg nem találja az elemzési igényeinek leginkább megfelelő megjelenítést.
{% endhint %}

### Iteratív finomítási munkafolyamat

**Tipikus LUT-optimalizálási munkafolyamat:**

1.**Válassza ki az indexréteget** (pl. RAW (Reflectance))
2. **Alkalmazzon indexet** – Válassza ki a kamera szűrőjét és az indexképletet, húzza a színes köröket a megfelelő helyre az indexképletben
3. **LUT-gradiens alkalmazása** – Kezdje az Red-Yellow-Green előre beállított értékkel
4. **Pixelesszámok ellenőrzése** – Mozgassa a kurzort, jegyezze meg az értéktartományokat
5. **Minimális/maximális értékek beállítása** – Szűkítse le a növényzetre való fókuszáláshoz (pl. 0,2–0,9)
6. **Válassza ki a kivágást** - Próbálja ki az „Eredeti háttér” opciót a kontextushoz
7. **Finomítsa a színeket** - Szükség esetén testreszabhatja a gradienst a kívánt hangsúlyozáshoz
8. **Véglegesítse a beállításokat**- Jegyezze fel a beállításokat, és másolja át a Projektbeállításokba az exportáláshoz

### Pixelértékek ellenőrzése

A tényleges pixelértékek megértése elengedhetetlen a hatékony LUT-tartományok beállításához:**Hogyan ellenőrizheti az értékeket:**

1. A pixelértékek akkor jelennek meg, ha a képen az Index, vagy az Index és a LUT**négyzetek be vannak jelölve**.
2. **Vigye az egérmutatót** a kép különböző területeire
3. **Figyelje meg a pixelértékeket**, amelyek a lebegő egérmutatóval a jelölőn jelennek meg
4. Nagyítson rá, hogy lássa az egyes pixeleket, amelyek lebegő értékkel vannak kiemelve
5. **Jegyezze fel** a különböző jellemzők értéktartományait:
   * **Egészséges növényzet**: pl. NDVI 0,55–0,85
   * **Stresszes növényzet**: pl. NDVI 0,30–0,50
   * **Csupasz talaj**: pl. NDVI 0,05–0,25
   * **Víz** (ha van): pl. NDVI -0,05–0,10**Pixelértékek használata a LUT-tartományok beállításához:**A pixelértékek ellenőrzése után állítsa be a LUT min/max értékeit ennek megfelelően:**Példa:*** **Megfigyelés**: Talajértékek = 0,05–0,25, Stresszes = 0,25–0,50, Egészséges = 0,50–0,85
* **Cél**: Csak a növények egészségi állapotának megjelenítése (a talaj kizárása)
* **LUT beállítások**: Min = `0.25`, Max = `0.85`
* **Kivágás**: „Eredeti háttér” a talaj természetes színének megjelenítéséhez
* **Eredmény**: A színátmenet csak a növényzetre vonatkozik, a talaj az eredeti kép szerint jelenik meg

{% hint style="info" %}
**Dinamikatartomány**: A különböző növénykultúrák, évszakok és növekedési szakaszok eltérő értéktartományokkal rendelkeznek. A LUT-tartományok beállítása előtt mindig ellenőrizze a konkrét adatkészlet pixelértékeit.
{% endhint %}

***

## Egyéni indexek (Chloros+)

### Egyéni indexképletek létrehozása

{% hint style="info" %}
**Hol hozható létre**: Az egyéni indexek a feldolgozás előtt a**Projektbeállítások** menüben, valamint a Képmegjelenítő sandbox oldalsávjában konfigurálhatók.
{% endhint %}

**Egyéni index létrehozása:**

1.**Nyissa meg a Projektbeállításokat** (a feldolgozás előtt) vagy az Image Viewer sandbox oldalsávját
2. Keresse meg az **Indexképlet legördülő menüt**

3. Keresse meg a**„Egyéni”** opciót (be kell jelentkeznie Chloros+ licenccel)
4. **Határozza meg a képletet** sávváltozók használatával:
   * Sávnevek: `NIR`, `Red`, `Green`, `Blue`, `RedEdge` stb.
   * Műveletek: `+`, `-`, `*`, `/`, `^` (exponens)
   * Funkciók: `sqrt()`, `abs()` stb. (ha támogatott)
   * Zárójelek: `()` a műveletek sorrendjéhez
5. **Nevezze el az indexet** (pl. „MyIndex” vagy „CustomNDVI”)
6. **Mentse el a konfigurációt**

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

{% hint style="warning" %}
**Képlet-ellenőrzés**: Győződjön meg arról, hogy a képlete a kamerájában elérhető sávokat használja. Például az RedEdge csak olyan kamerákon érhető el, amelyek RedEdge szűrővel rendelkeznek.
{% endhint %}

***

## Következő lépések

Most, hogy megértette az Index/LUT Sandbox működését:

* **Alkalmazás a feldolgozás során**: Használja a felfedezett beállításokat a [Projektbeállítások](../project-settings/project-settings.md) menüpontban
* **Tömeges feldolgozás**: Alkalmazzon optimalizált indexeket a teljes adatkészletekre
* **További információk**: Olvassa el a [Multispektrális indexképletek](../project-settings/multispectral-index-formulas.md) című részt

Kapcsolódó dokumentáció:

* [**Képrétegek**](image-layers.md) - Rétegek kezelése és megjelenítése
* [**Kép teljes képernyős megnyitása**](opening-an-image-full-screen.md) - A képnézegető alapjai
* [**Képek feldolgozása (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Teljes feldolgozási munkafolyamat
