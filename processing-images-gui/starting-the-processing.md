# A feldolgozás elindítása

Miután importálta a képeket, megjelölte a kalibrációs célpontokat és beállította a projektbeállításokat, készen áll a feldolgozás megkezdésére. Ez az oldal végigvezeti Önt az Chloros feldolgozási folyamat elindításán.

## Előfeldolgozási ellenőrzőlista

Mielőtt rákattintana a Start gombra, ellenőrizze, hogy minden készen áll-e:

* [ ] **Fájlok importálva** – Minden kép megjelenik a Fájlkezelőben
* [ ] **Célképek megjelölve** – A Cél oszlopban bejelölve vannak a kalibrációs képek (vagy importálva van egy `.daq` felvétel a LATTICE számára)
* [ ] **Kamera modellek felismerve** – A „Kamera modell” oszlop a megfelelő kamerákat jelzi
* [ ] **Beállítások konfigurálva** – A projektbeállításokat átnézték és módosították
* [ ] **Indexek kiválasztva** – A kívánt multispektrális indexek hozzáadva (ha szükséges)
* [ ] **Kiválasztott exportformátum** – A munkafolyamatához megfelelő kimeneti formátum

{% hint style="info" %}
**Tipp**: A feldolgozás megkezdése előtt kattintson át néhány képre a Fájlkezelőben, hogy ellenőrizze, hogy azok megfelelően betöltődtek-e.
{% endhint %}

***

## A feldolgozás elindítása

### A Start gomb megkeresése

A Start/Lejátszás gomb az Chloros felső fejlécsávjában található:

* Hely: Az ablak felső közepén
* Ikon: **Lejátszás/Start gomb** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Állapot: A gomb akkor aktív (világít), ha a feldolgozásra készen áll

### Kattintson a kezdéshez

1. Kattintson a felső fejlécben található **Lejátszás/Indítás gombra**

2. A feldolgozás azonnal megkezdődik
3. A gomb a feldolgozás során **Leállítás** gombbá válik
4. A haladási sáv frissül, jelezve a feldolgozás állapotát

{% hint style="success" %}
**Feldolgozás elindítva**: A gombra kattintás után az Chloros automatikusan elvégzi az összes feldolgozási lépést – célpont-felismerés, debayering, kalibrálás, indexszámítás és exportálás. A program automatikusan felismeri, hogy a projekt Survey3, LATTICE vagy ezek keveréke-e, és minden kamerához a megfelelő feldolgozási folyamatot alkalmazza.
{% endhint %}

***

## A feldolgozási módok ismertetése

Az Chloros a licencétől függően két különböző feldolgozási módban működik:

### Ingyenes mód (szekvenciális feldolgozás)

**Minden felhasználó számára elérhető**

**Hogyan működik:**

* A képeket egyenként, egymás után dolgozza fel
* Egy szálon futó működés
* Alacsonyabb memóriahasználat

**A haladási sáv 2 szakaszt jelenít meg:**

1.**Célpont-felismerés** – Kalibrációs célpontok keresése
2. **Feldolgozás** – Kalibrálás alkalmazása és a képek exportálása**Feldolgozási idő:**

* Sokkal lassabb, mint az Chloros+ párhuzamos üzemmód
* Kicsi és közepes méretű adatkészletekhez alkalmas (&lt; 200 kép)

### Chloros+ üzemmód (párhuzamos feldolgozás)

**Chloros+ licenc szükséges**

**Hogyan működik:**

* Több képet dolgoz fel egyszerre egy [4-szálas feldolgozási folyamat](../processing-architecture/processing-pipeline.md) segítségével
* A [dinamikus számítási adaptáció](../processing-architecture/dynamic-compute-adaptation.md) a futtatás indításakor automatikusan kiválasztja a hardverhez legmegfelelőbb stratégiát
* GPU (CUDA) gyorsítás NVIDIA grafikus kártyákkal (asztali és Jetson)
* **A munkaszálak száma a hardverhez igazodik**: a GPU-stratégiák**1–4 párhuzamos munkaszálat** futtatnak (a VRAM-méret függvényében — egy alacsony memóriájú Jetson 1-et, egy 12 GB-os vagy annál nagyobb memóriájú asztali GPU pedig legfeljebb 4-et futtat); a kizárólag CPU-val rendelkező rendszerek fizikai magonként egy munkaszálat futtatnak, mínusz egy**A haladási sáv 4 szakaszt jelenít meg** (a 4 folyamatmenet-szálnak megfelelően):

