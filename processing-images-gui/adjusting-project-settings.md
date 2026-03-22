# A projektbeállítások módosítása

A képek feldolgozása előtt fontos, hogy a projektbeállításokat a munkafolyamat követelményeinek megfelelően konfigurálja. A Projektbeállítások <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> panel átfogó vezérlést biztosít a kalibrálás, a feldolgozási opciók, a multispektrális indexek és az exportformátumok felett.

## A projektbeállítások eléréséhez

1. Nyissa meg a projektet az Chloros programban
2. Kattintson a **Projektbeállítások** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> ikonra a bal oldali sávban
3. A Projektbeállítások panel megjeleníti az összes konfigurációs opciót

{% hint style="info" %}
**A beállítások automatikusan mentésre kerülnek** a projekttel együtt. Amikor újra megnyit egy projektet, az összes beállítás visszaáll.
{% endhint %}

***

## Gyors beállítás általános munkafolyamatokhoz

### Alapbeállítások (a legtöbb felhasználó számára ajánlott)

A tipikus MAPIR Survey3 kamerás munkafolyamatokhoz az alapbeállítások jól működnek:

* ✅ **Vignette-korrekció**: Engedélyezve
* ✅ **Reflektancia-kalibrálás**: Engedélyezve (MAPIR célpontok képei szükségesek)
* ✅ **Debayer-módszer**: Standard (Gyors, Közepes minőség)
* ✅ **Exportálási formátum**: TIFF (16 bites)

Egyszerűen importálja a képeit, és kezdje el a feldolgozást ezekkel az alapértelmezett beállításokkal.

***

## A Projektbeállítások áttekintése

A Projektbeállítások panel több kategóriára van felosztva. Az alábbiakban összefoglaljuk az egyes szakaszokat. A teljes dokumentációt lásd a [Projektbeállítások](../project-settings/project-settings.md) alatt.

### Célpontfelismerés

Szabályozza, hogy az Chloros hogyan azonosítja a kalibrációs célpontokat a képein.

**Főbb beállítások:*** **Minimális kalibrációs mintafelület**: A célpontfelismerés méretküszöbértéke (alapértelmezett: 25 pixel)
* **Minimális célpont-csoportosítás**: A célterületek csoportosításának hasonlósági küszöbértéke (alapértelmezett: 60)**Mikor kell módosítani:**

* Növelje a minta területét, ha téves észleléseket kap
* Csökkentse, ha a célpontok nem kerülnek észlelésre
* Állítsa be az összevonást, ha a célpontok több észlelésre oszlanak

### Feldolgozás

Főbb képfeldolgozási és kalibrációs beállítások.

**Főbb beállítások:*** **Vignette korrekció**: Kompenzálja a lencse sötétedését a széleken ✅ Ajánlott
* **Reflektancia kalibráció**: Normalizálja az értékeket a kalibrációs célpontok segítségével ✅ Ajánlott
* **Debayer-módszer**: Algoritmus a RAW-fájlok 3-csatornás multispektrális formátumba konvertálásához
* **Minimális újrakalibrálási intervallum**: A kalibrációs célpontok használata közötti idő (0 = mindet használja)**Speciális beállítások:*** **Fényérzékelő időzóna eltolása**: PPK időszinkronizáláshoz (alapértelmezett: 0)
* **PPK-korrekciók alkalmazása**: A .daq fájlokból származó GPS/expozíciós pin adatokat használja
* **Expozíciós pin 1/2**: Kamerákat rendel az expozíciós pinekhez kettős kamerás felállások esetén

### Debayer-módszer

Jelenleg 2 debayering-módszert kínálunk az Chloros-ben:

#### Standard (Gyors, Közepes minőség)

A standard debayer gyorsan dolgozik, de debayering színzajt mutat, ami kevésbé pontos és zajosabb képeket eredményez.

#### Textúraérzékeny (lassú, legmagasabb minőség) \[Csak Chloros+]

