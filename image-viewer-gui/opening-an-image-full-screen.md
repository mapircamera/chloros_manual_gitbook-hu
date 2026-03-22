# Kép teljes képernyős megnyitása

Az Chloros Képmegjelenítő egy speciális, teljes képernyős felületet biztosít a multispektrális képek megtekintéséhez, elemzéséhez és szerkesztéséhez. Akár eredeti képeket, akár feldolgozott eredményeket tekint meg, a Képmegjelenítő hatékony eszközöket kínál a vizsgálathoz és az elemzéshez.

## Az Image Viewer elérés

### A Fájlkezelőből

A kép megnyitásának leggyakoribb módja az Image Viewerben:

1. Győződjön meg róla, hogy a **Fájlkezelő** fülön van <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Kattintson bármelyik **képminiatűrre** a képrácsban
3. A kép megnyílik a **fő előnézeti területen** (a képernyő közepén)
4. A kép betöltődött, és készen áll a teljes képernyős megtekintésre

### Az Image Viewer fül megnyitása

Miután a kép betöltődött az előnézeti területre:

1. Kattintson az **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> ikonra a bal oldali sávban
2. Megnyílik a Képmegjelenítő fül, amely teljes képernyőn jeleníti meg a kiválasztott képet
3. A bal oldali sávban elérhetővé válnak a fejlett megtekintési és elemzési eszközök

***

## A Képmegjelenítő felületének áttekintése

### Fő megjelenítési terület

A képernyő legnagyobb része a képet jeleníti meg:

* **Teljes felbontás**: A képek natív felbontásban jelennek meg
* **Nagyítható**: A nagyításhoz használja a vezérlőket vagy az egérgörgőt
* **Körbeforgatható**: Nagyításkor kattintson és húzza az egérrel a kép mozgatásához
* **Képarány megőrzése**: A képek arányosan méretezhetők***

## Megtekintési beállítások

### Alapvető képnavigáció

#### Képek között való böngészés

A képkészletben a billentyűparancsok vagy a gombok segítségével navigálhat:

* **Következő kép**: Kattintson a → gombra, vagy nyomja meg a**→** (jobb nyíl) billentyűt
* **Előző kép**: Kattintson a ← gombra, vagy nyomja meg a**←** (bal nyíl) billentyűt
* **Ugrás egy adott képre**: Térjen vissza a Fájlböngészőbe, és kattintson a kívánt miniatűrre

#### Nagyítási vezérlők

Állítsa be a nagyítást a kép részleteinek megtekintéséhez:

**Nagyítás:*** Kattintson a **+** (plusz) gombra
* Nyomja meg a **+**vagy**=** billentyűt
* Görgessen az egérkerékkel **felfelé**

**Kicsinyítés:*** Kattintson a **−** (mínusz) gombra
* Nyomja meg a **−** (mínusz) billentyűt
* Görgessen az egérkerékkel **lefelé**

#### Képmozgatás nagyításkor

Ha a képet a képernyő méreténél nagyobb méretre nagyította:

1. Vigye az egérmutatót a kép fölé
2. Kattintson és **tartsa lenyomva a bal egérgombot**

3.**Húzza** az egeret a kép mozgatásához
4. Engedje fel a gombot a képmozgatás leállításához

**Alternatív megoldás**: Használja a nyílgombokat a kép apró lépésekben történő mozgatásához***

## Pixelértékek megtekintése

### Pixelértékek megtekintése a kurzor alatt

Amint az egérkurzort a kép fölé viszi, a pixelértékek valós időben jelennek meg:**Az értékek megjelenítési helye:*** **Lebegő szám és piros vonal a jobb oldali index LUT gradiens legenda mellett*** **Ha tovább nagyít, lebegő érték a kurzor és a kijelölt pixel közelében*** Megmutatja a **kurzor alatt vagy kijelölt** pixel értékeit
* Az egér mozgatásával frissül

***

## Megtekinthető képformátumok

### JPG

**JPG képek a fényképezőgépről:**

* A JPG adatok megjelenítése előnézetként
* Az eredeti, korrekció nélküli értékek megjelenítése
* Hasznos a képminőség ellenőrzéséhez a feldolgozás előtt

### RAW (Eredeti)

### RAW (Fényvisszaverődés)

**Feldolgozás után:**

* Vignettálás korrigálva
* Reflektancia kalibrálva
* Többsávos TIFF (Red, Green, NIR stb.)
* Elemzésre kész tudományos adatok

### RAW (Index)

**NDVI, NDRE, GNDVI stb. (\_NDVI.tif fájlok):**