1. **Észlelés** (1. szál) – Kalibrációs célpontok keresése
2. **Elemzés** (2. szál) – A kép metaadatainak vizsgálata és a kalibráció kiszámítása
3. **Kalibrálás** (3. szál) – Debayering, vignettálás-korrekció, kalibrálás, indexszámítás
4. **Exportálás** (4. szál) – A feldolgozott képek és indexek mentése**A haladási sáv kezelése:*** **Vigye az egérmutatót** a sáv fölé a részletes, 4 lépésből álló legördülő panel megjelenítéséhez
* **Kattintson** az előrehaladási sávra a legördülő panel rögzítéséhez
* **Kattintson újra** a panel rögzítésének feloldásához és elrejtéséhez**Feldolgozási idő:**

* Jelentősen gyorsabb, mint az ingyenes mód
* A GPU-gyorsítás tovább növeli a sebességet

{% hint style="info" %}
**Chloros+ Sebesség**: A párhuzamos feldolgozás nagy adathalmazok esetén 5–10-szer gyorsabb lehet, mint a szekvenciális mód. Egy 500 képet tartalmazó projekt, amely ingyenes módban 2 órát vesz igénybe, az Chloros+ használatával 15–20 perc alatt elkészülhet.
{% endhint %}

***

## Mi történik a feldolgozás során?

### 1. szakasz: Célfelismerés

**Mit csinál az Chloros:**

* Beolvassa a „Cél” oszlopban bejelölt képeket (ha nincs bejelölve, akkor az összes képet)
* Azonosítja a kalibrációs paneleket az egyes célokban
* Kivonja a reflexiós értékeket a célpanelekből
* Rögzíti a célok időbélyegeit a kalibráció ütemezéséhez

**Időtartam:** 1–30 másodperc (megjelölt célok esetén), 5–30+ perc (megjelöletlen célok esetén)

### 2. szakasz: Debayering (RAW-konverzió)

**Az Chloros működése:**

* Konvertálja a RAW Bayer-mintázatú adatokat teljes 3-csatornás képekké (a LATTICE mono modulok egysávosak maradnak — ezek esetében a debayeringet kihagyja, és erről megjegyzést fűz a naplóba)
* Alkalmazza a kiválasztott demosaicing algoritmust
* Megőrzi a maximális képminőséget és a részleteket

**Időtartam:** A képek számától és a CPU/GPU sebességétől függően változik

### 3. lépés: Kalibrálás

**Az Chloros feladata:*** **Vignettálás-korrekció**: Eltávolítja a lencse szélein jelentkező sötétedést
* **Fényvisszaverődési kalibrálás**: Normalizálja a képet a cél fényvisszaverődési értékek és/vagy a DAQ lefelé irányuló sugárzási adatok felhasználásával
* A korrekciókat minden sávra/csatornára alkalmazza
* Minden képhez az időbélyeg alapján a megfelelő kalibrációs referenciát használja

**Időtartam:** A feldolgozási idő nagy része

### 4. szakasz: Indexszámítás

**Az Chloros funkciója:**

* Kiszámítja a beállított multispektrális indexeket (NDVI, NDRE stb.)
* Sávszámításokat alkalmaz a kalibrált képeken
* Indexképeket generál minden kiválasztott indexhez

**Időtartam:** Képenként néhány másodperc

### 5. szakasz: Exportálás

**Mit csinál az Chloros:**

* A feldolgozott képeket a kiválasztott formátumban menti
* **LATTICE fan-out**: minden nyers LATTICE-képkockát egy lépésben exportál az összes engedélyezett termék formájában — debayered, preview, radiance (mindig float32), reflectance
* A fájlokat a projekt kimeneti mappájába írja: `<project>/<camera>/<format>/<Product>_Images/`
* **Megtartja a forrásfájl nevét** — a mappa azonosítja a terméket, utótagot nem ad hozzá**Időtartam:** Az exportformátumtól és a fájlmérettől függően változik***

