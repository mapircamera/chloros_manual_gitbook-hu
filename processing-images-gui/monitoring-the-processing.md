# A feldolgozás nyomon követése

A feldolgozás megkezdése után az Chloros számos lehetőséget kínál a folyamat nyomon követésére, a problémák felismerésére és az adatkészlet állapotának megértésére. Ez az oldal bemutatja, hogyan lehet nyomon követni a feldolgozást, és hogyan kell értelmezni az Chloros által nyújtott információkat.

## A haladási sáv áttekintése

A felső fejlécben található haladási sáv valós időben mutatja a feldolgozás állapotát és a befejezés százalékos arányát.

### Haladási sáv szabad módban

Chloros+ licenc nélküli felhasználók számára:

**2 lépcsős haladási kijelzés:**

1.**Célpont felismerés** – Kalibrációs célpontok keresése a képeken
2. **Feldolgozás** – Korrekciók alkalmazása és exportálás**A haladási sáv a következőket mutatja:**

* Átfogó befejezési százalék (0–100%)
* Az aktuális szakasz neve
* Egyszerű vízszintes sávos megjelenítés

### Chloros+ haladási sáv

Chloros+ licenccel rendelkező felhasználók számára:

**4-fokozatú haladási kijelző:**

1.**Észlelés** – Kalibrációs célpontok keresése
2. **Elemzés** – Képek vizsgálata és a feldolgozási folyamat előkészítése
3. **Kalibrálás** – Vignett- és fényvisszaverődési korrekciók alkalmazása
4. **Exportálás** – A feldolgozott fájlok mentése**Interaktív funkciók:*** **Vigye az egérmutatót** a haladási sávra a kibővített 4-fokozatú panel megjelenítéséhez
* **Kattintson** a haladási sávra a kibővített panel rögzítéséhez/kipineléséhez
* **Kattintson újra** a rögzítés feloldásához és az egér elhagyásakor történő automatikus elrejtéshez
* Minden szakasz az egyéni haladást mutatja (0-100%)

***

## Az egyes feldolgozási szakaszok megértése

{% hint style="info" %}
**Pipeline architektúra**: Ez a 4 GUI szakasz a [4 szálas feldolgozási pipeline-nak](../processing-architecture/processing-pipeline.md) felel meg. GPU-gyorsítással rendelkező rendszereken a 3. szál (Kalibrálás) kihasználja a [Dinamikus számítási adaptáció](../processing-architecture/dynamic-compute-adaptation.md) előnyeit, amely optimalizálja a feldolgozást az Ön konkrét hardveréhez.
{% endhint %}

### 1. szakasz: Észlelés (célpont-észlelés)

**Mi történik:**

* Az Chloros beolvassa a Célpont jelölőnégyzetet bejelölt képeket
* A számítógépes látás algoritmusai azonosítják a 4 kalibrációs panelt
* Minden panelből kivonják a fényvisszaverődési értékeket
* A megfelelő kalibrációs ütemezés érdekében rögzítik a célpontok időbélyegeit

**Időtartam:**

* Megjelölt célpontokkal: 10–60 másodperc
* Megjelöletlen célpontokkal: 5–30+ perc (az összes képet átvizsgálja)

**Haladásjelző:**

* Felismerés: 0% → 100%
* Átvizsgált képek száma
* Megtalált célpontok száma

**Mire kell figyelni:**

* Ha a célpontok megfelelően vannak megjelölve, a folyamatnak gyorsan le kell zárulnia
* Ha túl sokáig tart, előfordulhat, hogy a célpontok nincsenek megjelölve
* Ellenőrizze a hibakeresési naplóban a „Célpont megtalálva” üzeneteket

### 2. szakasz: Elemzés

**Mi történik:**

* A kép EXIF-metadatainak olvasása (időbélyegek, expozíciós beállítások)
* A kalibrációs stratégia meghatározása a célpontok időbélyegei alapján
* A képfeldolgozási sor rendezése
* A párhuzamos feldolgozást végző munkások előkészítése (csak Chloros+)

**Időtartam:** 5–30 másodperc**Haladásjelző:**

* Elemzés: 0% → 100%
* Gyors szakasz, általában gyorsan befejeződik

**Mire kell figyelni:**

* A folyamatnak szünetek nélkül, egyenletesen kell haladnia
* A hiányzó metaadatokra vonatkozó figyelmeztetések megjelennek a Debug Log-ban

### 3. szakasz: Kalibrálás

**Mi történik:*** **Debayering**: A RAW Bayer-minta 3 csatornára történő konvertálása
* **Vignette-korrekció**: A lencse szélén jelentkező sötétedés eltávolítása
* **Reflektancia-kalibrálás**: Normalizálás a célértékekhez
* **Indexszámítás**: Multispektrális indexek kiszámítása
* Az egyes képek feldolgozása a teljes folyamat során

**Időtartam:** A teljes feldolgozási idő nagy része (60–80%)**Haladásjelző:**

* Kalibrálás: 0% → 100%
* Aktuálisan feldolgozott kép
* Befejezett képek / Összes kép

