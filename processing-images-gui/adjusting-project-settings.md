# A projektbeállítások módosítása

A képek feldolgozása előtt fontos, hogy a projektbeállításokat a munkafolyamat követelményeinek megfelelően konfigurálja. A **Projektbeállítások** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> panel átfogó vezérlési lehetőségeket biztosít a kalibrálás, a feldolgozási opciók, a multispektrális indexek és az exportformátumok tekintetében.

## A projektbeállítások megnyitása

1. Nyissa meg a projektjét a Chloros programban
2. Kattintson a bal oldali oldalsávon található **Projektbeállítások** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> ikonra
3. A Projektbeállítások panelen megjelennek az összes konfigurációs lehetőség

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>A Projektbeállítások panel – Megjelenítés, célpontfelismerés és feldolgozás</p></figcaption></figure>

{% hint style="info" %}
**A beállítások automatikusan mentésre kerülnek** a projekttel együtt. Amikor újra megnyit egy projektet, az összes beállítás visszaáll.
{% endhint %}

***

## Gyors beállítás a gyakori munkafolyamatokhoz

### Alapértelmezett beállítások (a legtöbb felhasználó számára ajánlott)

Az alapértelmezett beállítások jól működnek a tipikus Survey3- és LATTICE-munkafolyamatokhoz:

* ✅ **Vignettakorrekció**: Engedélyezve
* ✅ **Fényvisszaverődés-kalibrálás / fehéregyensúly**: Engedélyezve (MAPIR-célpontokat és/vagy DAQ fényérzékelő-adatokat használ)
* ✅ **Debayer-módszer**: Standard (Gyors, Közepes minőség)
* ✅ **Exportformátum**: TIFF (16 bites) (TIFF (8 bites) teljes 8 bites projektek esetén)
* ✅ **Minden exportált termék**: Engedélyezve (a LATTICE automatikusan rögzíti a fan out-ot debayered, előnézeti, sugárzási és reflektancia formátumban)

Egyszerűen importálja a képeit, és kezdje el a feldolgozást ezekkel az alapértelmezett beállításokkal.

***

## A Projektbeállítások áttekintése

A Projektbeállítások panel az alábbi szakaszokra van felosztva. Két további szakasz — **DAQ fényérzékelő**és**Tömb igazítás** — automatikusan megjelenik, ha a projekt tartalmazza a vonatkozó fájlokat. A teljes dokumentációt lásd a [Projektbeállítások](../project-settings/project-settings.md) alatt.

### Megjelenítés

* **Képminiatűr felbontás**: A képrács miniatűrjeinek felbontása. Beállítási lehetőségek:**Alapértelmezett (512 px)**,**1024 px**,**2048 px**,**Teljes felbontás**. Csak a megjelenítésre vonatkozik – soha nem befolyásolja a feldolgozást. A magasabb értékek nagyításkor élesebb képet adnak, de lassabban töltődnek be.

### Célpont-felismerés

Ez szabályozza, hogy a Chloros hogyan azonosítja a kalibrációs célpontokat a képein.

**Főbb beállítások:**

* **Minimális kalibrációs mintafelület (px)**: A célpont-felismerés méretküszöbe (alapértelmezett:**25**, tartomány: 0–10000)
* **Minimális célpont-csoportosítás (0–100)**: A célterületek csoportosításának hasonlósági küszöbértéke (alapértelmezett:**60**)**Mikor kell módosítani:**

* Növelje a minta területét, ha téves felismerések történnek
* Csökkentse, ha a célpontok nem kerülnek felismerésre
* Állítsa be az összevonást, ha a célpontok több felismerésre oszlanak szét

{% hint style="info" %}
Ezek a beállítások szürkén jelennek meg, ha a **Fényvisszaverődés-kalibrálás / fehér egyensúly** ki van kapcsolva — ennek kikapcsolt állapotában a célpont-felismerés egyáltalán nem fut.
{% endhint %}

### Feldolgozás

Főbb képfeldolgozási és kalibrálási beállítások.

**Főbb beállítások:**

* **Vignettakorrekció**: Kompenzálja az objektív szélein jelentkező sötétedést ✅ Ajánlott
* **Reflektancia-kalibrálás / fehéregyensúly**: Kalibrálja a képeket az észlelt célpontok (Survey3) és/vagy a DAQ fényérzékelő adatai (LATTICE) segítségével ✅ Ajánlott
* **Debayer-módszer**: Algoritmus a RAW-fájlok 3-csatornás multispektrális formátumba történő konvertálásához
* **Minimális újrakalibrálási intervallum**: A kalibrációs célpontok használata közötti minimális idő másodpercben kifejezve (alapértelmezett:**0** = mindet használja, tartomány: 0–3600)**Kalibrálatlan tartaléktermékek:**Ha egy képkocka reflektancia-kalibrálása nem lehetséges (nincs elérhető célpont, vagy a kalibrálás le van tiltva), akkor azt két tartaléktermék egyikeként exportálja a rendszer —**futásonként pontosan egy létezik a párból**, amelyet a Vignette-korrekció kapcsolója választ ki:

* **Érzékelőválasz exportálása**: `Sensor_Response_Images` fájlt ír – akkor használatos, ha a vignettakorrekció**ki** van kapcsolva
* **Vignette-korrekcióval exportálás**: az `Vignette_Corrected_Images` fájlt írja – akkor használatos, ha a Vignette-korrekció**be van kapcsolva**Az éppen nem aktív jelölőnégyzet szürkén jelenik meg. Az aktív jelölőnégyzet jelölésének megszüntetése teljesen leállítja az adott fájl írását.**LATTICE exporttermékek** (minden projektnél megjelennek; a LATTICE-felvételekre vonatkoznak):

* **Debayered export**: a lineáris debayered kép (`Debayered_Images`): a kalibrációs csomag sötét eltolásának eltávolítása után demosaicing, flat field vagy unmix nélkül — ugyanaz a fájl, amelyet a felvétel debayered szinten ment — az érzékelő natív tájolásában. RGB- és multispektrális modulokra vonatkozik.
* **Előnézet exportálása**: a képernyőn megjelenő előnézet (`Preview_Images`). RGB = fehéregyensúly (DAQ-megvilágítás, ha elérhető, egyébként szürke-világ) + gamma; multispektrális = hamis színű kiterjesztés.
* **Sugárzás exportálása**: float32 spektrális sugárzás (`Radiance_Images`, W/m²/sr/nm). Csak multispektrális moduloknál – nem alkalmazható RGB-mesterfájlokra.
* **Reflektancia exportálása**: uint16 reflektancia (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0), amennyiben egy `.daq` lefelé irányuló mérés vagy a képkockán belüli célpont lefedi a képkockát. Csak multispektrális modulok esetében.

Mind a négy **alapértelmezés szerint be van kapcsolva**— egy importált LATTICE nyers képkocka egyetlen feldolgozási lépésben minden engedélyezett és alkalmazható termékre kiterjed. Az**Export reflectance** jelölőnégyzet szürkén jelenik meg, ha a Reflectance calibration ki van kapcsolva. Azok a beállítások, amelyeket a szülőkapcsoló letilt, mindig szürkén jelennek meg, és egy eszköztipp jelzi, melyik kapcsolót kell megváltoztatni.

**Speciális beállítások:**

* **Fényérzékelő időzóna-eltolás**: Órák az UTC-től a fényérzékelő időszinkronizálásához (alapértelmezett: 0, tartomány −12 és +12 között)
* **PPK-korrekciók alkalmazása**: Az `.daq` fájlokból származó GPS/expozíciós pin-adatokat használja (alapértelmezés: ki)
* **Expozíciós pin 1/2**: Kamerákat rendel az expozíciós pinekhez kettős kamerás felállások esetén

{% hint style="info" %}
**A LATTICE bemeneti szintje automatikus.** A LATTICE-felvételek feldolgozási szintjét az XMP-metadatokban tárolják, és a feldolgozás mindig a nyers képkockánál lép be a feldolgozási folyamatba — a grafikus felhasználói felületen nincs mit beállítani. (Az `--input-level` CLI-jelző a haladó felhasználók számára szolgál a metadatákat elvesztett felvételek felülírására; lásd a [CLI-referenciát](../reference/cli-reference.md).)
{% endhint %}

### Debayer-módszer

Jelenleg két debayer-módszert kínálunk a Chloros-ban:

#### Standard (gyors, közepes minőség)

A Standard debayer gyorsan dolgozik, de debayering színzajt mutat, ami kevésbé pontos és zajosabb képeket eredményez.

#### Textúraérzékeny (lassú, legmagasabb minőség) \[Csak Chloros+]

