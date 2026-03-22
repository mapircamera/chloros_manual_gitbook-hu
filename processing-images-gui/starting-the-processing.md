# A feldolgozás elindítása

Miután importálta a képeket, megjelölte a kalibrációs célpontokat és beállította a projekt paramétereit, készen áll a feldolgozás megkezdésére. Ez az oldal végigvezeti Önt az Chloros feldolgozási folyamat elindításán.

## Előfeldolgozási ellenőrzőlista

Mielőtt rákattintana a Start gombra, ellenőrizze, hogy minden készen áll-e:

* [ ] **Fájlok importálva** - Minden kép megjelenik a Fájlkezelőben
* [ ] **Célképek megjelölve** - A kalibrációs képeknél bejelölve a Cél oszlop
* [ ] **Kamera modellek felismerve** - A Kamera modell oszlop a megfelelő kamerákat mutatja
* [ ] **Beállítások konfigurálva** - A projektbeállítások áttekintve és módosítva
* [ ] **Indexek kiválasztva** – A kívánt multispektrális indexek hozzáadva (ha szükséges)
* [ ] **Exportformátum kiválasztva** – A munkafolyamatához megfelelő kimeneti formátum

{% hint style="info" %}
**Tipp**: Kattintson át néhány képen a Fájlkezelőben, hogy ellenőrizze, hogy a feldolgozás előtt helyesen töltődtek-e be.
{% endhint %}

***

## A feldolgozás elindítása

### Keresse meg a Start gombot

A Start/Play gomb az Chloros felső fejlécsávjában található:

* Hely: Az ablak felső közepén
* Ikon: **Play/Start gomb** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Állapot: A gomb akkor aktív (világít), ha a feldolgozásra készen áll

### Kattintson a kezdéshez

1. Kattintson a felső fejlécben található **Lejátszás/Start gombra**

2. A feldolgozás azonnal megkezdődik
3. A gomb a feldolgozás alatt inaktívvá válik (szürkére vált)
4. A haladási sáv frissül, jelezve a feldolgozás állapotát

{% hint style="success" %}
**Feldolgozás elindítva**: A gombra kattintás után az Chloros automatikusan elvégzi az összes feldolgozási lépést – célpontfelismerés, debayering, kalibrálás, indexszámítás és exportálás.
{% endhint %}

***

## A feldolgozási módok megértése

Az Chloros a licencétől függően két különböző feldolgozási módban működik:

### Ingyenes mód (szekvenciális feldolgozás)

**Minden felhasználó számára elérhető**

**Hogyan működik:**

* A képeket egyenként, egymás után dolgozza fel
* Egy szálú működés
* Alacsonyabb memóriahasználat

**A haladási sáv 2 szakaszt mutat:**

1.**Célpont felismerés** – Kalibrációs célpontok keresése
2. **Feldolgozás** – Kalibráció alkalmazása és képek exportálása**Feldolgozási idő:**

* Sokkal lassabb, mint az Chloros+ párhuzamos mód
* Kis- és közepes méretű adathalmazokhoz alkalmas (&lt; 200 kép)

### Chloros+ mód (párhuzamos feldolgozás)

**Chloros+ licenc szükséges**

**Hogyan működik:**

* Több képet dolgoz fel egyszerre egy [4-szálas feldolgozási folyamat](../processing-architecture/processing-pipeline.md) segítségével
* A [dinamikus számítási adaptáció](../processing-architecture/dynamic-compute-adaptation.md) automatikusan kiválasztja a hardveréhez legmegfelelőbb stratégiát
* GPU (CUDA) gyorsítás NVIDIA grafikus kártyákkal (asztali és Jetson)
* Skálázható a Jetson Nano-tól (1 munkavégző) egy 12 GB-os vagy annál nagyobb GPU-val rendelkező asztali számítógépig (3-4 munkavégző)

**A haladási sáv 4 szakaszt mutat** (a 4 folyamatmenet szálnak megfelelően):