**Feldolgozási viselkedés:*** **Szabad mód**: Egyszerre egy képet dolgoz fel egymás után
* **Chloros+ mód**: Legfeljebb 16 képet dolgoz fel egyszerre
* **GPU-gyorsítás**: Jelentősen felgyorsítja ezt a szakaszt**Mire kell figyelni:**

* A képek számának folyamatos növekedése
* Ellenőrizze a Debug Logot az egyes képek befejezéséről szóló üzenetekért
* Figyelmeztetések a képminőséggel vagy kalibrálási problémákkal kapcsolatban

### 4. szakasz: Exportálás

**Mi történik:**

* A kalibrált képek írása a lemezre a kiválasztott formátumban
* Multispektrális indexképek exportálása LUT színekkel
* Kameramodell-almappák létrehozása
* Az eredeti fájlnevek megőrzése a megfelelő kiterjesztésekkel

**Időtartam:** a teljes feldolgozási idő 10–20%-a**Haladásjelző:**

* Exportálás: 0% → 100%
* Fájlok írása
* Export formátum és célhely

**Mire kell figyelni:**

* Lemezterület-figyelmeztetések
* Fájlírási hibák
* Az összes konfigurált kimenet befejezése

***

## Debug Log fül

A Debug Log részletes információkat nyújt a feldolgozás előrehaladásáról és az esetleges problémákról.

### A Debug Log eléréséhez

1. Kattintson a **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> ikonra a bal oldali sávban
2. Megnyílik a napló panel, amelyen a feldolgozás valós idejű üzenetei jelennek meg
3. Az oldal automatikusan görgetődik, hogy a legfrissebb üzeneteket mutassa

### A naplóüzenetek értelmezése

#### Információs üzenetek (fehér/szürke)

Normál feldolgozási frissítések:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Figyelmeztető üzenetek (sárga)

Nem kritikus problémák, amelyek nem állítják le a feldolgozást:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Tennivaló:** A feldolgozás után ellenőrizze a figyelmeztetéseket, de ne szakítsa meg a folyamatot

#### Hibaüzenetek (Red)

Kritikus problémák, amelyek a feldolgozás meghiúsulásához vezethetnek:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Tennivaló:** Állítsa le a feldolgozást, javítsa ki a hibát, indítsa újra

### Gyakori naplóüzenetek

| Üzenet                          | Jelentés                                | Szükséges intézkedés                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| „Célpont észlelve a \[fájlnév] fájlban” | Kalibrációs célpont sikeresen megtalálva  | Nincs – normális                                         |
| „X/Y kép feldolgozása”        | Aktuális előrehaladás frissítése                | Nincs – normális                                         |
| „Nincs célpont”               | Nincs kalibrációs célpont észlelve        | Jelölje meg a célképeket, vagy tiltsa le a reflexiós kalibrációt |
| „Nincs elegendő lemezterület”        | Nincs elegendő tárhely a kimenethez          | Szabadítson fel lemezterületet                                    |
| „A sérült fájlt kihagyom”        | A képfájl sérült                  | Másolja át újra a fájlt az SD-kártyáról                             |
| „PPK-adatok alkalmazva”               | A .daq fájlból származó GPS-korrekciók alkalmazva | Nincs – normális                                         |

### Naplóadatok másolása

A napló másolása hibaelhárítás vagy támogatás céljából:

1. Nyissa meg a Debug Log panelt
2. Kattintson a **&quot;Copy Log&quot;** gombra (vagy kattintson a jobb gombbal → Select All)
3. Illessze be egy szövegfájlba vagy e-mailbe
4. Szükség esetén küldje el az MAPIR támogatásnak

***

## Rendszererőforrás-figyelés

### CPU-használat

**Szabad mód:**

* 1 CPU-mag ~100%-os kihasználtsággal
* A többi mag tétlen vagy rendelkezésre áll
* A rendszer továbbra is reagál

**Chloros+ párhuzamos mód:**

* Több mag 80–100%-os kihasználtsággal (akár 16 mag)
* Magas általános CPU-kihasználtság
* A rendszer reagálóképessége romolhat

**Figyelés:**

* Windows Feladatkezelő (Ctrl+Shift+Esc)
* Teljesítmény fül → CPU szakasz
* Keresse meg az „Chloros” vagy „chloros-backend” folyamatokat

### Memória (RAM) használat

**Jellemző felhasználás:**

* Kis projektek (&lt; 100 kép): 2–4 GB
* Közepes projektek (100–500 kép): 4–8 GB
* Nagy projektek (500+ kép): 8–16 GB
* Az Chloros+ párhuzamos mód több RAM-ot használ

**Ha kevés a memória:**

* Feldolgozzon kisebb adagokat
* Zárjon be más alkalmazásokat
* Frissítse a RAM-ot, ha rendszeresen nagy adathalmazokat dolgoz fel

### GPU-használat (Chloros+ CUDA-val)

Ha a GPU-gyorsítás engedélyezve van:

* Az NVIDIA GPU magas kihasználtságot mutat (60–90%)
* A VRAM-használat növekszik (4 GB+ VRAM szükséges)
* A kalibrálási szakasz jelentősen gyorsabb

**Figyelemmel kísérni:**

