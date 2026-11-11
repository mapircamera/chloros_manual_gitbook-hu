# A feldolgozás elindítása

Miután importálta a képeket, megjelölte a kalibrációs célpontokat és beállította a projekt paramétereit, készen áll a feldolgozás megkezdésére. Ez az oldal végigvezeti Önt a Chloros feldolgozási folyamat elindításán.

## Előfeldolgozási ellenőrzőlista

Mielőtt rákattintana a „Start” gombra, ellenőrizze, hogy minden készen áll-e:

* [ ] **Fájlok importálva** – Minden kép megjelenik a Fájlkezelőben
* [ ] **Célképek megjelölve** – A „Cél” oszlopban bejelölve vannak a kalibrációs képek (vagy a LATTICE esetében importált `.daq` felvétel)
* [ ] **Kamera modellek felismerve** – A „Kamera modell” oszlop a megfelelő kamerákat jelzi
* [ ] **Beállítások konfigurálva** – A projektbeállításokat átnéztük és módosítottuk
* [ ] **Indexek kiválasztva** – A kívánt multispektrális indexeket hozzáadtuk (ha szükséges)
* [ ] **Kiválasztott exportformátum** – A munkafolyamatához megfelelő kimeneti formátum

{% hint style="info" %}
**Tipp**: Kattintson végig néhány képen a Fájlkezelőben, hogy ellenőrizze, hogy a feldolgozás előtt helyesen töltődtek-e be.
{% endhint %}

***

## A feldolgozás elindítása

### Keresse meg a Start gombot

A Start/Lejátszás gomb a Chloros felső fejlécsávjában található:

* Hely: Az ablak felső közepén
* Ikon: **Lejátszás/Start gomb** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Állapot: A gomb aktív (világít), ha a feldolgozásra készen áll

### Kattintson a kezdéshez

1. Kattintson a felső fejlécben található **Lejátszás/Indítás gombra**

2. A feldolgozás azonnal megkezdődik
3. A gomb a feldolgozás során **Leállítás** gombbá válik
4. A haladási sáv frissül, jelezve a feldolgozás állapotát

{% hint style="success" %}
**Feldolgozás elindítva**: A gombra kattintás után a Chloros automatikusan elvégzi az összes feldolgozási lépést – célpontfelismerés, debayering, kalibrálás, indexszámítás és exportálás. A program automatikusan felismeri, hogy a projekt Survey3, LATTICE vagy ezek keveréke-e, és minden kamerához a megfelelő feldolgozási folyamatot alkalmazza.
{% endhint %}

***

## A feldolgozási módok ismertetése A

Chloros a licencétől függően két különböző feldolgozási módban működik:

### Ingyenes mód (szekvenciális feldolgozás)

**Minden felhasználó számára elérhető**

**Hogyan működik:**

* A képeket egyenként, egymás után dolgozza fel
* Egy szálon futó művelet
* Alacsonyabb memóriahasználat

**A haladási sáv 2 szakaszt jelenít meg:**

1.**Célpont-felismerés** – Kalibrációs célpontok keresése
2. **Feldolgozás** – Kalibrálás alkalmazása és képek exportálása**Feldolgozási idő:**

* Sokkal lassabb, mint a Chloros+ párhuzamos módja
* Kis és közepes méretű adatkészletekhez alkalmas (&lt; 200 kép)

### Chloros+ mód (párhuzamos feldolgozás)

**Chloros+ licenc szükséges**

**Hogyan működik:**

* Több képet dolgoz fel egyszerre egy [4-szálas feldolgozási folyamat](../processing-architecture/processing-pipeline.md) segítségével
* A [dinamikus számítási adaptáció](../processing-architecture/dynamic-compute-adaptation.md) a futtatás indításakor automatikusan kiválasztja a hardveréhez legmegfelelőbb stratégiát
* GPU (CUDA) gyorsítás NVIDIA grafikus kártyákkal
* **A munkaszálak száma a hardverhez igazodik**: a GPU-stratégiák**2–4 párhuzamos munkaszálat** futtatnak (a VRAM-méret függvényében — 3 egy 2–12 GB-os GPU-n, 2–4 egy 12 GB-nál nagyobb GPU-n); A kizárólag CPU-val rendelkező rendszerek fizikai magonként egy munkaszálat futtatnak, mínusz egyet**A haladási sáv 4 szakaszt jelenít meg** (a 4 folyamatmenet-szálnak megfelelően):

