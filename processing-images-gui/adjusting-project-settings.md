# Projektbeállítások módosítása

A képek feldolgozása előtt fontos, hogy a projektbeállításokat a munkafolyamat követelményeinek megfelelően konfigurálja. A Projektbeállítások <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> panel átfogó ellenőrzést biztosít a kalibrálás, a feldolgozási opciók, a multispektrális indexek és az exportformátumok felett.

## A projektbeállítások elérés

1. Nyissa meg a projektet az Chloros programban
2. Kattintson a **Projektbeállítások** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> ikonra a bal oldali sávban
3. A Projektbeállítások panel minden konfigurációs opciót megjeleníti

{% hint style=&quot;info&quot; %}
**A beállítások automatikusan mentésre kerülnek** a projekttel együtt. Amikor újra megnyitja a projektet, az összes beállítás visszaáll.
{% endhint %}

***

## Gyors beállítás a gyakori munkafolyamatokhoz

### Alapértelmezett beállítások (a legtöbb felhasználó számára ajánlott)

A tipikus MAPIR Survey3 kamera munkafolyamatokhoz az alapértelmezett beállítások jól működnek:

* ✅ **Vignette korrekció**: Engedélyezve
* ✅ **Reflektancia kalibrálás**: Engedélyezve (MAPIR célok képei szükségesek)
* ✅ **Debayer-módszer**: Kiváló minőség (gyorsabb)
* ✅ **Export formátum**: TIFF (16 bites)

Egyszerűen importálja a képeket, és kezdje el a feldolgozást ezekkel az alapértelmezett beállításokkal.

***

## Projektbeállítások áttekintése

A Projektbeállítások panel több kategóriára van felosztva. Az alábbiakban összefoglaljuk az egyes szakaszokat. A teljes dokumentációt lásd a [Projektbeállítások](../project-settings/project-settings.md) alatt.

### Célfelismerés

Ez szabályozza, hogy az Chloros hogyan azonosítja a kalibrációs célokat a képeken.

**Főbb beállítások:**

* **Minimális kalibrációs minta terület**: A célfelismerés méretküszöbe (alapértelmezett: 25 pixel)
* **Minimális célcsoportosítás**: A célterületek csoportosításának hasonlósági küszöbe (alapértelmezett: 60)

**Mikor kell módosítani:**

* Növelje a minta területét, ha téves felismerések történnek.
* Csökkentse, ha a célok nem kerülnek felismerésre.
* Módosítsa a csoportosítást, ha a célok több felismerésre oszlanak.

### Feldolgozás

Fő képfeldolgozási és kalibrálási beállítások.

**Főbb beállítások:**

* **Vignette-korrekció**: Kompenzálja a lencse sötétedését a széleken ✅ Ajánlott
* **Reflektancia-kalibrálás**: Normalizálja az értékeket a kalibrálási célok segítségével ✅ Ajánlott
* **Debayer-módszer**: Algoritmus a RAW 3-csatornás multispektrális formátumba konvertálásához
* **Minimális újrakalibrálási intervallum**: A kalibrációs célok használata közötti idő (0 = mindet használja)

**Speciális beállítások:**

* **Fényérzékelő időzóna eltolás**: PPK időszinkronizáláshoz (alapértelmezett: 0)
* **PPK korrekciók alkalmazása**: A .daq fájlokból származó GPS/expozíciós pin adatokat használja
* **Expozíciós pin 1/2**: kamerákat rendel az expozíciós pinekhez kettős kamera beállítások esetén

### Index (multispektrális indexek)

Konfigurálja, mely növényzetindexeket kívánja kiszámítani és exportálni.

**Indexek hozzáadása:**

1. Kattintson az **„Index hozzáadása”** gombra
2. Válasszon ki egy indexet a legördülő menüből (NDVI, NDRE, GNDVI stb.)
3. Konfigurálja a megjelenítési beállításokat (LUT színek, értéktartományok)
4. Szükség szerint adjon hozzá több indexet

**Népszerű indexek:**

* **NDVI**: Általános növényállomány-egészség (a leggyakoribb)
* **NDRE**: Korai stresszfelismerés az RedEdge segítségével
* **GNDVI**: Klorofillkoncentrációra érzékeny
* **OSAVI**: Jól működik látható talajjal
* **EVI**: Magas levélterület-indexű (LAI) régiók

**Egyéni képletek (csak Chloros+):**

* Egyedi multispektrális index képletek létrehozása
* Sávmatematika használata az összes képcsatornával
* Egyedi képletek mentése újrafelhasználás céljából

Az összes elérhető index és képlet megtalálható a [Multispektrális index képletek](../project-settings/multispectral-index-formulas.md) oldalon.

### Exportálás

A kimeneti fájl formátumának és minőségének vezérlése.

**Elérhető formátumok:**