1. **Észlelés** (1. szál) – Kalibrációs célpontok keresése
2. **Elemzés** (2. szál) – A kép metaadatainak vizsgálata és a kalibráció kiszámítása
3. **Kalibrálás** (3. szál) – GPU-val történő debayering, vignettás korrekció, index kiszámítása
4. **Exportálás** (4. szál) – A feldolgozott képek és indexek mentése**A haladási sáv használata:*** **Vigye az egérmutatót** a sáv fölé a részletes, 4 lépcsős legördülő panel megjelenítéséhez
* **Kattintson** a haladási sávra a legördülő panel rögzítéséhez
* **Kattintson újra** a panel feloldásához és elrejtéséhez**Feldolgozási idő:**

* Jelentősen gyorsabb, mint az ingyenes mód
* A CPU magok számával arányosan skálázódik
* A GPU-gyorsítás tovább növeli a sebességet

{% hint style="info" %}
**Chloros+ sebesség**: A párhuzamos feldolgozás nagy adathalmazok esetén 5-10-szer gyorsabb lehet, mint a szekvenciális mód. Egy 500 képet tartalmazó projekt, amely ingyenes módban 2 órát vesz igénybe, az Chloros+ használatával 15-20 perc alatt elkészülhet.
{% endhint %}

***

## Mi történik a feldolgozás során

### 1. szakasz: Célfelismerés

**Mit csinál az Chloros:**

* Beolvassa a megjelölt célképeket (vagy az összes képet, ha nincs megjelölve)
* Azonosítja a 4 kalibrációs panelt minden célban
* Kivonja a reflexiós értékeket a célpanelekből
* Rögzíti a cél időbélyegeit a kalibrációs ütemezéshez

**Időtartam:** 1–30 másodperc (jelölt célpontok esetén), 5–30+ perc (jelöletlen célpontok esetén)

### 2. szakasz: Debayering (RAW-konverzió)

**Mit csinál az Chloros:**

* Konvertálja a RAW Bayer-mintázatú adatokat teljes RGB képekké
* Kiváló minőségű demosaicing algoritmust alkalmaz
* Megőrzi a maximális képminőséget és a részleteket

**Időtartam:** A képek számától és a CPU sebességétől függően változik

### 3. szakasz: Kalibrálás

**Mit csinál az Chloros:*** **Vignettakorrekció**: Eltávolítja a lencse sötétedését a széleken
* **Reflektancia-kalibrálás**: Normalizálja a célreflektancia-értékek felhasználásával
* Korrekciókat alkalmaz az összes sávra/csatornára
* Időbélyeg alapján minden képhez a megfelelő kalibrációs célt használja

**Időtartam:** A feldolgozási idő nagy része

### 4. szakasz: Indexszámítás

**Az Chloros feladata:**

* Kiszámítja a konfigurált multispektrális indexeket (NDVI, NDRE stb.)
* Sávszámításokat alkalmaz a kalibrált képekre
* Indexképeket generál minden kiválasztott indexhez

**Időtartam:** Néhány másodperc képként

### 5. szakasz: Exportálás

**A Chloros feladata:**

* A kalibrált képek mentése a kiválasztott formátumban
* Az indexképek exportálása a konfigurált LUT-színekkel
* A fájlok írása a kameramodell alkönyvtáraiba
* Az eredeti fájlnevek megőrzése utótagokkal

**Időtartam:** Az export formátumtól és a fájl méretétől függően változik***

## Feldolgozási viselkedés

### Automatikus feldolgozási folyamat

Elindítás után a teljes folyamat automatikusan fut:

* Nincs szükség felhasználói beavatkozásra
* Az összes beállított lépés egymás után fut le
* A folyamat állása valós időben látható

### Számítógép-használat a feldolgozás során

**Szabad mód:**

* Viszonylag alacsony CPU-használat (egyszálas)
* A számítógép továbbra is reagál más feladatokra
* Biztonságosan minimalizálható az Chloros, és más alkalmazásokban is lehet dolgozni

**Chloros+ Párhuzamos mód:**