1. **Észlelés** (1. szál) – Kalibrációs célpontok keresése
2. **Elemzés** (2. szál) – A kép metaadatainak vizsgálata és a kalibrálás kiszámítása
3. **Kalibrálás** (3. szál) – Debayering, vignettakorrekció, kalibrálás, indexszámítás
4. **Exportálás** (4. szál) – A feldolgozott képek és indexek mentése**A haladási sáv kezelése:*** **Vigye az egérmutatót** a sáv fölé a részletes, 4 szakaszból álló legördülő panel megjelenítéséhez
* **Kattintson** a haladási sávra a legördülő panel rögzítéséhez
* **Kattintson újra** a panel rögzítésének feloldásához és elrejtéséhez**Feldolgozási idő:**

* Jelentősen gyorsabb, mint az ingyenes mód
* A GPU-gyorsítás tovább növeli a sebességet

{% hint style="info" %}
**Chloros+ sebesség**: Nagy adathalmazok esetén a párhuzamos feldolgozás 5–10-szer gyorsabb lehet, mint a szekvenciális mód. Egy 500 képet tartalmazó projekt, amely ingyenes módban 2 órát vesz igénybe, a Chloros+ használatával 15–20 perc alatt elkészülhet.
{% endhint %}

***

## Mi történik a feldolgozás során

### 1. szakasz: Célpont-felismerés

**A Chloros feladata:**

* Beolvassa a „Célpont” oszlopban bejelölt képeket (ha egyik sem jelölve, akkor az összes képet)
* Azonosítja a kalibrációs paneleket az egyes célpontokon
* Kivonja a reflexiós értékeket a célpanelekből
* Rögzíti a célok időbélyegeit a kalibrálás ütemezéséhez

**Időtartam:** 1–30 másodperc (megjelölt célok esetén), 5–30+ perc (megjelöletlenek esetén)

### 2. szakasz: Debayering (RAW-konverzió)

**A Chloros feladata:**

* Konvertálja a RAW Bayer-mintázatú adatokat teljes 3-csatornás képekké (a LATTICE mono modulok egysávosak maradnak — ezek esetében a debayeringet kihagyja, és erről megjegyzést fűz a naplóba)
* Alkalmazza a kiválasztott demosaicing algoritmust
* Megőrzi a maximális képminőséget és a részleteket

**Időtartam:** A képek számától és a CPU/GPU sebességétől függően változik

### 3. szakasz: Kalibrálás

**A Chloros feladata:*** **Vignettálás-korrekció**: Eltávolítja a lencse szélein jelentkező sötétedést
* **Fényvisszaverődési kalibrálás**: Normalizálja a képet a célfényvisszaverődési értékek és/vagy a DAQ lefelé irányuló sugárzási adatok alapján
* A korrekciókat minden sávra/csatornára alkalmazza
* Időbélyeg alapján minden képhez a megfelelő kalibrációs referenciát használja

**Időtartam:** A feldolgozási idő nagy része

### 4. szakasz: Indexszámítás

**A Chloros feladata:**

* Kiszámítja a beállított multispektrális indexeket (NDVI, NDRE stb.)
* Alkalmazza a sávszámításokat a kalibrált képekre
* Indexképeket generál minden kiválasztott indexhez

**Időtartam:** Képenként néhány másodperc

### 5. lépés: Exportálás

**A Chloros feladata:**

* A feldolgozott képeket a kiválasztott formátumban menti
* **LATTICE fan-out**: minden nyers LATTICE-képkockát egy lépésben exportál minden engedélyezett termékként — debayered, preview, radiance (mindig float32), reflectance
* A fájlokat a projekt kimeneti mappájába írja: `<project>/<camera>/<format>/<Product>_Images/`
* **Megtartja a forrásfájl nevét** — a mappa azonosítja a terméket, utótagot nem ad hozzá**Időtartam:** Az exportformátumtól és a fájlmérettől függően változik***

## Feldolgozási viselkedés

### Automatikus feldolgozási folyamat

Az indítás után a teljes folyamat automatikusan lefut:

* Nincs szükség felhasználói beavatkozásra
* Az összes beállított lépés sorrendben hajtódik végre
* A folyamat állása valós időben jelenik meg
* Az exportált fájlok elkészülésükkor azonnal a lemezre kerülnek — a kész kimeneteket már a feldolgozás közben is megnyithatja

### A számítógép terhelése a feldolgozás során

**Szabad mód:**

* Viszonylag alacsony CPU-terhelés (egyszálas)
* A számítógép továbbra is reagál más feladatokra
* Biztonságosan minimalizálható a Chloros, és más alkalmazásokban is lehet dolgozni

**Chloros+ párhuzamos mód:**

* Magas CPU-használat a stratégia munkavállalói pooljában
* GPU-gyorsítással: Magas GPU-használat
* A számítógép a feldolgozás alatt kevésbé reagálhat
* Kerülje más, CPU-igényes feladatok elindítását

{% hint style="warning" %}
**Teljesítménytipp**: A Chloros+ legjobb teljesítményének elérése érdekében zárja be az egyéb alkalmazásokat, és hagyja, hogy a Chloros a rendszer erőforrásait teljes mértékben kihasználja.
{% endhint %}

### A feldolgozás nem állítható szünetre (de a leállítás tiszta)

* Az indítás után a feldolgozás nem állítható szünetre, majd később nem folytatható
* A **Stop** gombra kattintva a futtatás az első kattintáskor tisztán leáll
* A leállítás előtt már exportált termékek a lemezen maradnak
* A leállított futtatás pontosan jelenti, hogy mit végzett el (lásd az `[RUN-SUMMARY]` sorokat a naplóban)
* Egy új futtatás az elejétől indítja el a feldolgozási folyamatot

**Tervezési tipp:** Nagyon nagy projektek esetén fontolja meg a feldolgozás kötegenkénti elvégzését vagy a CLI használatát a jobb ellenőrzés érdekében.***

## A feldolgozás figyelemmel kísérése

A feldolgozás futása közben a következőket teheti:

* **A haladási sáv figyelése** – Az összességében elért befejezési százalék megtekintése
* **Az aktuális szakasz megtekintése** – Felismerés, elemzés, kalibrálás vagy exportálás
* **A napló fül ellenőrzése** – A feldolgozással kapcsolatos részletes üzenetek és figyelmeztetések megtekintése
* **A befejezett képek előnézete** – Az exportált fájlok a feldolgozás során megjelennek a lemezen

A figyelemmel kíséréssel kapcsolatos részletes információkért lásd: [A feldolgozás figyelemmel kísérése](monitoring-the-processing.md).

***

## A feldolgozás leállítása

Ha le kell állítania a feldolgozást:

### A leállítás módja

1. Keresse meg a **Leállítás gombot** (a feldolgozás során ez váltja fel az Indítás gombot)
2. Kattintson rá egyszer — a sávon **„Leállítás...”** felirat jelenik meg, amíg a feldolgozás alatt álló kép befejeződik
3. A futtatás végleges leállított állapotban ér véget, és a naplóban egy `[RUN-SUMMARY]` kód jelenik meg, amely pontosan leírja, mi készült el

### Mikor kell leállítani

**Érvényes okok a leállításra:**

* Rájött, hogy helytelen beállításokat használt
* Elfelejtette megjelölni a célképeket
* Rossz képeket importált
* A rendszer túl lassan fut, vagy nem reagál

**A leállítás után:**

* A leállítás előtt exportált termékek a lemezen maradnak
* Ellenőrizze és javítsa ki az esetleges problémákat, szükség szerint módosítsa a beállításokat
* Indítsa újra a feldolgozást — a futtatás az elejétől kezdődik

***

## A feldolgozási idő becslése

A tényleges feldolgozási idő nagymértékben függ a következőktől:

* A képek száma
* A képek felbontása
* RAW vagy JPG bemeneti formátum
* Feldolgozási mód (Ingyenes vs. Chloros+)
* A CPU sebessége és magjainak száma
* GPU rendelkezésre állása (csak Chloros+ esetén)
* A kiszámítandó indexek száma
* Az engedélyezett exporttermékek száma (LATTICE)

### Hozzávetőleges becslések (Chloros+, 12 MP-es képek, modern CPU)