A textúraérzékeny módszer egy kiváló minőségű, élekkel számoló debayert használ, amelyhez egy AI/ML zajszűrő modell társul, amely szinte az összes debayering zajt eltávolítja. A Textúraérzékeny modell futtatásához GPU-memória (VRAM) szükséges. A gyorsabb feldolgozás érdekében azt javasoljuk, hogy akkor használja, ha &gt;4 GB VRAM áll rendelkezésre.

### Index (Multispektrális indexek)

Állítsa be, mely növényzetindexeket kívánja kiszámítani és exportálni.

**Indexek hozzáadása:**

1. Kattintson az**„Index hozzáadása”** gombra
2. Válasszon ki egy indexet a legördülő menüből (NDVI, NDRE, GNDVI stb.)
3. Konfigurálja a megjelenítési beállításokat (LUT színek, értéktartományok)
4. Szükség szerint adjon hozzá több indexet

**Népszerű indexek:*** **NDVI**: Általános növényállapot (a leggyakoribb)
* **NDRE**: Korai stresszfelismerés az RedEdge-szel
* **GNDVI**: Klorofillkoncentráció-érzékeny
* **OSAVI**: Jól működik látható talajjal
* **EVI**: Magas levélfelületi indexű (LAI) régiók**Egyéni képletek (csak Chloros+):**

* Egyéni multispektrális indexképletek létrehozása
* Sávszámítás használata az összes képcsatornával
* Egyéni képletek mentése újrafelhasználás céljából

Az összes elérhető index és képlet megtekintéséhez lásd [Multispektrális indexképletek](../project-settings/multispectral-index-formulas.md).

### Export

A kimeneti fájl formátumának és minőségének szabályozása.

**Elérhető formátumok:*** **TIFF (16 bites)**: Ajánlott GIS-hez és tudományos elemzésekhez (0–65 535 tartomány)
* **TIFF (32 bites, százalék)**: Lebegőpontos reflexiós értékek (0,0–1,0 tartomány)
* **PNG (8 bites)**: Veszteségmentes tömörítés vizualizáláshoz (0–255 tartomány)
* **JPG (8 bites)**: A legkisebb fájlok, veszteséges tömörítés (0–255 tartomány)***

## Beállítások mentése és betöltése

### Projekt sablon mentése

Hozzon létre újrafelhasználható sablonokat az egységes munkafolyamatok érdekében:

1. Konfigurálja az összes kívánt beállítást a Projektbeállítások panelen
2. Görgessen le az alján található **„Projektsablon mentése”** szakaszhoz
3. Adjon meg egy leíró sablonnevet (pl. „Survey3N\_RGN\_Agriculture”)
4. Kattintson a mentés ikonra

**Előnyök:**

* Azonos beállítások alkalmazása több projektben
* A konfigurációk megosztása a csapattagokkal
* Az ismételt felmérések konzisztenciájának fenntartása

### Sablon betöltése új projektbe

Új projekt létrehozásakor:

1. Válassza a **„Új projekt”** menüpontot a főmenüből
2. Válassza a **„Betöltés sablonból”** opciót
3. Válassza ki a mentett sablont
4. Az összes beállítás automatikusan alkalmazásra kerül

### Munkakönyvtár

A **&quot;Projektmappa mentése&quot;** beállítás határozza meg, hogy alapértelmezés szerint hol jönnek létre az új projektek:

* **Alapértelmezett hely**: `C:\Users\[Username]\Chloros Projects`
* **Hely megváltoztatása**: Kattintson a szerkesztés ikonra, és válassza ki az új mappát
* **Mikor érdemes megváltoztatni**:
  * Hálózati meghajtó csapatmunkához
  * Másik meghajtó, nagyobb tárhellyel
  * Év/ügyfél szerint rendezett mappaszerkezet

***

## PPK (utólagosan feldolgozott kinematikus) beállítás

Ha MAPIR DAQ rögzítőket használ GPS-szel a pontos földrajzi helymeghatározáshoz:

### Előfeltételek

* MAPIR DAQ GPS (GNSS) modullal
* .daq naplófájl expozíciós pin bejegyzésekkel
* A rögzítési munkamenet alatt a DAQ expozíciós pinjeihez csatlakoztatott kamera

