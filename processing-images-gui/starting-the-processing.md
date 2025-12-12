# A feldolgozás megkezdése

Miután importálta a képeket, megjelölte a kalibrációs célpontokat és konfigurálta a projekt beállításait, készen áll a feldolgozás megkezdésére. Ez az oldal végigvezeti Önt az Chloros feldolgozási folyamat elindításán.

## Előfeldolgozási ellenőrzőlista

Mielőtt rákattintana a Start gombra, ellenőrizze, hogy minden készen áll-e:

* [ ] **Fájlok importálva** - Minden kép megjelenik a Fájlböngészőben
* [ ] **Célképek megjelölve** - A Cél oszlop bejelölve a kalibrációs képekhez
* [ ] **A kameramodellek felismerve** - A Kameramodell oszlopban a megfelelő kamerák jelennek meg
* [ ] **Beállítások konfigurálva** - A projektbeállítások áttekintve és módosítva
* [ ] **Indexek kiválasztva** - A kívánt multispektrális indexek hozzáadva (ha szükséges)
* [ ] **Exportformátum kiválasztva** - A munkafolyamatához megfelelő kimeneti formátum

{% hint style=&quot;info&quot; %}
**Tipp**: Kattintson néhány képre a Fájlböngészőben, hogy ellenőrizze, hogy azok megfelelően töltődtek-e be a feldolgozás előtt.
{% endhint %}

***

## A feldolgozás elindítása

### Keresse meg a Start gombot

A Start/Lejátszás gomb az Chloros felső fejlécében található:

* Hely: az ablak felső közepén
* Ikon: **Lejátszás/Start gomb** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">
* Állapot: A gomb aktív (világos), ha készen áll a feldolgozásra

### Kattintson a Start gombra

1. Kattintson a **Lejátszás/Start gombra** a felső fejlécben
2. A feldolgozás azonnal megkezdődik
3. A gomb a feldolgozás alatt inaktívvá válik (szürkén jelenik meg)
4. A haladási sáv frissül, és megmutatja a feldolgozás állapotát

{% hint style=&quot;success&quot; %}
**Feldolgozás elindítva**: A gombra kattintás után az Chloros automatikusan elvégzi az összes feldolgozási lépést – célfelismerés, debayering, kalibrálás, indexszámítás és exportálás.
{% endhint %}

***

## A feldolgozási módok megértése

Az Chloros a licencétől függően két különböző feldolgozási módban működik:

### Ingyenes mód (szekvenciális feldolgozás)

**Minden felhasználó számára elérhető**

**Hogyan működik:**

* A képeket egyenként, egymás után dolgozza fel.
* Egy szálas működés.
* Alacsonyabb memóriahasználat.

**A haladási sáv 2 szakaszt mutat:**

1. **Célfelismerés** – Kalibrációs célok keresése.
2. **Feldolgozás** – Kalibráció alkalmazása és képek exportálása.

**Feldolgozási idő:**

* Sokkal lassabb, mint az Chloros+ párhuzamos mód
* Kis és közepes méretű adathalmazokhoz alkalmas (&lt; 200 kép)

### Chloros+ mód (párhuzamos feldolgozás)

**Chloros+ licenc szükséges**

**Hogyan működik:**

* Több kép egyidejű feldolgozása
* Többszálas működés (legfeljebb 16 párhuzamos munkás)
* Több CPU mag felhasználása
* Opcionális GPU (CUDA) gyorsítás NVIDIA grafikus kártyákkal

**A haladási sáv 4 szakaszt mutat:**

1. **Észlelés** – Kalibrációs célok keresése
2. **Elemzés** – A kép metaadatainak vizsgálata és a folyamat előkészítése
3. **Kalibrálás** – Korrekciók és kalibrálások alkalmazása
4. **Exportálás** – A feldolgozott képek és indexek mentése

**A haladási sáv interakciója:**

* **Vigye az egérmutatót** a sáv fölé, hogy megtekintse a részletes, 4 szakaszból álló legördülő panelt
* **Kattintson** a haladási sávra, hogy a legördülő panelt a helyén rögzítse
* **Kattintson újra** a panel feloldásához és elrejtéséhez

**Feldolgozási idő:**

* Jelentősen gyorsabb, mint az ingyenes mód
* A CPU magszámával arányosan skálázódik
* A GPU-gyorsítás tovább javítja a sebességet

{% hint style=&quot;info&quot; %}
**Chloros+ sebesség**: A párhuzamos feldolgozás nagy adathalmazok esetén 5-10-szer gyorsabb lehet, mint a szekvenciális mód. Egy 500 képet tartalmazó projekt, amely ingyenes módban 2 órát vesz igénybe, Chloros+ használatával 15-20 perc alatt elkészülhet.
{% endhint %}

***

## Mi történik a feldolgozás során?

### 1. szakasz: Célfelismerés

**Mit csinál az Chloros:**

* Beolvassa a megjelölt célképeket (vagy az összes képet, ha nincs megjelölve)
* Azonosítja a 4 kalibrációs panelt minden célpontban
* Kivonja a visszaverődési értékeket a célpont panelekből
* Rögzíti a célpont időbélyegzőit a kalibráció ütemezéséhez