## Feldolgozási viselkedés

### Automatikus feldolgozási folyamat

Az indítás után a teljes folyamat automatikusan lefut:

* Nincs szükség felhasználói beavatkozásra
* Az összes beállított lépés egymás után fut le
* A folyamat állása valós időben jelenik meg
* Az exportált fájlok elkészülésükkor azonnal a lemezre kerülnek — a futás folytatása közben megnyithatja a kész kimeneti fájlokat

### A számítógép használata a feldolgozás során

**Szabad mód:**

* Viszonylag alacsony CPU-terhelés (egyszálas)
* A számítógép továbbra is reagál más feladatokra
* Biztonságosan minimalizálható az Chloros ablak, és más alkalmazásokban is dolgozhat

**Chloros+ Párhuzamos mód:**

* Magas CPU-terhelés a stratégia munkavállalói csoportjában
* GPU-gyorsítással: Magas GPU-terhelés
* A számítógép a feldolgozás alatt kevésbé reagálhat
* Kerülje más CPU-igényes feladatok elindítását

{% hint style="warning" %}
**Teljesítménytípus**: Az Chloros+ optimális teljesítményéhez zárja be az egyéb alkalmazásokat, és hagyja, hogy az Chloros a rendszer összes erőforrását kihasználja.
{% endhint %}

### A feldolgozás nem szüneteltethető (de a leállítás tiszta)

* A feldolgozás elindítása után nem szüneteltethető, majd később nem folytatható
* A **Stop** gombra kattintva a futtatás az első kattintással tiszta módon leáll
* A leállítás előtt már exportált termékek a lemezen maradnak
* A leállított futtatás pontosan jelenti, hogy mit végzett el (lásd az `[RUN-SUMMARY]` sorokat a naplóban)
* Egy új futtatás az elejétől indítja el a feldolgozási folyamatot

**Tervezési tipp:** Nagyon nagy projektek esetén fontolja meg a feldolgozás köteges elvégzését, vagy az CLI használatát a jobb ellenőrzés érdekében.***

## A feldolgozás figyelemmel kísérése

A feldolgozás futása közben a következőket teheti:

* **A haladási sáv figyelése** – Az általános befejezettségi százalék megtekintése
* **Az aktuális szakasz megtekintése** – Felismerés, elemzés, kalibrálás vagy exportálás
* **A napló fül ellenőrzése** – A feldolgozással kapcsolatos részletes üzenetek és figyelmeztetések megtekintése
* **A befejezett képek előnézete** – Az exportált fájlok a feldolgozás során megjelennek a lemezen

A figyelemmel kíséréssel kapcsolatos részletes információkért lásd: [A feldolgozás figyelemmel kísérése](monitoring-the-processing.md).

***

## A feldolgozás leállítása

Ha le kell állítania a feldolgozást:

### A leállítás módja

1. Keresse meg a **Leállítás gombot** (a feldolgozás során ez váltja fel a Start gombot)
2. Kattintson rá egyszer — a sávon **„Leállítás...”** felirat jelenik meg, amíg a feldolgozás alatt álló kép befejeződik
3. A futtatás végleges leállított állapotban ér véget, és a naplóban egy `[RUN-SUMMARY]` kód jelenik meg a befejezett műveletekről

### Mikor kell leállítani?

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
* Feldolgozási mód (Ingyenes vagy Chloros+)
* A CPU sebessége és magszáma
* GPU rendelkezésre állása (csak Chloros+)
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
**Első futtatás**: Az első feldolgozás hosszabb ideig tarthat, mivel az Chloros cache-eket és profilokat épít fel. A hasonló adatkészletek későbbi feldolgozása gyorsabb lesz.
{% endhint %}

***

## Gyakori problémák az indításkor

### Az Indítás gomb le van tiltva (szürkén jelenik meg)

**Lehetséges okok:**

* Nincs importált kép
* A háttérprogram nem indult el teljesen
* Az előző feldolgozás még fut
* A projekt nem töltődött be teljesen