* Egycsatornás szürkeárnyalatos képek
* A képpontértékek az indexszámítás eredményeit jelzik
* A normalizált indexek értéke általában -1 és +1 között mozog
* A vizualizáláshoz színes LUT-ok alkalmazhatók

***

## Index és LUT alkalmazás

Multispektrális indexek és színes Look-Up Tables alkalmazása:

1. Keresse meg az **Index/LUT Sandbox**elemet az**Image Viewer**-ben <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> oldalsávon
2. Válassza ki a vegetációs indexet (NDVI, NDRE stb.)
3. Válasszon multispektrális képletet, vagy hozzon létre saját egyéni képletet (csak Chloros+)
4. Alkalmazzon színes LUT-gradienst a vizualizáláshoz
5. Állítsa be az értéktartományokat és a küszöbértékeket

Részletes utasításokat az [Index/LUT Sandbox](index-lut-sandbox.md) oldalon talál.

***

## Billentyűparancsok

### Navigáció

* **→** (Jobb nyíl): Következő kép
* **←** (Bal nyíl): Előző kép
* **Home**: A lista első képe
* **End**: A lista utolsó képe

### Nagyítás

* **+**vagy**=**: Nagyítás
* **−**: Kicsinyítés
* **Egérkerék**: Nagyítás/kicsinyítés***

### Az indexszámítások ellenőrzése

Ellenőrizze, hogy az indexek helyesen lettek-e kiszámítva:

1. Nyissa meg az NDVI vagy más indexképet
2. Ellenőrizze a növényzet területeit:
   * **NDVI**: Egészséges növények esetén 0,4–0,9 értéket kell mutatnia
   * **NDRE**: Erőteljes növekedés esetén magasabb értékeket mutat
   * **GNDVI**: Hasonló az NDVI-hez, de klorofill-érzékeny
3. Ellenőrizze a nem növényzetet:
   * **Talaj**: 0 közelében vagy enyhén negatív
   * **Víz**: Negatív értékek (-0,5 és 0 között)***

## Megjelenítési problémák elhárítása

### A kép nem nyílik meg

**Lehetséges okok:**

* A fájl feldolgozás közben megsérült
* Nem támogatott fájlformátum
* Nincs elegendő memória a nagy képhez

**Megoldások:**

1. Próbálja meg megnyitni egy külső nézőprogramban, hogy ellenőrizze a fájl integritását
2. Ellenőrizze, hogy a fájlformátum megfelel-e a várt típusnak
3. Zárjon be más alkalmazásokat a memória felszabadítása érdekében
4. Próbálkozzon kisebb/más képekkel

### Fekete vagy fehér kép megjelenítése

**Lehetséges okok:**

* A megjelenítési képességeken kívüli értéktartomány
* Szokatlan értékeket tartalmazó 32 bites lebegőpontos kép
* Indexszámítási hiba

**Megoldások:**

1. Ellenőrizze a pixelértékeket – ha mind nagyon alacsonyak vagy nagyon magasak, állítsa be a megjelenítési tartományt
2. Próbálja meg megnyitni a QGIS-ben vagy hasonló programban, automatikus tartománybeállítással
3. Ellenőrizze a feldolgozás hibajelentését

### A pixelértékek helytelennek tűnnek

**Lehetséges okok:**

* Rossz kép megtekintése (eredeti vs. feldolgozott)
* A kalibrálás nem sikerült megfelelően
* A fényérzékelő adatai nem kerültek be a bemenetbe
* A százalékos mód helytelenül van beállítva

**Megoldások:**

1. Ellenőrizze, hogy a feldolgozott kimenetet nézi-e (ellenőrizze a fájlnév kiterjesztését)
2. Ellenőrizze a százalékos mód gomb állapotát
3. Hasonlítsa össze ugyanazon adatkészletből származó, ismert jó képekkel

***

## Következő lépések

Most, hogy már teljes képernyőn tudja megtekinteni a képeket:

* [**Képrétegek**](image-layers.md) – Ismerje meg a többsávos megjelenítést
* [**Index/LUT Sandbox**](index-lut-sandbox.md) – Alkalmazzon egyéni indexeket és színleképezést
* [**Multispektrális indexképletek**](../project-settings/multispectral-index-formulas.md) – Ismerje meg a rendelkezésre álló indexeket

A feldolgozási munkafolyamatot lásd:

* [**Képek feldolgozása (GUI)**](../processing-images-gui/adding-files-to-a-project.md) – Teljes feldolgozási útmutató