**Időtartam:** 1-30 másodperc (jelölt célpontok esetén), 5-30+ perc (jelöletlen célpontok esetén)

### 2. szakasz: Debayering (RAW konverzió)

**Az Chloros funkciói:**

* RAW Bayer-minta adatokat konvertál teljes RGB képekké
* Kiváló minőségű demosaicing algoritmust alkalmaz
* Megőrzi a maximális képminőséget és részletességet

**Időtartam:** A képek számától és a CPU sebességétől függően változik

### 3. szakasz: Kalibrálás

**Az Chloros funkciói:**

* **Vignette-korrekció**: Eltávolítja a lencse sötétedését a széleken
* **Reflektancia-kalibrálás**: Normalizálja a célreflektancia-értékek felhasználásával
* Korrekciókat alkalmaz minden sávon/csatornán
* Az időbélyeg alapján minden képhez a megfelelő kalibrációs célt használja

**Időtartam:** A feldolgozási idő nagy része

### 4. szakasz: Index számítás

**Az Chloros funkciója:**

* Kiszámítja a konfigurált multispektrális indexeket (NDVI, NDRE stb.)
* Sávmatematikát alkalmaz a kalibrált képekre
* Indexképeket generál minden kiválasztott indexhez

**Időtartam:** Néhány másodperc képenként

### 5. szakasz: Exportálás

**Az Chloros feladata:**

* A kalibrált képeket a kiválasztott formátumban menti
* Az indexképeket a konfigurált LUT színekkel exportálja
* A fájlokat a kamera modell alkönyvtáraiba írja
* Megőrzi az eredeti fájlneveket a kiterjesztésekkel

**Időtartam:** Az export formátumtól és a fájl méretétől függően változik

***

## Feldolgozási viselkedés

### Automatikus feldolgozási folyamat

A folyamat elindítása után az egész folyamat automatikusan lefut:

* Nincs szükség felhasználói beavatkozásra
* Az összes konfigurált lépés egymás után fut le
* A folyamat előrehaladása valós időben látható

### Számítógép használata a feldolgozás során

**Szabad mód:**

* Viszonylag alacsony CPU-használat (egyszálas)
* A számítógép továbbra is reagál más feladatokra
* Biztonságos minimalizálni az Chloros programot és más alkalmazásokban dolgozni

**Chloros+ párhuzamos mód:**

* Magas CPU-használat (többszálas, akár 16 mag)
* GPU-gyorsítással: magas GPU-használat
* A számítógép a feldolgozás során kevésbé reagálhat
* Kerülje más CPU-igényes feladatok elindítását

{% hint style=&quot;warning&quot; %}
**Teljesítménytipp**: A legjobb Chloros+ teljesítmény érdekében zárjon be más alkalmazásokat, és hagyja, hogy az Chloros a teljes rendszer erőforrásait felhasználja.
{% endhint %}

### A feldolgozás nem állítható le

**Fontos korlátozások:**

* A feldolgozás elindítása után nem állítható le.
* A feldolgozás törölhető, de a haladás elveszik.
* A részleges eredmények nem kerülnek mentésre.
* Törlés esetén az elejétől kell újrakezdeni.

**Tervezési tipp:** Nagyon nagy projektek esetén fontolja meg a feldolgozás kötegelt feldolgozását vagy az CLI használatát a jobb ellenőrzés érdekében.

***

## A feldolgozás figyelemmel kísérése

A feldolgozás futása közben a következőket teheti:

* **A haladási sáv megtekintése** – Az általános befejezési százalék megtekintése
* **Az aktuális szakasz megtekintése** – Észlelés, elemzés, kalibrálás vagy exportálás
* **A napló fül megtekintése** – A feldolgozással kapcsolatos részletes üzenetek és figyelmeztetések megtekintése
* **A befejezett képek előnézete** – Néhány exportált fájl a feldolgozás során megjelenhet

A figyelemmel kísérésről részletes információkat a [A feldolgozás figyelemmel kísérése](monitoring-the-processing.md) című részben talál.

***

## A feldolgozás megszakítása

Ha meg kell szakítania a feldolgozást:

### Megszakítás módja

1. Keresse meg a **Stop/Cancel gombot** (a feldolgozás során ez helyettesíti a Start gombot)
2. Kattintson a Stop gombra
3. A feldolgozás azonnal leáll
4. A részleges eredmények elvetésre kerülnek

### Mikor kell megszakítani

**Érvényes okok a megszakításra:**

* Rájött, hogy helytelen beállításokat használt
* Elfelejtette megjelölni a célképeket
* Rossz képeket importált
* A rendszer túl lassú vagy nem reagál

**A megszakítás után:**

* Ellenőrizze és javítsa ki az esetleges problémákat
* Szükség szerint módosítsa a beállításokat
* Indítsa újra a feldolgozást az elejétől
* A legtisztább élmény érdekében teljesen zárja be az Chloros programot, és indítsa újra.

{% hint style=&quot;warning&quot; %}
**Nincs részleges eredmény**: A lemondás az összes előrehaladást elveti. Az Chloros nem menti a részlegesen feldolgozott képeket.
{% endhint %}

