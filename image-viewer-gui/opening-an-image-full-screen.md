# Kép teljes képernyős megnyitása

Az Chloros Image Viewer egy speciális teljes képernyős felületet biztosít a multispektrális képek megtekintéséhez, elemzéséhez és manipulálásához. Akár eredeti képeket, akár feldolgozott kimeneteket tekint meg, az Image Viewer hatékony eszközöket kínál a vizsgálathoz és elemzéshez.

## Hozzáférés a képnézegetőhöz

### A fájlböngészőből

A képnézegetőben a kép megnyitásának leggyakoribb módja:

1. Győződjön meg arról, hogy a **Fájlböngésző** fülön van. <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Kattintson bármelyik **képminiatűrre** a képrácsban
3. A kép megnyílik a **fő előnézeti területen** (a képernyő közepén)
4. A kép most már betöltődött és készen áll a teljes képernyős megtekintésre

### Az Image Viewer fül megnyitása

Miután a kép betöltődött az előnézeti területen:

1. Kattintson az **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> ikonra a bal oldali sávban.
2. Megnyílik a Képnézegető fül, amely teljes képernyős nézetben jeleníti meg a kiválasztott képet.
3. A bal oldali sávban megjelennek a fejlett megtekintési és elemzési eszközök.

***

## A Képnézegető felületének áttekintése

### Fő megjelenítési terület

A képernyő legnagyobb részén a kép látható:

* **Teljes felbontás**: A képek natív felbontásban jelennek meg.
* **Nagyítható**: A nagyításhoz használja a vezérlőket vagy az egér görgőjét
* **Pannable**: Nagyításkor kattintson és húzza az egérmutatót a kép mozgatásához
* **Képarány megőrzése**: A képek arányosan méretezhetők

***

## Megtekintési beállítások

### Alapvető képnavigáció

#### Képek böngészése

A képek között billentyűparancsokkal vagy gombokkal navigálhat:

* **Következő kép**: Kattintson a → gombra, vagy nyomja meg a **→** (jobb nyíl) billentyűt
* **Előző kép**: Kattintson a ← gombra, vagy nyomja meg a **←** (bal nyíl) billentyűt
* **Ugrás egy adott képre**: Térjen vissza a Fájlböngészőbe, és kattintson a kívánt miniatűrre

#### Nagyítási vezérlők

Állítsa be a nagyítást a kép részleteinek megtekintéséhez:

**Nagyítás:**

* Kattintson a **+** (plusz) gombra.
* Nyomja meg a **+** vagy **=** billentyűt.
* Görgessen az egérkerékkel **felfelé**.

**Kicsinyítés:**

* Kattintson a **−** (mínusz) gombra.
* Nyomja meg a **−** (mínusz) billentyűt.
* Görgessen az egérkerékkel **lefelé**.

**Képernyőhöz igazítás:**

* Kattintson a **↔** (Igazítás) gombra.
* Nyomja meg a **0** (nulla) billentyűt.
* Kattintson duplán a képre.

#### Nagyításkor történő mozgatás

Ha a képet a képernyő méretén túl nagyítja:

1. Vigye az egérmutatót a kép fölé.
2. Kattintson, és **tartsa lenyomva a bal egérgombot**.
3. **Húzza** a képet a mozgatáshoz
4. Engedje el a gombot a pásztázás leállításához

**Alternatíva**: A nyílgombokkal apránként pásztázhat

***

## Pixelértékek ellenőrzése

### Pixelértékek megtekintése a kurzor alatt

Amikor az egérmutatót a kép fölé viszi, a pixelértékek valós időben megjelennek:

**Az értékek megjelenítési helye:**

* **Lebegő szám és piros vonal a jobb oldali index LUT gradiens legenda**
* **További nagyítás esetén a kurzor közelében lebegő érték és kiemelt pixel**
* Megjeleníti a kurzor **alatti vagy kiemelt** pixel értékeit
* Az egér mozgatásával frissül

***

## Megtekinthető képtípusok

### Eredeti képek (előfeldolgozás)

**RAW + JPG képek a fényképezőgépről:**

* A RAW adatok előnézetként jelennek meg
* Az eredeti, korrigálatlan értékek megjelenítése
* Hasznos a képminőség ellenőrzéséhez feldolgozás előtt

### Kalibrált reflektancia képek

**Feldolgozás után:**

* Vignette-korrekció
* Kalibrált reflektancia
* Többsávos TIFF (Red, Green, NIR stb.)
* Tudományos adatok elemzésre készen

### Indexképek

**NDVI, NDRE, GNDVI stb. (\_NDVI.tif fájlok):**