### Beállítási lépések

1. Helyezze a .daq naplófájlt a projektmappájába
2. A Projektbeállításokban jelölje be az **„PPK-korrekciók alkalmazása”** jelölőnégyzetet
3. Szükség esetén állítsa be a **„Fényérzékelő időzóna eltolása”** értéket (alapértelmezett: 0 az UTC-hez)
4. Rendelje hozzá a kamerákat az expozíciós csatlakozókhoz:
   * **Egy kamera**: automatikusan a 1. csatlakozóhoz rendelődik
   * **Két kamera**: manuálisan rendelje hozzá az egyes kamerákat a megfelelő csatlakozókhoz**Expozíciós csatlakozók hozzárendelése:*** **

1. expozíciós pin**: Válassza ki a kameramodellt a legördülő menüből
* **

2. expozíciós pin**: Válassza ki a második kamerát vagy a „Ne használja” opciót
* Ugyanaz a kamera nem rendelhető mindkét pinhez

{% hint style="warning" %}
**Fontos**: Az expozíciós pineket helyesen kell hozzárendelni a megfelelő kamerákhoz. A helytelen hozzárendelés hibás földrajzi helyadatokhoz vezet.
{% endhint %}

***

## Speciális esetek

### Többkamerás projektek

Ha egy projektben több MAPIR kamerából származó képeket dolgoz fel:

1. Az Chloros automatikusan felismeri az egyes kameramodelleket
2. Minden kamera megkapja a megfelelő feldolgozási profilt
3. PPK: Manuálisan rendelje hozzá az egyes kamerákat a megfelelő expozíciós csaphoz
4. Minden kamera ugyanazt az exportformátumot és indexeket használja

**Példa**: Survey3W RGN + Survey3N OCN kétkamerás rendszer

### Idősoros vagy több dátumot tartalmazó felmérések

Ugyanazon terület időbeli ismételt felméréseihez:

1. Készítsen sablont a szokásos beállításokkal
2. Minden munkamenetben használjon egységes kalibrációs célpont-beállítást
3. Minden dátumot külön projektként dolgozzon fel
4. Az összehasonlítható eredmények érdekében használjon azonos beállításokat
5. Az időbeli elemzéshez ugyanabban a formátumban exportáljon

### Nagy adathalmazok

Sok képet (500+) tartalmazó projektek esetén:

* Fontolja meg a kisebb projektekre való felosztást dátum vagy terület szerint
* Használja az Chloros+ párhuzamos feldolgozást a gyorsabb eredmények érdekében
* Fontolja meg az CLI vagy az API használatát a kötegelt automatizáláshoz
* Állítsa be a minimális újrakalibrálási intervallumot a célpont-felismerési idő csökkentése érdekében

***

## A beállítások ellenőrzése

A feldolgozás megkezdése előtt ellenőrizze a következő kulcsfontosságú beállításokat:

* [ ] A kamera modellje helyesen felismerve a Fájlkezelőben
* [ ] Vignett-korrekció engedélyezve
* [ ] Reflektancia-kalibrálás engedélyezve
* [ ] Legalább egy kalibrációs célkép importálva
* [ ] A kívánt multispektrális indexek hozzáadva
* [ ] A munkafolyamatához megfelelő exportformátum
* [ ] PPK-beállítások konfigurálva (ha .daq fájlt használ expozíciós eseményekkel)

***

## Következő lépések

A beállítások konfigurálása után:

1. **Jelölje meg a kalibrációs célképeket** – Lásd: [Célképek kiválasztása](choosing-target-images.md)
2. **Indítsa el a feldolgozást** – Lásd: [A feldolgozás elindítása](starting-the-processing.md)
3. **Kövesse nyomon a folyamatot** – Lásd: [A feldolgozás nyomon követése](monitoring-the-processing.md)

Az összes rendelkezésre álló beállításról részletes információkat a [Projektbeállítások](../project-settings/project-settings.md) referencia-dokumentációban talál.