| Képek száma | Ingyenes mód | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 kép   | 15–20 perc | 5–8 perc        | 3–5 perc        |
| 100 kép  | 30–40 perc | 10–15 perc      | 5–8 perc        |
| 200 kép  | 1–1,5 óra | 20–30 perc      | 10–15 perc      |
| 500 kép  | 2–3 óra   | 45–60 perc      | 20–30 perc      |
| 1000 kép | 4–6 óra   | 1,5–2 óra      | 40–60 perc      |

{% hint style="info" %}
**Első futtatás**: Az első feldolgozás hosszabb ideig tarthat, mivel a Chloros cache-eket és profilokat épít. A hasonló adatkészletek későbbi feldolgozása gyorsabb lesz.
{% endhint %}

***

## Gyakori problémák az indításkor

### Az Indítás gomb letiltva (szürkén jelenik meg)

**Lehetséges okok:**

* Nincs importált kép
* A háttérprogram nem indult el teljesen
* Az előző feldolgozás még fut
* A projekt nem töltődött be teljesen

**Megoldások:**

1. Várja meg, amíg a háttérrendszer teljesen inicializálódik (ellenőrizze a főmenü ikonját)
2. Ellenőrizze, hogy a képek be vannak-e importálva a Fájlkezelőben
3. Indítsa újra a Chlorost, ha a gomb továbbra is letiltott
4. Ellenőrizze a hibajelentéseket a hibakeresési naplóban

### A feldolgozás elindul, majd azonnal meghiúsul

**Lehetséges okok:**

* Nincsenek érvényes képek a projektben
* Sérült képfájlok
* Nincs elegendő lemezterület
* Nincs elegendő memória (RAM)

**Megoldások:**

1. Ellenőrizze a hibajelentéseket a hibakeresési naplóban <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">
2. Ellenőrizze a rendelkezésre álló lemezterületet
3. Próbálja meg feldolgozni a képek egy kisebb részhalmazát
4. Ellenőrizze, hogy a képek nem sérültek-e meg

### A futtatás befejeződik, de nem ír ki képeket

Az a futtatás, amely képtermékeket kért, de egyet sem írt ki, **kudarcnak, nem sikernek** minősül — a Chloros ezt hangosan jelzi:

* A grafikus felület naplója `[RUN-SUMMARY]` hibakódokat jelenít meg, amelyek utalnak a valószínűsíthető okra — nem importáltak képeket, nem észlelt célpontot, vagy minden kért terméket alkalmazhatatlannak minősítve kihagytak (pl. sugárzási/visszaverődési értékek kérése kizárólag RGB-kamerákról)
* Az CLI-megfelelő (`chloros-cli process`) az `Processing finished but wrote no image products.` üzenetet írja ki, és **nem nulla értékkel lép ki**, így a szkriptek felismerhetik
* A szándékosan kizárólag metaadatokra korlátozott futtatás (minden exporttermék letiltva, nincsenek indexek) továbbra is sikernek számít