**Megoldások:**

1. Várja meg, amíg a háttérrendszer teljesen inicializálódik (ellenőrizze a főmenü ikonját)
2. Ellenőrizze, hogy a képek be vannak-e importálva a Fájlkezelőbe
3. Indítsa újra az Chloros programot, ha a gomb továbbra is letiltva van
4. Ellenőrizze a hibajelentéseket a hibakeresési naplóban

### A feldolgozás elindul, majd azonnal meghiúsul

**Lehetséges okok:**

* Nincsenek érvényes képek a projektben
* Sérült képfájlok
* Nem elegendő lemezterület
* Nem elegendő memória (RAM)

**Megoldások:**

1. Ellenőrizze a hibajelentéseket a hibakeresési naplóban (<img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">)
2. Ellenőrizze a rendelkezésre álló lemezterületet
3. Próbálja meg feldolgozni a képek egy kisebb részhalmazát
4. Ellenőrizze, hogy a képek nem sérültek-e

### A futtatás befejeződik, de nem ír ki képeket

Az a futtatás, amely képtermékeket kért, de egyet sem írt ki, **kudarcnak, nem sikernek** minősül — az Chloros ezt egyértelműen jelzi:

* A grafikus felület naplója az `[RUN-SUMMARY]` kóddal jelzi a valószínű okot — nem importáltak képeket, nem észlelték a célt, vagy minden kért terméket kihagytak, mert nem alkalmazható (pl. sugárzási/reflektanciát kértünk kizárólag RGB kamerákból)
* Az CLI megfelelője (`chloros-cli process`) kiírja az `Processing finished but wrote no image products.` üzenetet és **nem nulla értékkel lép ki**, így a szkriptek felismerhetik
* A szándékosan csak metaadatokra korlátozott futtatás (minden exporttermék letiltva, indexek nélkül) is sikernek számít

A teljes szemantikaért lásd [az CLI-referenciát](../reference/cli-reference.md#a-run-that-writes-no-images-fails).

### „Nincs célpont észlelve” figyelmeztetés

**Lehetséges okok:**

* Elfelejtette megjelölni a célképeket
* A célképek nem tartalmaznak látható célpontokat
* A célpont-felismerési beállítások túl szigorúak

**Megoldások:**

1. Olvassa el a [Célképek kiválasztása](choosing-target-images.md) című részt
2. Jelölje meg a megfelelő képeket a „Cél” oszlopban
3. Ellenőrizze, hogy a megjelölt képeken láthatók-e a célok
4. Szükség esetén módosítsa a célfelismerési beállításokat

***

## Tippek a sikeres feldolgozáshoz

### A kezdés előtt

1. **Először egy kis részhalmazzal teszteljen** – Dolgozzon fel 10–20 képet a beállítások ellenőrzéséhez
2. **Ellenőrizze a rendelkezésre álló lemezterületet** – Gondoskodjon az adatkészlet méretének 2–3-szorosáról (többet, ha az összes LATTICE termék engedélyezve van)
3. **Zárja be a felesleges alkalmazásokat** – Szabadítson fel rendszererőforrásokat
4. **Ellenőrizze a célképeket** – Tekintse meg előzetesen a megjelölt célokat a minőség biztosítása érdekében
5. **Mentse el a projektet** – A projekt automatikusan mentődik, de célszerű manuálisan is elmenteni

### A feldolgozás során

1. **Kerülje a rendszer alvó üzemmódját** – Kapcsolja ki az energiatakarékos üzemmódokat
2. **Tartsa az Chloros programot az előtérben** – Vagy legalábbis láthatóan a tálcán
3. **Időnként ellenőrizze a folyamatot** – Figyeljen a figyelmeztetésekre és hibákra
4. **Ne indítson el más erőforrás-igényes alkalmazásokat** – Különösen az Chloros+ párhuzamos üzemmódban

### Chloros+ GPU-gyorsítás

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

A feldolgozás alatt teendőkről szóló információkat a [A feldolgozás figyelemmel kísérése](monitoring-the-processing.md) című részben találja.
