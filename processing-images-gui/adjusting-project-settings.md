# Projektbeállítások módosítása

A képek feldolgozása előtt fontos, hogy a projektbeállításokat a munkafolyamat követelményeinek megfelelően konfigurálja. A Projektbeállítások <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> panel átfogó ellenőrzést biztosít a kalibrálás, a feldolgozási opciók, a multispektrális indexek és az exportformátumok felett.

## A projektbeállítások elérés

1. Nyissa meg a projektet az Chloros
2. Kattintson a **Projektbeállítások** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> ikonra a bal oldali sávban
3. A Projektbeállítások panel minden konfigurációs opciót megjeleníti

{% hint style="info" %}
**A beállítások automatikusan mentésre kerülnek** a projekttel együtt. Amikor újra megnyit egy projektet, minden beállítás visszaáll.
{% endhint %}

***

## Gyors beállítás gyakori munkafolyamatokhoz

### Alapértelmezett beállítások (a legtöbb felhasználó számára ajánlott)

A tipikus MAPIR Survey3 kamera munkafolyamatokhoz az alapértelmezett beállítások jól működnek:

* ✅ **Vignette korrekció**: Engedélyezve
* ✅ **Reflektancia kalibrálás**: Engedélyezve (MAPIR célok képei szükségesek)
* ✅ **Debayer-módszer**: Standard (gyors, közepes minőség)
* ✅ **Export formátum**: TIFF (16 bites)

Egyszerűen importálja a képeket, és kezdje el a feldolgozást ezekkel az alapértelmezett beállításokkal.

***

## Projektbeállítások áttekintése

A Projektbeállítások panel több kategóriára van felosztva. Az alábbiakban összefoglaljuk az egyes szakaszokat. A teljes dokumentációt lásd: [Projektbeállítások](../project-settings/project-settings.md).

### Célfelismerés

Ez szabályozza, hogy az Chloros hogyan azonosítja a kalibrációs célokat a képeken.

**Főbb beállítások:*** **Minimális kalibrációs mintaterület**: A célfelismerés méretküszöbértéke (alapértelmezett: 25 pixel)
* **Minimális célcsoportosítás**: A célterületek csoportosításának hasonlósági küszöbértéke (alapértelmezett: 60)**Mikor kell módosítani:**

* Növelje a minta területét, ha téves észlelések történnek.
* Csökkentse, ha a célpontok nem kerülnek észlelésre.
* Állítsa be a csoportosítást, ha a célpontok több észlelésre oszlanak.

### Feldolgozás

Fő képfeldolgozási és kalibrációs beállítások.

**Főbb beállítások:*** **Vignette-korrekció**: Kompenzálja a lencse sötétedését a széleken ✅ Ajánlott
* **Reflektancia-kalibráció**: A kalibrációs célok segítségével normalizálja az értékeket ✅ Ajánlott
* **Debayer-módszer**: Algoritmus a RAW 3-csatornás multispektrális formátumba konvertálásához
* **Minimális újrakalibrálási intervallum**: A kalibrációs célok használata közötti idő (0 = mindet használja)**Speciális beállítások:*** **Fényérzékelő időzóna eltolása**: PPK időszinkronizáláshoz (alapértelmezett: 0)
* **PPK-korrekciók alkalmazása**: A .daq fájlokból származó GPS/expozíciós pin adatokat használja
* **Expozíciós pin 1/2**: A kamerákat expozíciós pinekhez rendeli a kettős kamera beállításokhoz

### Debayer-módszer

Jelenleg 2 debayering módszert kínálunk az Chloros-ben:

#### Standard (gyors, közepes minőség)

A standard debayer gyorsan feldolgozza az adatokat, de debayering színes zajt eredményez, ami kevésbé pontos és zajosabb képeket eredményez.

#### Textúraérzékeny (lassú, legmagasabb minőség) \[Chloros+ csak]