A teljes szemantikaért lásd [a CLI-referenciát](../reference/cli-reference.md#a-run-that-writes-no-images-fails).

### A projektmappa túl mélyen van ezen a számítógépen

**Csak Windows, újdonság a 1.3.0-s verzióban.** A Windows-ban alapértelmezés szerint a hosszú elérési utak ki vannak kapcsolva, így egyetlen fájl elérési útja sem lehet hosszabb 260 karakternél. A Chloros-termékek a projektmappa alatt körülbelül 130 karakterrel helyezkednek el, így egy ennél mélyebben elhelyezkedő projektmappa nem tudná fogadni a termékeket. Ahelyett, hogy futna és semmit sem írna, a Chloros most a feldolgozás megkezdése előtt ellenőrzi:

* A „Start” gombra kattintás nem indítja el a futtatást: a haladási sáv megmutatja az okot, a hibakeresési napló pedig az `[PROCESS] refused: The project folder is too deep for this computer…` kódot jeleníti meg. Az üzenet megadja a projektmappa hosszát és a leghosszabb, még befogadható projektmappát.
* A projekt létrehozása vagy megnyitása olyan mappában, amely még a projekt saját fájljai számára is túl mély, szintén elutasításra kerül. Egy üzenetablak jelzi az okot, feltüntetve a leghosszabb megengedett projektmappát és azt, hogyan lehet engedélyezni a hosszú elérési utakat, és semmi sem jön létre, illetve nem nyílik meg; a hibakeresési naplóban `[PROJECT] refused: …` kóddal kerül rögzítésre. A projekt hosszabb névvel történő másolása szintén túllépheti a korlátot: a másolat elkészül, de nem nyílik meg, és ugyanaz az üzenet jelzi az okot.

**Megoldások:**

1. Tárolja a projekteket rövidebb mappában — módosítsa a**Munkakönyvtárat** a [Projektbeállítások](../project-settings/project-settings.md) menüpontban, majd hozza létre újra a projektet, vagy
2. Kapcsolja be a Windows hosszú elérési útjait, és indítsa újra a Chloros programot: futtassa újra a Chloros telepítőjét az **Enable Windows long paths** (Windows hosszú elérési utak engedélyezése) jelölőnégyzet bejelölésével (ez alapértelmezés szerint be van jelölve), vagy használja a Csoportházirendet: *Számítógép konfigurációja &gt; Felügyeleti sablonok &gt; Rendszer &gt; Fájlrendszer &gt; Win32 hosszú elérési utak engedélyezése*.

### „Nincs célpont észlelve” figyelmeztetés

**Lehetséges okok:**

* Elfelejtette megjelölni a célképeket
* A célképek nem tartalmaznak látható célpontokat
* A célpont-felismerési beállítások túl szigorúak

**Megoldások:**

1. Olvassa el a [Célképek kiválasztása](choosing-target-images.md)
2. Jelölje meg a megfelelő képeket a „Cél” oszlopban
3. Ellenőrizze, hogy a megjelölt képeken láthatók-e a célok
4. Szükség esetén módosítsa a célfelismerési beállításokat

***

## Tippek a sikeres feldolgozáshoz

### Indítás előtt

1. **Először kis adathalmazzal teszteljen** – Dolgozzon fel 10–20 képet a beállítások ellenőrzéséhez
2. **Ellenőrizze a rendelkezésre álló lemezterületet** – Gondoskodjon az adathalmaz méretének 2–3-szorosáról (több, ha az összes LATTICE-termék engedélyezve van)
3. **Zárja be a felesleges alkalmazásokat** – Szabadítson fel rendszererőforrásokat
4. **Ellenőrizze a célképeket** – Tekintse át a megjelölt célokat a minőség biztosítása érdekében
5. **Mentse el a projektet** – A projekt automatikusan mentődik, de célszerű manuálisan is elmenteni

### A feldolgozás során

1. **Kerülje a rendszer alvó üzemmódját** – Kapcsolja ki az energiatakarékos üzemmódokat
2. **Tartsa az Chloros-t az előtérben** – Vagy legalábbis láthatóan a tálcán
3. **Időnként ellenőrizze a folyamatot** – Ellenőrizze, hogy nincsenek-e figyelmeztetések vagy hibák
4. **Ne indítson el más erőforrásigényes alkalmazásokat** – Különösen az Chloros+ párhuzamos üzemmódban

### A Chloros+ GPU-gyorsítása

NVIDIA GPU-gyorsítás használata esetén:

1. Frissítse az NVIDIA illesztőprogramokat a legújabb verzióra
2. Győződjön meg arról, hogy a GPU rendelkezik legalább 4 GB VRAM-mal (7 GB vagy több a párhuzamos Texture Aware debayeringhez)
3. Zárja be a GPU-igényes alkalmazásokat (játékok, videószerkesztés)
4. Figyelje a GPU hőmérsékletét (gondoskodjon megfelelő hűtésről)

***

## Következő lépések

A feldolgozás megkezdése után:

1. **Kövesse nyomon a folyamatot** – Lásd: [A feldolgozás nyomon követése](monitoring-the-processing.md)
2. **Várja meg a befejezést** – A feldolgozás automatikusan fut
3. **Ellenőrizze az eredményeket** – Lásd: [A feldolgozás befejezése](finishing-the-processing.md)

A feldolgozás alatt teendőkről szóló információkat lásd: [A feldolgozás figyelemmel kísérése](monitoring-the-processing.md).