* Magas CPU-használat (többszálas, akár 16 mag)
* GPU-gyorsítással: Magas GPU-használat
* A számítógép a feldolgozás alatt kevésbé reagálhat
* Kerülje más CPU-igényes feladatok elindítását

{% hint style="warning" %}
**Teljesítménytipp**: A legjobb Chloros+ teljesítmény érdekében zárja be az egyéb alkalmazásokat, és hagyja, hogy az Chloros a rendszer teljes erőforrásait felhasználja.
{% endhint %}

### A feldolgozás nem állítható le

**Fontos korlátozások:**

* A feldolgozás elindítása után nem állítható le
* A feldolgozás megszakítható, de az elért eredmények elvesznek
* A részleges eredmények nem kerülnek mentésre
* Megszakítás esetén az elejétől kell újraindítani

**Tervezési tipp:** Nagyon nagy projektek esetén fontolja meg a feldolgozás köteges végrehajtását vagy az CLI használatát a jobb ellenőrzés érdekében.***

## A feldolgozás figyelemmel kísérése

A feldolgozás futása közben a következőket teheti:

* **A haladási sáv figyelése** – Az általános befejezési százalék megtekintése
* **Az aktuális szakasz megtekintése** – Felismerés, elemzés, kalibrálás vagy exportálás
* **Ellenőrizze a Napló fület** – A feldolgozással kapcsolatos részletes üzenetek és figyelmeztetések megtekintése
* **A kész képek előnézete** – Egyes exportfájlok már a feldolgozás során megjelenhetnek

A figyelemmel kíséréssel kapcsolatos részletes információkért lásd: [A feldolgozás figyelemmel kísérése](monitoring-the-processing.md).

***

## A feldolgozás megszakítása

Ha le kell állítania a feldolgozást:

### A megszakítás módja

1. Keresse meg a **Stop/Cancel gombot** (a feldolgozás során ez váltja fel a Start gombot)
2. Kattintson a Stop gombra
3. A feldolgozás azonnal leáll
4. A részleges eredmények elvésznek

### Mikor kell megszakítani

**Érvényes okok a megszakításra:**

* Rájött, hogy helytelen beállításokat használt
* Elfelejtette megjelölni a célképeket
* Rossz képeket importált
* A rendszer túl lassan fut vagy nem reagál

**A lemondás után:**

* Ellenőrizze és javítsa ki az esetleges problémákat
* Szükség szerint módosítsa a beállításokat
* Indítsa újra a feldolgozást az elejétől
* A legtisztább élmény érdekében teljesen zárja be az Chloros programot, majd indítsa újra

{% hint style="warning" %}
**Nincs részleges eredmény**: A lemondás az összes eddigi eredményt törli. Az Chloros nem menti a részlegesen feldolgozott képeket.
{% endhint %}

***

## A feldolgozási idő becslése

A tényleges feldolgozási idő nagymértékben függ a következőktől:

* A képek száma
* A képek felbontása
* RAW vagy JPG bemeneti formátum
* Feldolgozási mód (Ingyenes vs. Chloros+)
* A CPU sebessége és magjainak száma
* A GPU rendelkezésre állása (csak Chloros+)
* A kiszámítandó indexek száma
* Az exportformátum összetettsége

### Nagyjából becsült értékek (Chloros+, 12 MP-es képek, modern CPU)

| Képek száma | Ingyenes mód | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 kép   | 15–20 perc | 5–8 perc        | 3–5 perc        |
| 100 kép  | 30–40 perc | 10–15 perc      | 5–8 perc        |
| 200 kép  | 1–1,5 óra | 20–30 perc      | 10–15 perc      |
| 500 kép  | 2-3 óra   | 45-60 perc      | 20-30 perc      |
| 1000 kép | 4-6 óra   | 1,5-2 óra      | 40-60 perc      |

{% hint style="info" %}
**Első futtatás**: Az első feldolgozás hosszabb ideig tarthat, mivel az Chloros cache-eket és profilokat épít. A hasonló adatkészletek későbbi feldolgozása gyorsabb lesz.
{% endhint %}

***