A textúraérzékeny módszer egy magas minőségű, élekérzékeny debayert használ, amelyhez egy AI/ML zajszűrési modell társul, amely szinte az összes debayering zajt eltávolítja. A textúraérzékeny modell futtatásához GPU memória (VRAM) szükséges. A gyorsabb feldolgozás érdekében azt javasoljuk, hogy csak akkor használja, ha rendelkezésre áll &gt;4 GB VRAM.

### Index (multispektrális indexek)

Állítsa be, hogy mely növényzetindexeket kívánja kiszámítani és exportálni.

**Indexek hozzáadása:**

1. Kattintson az**„Index hozzáadása”** gombra
2. Válasszon egy indexet a legördülő menüből (NDVI, NDRE, GNDVI stb.)
3. Konfigurálja a megjelenítési beállításokat (LUT színek, értéktartományok)
4. Szükség szerint adjon hozzá több indexet

**Népszerű indexek:*** **NDVI**: Általános növényállomány-egészség (a leggyakoribb)
* **NDRE**: Korai stresszfelismerés az RedEdge segítségével
* **GNDVI**: Klorofillkoncentrációra érzékeny
* **OSAVI**: Jól működik látható talajjal
* **EVI**: Magas levélterületi indexű (LAI) régiók**Egyéni képletek (csak Chloros+):**

* Egyedi multispektrális index képletek létrehozása
* Sávmatematika használata az összes képcsatornával
* Egyedi képletek mentése újrafelhasználás céljából

Az összes elérhető index és képlet megtalálható a [Multispektrális index képletek](../project-settings/multispectral-index-formulas.md) oldalon.

### Exportálás

A kimeneti fájl formátumának és minőségének vezérlése.

**Elérhető formátumok:*** **TIFF (16 bites)**: Ajánlott GIS és tudományos elemzésekhez (0–65 535 tartomány)
* **TIFF (32 bites, százalék)**: Lebegőpontos visszaverődési értékek (0,0–1,0 tartomány)
* **PNG (8 bites)**: Veszteségmentes tömörítés vizualizáláshoz (0–255 tartomány)
* **JPG (8 bites)**: Legkisebb fájlok, veszteséges tömörítés (0–255 tartomány)***

## Beállítások mentése és betöltése

### Projekt sablon mentése

Hozzon létre újrafelhasználható sablonokat a konzisztens munkafolyamatokhoz:

1. Konfigurálja az összes kívánt beállítást a Projektbeállítások panelen.
2. Görgessen le az alján található **„Projekt sablon mentése”** szakaszhoz.
3. Adjon meg egy leíró sablonnevet (pl. „Survey3N\_RGN\_Agriculture”).
4. Kattintson a mentés ikonra.

**Előnyök:**

* Azonos beállítások alkalmazása több projektben
* A konfigurációk megosztása a csapat tagjaival
* Az ismételt felmérések konzisztenciájának fenntartása

### Sablon betöltése új projektbe

Új projekt létrehozásakor:

1. Válassza a főmenüből az **„Új projekt”** lehetőséget.
2. Válassza a **„Betöltés sablonból”** lehetőséget.
3. Válassza ki a mentett sablont.
4. Az összes beállítás automatikusan alkalmazásra kerül.

### Munkakönyvtár

A **„Projektmappa mentése”** beállítás határozza meg, hogy alapértelmezés szerint hol jönnek létre az új projektek:

* **Alapértelmezett hely**: `C:\Users\[Username]\Chloros Projects`
* **Hely módosítása**: Kattintson a szerkesztés ikonra, és válassza ki az új mappát
* **Mikor érdemes megváltoztatni**:
  * Hálózati meghajtó csapatmunkához
  * Másik meghajtó, amely több tárhelyet biztosít
  * Év/ügyfél szerint szervezett mappaszerkezet

***

## PPK (utólag feldolgozott kinematika) beállítás

Ha MAPIR DAQ rögzítőket használ GPS-szel a pontos földrajzi helymeghatározáshoz:

### Előfeltételek

* MAPIR DAQ GPS (GNSS) modullal
* .daq naplófájl expozíciós pin bejegyzésekkel
* A kamera a DAQ expozíciós pinjeihez csatlakoztatva a rögzítési munkamenet alatt

