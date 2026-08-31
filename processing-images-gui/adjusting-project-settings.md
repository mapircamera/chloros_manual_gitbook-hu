# A projektbeállítások módosítása

A képek feldolgozása előtt fontos, hogy a projektbeállításokat a munkafolyamat igényeihez igazítsa. A „Projektbeállítások” (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">) panel átfogó vezérlési lehetőségeket biztosít a kalibrálás, a feldolgozási beállítások, a multispektrális indexek és az exportformátumok tekintetében.

## A projektbeállítások megnyitása

1. Nyissa meg a projektet az Chloros alkalmazásban.
2. Kattintson a bal oldali sávon található **Projektbeállítások** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> ikonra.
3. A Projektbeállítások panelen megjelennek az összes konfigurációs lehetőségek.

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>A Projektbeállítások panel – Megjelenítés, célpontfelismerés és feldolgozás</p></figcaption></figure>{% hint style="info" %}
**A beállítások automatikusan mentésre kerülnek** a projekttel együtt. Amikor újra megnyit egy projektet, az összes beállítás visszaáll.
{% endhint %}

***

## Gyors beállítás a leggyakoribb munkafolyamatokhoz

### Alapértelmezett beállítások (a legtöbb felhasználó számára ajánlott)

Az alapértelmezett beállítások jól működnek a tipikus Survey3 és LATTICE munkafolyamatoknál:

* ✅ **Vignettakorrekció**: Engedélyezve
* ✅ **Fényvisszaverődés-kalibrálás / fehéregyensúly**: Engedélyezve (MAPIR célpontokat és/vagy DAQ fényérzékelő adatait használja)
* ✅ **Debayer-módszer**: Standard (Gyors, Közepes minőség)
* ✅ **Exportformátum**: TIFF (16 bites)
* ✅ **Minden exportált termék**: Engedélyezve (a LATTICE automatikusan rögzíti a fan out-ot debayered, preview, radiance és reflectance formátumokban)

Egyszerűen importálja a képeit, és kezdje el a feldolgozást ezekkel az alapértelmezett beállításokkal.

***

## A Projektbeállítások áttekintése

A Projektbeállítások panel az alábbi szakaszokból áll. Két további szakasz – **DAQ fényérzékelő**és**Tömb igazítás** – automatikusan megjelenik, ha a projekt tartalmazza a vonatkozó fájlokat. A teljes dokumentációt lásd a [Projektbeállítások](../project-settings/project-settings.md) oldalon.

### Megjelenítés

* **Képminiatűr felbontás**: A képrács miniatűrjeinek felbontása. Beállítások:**Alapértelmezett (512 px)**,**1024 px**,**2048 px**,**Teljes felbontás**. Csak megjelenítés — soha nem befolyásolja a feldolgozást. A magasabb értékek nagyításkor élesebb képet adnak, de lassabban töltődnek be.

### Célpont-felismerés

Szabályozza, hogy az Chloros hogyan azonosítja a kalibrációs célpontokat a képein.

**Főbb beállítások:*** **Minimális kalibrációs mintaterület (px)**: A célpont-felismerés méretküszöbe (alapértelmezett:**25**, tartomány: 0–10000)
* **Minimális célpont-csoportosítás (0–100)**: A célterületek csoportosításának hasonlósági küszöbértéke (alapértelmezett:**60**)**Mikor kell módosítani:**

* Növelje a mintaterületet, ha téves felismerések történnek
* Csökkentse, ha a célokat nem ismeri fel a rendszer
* Állítsa be a csoportosítást, ha a célok több felismerésre oszlanak szét

{% hint style="info" %}
Ezek a beállítások szürkén jelennek meg, ha a **Fényvisszaverődés-kalibrálás / fehéregyensúly** ki van kapcsolva — ennek kikapcsolt állapotában a célpont-felismerés egyáltalán nem fut.
{% endhint %}

### Feldolgozás

Főbb képfeldolgozási és kalibrációs beállítások.

**Főbb beállítások:*** **Vignette-korrekció**: Kompenzálja a lencse szélein jelentkező sötétedést ✅ Ajánlott
* **Reflektancia-kalibrálás / fehéregyensúly**: A képeket az észlelt célpontok (Survey3) és/vagy a DAQ fényérzékelő adatai (LATTICE) alapján kalibrálja ✅ Ajánlott
* **Debayer-módszer**: Algoritmus a RAW-fájlok 3-csatornás multispektrális formátumba történő konvertálásához
* **Minimális újrakalibrálási intervallum**: A kalibrációs célpontok használata közötti minimális idő másodpercben kifejezve (alapértelmezett:**0** = mindet használja, tartomány: 0–3600)**Kalibrálatlan tartaléktermékek:**Ha egy képkocka nem kalibrálható reflektancia alapján (nincs célpont, vagy a kalibrálás le van tiltva), akkor két tartaléktermék egyikeként kerül exportálásra —**futásonként pontosan az egyik létezik a párból**, amelyet a Vignette-korrekciós kapcsoló választ ki:

* **Érzékelőválasz exportálása**: `Sensor_Response_Images` fájlt ír – akkor használatos, ha a vignettakorrekció**ki** van kapcsolva
* **Vignette-korrekcióval exportálás**: az `Vignette_Corrected_Images` fájlt írja – akkor használatos, ha a Vignette-korrekció**be van kapcsolva**Az éppen nem aktív jelölőnégyzet szürkére van állítva. Az aktív jelölőnégyzet jelölésének megszüntetése teljesen leállítja az adott fájl írását.**LATTICE exporttermékek** (minden projektnél megjelennek; a LATTICE-felvételekre vonatkoznak):

* **Debayered export**: a lineáris debayered kép (`Debayered_Images`). Az RGB-re és a multispektrális modulokra vonatkozik.
* **Előnézet exportálása**: a képernyőn megjelenő előnézet (`Preview_Images`). RGB = fehéregyensúly (DAQ-megvilágító, ha elérhető, egyébként szürke-világ) + gamma; multispektrális = hamis színű kiterjesztés.
* **Sugárzás exportálása**: float32 spektrális sugárzás (`Radiance_Images`, W/m²/sr/nm). Csak multispektrális moduloknál — nem alkalmazható az RGB mesterfájlokra.
* ****Reflektancia exportálása**: uint16 reflektancia (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0), ha egy `.daq` lefelé irányuló mérés vagy a képkockán belüli célpont lefedi a képkockát. Csak multispektrális moduloknál.

Mind a négy **alapértelmezés szerint be van kapcsolva**— egy importált LATTICE nyers képkocka egyetlen feldolgozási lépésben szétosztódik minden engedélyezett és alkalmazható termékre. A**Reflektancia exportálása** jelölőnégyzet szürkén jelenik meg, ha a Reflektancia-kalibrálás ki van kapcsolva. Azok a beállítások, amelyek szülőkapcsolójuk miatt nem használhatók, mindig szürkén jelennek meg, és egy eszköztipp jelzi, melyik kapcsolót kell módosítani.**Speciális beállítások:*** **Fényérzékelő időzóna-eltolás**: Órák az UTC-hez viszonyítva a fényérzékelő időszinkronizálásához (alapértelmezett: 0, tartomány −12 és +12 között)
* **PPK-korrekciók alkalmazása**: Az `.daq` fájlokból származó GPS/expozíciós csapadatokat használja (alapértelmezés: ki)
* **Expozíciós csap 1/2**: A kamerákat expozíciós csapokhoz rendeli a kétkamerás felállásokhoz

{% hint style="info" %}
**A LATTICE bemeneti szintje automatikus.** A LATTICE-felvételek feldolgozási szintjüket az XMP-metadatokban hordozzák, és a feldolgozás mindig a nyers képkockánál lép be a feldolgozási folyamatba — a grafikus felhasználói felületen nincs mit beállítani. (Az CLI jelző, az `--input-level` a metaadatok elvesztése esetén a felvételekre vonatkozó, haladó felhasználók számára elérhető felülírási lehetőség; lásd az [CLI Referenciát](../reference/cli-reference.md).)
{% endhint %}

### Debayer-módszer

Jelenleg 2 debayer-módszert kínálunk az Chloros-ben:

#### Standard (Gyors, közepes minőség)

A Standard debayer gyorsan dolgozik, de a debayering során színzajt mutat, ami kevésbé pontos és zajosabb képeket eredményez.

#### Textúraérzékeny (lassú, legmagasabb minőség) \[Csak Chloros+]

A textúraérzékeny módszer egy kiváló minőségű, élekkel számoló debayeringet használ, amelyet egy AI/ML zajszűrő modellel kombinálnak, így szinte az összes debayering-zajt eltávolítja. A modell futtatásához GPU-memória (VRAM) szükséges: **7 GB vagy annál több VRAM** esetén több képet is képes egyszerre feldolgozni; 7 GB alatt egyszerre csak egy képet dolgoz fel (ez észrevehetően lassabb). Lásd: [Dinamikus számítási adaptáció](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**A LATTICE-felvételek mindig a Standard demosaic-ot használják.** Nincs LATTICE-re betanított Texture Aware modell, ezért a lehetőség nem elérhető a LATTICE-képek esetében — ugyanazon projektben található Survey3 képek azonban továbbra is használhatják.
{% endhint %}

### Index (multispektrális indexek)

Állítsa be, mely vegetációs indexeket kívánja kiszámítani és exportálni. A grafikus felület legördülő menüje **27 előre definiált indexképletet** kínál.**Indexek hozzáadása:**

1. Kattintson az**„Index hozzáadása”** gombra
2. Válasszon ki egy indexet a legördülő menüből (NDVI, NDRE, GNDVI stb.)
3. Állítsa be a megjelenítési beállításokat (LUT-színek, értéktartományok)
4. Szükség szerint adjon hozzá több indexet

**Gyakori indexek:*** **NDVI**: Általános növényzetállapot (a leggyakoribb)
* **NDRE**: Korai stresszfelismerés az RedEdge-szel együtt
* **GNDVI**: Klorofill-koncentrációra érzékeny
* **OSAVI**: Jól működik látható talaj esetén
* **EVI**: Magas levélfelületi indexű (LAI) régiók**Egyéni képletek:**

* Hozzon létre egyéni multispektrális indexképleteket a sávok matematikai műveleteivel az összes képcsatornán
* Mentse el az egyéni képleteket későbbi újrafelhasználás céljából
* Az egyéni képletek az Chloros+ szolgáltatás részei; elérhetőségük a választott csomagszinttől függ

Az összes elérhető index és képlet – beleértve azt is, hogy melyek csak a grafikus felületen (GUI) használhatók, és melyek működnek az CLI/SDK rendszerekben is – megtalálható a [Multispektrális indexképletek](../project-settings/multispectral-index-formulas.md) című részben.

### Export

A kimeneti fájl formátumát szabályozza.

**Elérhető formátumok**(beállítás:**Kalibrált képformátum**, alapértelmezett**TIFF (16-bit)**):

* **TIFF (16 bites)**: GIS-hez és tudományos elemzésekhez ajánlott
* **TIFF (32 bites, százalék)**: Lebegőpontos értékek
* **PNG (8 bites)**: Veszteségmentes tömörítés megjelenítéshez
* **JPG (8 bites)**: A legkisebb fájlok, veszteséges tömörítés

A kimeneti fájlok a projektmappa alatt kerülnek mentésre, kameránként és formátumonként csoportosítva: `<project>/<camera>/<format>/<Product>_Images/`. A Radiance értéke **mindig** float32 típusúként kerül mentésre az `tiff32` mappába, függetlenül ettől a beállítástól. Az exportált fájlok megtartják a forrásfájl nevét – a mappa azonosítja a terméket. A teljes kimeneti fa szerkezetét lásd a [Feldolgozás befejezése](finishing-the-processing.md) részben.

{% hint style="warning" %}
**Reflektanciaértékek olvasása**: az a DN-érték, amelynél ρ = 1,0, a forráskamerától függ — a LATTICE 32768-at használ (XMP `Chloros:PixelScale` néven rögzítve), az Survey3 pedig 65535-öt. Ne feltételezzünk állandó értéket, hanem olvassuk ki a címkét. Lásd [Kimeneti képformátumok](../output-image-formats.md).
{% endhint %}

### DAQ fényérzékelő

Ez a szakasz felsorolja a projektben található összes DAQ lefelé irányuló sugárzási fájlt (`.daq` / `.csv`), fájlonként egy sorban, feltüntetve az érzékelő modellt, a fájlnevet és az adott fájlra érvényes diffúzor **fedél** korrekciót.

* **Felső határ felülírása (minden fájl)**: egyetlen, a teljes projektre kiterjedő legördülő menü. Az**Auto** (alapértelmezett) beállítás az egyes fájlokban rögzített felső határt használja — amennyiben nincs rögzítve érték, a rendszer napfényt feltételez, mivel minden MAPIR DAQ a napfény-korrektorral együtt kerül forgalomba. A felső határ kiválasztása minden fájlt felülír: a nyers felvételeket ezzel korrigálja, és a már felső határral rendelkező felvételeket újra hivatkozza (a rögzített korrekciót visszavonja, a kiválasztott felső határt alkalmazza).
* A sorok figyelmeztetnek, ha a rögzített felső határ a hub által feltételezett alapértelmezett érték volt, nem pedig a kezelő által megerősített, valamint ha a kiválasztott felső határnak nincs profilja az adott eszközmodellhez (a felülírás elutasításra kerül az adott fájl esetében).

A „Fényérzékelők” fülön készített DAQ-felvételek automatikusan hozzáadódnak a megnyitott projekthez, és az importált `.daq` / `.csv` fájlok itt jelennek meg, amint hozzáadódnak.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Alsó projektbeállítások — Index, Exportálási formátum, a DAQ Fényérzékelők szakasz, valamint a projektsablon/mappa vezérlői</p></figcaption></figure>### Tömb igazítás

Ez a szakasz **csak** akkor jelenik meg, ha a projektben legalább egy kép tartalmazza azt a modulok közötti igazítási transzformációt, amelyet a LATTICE-tömbök a rögzítéskor jelölnek meg (`Chloros:Alignment*` XMP). Megmutatja, hány kép tartalmaz címkéket, és melyik kamera a referencia, a következő vezérlőkkel:

* **Tömb igazítás alkalmazása** (alapértelmezett: be): minden feldolgozott terméket (debayered / előnézet / sugárzás / visszaverődés / index) a tömb közös referencia-geometriájához igazít. Ki = exportálás a szenzor natív geometriájában.
* **Kivágás a közös átfedésre** (alapértelmezés: be): az igazított exportokat kivágnja arra a területre, amelyet minden modul megoszt, így minden sávnak azonos lábnyoma lesz. Ki kapcsolás esetén az érzékelő teljes felülete megmarad (a forráson kívül fekete kitöltéssel).
* **Újramintavétel**:**Bilineáris (sima, alapértelmezett)**,**Legközelebbi (a pontos értékek megőrzése)**— nincs pixelek közötti keverés, szigorú radiometrikus elemzéshez — vagy**Kubikus (legélesebb)**.***

## Beállítások mentése és betöltése

### Projekt sablon mentése

Hozzon létre újrafelhasználható sablonokat az egységes munkafolyamatok érdekében:

1. Állítsa be az összes kívánt beállítást a Projektbeállítások panelen
2. Görgessen le a lap alján található **„Projekt sablon mentése”** szakaszhoz
3. Adjon meg egy leíró sablonnevet (pl. „Survey3N\_RGN\_Agriculture”)
4. Kattintson a mentés ikonra

**Előnyök:**

* Azonos beállítások alkalmazása több projektben
* A konfigurációk megosztása a csapattagokkal
* Az ismételt felmérések következetességének biztosítása

### Sablon betöltése új projektbe

Új projekt létrehozásakor:

1. Válassza a **„Új projekt”** menüpontot a főmenüből
2. Válasszon ki egy projektsablont az opcionális sablonválasztóból
3. A sablon összes beállítása automatikusan alkalmazásra kerül

### Munkakönyvtár

A **„Munkakönyvtár”** beállítás határozza meg, hogy alapértelmezés szerint hol jönnek létre az új projektek:

* **Alapértelmezett hely**: `C:\Users\[Username]\Chloros Projects`
* **Hely módosítása**: Kattintson a szerkesztés ikonra, és válasszon ki egy új mappát
* **Megosztva az CLI-szel**: Az `chloros-cli` ugyanazt az alapértelmezett projektmappa-beállítást használja
* **Mikor érdemes megváltoztatni**:
  * Hálózati meghajtó csapatmunkához
  * Másik meghajtó nagyobb tárhellyel
  * Év/ügyfél szerint rendezett mappaszerkezet

***

## PPK (utólagosan feldolgozott kinematikus) beállítás

Ha MAPIR DAQ-felvevőket használ GPS-szel a pontos földrajzi helymeghatározáshoz:

### Előfeltételek

* MAPIR DAQ GPS (GNSS) modullal
* .daq naplófájl expozíciós csatlakozóbeállításokkal
* A felvétel készítése során a kamerát a DAQ expozíciós csatlakozóihoz csatlakoztatva

### Beállítási lépések

1. Helyezze a .daq naplófájlt a projektmappájába
2. A Projektbeállítások menüben jelölje be az **„PPK-korrekciók alkalmazása”** jelölőnégyzetet
3. Szükség esetén állítsa be a **„Fényérzékelő időzóna-eltolódását”** (alapértelmezett: 0 az UTC-hez)
4. Rendelje hozzá a kamerákat az expozíciós csatlakozókhoz:
   * **Egy kamera**: automatikusan a 1. csatlakozóhoz kerül
   * **Két kamera**: minden kamerát manuálisan rendeljen a megfelelő csatlakozóhoz**Expozíciós csatlakozók hozzárendelése:*** **

1. expozíciós pin**: Válassza ki a kamera modelljét a legördülő menüből
* **

2. expozíciós pin**: Válassza ki a második kamerát vagy a „Ne használja” opciót
* Ugyanaz a kamera nem rendelhető mindkét pinhez

{% hint style="warning" %}
**Fontos**: Az expozíciós csatlakozókat helyesen kell hozzárendelni a megfelelő kamerákhoz. A helytelen hozzárendelés hibás földrajzi helyadatokhoz vezet.
{% endhint %}

***

## Speciális esetek

### Többkamerás projektek

Ha egy projektben több MAPIR kamerából származó képeket dolgoz fel:

1. Az Chloros automatikusan felismeri az egyes kameramodelleket (az Survey3-et és a LATTICE-t egyaránt)
2. Minden kamera megkapja a megfelelő feldolgozási profilokat, és minden kamera saját kimeneti mappaszerkezetet kap
3. PPK: Minden Survey3 kamerát manuálisan rendeljen a megfelelő expozíciós csaphoz
4. Minden kamera ugyanazt az exportformátumot és indexeket használja

**Példák**: Survey3W, RGN + Survey3N, OCN kétkamerás rendszer, vagy egy LATTICE-rendszer, amely egy RGB főkamerát keskenysávú modulokkal kombinál

### Idősoros vagy több időpontban végzett felmérések

Ugyanazon terület időbeli ismételt felméréséhez:

1. Hozzon létre egy sablont a szokásos beállításokkal
2. Minden munkamenetben használjon egységes kalibrációs célpont-beállítást
3. Minden dátumot külön projektként dolgozzon fel
4. Az összehasonlítható eredmények érdekében használjon azonos beállításokat
5. Az időbeli elemzéshez exportálja ugyanabban a formátumban

### Nagy adatállományok

Sok képet (500+) tartalmazó projektek esetén:

* Fontolja meg a kisebb projektekre való felosztást dátum vagy terület szerint
* Használja az Chloros+ párhuzamos feldolgozást a gyorsabb eredmények érdekében
* Fontolja meg az CLI vagy az API használatát a kötegelt feldolgozás automatizálásához
* Állítsa be a minimális újrakalibrálási intervallumot a célpont-felismerési idő csökkentése érdekében

***

## A beállítások ellenőrzése

A feldolgozás megkezdése előtt ellenőrizze az alábbi kulcsfontosságú beállításokat:

* [ ] A kamera modellje helyesen lett felismerve a Fájlkezelőben
* [ ] A vignettálás-korrekció engedélyezve
* [ ] A reflektancia-kalibrálás engedélyezve
* [ ] Survey3 esetén: legalább egy kalibrációs célkép importálva és ellenőrizve; LATTICE esetén: cél és/vagy `.daq` lefelé irányuló felvétel jelen van
* [ ] A kívánt multispektrális indexek hozzáadva
* [ ] A munkafolyamatához megfelelő exportformátum
* [ ] A PPK-beállítások konfigurálva (ha .daq fájlt használ expozíciós eseményekkel)

***

## Következő lépések

A beállítások konfigurálása után:

1. **Jelölje meg a kalibrációs célképeket** – Lásd: [Célképek kiválasztása](choosing-target-images.md)
2. **Indítsa el a feldolgozást** – Lásd: [A feldolgozás elindítása](starting-the-processing.md)
3. **Kövesse nyomon a feldolgozás előrehaladását** – Lásd: [A feldolgozás nyomon követése](monitoring-the-processing.md)

Az összes rendelkezésre álló beállításról szóló részletes információkat a [Projektbeállítások](../project-settings/project-settings.md) referencia-dokumentációban találja.