## Gyakori problémák az indításkor

### A Start gomb letiltva (szürkén jelenik meg)

**Lehetséges okok:**

* Nincs importált kép
* A háttérprogram nem indult el teljesen
* Az előző feldolgozás még fut
* A projekt nem töltődött be teljesen

**Megoldások:**

1. Várja meg, amíg a háttérprogram teljesen inicializálódik (ellenőrizze a főmenü ikonját)
2. Ellenőrizze, hogy a képek importálva vannak-e a Fájlkezelőben
3. Indítsa újra az Chloros programot, ha a gomb továbbra is letiltva van
4. Ellenőrizze a hibajelentéseket a Debug Logban

### A feldolgozás elindul, majd azonnal meghiúsul

**Lehetséges okok:**

* Nincsenek érvényes képek a projektben
* Sérült képfájlok
* Nem elegendő lemezterület
* Nem elegendő memória (RAM)

**Megoldások:**

1. Ellenőrizze a hibakeresési naplót <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> a hibaüzenetek miatt
2. Ellenőrizze a rendelkezésre álló lemezterületet
3. Próbálja meg feldolgozni a képek egy kisebb részét
4. Ellenőrizze, hogy a képek nem sérültek-e

### „Nincs célpont észlelve” figyelmeztetés

**Lehetséges okok:**

* Elfelejtette megjelölni a célképeket
* A célképek nem tartalmaznak látható célpontokat
* A célpont-felismerési beállítások túl szigorúak

**Megoldások:**

1. Olvassa el a [Célképek kiválasztása](choosing-target-images.md) című részt
2. Jelölje meg a megfelelő képeket a Cél oszlopban
3. Ellenőrizze, hogy a megjelölt képeken láthatóak-e a célok
4. Szükség esetén módosítsa a célfelismerési beállításokat

***

## Tippek a sikeres feldolgozáshoz

### Indítás előtt

1. **Először teszteljen egy kis részhalmazzal** – Dolgozzon fel 10–20 képet a beállítások ellenőrzéséhez
2. **Ellenőrizze a rendelkezésre álló lemezterületet** – Gondoskodjon az adatkészlet méretének 2–3-szorosáról
3. **Zárja be a felesleges alkalmazásokat** – Szabadítson fel rendszererőforrásokat
4. **Ellenőrizze a célképeket** – Tekintse meg a megjelölt célpontok előnézetét a minőség biztosítása érdekében
5. **Mentse el a projektet** – A projekt automatikusan mentődik, de célszerű manuálisan is elmenteni

### A feldolgozás során

1. **Kerülje a rendszer alvó módját** – Kapcsolja ki az energiatakarékos módokat
2. **Tartsa az Chloros programot az előtérben** – Vagy legalábbis láthatóan a tálcán
3. **Ellenőrizze időnként a folyamatot** – Ellenőrizze, hogy nincsenek-e figyelmeztetések vagy hibák
4. **Ne töltsön be más erőforrásigényes alkalmazásokat** – Különösen az Chloros+ párhuzamos mód használata esetén

### Chloros+ GPU-gyorsítás

NVIDIA GPU-gyorsítás használata esetén:

1. Frissítse az NVIDIA illesztőprogramokat a legújabb verzióra
2. Győződjön meg arról, hogy a GPU rendelkezik legalább 4 GB VRAM-mal
3. Zárja be a GPU-igényes alkalmazásokat (játékok, videószerkesztés)
4. Figyelje a GPU hőmérsékletét (biztosítsa a megfelelő hűtést)

***

## Következő lépések

A feldolgozás megkezdése után:

1. **Figyelje a folyamatot** - Lásd [A feldolgozás figyelése](monitoring-the-processing.md)
2. **Várja meg a befejezést** – A feldolgozás automatikusan fut
3. **Ellenőrizze az eredményeket** – Lásd: [A feldolgozás befejezése](finishing-the-processing.md)

A feldolgozás alatt teendőkről szóló információkat lásd: [A feldolgozás figyelemmel kísérése](monitoring-the-processing.md).