* NVIDIA tálcaikon
* Feladatkezelő → Teljesítmény → GPU
* GPU-Z vagy hasonló felügyeleti eszköz

### Lemez I/O

**Mire számíthat:**

* Magas lemezolvasási terhelés az elemzési szakaszban
* Magas lemezírási terhelés az exportálási szakaszban
* Az SSD jelentősen gyorsabb, mint a HDD

**Teljesítménytipp:**

* Ha lehetséges, használjon SSD-t a projektmappához
* Kerülje a hálózati meghajtókat nagy adathalmazok esetén
* Győződjön meg arról, hogy a lemez nem közelíti meg a kapacitáshatárt (ez befolyásolja az írási sebességet)

***

## Problémák észlelése a feldolgozás során

### Figyelmeztető jelek

**A folyamat leáll (5 percnél hosszabb ideig nincs változás):**

* Ellenőrizze a hibajelentéseket a Debug Logban
* Ellenőrizze a rendelkezésre álló lemezterületet
* Ellenőrizze a Feladatkezelőben, hogy az Chloros fut-e

**Gyakran jelennek meg hibaüzenetek:**

* Állítsa le a feldolgozást, és vizsgálja meg a hibákat
* Gyakori okok: lemezterület, sérült fájlok, memóriaproblémák
* Lásd az alábbi Hibaelhárítás részt

**A rendszer nem reagál:**

* Az Chloros+ párhuzamos mód túl sok erőforrást használ
* Fontolja meg az egyidejű feladatok számának csökkentését vagy a hardver frissítését
* A szabad mód kevésbé erőforrás-igényes

### Mikor kell leállítani a feldolgozást

Állítsa le a feldolgozást, ha a következőket látja:

* ❌ „Lemez megtelt” vagy „Fájl írása nem lehetséges” hibák
* ❌ Ismétlődő képfájl-sérülési hibák
* ❌ A rendszer teljesen lefagyott (nem reagál)
* ❌ Rájött, hogy helytelen beállításokat konfigurált
* ❌ Helytelen képeket importált

**Hogyan állítsa le:**

1. Kattintson a**Stop/Cancel gombra** (a Start gomb helyett)
2. A feldolgozás leáll, az elért eredmények elvesznek
3. Javítsa ki a problémákat, és indítsa újra az elejétől

***

## Hibaelhárítás a feldolgozás során

### A feldolgozás nagyon lassú

**Lehetséges okok:**

* Megjelölés nélküli célképek (az összes kép beolvasása)
* HDD tároló SSD helyett
* Elégtelen rendszererőforrások
* Sok index beállítva
* Hálózati meghajtóhoz való hozzáférés

**Megoldások:**

1. Ha éppen most indult el és az Észlelés szakaszban van: Mégse, jelölje meg a célokat, indítsa újra
2. A jövőben: Használjon SSD-t, csökkentse az indexek számát, frissítse a hardvert
3. Fontolja meg az CLI használatát nagy adathalmazok kötegelt feldolgozásához

### „Lemezterület” figyelmeztetések

**Megoldások:**

1. Azonnal szabadítson fel lemezterületet
2. Helyezze át a projektet egy nagyobb lemezterülettel rendelkező meghajtóra
3. Csökkentse az exportálandó indexek számát
4. Használjon JPG formátumot az TIFF helyett (kisebb fájlok)

### Gyakori „Sérült fájl” üzenetek

**Megoldások:**

1. Másolja át újra a képeket az SD-kártyáról az integritás biztosítása érdekében
2. Ellenőrizze az SD-kártyát hibák szempontjából
3. Távolítsa el a sérült fájlokat a projektből
4. Folytassa a fennmaradó képek feldolgozását

### A rendszer túlmelegedése / teljesítménycsökkentés

**Megoldások:**

1. Gondoskodjon megfelelő szellőzésről
2. Tisztítsa meg a számítógép szellőzőnyílásait a portól
3. Csökkentse a feldolgozási terhelést (használja a Free módot az Chloros+ helyett)
4. A feldolgozást végezze a nap hűvösebb óráiban

***

## Értesítés a feldolgozás befejezéséről

A feldolgozás befejezésekor:

* A haladási sáv eléri a 100%-ot
* **„Feldolgozás befejezve”** üzenet jelenik meg a hibakeresési naplóban
* A Start gomb újra használhatóvá válik
* Az összes kimeneti fájl a kamera modelljének almappájában található

***

## Következő lépések

A feldolgozás befejezése után:

1. **Ellenőrizze az eredményeket** – Lásd [A feldolgozás befejezése](finishing-the-processing.md)
2. **Ellenőrizze a kimeneti mappát** – Ellenőrizze, hogy az összes fájl helyesen lett-e exportálva
3. **Ellenőrizze a hibakeresési naplót** – Ellenőrizze, hogy vannak-e figyelmeztetések vagy hibák
4. **Nézze meg a feldolgozott képeket** – Használja a Képmegjelenítőt vagy külső szoftvert

A feldolgozott eredmények áttekintésével és használatával kapcsolatos információkért lásd: [A feldolgozás befejezése](finishing-the-processing.md).