* **TIFF (16 bites)**: Ajánlott GIS és tudományos elemzésekhez (0–65 535 tartomány)
* **TIFF (32 bites, százalék)**: Lebegőpontos visszaverődési értékek (0,0–1,0 tartomány)
* **PNG (8 bites)**: Veszteségmentes tömörítés vizualizáláshoz (0–255 tartomány)
* **JPG (8 bites)**: Legkisebb fájlok, veszteséges tömörítés (0–255 tartomány)

***

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

A **„Projektmappa mentése”** beállítás meghatározza, hogy alapértelmezés szerint hol jönnek létre az új projektek:

* **Alapértelmezett hely**: `C:\Users\[Username]\Chloros Projects`
* **Hely megváltoztatása**: Kattintson a szerkesztés ikonra, és válassza ki az új mappát.
* **Mikor érdemes megváltoztatni**:
  * Hálózati meghajtó a csapatmunkához
  * Másik meghajtó, amely több tárhelyet biztosít
  * Év/ügyfél szerint szervezett mappaszerkezet

***

## PPK (utólagos kinematikai feldolgozás) beállítás

Ha MAPIR DAQ rögzítőket használ GPS-szel a pontos földrajzi helymeghatározáshoz:

### Előfeltételek

* MAPIR DAQ GPS (GNSS) modullal
* .daq naplófájl expozíciós pin bejegyzésekkel
* A felvételkészítés során a kamera csatlakoztatva van a DAQ expozíciós csapjaihoz

### Konfigurációs lépések

1. Helyezze a .daq naplófájlt a projektmappájába.
2. A Projektbeállításokban jelölje be az **„PPK-korrekciók alkalmazása”** jelölőnégyzetet.
3. Szükség esetén állítsa be a **„Fényérzékelő időzóna eltolása”** értéket (alapértelmezett: 0 UTC esetén).
4. Rendelje hozzá a kamerákat az expozíciós csapokhoz:
   * **Egyetlen kamera**: Automatikusan hozzárendelve az 1. csaphoz
   * **Két kamera**: Manuálisan rendelje hozzá az egyes kamerákat a megfelelő csapokhoz

**Expozíciós csapok hozzárendelése:**

* **Expozíciós csap 1**: Válassza ki a kamera modellt a legördülő menüből
* **Expozíciós csap 2**: Válassza ki a második kamerát vagy a „Ne használja” opciót
* Ugyanaz a kamera nem rendelhető hozzá mindkét csaphoz

{% hint style=&quot;warning&quot; %}
**Fontos**: Az expozíciós csapokat helyesen kell hozzárendelni a megfelelő kamerákhoz. A helytelen hozzárendelés hibás földrajzi helyadatokhoz vezet.
{% endhint %}

***

## Haladó forgatókönyvek

### Többkamerás projektek

Ha egy projektben több MAPIR kamera képét dolgozza fel:

1. Az Chloros automatikusan felismeri az egyes kameramodelleket
2. Minden kamera megkapja a megfelelő feldolgozási profilt
3. PPK: Manuálisan rendelje hozzá az egyes kamerákat a megfelelő expozíciós csaphoz
4. Az összes kamera ugyanazt az exportformátumot és indexeket használja

**Példa**: Survey3W RGN + Survey3N OCN kettős kamerás szerelvény

### Idősoros vagy több dátumú felmérések

Ugyanazon terület időbeli ismételt felméréseihez:

1. Hozzon létre egy sablont a standard beállításokkal.
2. Minden munkamenetben használjon konzisztens kalibrációs célbeállítást.
3. Minden dátumot külön projektként dolgozzon fel.
4. Az összehasonlítható eredmények érdekében használjon azonos beállításokat.
5. Az időbeli elemzéshez ugyanabban a formátumban exportáljon.

### Nagy adatállományok

Sok képpel rendelkező projektek (500+) esetén:

* Fontolja meg a projektek kisebb projektekre bontását dátum vagy terület szerint.
* Használja az Chloros+ párhuzamos feldolgozást a gyorsabb eredmények érdekében.
* Fontolja meg az CLI vagy az API használatát a kötegelt automatizáláshoz.
* Állítsa be a minimális újrakalibrálási intervallumot a célpontok észlelési idejének csökkentése érdekében.

***

## A beállítások ellenőrzése

A feldolgozás megkezdése előtt ellenőrizze a következő fontos beállításokat:

* [ ] A kamera modellje helyesen lett felismerve a Fájlkezelőben
* [ ] A vignettás korrekció engedélyezve
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
3. **Figyelje a folyamatot** – Lásd: [A feldolgozás figyelése](monitoring-the-processing.md)

Az összes rendelkezésre álló beállításról részletes információkat a [Projektbeállítások](../project-settings/project-settings.md) referencia dokumentációban talál.