***

## Becsült feldolgozási idő

A tényleges feldolgozási idő nagyban függ a következőktől:

* Képek száma
* Képfelbontás
* RAW vagy JPG bemeneti formátum
* Feldolgozási mód (Free vagy Chloros+)
* CPU sebesség és magok száma
* GPU rendelkezésre állása (csak Chloros+)
* Számítandó indexek száma
* Export formátum komplexitása

### Hozzávetőleges becslések (Chloros+, 12 MP-es képek, modern CPU)

| Képek száma | Ingyenes mód | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 kép   | 15-20 perc | 5-8 perc        | 3-5 perc        |
| 100 kép  | 30-40 perc | 10-15 perc      | 5-8 perc        |
| 200 kép  | 1-1,5 óra | 20-30 perc      | 10-15 perc      |
| 500 kép  | 2-3 óra   | 45-60 perc      | 20-30 perc      |
| 1000 kép | 4-6 óra   | 1,5-2 óra      | 40-60 perc      |

{% hint style=&quot;info&quot; %}
**Első futtatás**: Az első feldolgozás hosszabb ideig tarthat, mivel az Chloros cache-eket és profilokat hoz létre. A hasonló adatkészletek későbbi feldolgozása gyorsabb lesz.
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

1. Várja meg, amíg a háttérprogram teljesen inicializálódik (ellenőrizze a főmenü ikont)
2. Ellenőrizze, hogy a képek importálva vannak-e a Fájlböngészőben
3. Indítsa újra az Chloros programot, ha a gomb továbbra is letiltva marad
4. Ellenőrizze a hibajelentéseket a hibakeresési naplóban

### A feldolgozás elindul, majd azonnal megszakad

**Lehetséges okok:**

* Nincs érvényes kép a projektben
* Megsérült képfájlok
* Nincs elegendő lemezterület
* Nincs elegendő memória (RAM)

**Megoldások:**

1. Ellenőrizze a hibakeresési naplót <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> a hibaüzenetekért
2. Ellenőrizze a rendelkezésre álló lemezterületet
3. Próbálja meg feldolgozni a képek kisebb részét
4. Ellenőrizze, hogy a képek nem sérültek-e

### „Nincs célpont észlelve” figyelmeztetés

**Lehetséges okok:**

* Elfelejtette megjelölni a célképeket
* A célképek nem tartalmaznak látható célpontokat
* A célpont-észlelési beállítások túl szigorúak

**Megoldások:**

1. Olvassa el a [Célképek kiválasztása](choosing-target-images.md) című részt.
2. Jelölje meg a megfelelő képeket a Cél oszlopban.
3. Ellenőrizze, hogy a célok láthatók-e a megjelölt képeken.
4. Szükség esetén módosítsa a célfelismerési beállításokat.

***

## Tippek a sikeres feldolgozáshoz

### Mielőtt elkezdené

1. **Először kis részhalmazzal teszteljen** – Feldolgozzon 10-20 képet a beállítások ellenőrzéséhez.
2. **Ellenőrizze a rendelkezésre álló lemezterületet** – Gondoskodjon arról, hogy a adatállomány méretének 2-3-szorosának megfelelő szabad hely álljon rendelkezésre.
3. **Zárjon be felesleges alkalmazásokat** – Szabadítson fel rendszererőforrásokat.
4. **Ellenőrizze a célképeket** – Nézze meg a megjelölt célok előnézetét a minőség ellenőrzése érdekében.
5. **Mentse el a projektet** – A projekt automatikusan mentésre kerül, de célszerű manuálisan is elmenteni.

### Feldolgozás közben

1. **Kerülje a rendszer alvó állapotát** – Kapcsolja ki az energiatakarékos módokat.
2. **Tartsa az Chloros programot az előtérben** – Vagy legalábbis látható helyen a tálcán.
3. **Időnként ellenőrizze a folyamatot** – Ellenőrizze, hogy nincs-e figyelmeztetés vagy hiba.
4. **Ne töltsön be más nagy terhelésű alkalmazásokat** – Különösen Chloros+ párhuzamos módban

### Chloros+ GPU-gyorsítás

NVIDIA GPU-gyorsítás használata esetén:

1. Frissítse az NVIDIA illesztőprogramokat a legújabb verzióra
2. Győződjön meg arról, hogy a GPU rendelkezik 4 GB+ VRAM-mal
3. Zárja be a GPU-igényes alkalmazásokat (játékok, videószerkesztés)
4. Figyelje a GPU hőmérsékletét (biztosítsa a megfelelő hűtést)

***

## Következő lépések

A feldolgozás megkezdése után:

1. **Figyelje a folyamatot** - Lásd [A feldolgozás figyelése](monitoring-the-processing.md)
2. **Várja meg a befejezést** – A feldolgozás automatikusan fut.
3. **Ellenőrizze az eredményeket** – Lásd: [A feldolgozás befejezése](finishing-the-processing.md)

A feldolgozás során teendőkről további információkat a [A feldolgozás figyelése](monitoring-the-processing.md) című részben talál.