* Egycsatornás szürkeárnyalatos képek
* A pixelértékek az index számítási eredményeit képviselik
* A normalizált indexek tartománya általában -1 és +1 között van
* Színes LUT-ok alkalmazhatók a vizualizáláshoz

***

## Index és LUT alkalmazás

Többspektrális indexek és színes Look-Up táblák alkalmazása:

1. Keresse meg az **Index/LUT Sandbox** elemet az **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> oldalsávban
2. Válassza ki a vegetációs indexet (NDVI, NDRE stb.)
3. Válassza ki a multispektrális képletet, vagy hozzon létre saját egyéni képletet (csak Chloros+)
4. Alkalmazzon szín LUT gradienst a vizualizáláshoz
5. Állítsa be az értéktartományokat és a küszöbértékeket

Részletes utasításokat lásd az [Index/LUT Sandbox](index-lut-sandbox.md) oldalon.

***

## Billentyűparancsok

### Navigáció

* **→** (Jobb nyíl): Következő kép
* **←** (Bal nyíl): Előző kép
* **Home**: Az első kép a listában
* **End**: Utolsó kép a listában

### Nagyítás

* **+** vagy **=**: Nagyítás
* **−**: Kicsinyítés
* **0** (Nulla): Képernyőhöz igazítás
* **Egérkerék**: Nagyítás/kicsinyítés

### Megjelenítés vezérlése

* **P**: Pixel százalékos mód váltása
* **L**: Rétegek panel váltása
* **Esc**: Teljes képernyős mód bezárása vagy visszatérés a Fájlböngészőhöz

### Egyéb

* **Ctrl+S**: Aktuális kép mentése
* **F**: Teljes képernyős mód (ha elérhető)

***

### Indexszámítások ellenőrzése

Ellenőrizze, hogy az indexek helyesen lettek-e kiszámítva:

1. Nyissa meg az NDVI vagy más indexképet.
2. Ellenőrizze a növényzet területét:
   * **NDVI**: Egészséges növények esetén 0,4–0,9 értéket kell mutatnia.
   * **NDRE**: Erőteljes növekedés esetén magasabb értékek
   * **GNDVI**: Hasonló az NDVI-hez, de klorofillérzékeny
3. Ellenőrizze a nem növényzetet:
   * **Talaj**: 0 közelében vagy enyhén negatív
   * **Víz**: Negatív értékek (-0,5 és 0 között)

***

## Hibaelhárítás Megjelenítési problémák

### A kép nem nyílik meg

**Lehetséges okok:**

* A fájl feldolgozás közben megsérült
* Nem támogatott fájlformátum
* Nincs elegendő memória a nagy képekhez

**Megoldások:**

1. Próbálja meg megnyitni egy külső nézőprogrammal, hogy ellenőrizze a fájl integritását.
2. Ellenőrizze, hogy a fájlformátum megfelel-e a várt típusnak.
3. Zárjon be más alkalmazásokat, hogy memóriát szabadítson fel.
4. Próbáljon kisebb/másik képet.

### Fekete vagy fehér kép megjelenítése

**Lehetséges okok:**

* Az érték tartománya meghaladja a kijelző képességeit.
* 32 bites lebegőpontos kép szokatlan értékekkel.
* Index számítási hiba

**Megoldások:**

1. Ellenőrizze a pixelértékeket – ha mind nagyon alacsonyak vagy nagyon magasak, állítsa be a kijelzési tartományt.
2. Próbálja meg megnyitni a QGIS-ben vagy hasonló programban az automatikus tartománybeállítással.
3. Ellenőrizze a feldolgozás hibakeresési naplóját.

### A pixelértékek helytelennek tűnnek

**Lehetséges okok:**

* Helytelen kép megjelenítése (eredeti vs. feldolgozott)
* A kalibrálás nem megfelelően történt
* A fényérzékelő adatai nem kerültek be a bemenetbe
* A százalékos mód helytelenül váltott

**Megoldások:**

1. Ellenőrizze, hogy a feldolgozott kimenetet nézi-e (ellenőrizze a fájlnév kiterjesztését)
2. Ellenőrizze a százalékos mód gomb állapotát
3. Hasonlítsa össze ugyanazon adatkészletből származó, ismert jó képekkel

***

## Következő lépések

Most, hogy teljes képernyőn tudja megtekinteni a képeket:

* [**Képrétegek**](image-layers.md) – Ismerje meg a többsávos vizualizációt
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – Alkalmazzon egyéni indexeket és színleképezést
* [**Multispektrális index képletek**](../project-settings/multispectral-index-formulas.md) – Ismerje meg a rendelkezésre álló indexeket

A feldolgozási munkafolyamatot lásd:

* [**Képek feldolgozása (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Teljes feldolgozási útmutató