### Konfigurációs lépések

1. Helyezze a .daq naplófájlt a projektmappájába
2. A Projektbeállításokban jelölje be az **„PPK korrekciók alkalmazása”** jelölőnégyzetet
3. Szükség esetén állítsa be a **„Fényérzékelő időzóna eltolása”** értéket (alapértelmezett: 0 UTC esetén)
4. Rendelje hozzá a kamerákat az expozíciós csapokhoz:
   * **Egyetlen kamera**: automatikusan a 1. csaphoz rendelve
   * **Két kamera**: manuálisan rendelje hozzá az egyes kamerákat a megfelelő csapokhoz**Expozíciós csapok hozzárendelése:*** **Expozíciós csatlakozó 1**: Válassza ki a kamera modellt a legördülő menüből
* **Expozíciós csatlakozó 2**: Válassza ki a második kamerát vagy a „Ne használja” opciót
* Ugyanaz a kamera nem rendelhető mindkét csatlakozóhoz

{% hint style="warning" %}
**Fontos**: Az expozíciós csatlakozókat helyesen kell hozzárendelni a megfelelő kamerákhoz. A helytelen hozzárendelés hibás földrajzi helyadatokhoz vezet.
{% endhint %}

***

## Haladó forgatókönyvek

### Többkamerás projektek

Ha egy projektben több MAPIR kamera képét dolgozza fel:

1. Az Chloros automatikusan felismeri az egyes kameramodelleket
2. Minden kamera a megfelelő feldolgozási profilt kapja
3. PPK: Manuálisan rendelje hozzá az egyes kamerákat a megfelelő expozíciós csaphoz
4. Az összes kamera ugyanazt az exportformátumot és indexeket használja

**Példa**: Survey3W RGN + Survey3N OCN kettős kamerás szerelvény

### Idősoros vagy több dátumú felmérések

Ugyanazon terület időbeli ismételt felméréseihez:

1. Hozzon létre egy sablont a standard beállításaival.
2. Minden munkamenetben használjon konzisztens kalibrációs célbeállítást.
3. Minden dátumot külön projektként dolgozzon fel.
4. Az összehasonlítható eredmények érdekében használjon azonos beállításokat.
5. Az időbeli elemzéshez ugyanabban a formátumban exportáljon.

### Nagy adatállományok

Sok képet (500+) tartalmazó projektek esetén:

* Fontolja meg a projektek kisebb projektekre bontását dátum vagy terület szerint.
* Használja az Chloros+ párhuzamos feldolgozást a gyorsabb eredmények érdekében.
* Fontolja meg az CLI vagy az API használatát a kötegelt automatizáláshoz.
* Állítsa be a minimális újrakalibrálási intervallumot a célpontok észlelési idejének csökkentése érdekében.

***

## A beállítások ellenőrzése

A feldolgozás megkezdése előtt ellenőrizze a következő fontos beállításokat:

* [ ] A kamera modellje helyesen lett felismerve a Fájlkezelőben
* [ ] A vignettázás korrekció engedélyezve
* [ ] A visszaverődés kalibrálása engedélyezve
* [ ] Legalább egy kalibrációs célkép importálva
* [ ] A kívánt multispektrális indexek hozzáadva
* [ ] A munkafolyamatához megfelelő exportformátum
* [ ] PPK beállítások konfigurálva (ha .daq fájlt használ expozíciós eseményekkel)

***

## Következő lépések

A beállítások konfigurálása után:

1. **Jelölje meg a kalibrációs célképeket** – Lásd: [Célképek kiválasztása](choosing-target-images.md)
2. **Indítsa el a feldolgozást** – Lásd: [A feldolgozás elindítása](starting-the-processing.md)
3. **Figyelje a folyamatot** – Lásd [A feldolgozás figyelése](monitoring-the-processing.md)

Az összes rendelkezésre álló beállítás részletes leírását lásd a [Projektbeállítások](../project-settings/project-settings.md) referencia dokumentációban.