A textúraérzékeny módszer egy kiváló minőségű, élekkel számoló debayeringet használ, amelyhez egy AI/ML zajszűrő modell társul, amely szinte az összes debayering-zajt eltávolítja. A modell futtatásához GPU-memória (VRAM) szükséges: **7 GB vagy annál több VRAM** esetén több képet is képes egyszerre feldolgozni; 7 GB alatt egyszerre csak egy képet dolgoz fel (ez észrevehetően lassabb). Lásd: [Dinamikus számítási adaptáció](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**A LATTICE-felvételek mindig a Standard demosaic-ot használják.** Nincs LATTICE-re betanított Texture Aware modell, ezért ez az opció nem elérhető a LATTICE-felvételek esetében — ugyanazon projektben található Survey3-felvételek azonban továbbra is használhatják.
{% endhint %}

### Index (multispektrális indexek)

Állítsa be, mely vegetációs indexeket kívánja kiszámítani és exportálni. A grafikus felület legördülő menüje **27 előre definiált indexképletet** kínál.**Indexek hozzáadása:**

1. Kattintson az**„Index hozzáadása”** gombra
2. Válasszon ki egy indexet a legördülő menüből (NDVI, NDRE, GNDVI stb.)
3. Állítsa be a megjelenítési beállításokat (LUT-színek, értéktartományok)
4. Szükség szerint adjon hozzá több indexet is

**Gyakori indexek:**

* **NDVI**: Általános növényzetállapot (a leggyakoribb)
* **NDRE**: Korai stresszfelismerés RedEdge segítségével* **GNDVI**: Klorofill-koncentrációra érzékeny
* **OSAVI**: Jól működik látható talaj esetén
* **EVI**: Magas levélfelületi indexű (LAI) régiók

**Egyéni képletek:**

* Hozzon létre egyéni multispektrális indexképleteket a sávok közötti matematikai műveletekkel az összes képcsatornán
* Mentse el az egyéni képleteket későbbi újrafelhasználás céljából
* Az egyéni képletek az Chloros+ szolgáltatásai; elérhetőségük a választott csomagszinttől függ

Az összes elérhető index és képlet – beleértve azt is, hogy mely nevek csak a grafikus felhasználói felületen (GUI) használhatók, és melyek működnek a parancssorban (CLI) vagy az SDK-ben is – megtalálható a [Multispektrális indexképletek](../project-settings/multispectral-index-formulas.md) oldalon.

### Exportálás

A kimeneti fájlformátum beállítása.

**Elérhető formátumok**(beállítás:**Kalibrált képformátum**, alapértelmezett:**TIFF (16-bit)**):

* **TIFF (16-bit)**: GIS-hez és tudományos elemzésekhez ajánlott
* **TIFF (8-bit)**: 8-bites TIFF
* **TIFF (32-bit, százalék)**: Lebegőpontos értékek
* **PNG (8-bit)**: Veszteségmentes tömörítés vizualizációhoz
* **JPG (8-bit)**: A legkisebb fájlméret, veszteséges tömörítés

A lista bemutatja, milyen képformátumokat támogat a projekt: az Chloros nem növeli a bitmélységet. A kizárólag 8-bites képek esetében csak a TIFF (8-bites), a PNG (8-bites) és a JPG (8-bites) formátumok állnak rendelkezésre, alapértelmezés szerint a TIFF (8-bites) formátumot használja a rendszer. Vegyes projektben minden felvétel a saját mélységében vagy annál alacsonyabb mélységben kerül exportálásra. A Radiance értéke 32-bites lebegőpontos marad.

A kimeneti fájlok a projektmappa alá kerülnek, kameránként és formátumonként csoportosítva: `<project>/<camera>/<format>/<Product>_Images/`. A Radiance értéket **mindig** float32 formátumban írja a rendszer az `tiff32` mappába, függetlenül ettől a beállítástól. Az exportált fájlok megtartják a forrásfájl nevét — a mappa azonosítja a terméket. A LATTICE multispektrális exportok pixel-alapú korlátozó maszkokat is írhatnak a termékmappák mellett található `clamp_masks/` mappába. A teljes kimeneti fa megtekintéséhez lásd a [Feldolgozás befejezése](finishing-the-processing.md) című részt.

{% hint style="warning" %}
**A reflexiós értékek kiolvasása**: az a DN-érték, amelynél ρ = 1,0, a forráskamerától függ — a LATTICE 32768-at használ (XMP `Chloros:PixelScale` néven megjelölve), a Survey3 pedig 65535-öt. Ne feltételezzünk állandó értéket, hanem olvassuk ki a címkét. Lásd [Kimeneti képformátumok](../output-image-formats.md).
{% endhint %}

### DAQ fényérzékelő

Ez a szakasz felsorolja a projektben található összes DAQ lefelé irányuló sugárzási fájlt (`.daq` / `.csv`), fájlonként egy sorban, feltüntetve az érzékelő modelljét, a fájlnevet és az adott fájlra érvényes diffúzor **fedél** korrekciót.

* **Fedél felülírás (minden fájl)**: egyetlen, a teljes projektre kiterjedő legördülő menü.**Auto** (alapértelmezett) az egyes fájlokban rögzített felső határt használja — ha nincs rögzítve érték, akkor napfényt feltételez, de a DAQ-U, DAQ-M és DAQ-E felső határ nélkül kerül forgalomba, és a Chloros nem látja, mi lett beállítva, ezért a feldolgozás figyelmeztetést ad a feltételezett felső határra vonatkozóan: válassza ki a beállított felső határt. A felső határ kiválasztása minden fájlt felülír, kivéve a DAQ-E-S-ét, amelynek diffúzora a kalibrációjának része: a nyers felvételeket ezzel korrigálják, a már felső határral rendelkező felvételeket pedig újra referenciálják (a rögzített korrekciót visszavonják, a kiválasztott felső határt alkalmazzák).
* A sorok figyelmeztetést jelenítenek meg, ha a rögzített fedél a hub feltételezett alapértelmezése volt, nem pedig a kezelő által megerősített, valamint ha a kiválasztott fedélnek nincs profilja az adott eszközmodellhez (a felülírás elutasításra kerül az adott fájl esetében).

A „Fényérzékelők” fülön készített DAQ-felvételek automatikusan hozzáadódnak a nyitott projekthez, az importált `.daq` / `.csv` fájlok pedig azonnal megjelennek itt, amint hozzáadódnak.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Alsó projektbeállítások — Index, Exportálási formátum, a DAQ Fényérzékelők szakasz, valamint a projektsablon/mappa vezérlői</p></figcaption></figure>### Tömb igazítás

Ez a szakasz **csak** akkor jelenik meg, ha a projektben legalább egy kép tartalmazza azt a modulok közötti igazítási transzformációt, amelyet a LATTICE-tömbök a rögzítéskor bejegyznek (`Chloros:Alignment*` XMP). Megmutatja, hány kép tartalmaz címkéket, és melyik kamera a referencia, a következő vezérlőkkel:

* **Tömb igazítás alkalmazása** (alapértelmezés: be): minden feldolgozott terméket (debayered / előnézet / sugárzás / visszaverődés / index) a tömb közös referencia-geometriájához igazít. Ki = exportálás a szenzor natív geometriájában.
* **Kivágás a közös átfedésre** (alapértelmezés: be): az igazított exportált adatokat a modulok által közösen használt területre vágja ki, így minden sáv lábnyoma megegyezik. Ki kapcsolás esetén az érzékelő teljes felülete megmarad (a forráson kívül fekete kitöltés).
* **Újramintavétel**:**Bilineáris (sima, alapértelmezett)**,**Legközelebbi (a pontos értékek megőrzése)**— nincs pixelek közötti keverés, szigorú radiometrikus elemzéshez — vagy**Kubikus (legélesebb)**.

***

## Beállítások mentése és betöltése

### Projekt sablon mentése

Hozzon létre újrafelhasználható sablonokat az egységes munkafolyamatok érdekében:

1. Állítsa be az összes kívánt beállítást a Projektbeállítások panelen
2. Görgessen le az alján található **„Projekt sablon mentése”** szakaszhoz
3. Adjon meg egy leíró sablonnevet (pl. „Survey3N_RGN_Agriculture”)
4. Kattintson a mentés ikonra

**Előnyök:**

* Azonos beállítások alkalmazása több projektben
* A konfigurációk megosztása a csapattagokkal
* Az ismételt felmérések következetességének fenntartása

### Sablon betöltése új projektbe

Új projekt létrehozásakor:

1. Válassza a **„Új projekt”** menüpontot a főmenüből
2. Válasszon ki egy projektsablont az opcionális sablonválasztóból
3. A sablon összes beállítása automatikusan alkalmazásra kerül

### Munkakönyvtár

A **„Munkakönyvtár”** beállítás határozza meg, hogy alapértelmezés szerint hol jönnek létre az új projektek:

* **Alapértelmezett hely**: `Documents\MAPIR\Chloros Projects` a Windows Dokumentumok mappájában (általában `C:\Users\[Username]\Documents\MAPIR\Chloros Projects`; követi a Dokumentumok mappát, ha az OneDrive vagy egy vándorprofil áthelyezi)
* **Hely megváltoztatása**: Kattintson a szerkesztés ikonra, és válasszon ki egy új mappát
* **Megosztva a CLI-vel**: Az `chloros-cli` ugyanazt az alapértelmezett projektmappa-beállítást használja
* **Windows rendszeren rövidítsd le**: amennyiben a Windows hosszú elérési útvonalak funkciója nincs bekapcsolva, a feldolgozás előtt elutasításra kerül minden olyan projektmappa, amelynek elérési útja meghaladja a 130 karaktert — lásd [A feldolgozás elindítása](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer)
* **Mikor érdemes megváltoztatni**:
  * Hálózati meghajtó csapatmunkához
  * Másik meghajtó nagyobb tárhellyel
  * Év/ügyfél szerint rendezett mappaszerkezet

***

## PPK (utólagosan feldolgozott kinematikus) beállítás

MAPIR DAQ-rögzítők használata esetén GPS-szel a pontos földrajzi helymeghatározáshoz:

### Előfeltételek

* GPS (GNSS) modullal ellátott MAPIR DAQ
* Expozíciós csatlakozók adatait tartalmazó .daq naplófájl
* A felvétel készítése során a DAQ expozíciós csatlakozóihoz csatlakoztatott kamera

### Beállítási lépések

1. Helyezze a .daq naplófájlt a projektmappájába
2. A Projektbeállítások menüben jelölje be a **„PPK-korrekciók alkalmazása”** jelölőnégyzetet
3. Szükség esetén állítsa be a **„Fényérzékelő időzóna-eltolódás”** értéket (alapértelmezett: 0 az UTC-hez)
4. Rendelje hozzá a kamerákat az expozíciós csatlakozókhoz:
   * **Egy kamera**: Automatikusan a 1. csatlakozóhoz kerül
   * **Két kamera**: Minden kamerát manuálisan rendeljen a megfelelő csatlakozóhoz

**Expozíciós csatlakozók hozzárendelése:**

* **

1. expozíciós csatlakozó**: Válassza ki a kamera modelljét a legördülő menüből
* **

2. expozíciós csatlakozó**: Válassza ki a második kamerát vagy a „Ne használja” opciót
* Ugyanaz a kamera nem rendelhető mindkét csatlakozóhoz

{% hint style="warning" %}
**Fontos**: Az expozíciós csatlakozókat helyesen kell hozzárendelni a megfelelő kamerákhoz. A helytelen hozzárendelés hibás földrajzi helyadatokhoz vezet.
{% endhint %}

***

## Speciális esetek

### Többkamerás projektek

Ha egy projektben több MAPIR-kamerából származó képeket dolgoz fel:

1. A Chloros automatikusan felismeri az egyes kameramodelleket (Survey3-t és LATTICE-t egyaránt)
2. Minden kamera megkapja a megfelelő feldolgozási profilokat, és minden kamera saját kimeneti mappaszerkezetet kap
3. PPK: Minden Survey3-kamerát manuálisan kell hozzárendelni a megfelelő expozíciós csatlakozóhoz
4. Minden kamera ugyanazt az exportformátumot és indexeket használja

**Példák**: Survey3W RGN + Survey3N OCN kettős kamerás rendszer, vagy egy LATTICE-rendszer, amely RGB-mesterkamerát keskenysávú modulokkal kombinál

### Idősoros vagy több időpontban végzett felmérések

Ugyanazon terület időben ismételt felmérései esetén:

1. Hozzon létre egy sablont a szokásos beállításokkal
2. Minden munkamenet során használjon egységes kalibrációs célpont-beállítást
3. Minden dátumot külön projektként dolgozzon fel
4. Az összehasonlítható eredmények érdekében használjon azonos beállításokat
5. Az időbeli elemzés érdekében ugyanabban a formátumban exportáljon

### Nagy adatállományok

Sok képet (500+) tartalmazó projektek esetén:

* Fontolja meg a kisebb projektekre való felosztást dátum vagy terület szerint
* Használja az Chloros+ párhuzamos feldolgozását a gyorsabb eredmények érdekében
* Fontolja meg a CLI vagy az API használatát a kötegelt feldolgozás automatizálásához
* Állítsa be a minimális újrakalibrálási intervallumot a célpontok felismerési idejének csökkentése érdekében

***

## A beállítások ellenőrzése

A feldolgozás megkezdése előtt ellenőrizze az alábbi kulcsfontosságú beállításokat:

* [ ] A kamera modellje helyesen felismerve a Fájlkezelőben
* [ ] A vignettakorrekció engedélyezve
* [ ] A reflexiós kalibrálás engedélyezve
* [ ] A Survey3 esetében: legalább egy kalibrációs célkép importálva és ellenőrizve; a LATTICE esetében: célkép és/vagy `.daq` lefelé irányuló felvétel jelen van
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
