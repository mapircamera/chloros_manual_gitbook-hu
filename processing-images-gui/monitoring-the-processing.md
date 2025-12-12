# A feldolgozás figyelemmel kísérése

A feldolgozás megkezdése után az Chloros többféle lehetőséget kínál a folyamat figyelemmel kísérésére, a problémák ellenőrzésére és az adatkészletben zajló események megértésére. Ez az oldal elmagyarázza, hogyan lehet nyomon követni a feldolgozást és értelmezni az Chloros által nyújtott információkat.

## A haladási sáv áttekintése

A fejlécben található haladási sáv valós időben mutatja a feldolgozás állapotát és a befejezés százalékos arányát.

### Ingyenes mód haladási sávja

Chloros+ licenccel nem rendelkező felhasználók számára:

**Kétlépcsős haladási kijelzés:**

1. **Célfelismerés** – Kalibrációs célok keresése a képeken
2. **Feldolgozás** – Javítások alkalmazása és exportálás

**A haladási sáv a következőket jeleníti meg:**

* Átfogó befejezési százalék (0-100%)
* Aktuális szakasz neve
* Egyszerű vízszintes sávos megjelenítés

### Chloros+ haladási sáv

Chloros+ licenccel rendelkező felhasználók számára:

**4 szakaszos haladási kijelzés:**

1. **Észlelés** – Kalibrációs célpontok keresése
2. **Elemzés** – Képek vizsgálata és folyamat előkészítése
3. **Kalibrálás** – Vignettázás és visszaverődés korrekciók alkalmazása
4. **Exportálás** – Feldolgozott fájlok mentése

**Interaktív funkciók:**

* **Vigye az egérmutatót** a haladásjelző sávra a kibővített 4 szakaszos panel megjelenítéséhez
* **Kattintson** a haladási sávra a kibővített panel befagyasztásához/rögzítéséhez
* **Kattintson újra** a befagyasztás feloldásához és az egér elmozdításakor az automatikus elrejtéshez
* Minden szakasz az egyéni haladást mutatja (0-100%)

***

## Az egyes feldolgozási szakaszok megértése

### 1. szakasz: Észlelés (célpontok észlelése)

**Mi történik:**

* Az Chloros beolvassa a Cél jelölőnégyzettel megjelölt képeket
* A számítógépes látás algoritmusok azonosítják a 4 kalibrációs panelt
* Az egyes panelekből kivont visszaverődési értékek
* A megfelelő kalibrációs ütemezéshez rögzített cél időbélyegek

**Időtartam:**

* Megjelölt célokkal: 10-60 másodperc
* Megjelölt célok nélkül: 5-30+ perc (az összes kép beolvasása)

**Haladásjelző:**

* Észlelés: 0% → 100%
* Beolvasott képek száma
* Megtalált célok száma

**Mire kell figyelni:**

* Ha a célok megfelelően vannak megjelölve, a folyamat gyorsan befejeződik.
* Ha túl sokáig tart, előfordulhat, hogy a célok nincsenek megjelölve.
* Ellenőrizze a hibakeresési naplóban a „Cél megtalálva” üzeneteket.

### 2. szakasz: Elemzés

**Mi történik:**

* A kép EXIF metaadatainak olvasása (időbélyegek, expozíciós beállítások)
* A cél időbélyegei alapján a kalibrációs stratégia meghatározása
* A képfeldolgozási sor rendezése
* Párhuzamos feldolgozási munkások előkészítése (csak Chloros+)

**Időtartam:** 5-30 másodperc

**Haladásjelző:**

* Elemzés: 0% → 100%
* Gyors szakasz, általában gyorsan befejeződik

**Mire kell figyelni:**

* A folyamatnak szünet nélkül, egyenletesen kell haladnia
* A hiányzó metaadatokra vonatkozó figyelmeztetések a hibakeresési naplóban jelennek meg

### 3. szakasz: Kalibrálás

**Mi történik:**

* **Debayering**: RAW Bayer-minta konvertálása 3 csatornára
* **Vignette-korrekció**: A lencse szélének elsötétülésének eltávolítása
* **Reflektancia-kalibrálás**: Normalizálás a célértékekkel
* **Index számítás**: Multispektrális indexek kiszámítása
* Minden kép feldolgozása a teljes folyamaton keresztül

**Időtartam:** A teljes feldolgozási idő nagy része (60-80%)

**Haladásjelző:**

* Kalibrálás: 0% → 100%
* Jelenleg feldolgozott kép
* Befejezett képek / Összes kép

**Feldolgozási viselkedés:**

* **Szabad mód**: Egyszerre egy képet dolgoz fel egymás után
* **Chloros+ mód**: Egyszerre legfeljebb 16 képet dolgoz fel
* **GPU-gyorsítás**: jelentősen felgyorsítja ezt a szakaszt

**Mire kell figyelni:**

* A képek számának folyamatos növekedése
* A képek befejezéséről szóló üzenetek ellenőrzése a hibakeresési naplóban
* Figyelmeztetések a képminőségről vagy a kalibrálási problémákról

### 4. szakasz: Exportálás

**Mi történik:**

* A kalibrált képek írása a kiválasztott formátumban a lemezre
* Multispektrális indexképek exportálása LUT színekkel
* Kamera modell alkönyvtárak létrehozása
* Az eredeti fájlnevek megőrzése a megfelelő kiterjesztésekkel

**Időtartam:** A teljes feldolgozási idő 10-20%-a

**Haladásjelző:**

* Exportálás: 0% → 100%
* Fájlok írása
* Export formátum és célhely

**Mire kell figyelni:**

* Lemezterület figyelmeztetések
* Fájl írási hibák
* Az összes konfigurált kimenet befejezése

***

## Debug Log fül

A Debug Log részletes információkat nyújt a feldolgozás előrehaladásáról és az esetleges problémákról.

### A Debug Log elérés

1. Kattintson a **Debug Log** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> ikonra a bal oldali sávban.
2. Megnyílik a napló panel, amely valós idejű feldolgozási üzeneteket jelenít meg.
3. Automatikusan görget, hogy a legújabb üzeneteket mutassa.

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

**Művelet:** A feldolgozás után ellenőrizze a figyelmeztetéseket, de ne szakítsa meg a feldolgozást.

#### Hibaüzenetek (Red)

Kritikus problémák, amelyek a feldolgozás meghibásodását okozhatják:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Teendő:** Állítsa le a feldolgozást, oldja meg a hibát, indítsa újra.

### Általános naplóüzenetek

| Üzenet                          | Jelentés                                | Szükséges teendő                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| „Célpont észlelve a \[filename] fájlban” | Kalibrációs célpont sikeresen megtalálva  | Nincs – normális                                         |
| „Y kép X feldolgozása”        | Aktuális előrehaladás frissítése                | Nincs – normális                                         |
| „Nincs célpont”               | Nincs kalibrációs célpont észlelve        | Jelölje meg a célképeket, vagy tiltsa le a visszaverődés kalibrálását |
| „Nincs elegendő lemezterület”        | Nincs elegendő tárhely a kimenethez          | Felszabadítson lemezterületet                                    |
| „Sérült fájl kihagyása”        | A képfájl sérült                  | Másolja át a fájlt az SD-kártyáról                             |
| „PPK-adatok alkalmazva”               | .daq fájlból származó GPS-korrekciók alkalmazva | Nincs - normális                                         |

### Naplóadatok másolása

A napló másolása hibaelhárítás vagy támogatás céljából:

1. Nyissa meg a Debug Log (Hibakeresési napló) panelt.
2. Kattintson a **„Copy Log”** (Napló másolása) gombra (vagy kattintson a jobb gombbal → Select All (Összes kijelölése)).
3. Illessze be egy szövegfájlba vagy e-mailbe.
4. Szükség esetén küldje el az MAPIR támogatásnak.

***

## Rendszererőforrás-figyelés

### CPU-használat

**Szabad mód:**

* 1 CPU-mag ~100%
* A többi mag üresjáratban vagy rendelkezésre áll
* A rendszer továbbra is reagál

**Chloros+ párhuzamos mód:**

* Több mag 80-100% (legfeljebb 16 mag)
* Magas általános CPU-kihasználtság
* A rendszer kevésbé reagálhat

**Megfigyelés:**

* Windows Feladatkezelő (Ctrl+Shift+Esc)
* Teljesítmény fül → CPU szakasz
* Keresse meg az „Chloros” vagy „chloros-backend” folyamatokat

### Memória (RAM) használat

**Tipikus használat:**

* Kis projektek (&lt; 100 kép): 2-4 GB
* Közepes projektek (100-500 kép): 4-8 GB
* Nagy projektek (500+ kép): 8-16 GB
* Az Chloros+ párhuzamos mód több RAM-ot használ

**Ha kevés a memória:**

* Kisebb adagokat dolgozzon fel
* Zárjon be más alkalmazásokat
* Ha rendszeresen nagy adathalmazokat dolgoz fel, bővítse a RAM-ot

### GPU-használat (Chloros+ CUDA-val)

Ha a GPU-gyorsítás engedélyezve van:

* Az NVIDIA GPU magas kihasználtságot mutat (60-90%)
* A VRAM használata nő (4 GB+ VRAM szükséges)
* A kalibrálási szakasz jelentősen gyorsabb

**Megfigyelés:**

* NVIDIA tálcaikon
* Feladatkezelő → Teljesítmény → GPU
* GPU-Z vagy hasonló megfigyelő eszköz

### Lemez I/O

**Mire számíthat:**

* Magas lemezolvasási sebesség az elemzési szakaszban
* Magas lemezírási sebesség az exportálási szakaszban
* Az SSD jelentősen gyorsabb, mint a HDD

**Teljesítménytipp:**

* Ha lehetséges, SSD-t használjon a projektmappához
* Kerülje a hálózati meghajtókat nagy adathalmazok esetén
* Győződjön meg arról, hogy a lemez nem közelíti meg a kapacitását (ez befolyásolja az írási sebességet)

***

## Problémák észlelése a feldolgozás során

### Figyelmeztető jelek

**A folyamat leáll (5 percnél hosszabb ideig nincs változás):**

* Ellenőrizze a hibakeresési naplót hibák után
* Ellenőrizze a rendelkezésre álló lemezterületet
* Ellenőrizze a Feladatkezelőt, hogy az Chloros fut-e

**Gyakran jelennek meg hibaüzenetek:**

* Állítsa le a feldolgozást, és ellenőrizze a hibákat
* Gyakori okok: lemezterület, sérült fájlok, memóriaproblémák
* Lásd az alábbi Hibaelhárítás részt

**A rendszer nem reagál:**

* Az Chloros+ párhuzamos mód túl sok erőforrást használ
* Fontolja meg az egyidejű feladatok számának csökkentését vagy a hardver frissítését
* A szabad mód kevésbé erőforrás-igényes

### Mikor kell leállítani a feldolgozást

Állítsa le a feldolgozást, ha a következőket látja:

* ❌ „Lemez megtelt” vagy „Nem lehet fájlt írni” hibák
* ❌ Ismétlődő képfájl-sérülési hibák
* ❌ A rendszer teljesen lefagyott (nem reagál)
* ❌ Rossz beállítások konfigurálása
* ❌ Rossz képek importálása

**A leállítás módja:**

1. Kattintson a **Stop/Cancel (Leállítás/Mégse) gombra** (a Start gomb helyett)
2. A feldolgozás leáll, az előrehaladás elveszik
3. Javítsa ki a problémákat, és indítsa újra az elejétől

***

## Hibaelhárítás a feldolgozás során

### A feldolgozás nagyon lassú

**Lehetséges okok:**

* Nem jelölt célképek (az összes kép beolvasása)
* HDD SSD tároló helyett
* Nem elegendő rendszererőforrás
* Sok index konfigurálva
* Hálózati meghajtóhoz való hozzáférés

**Megoldások:**

1. Ha csak most indult és az Észlelés szakaszban van: Mégse, jelölje meg a célokat, indítsa újra
2. A jövőben: Használjon SSD-t, csökkentse az indexek számát, frissítse a hardvert
3. Fontolja meg az CLI használatát nagy adathalmazok kötegelt feldolgozásához

### „Lemezterület” figyelmeztetések

**Megoldások:**

1. Azonnal szabadítson fel lemezterületet
2. Helyezze át a projektet egy nagyobb lemezterületű meghajtóra
3. Csökkentse az exportálandó indexek számát.
4. Használjon JPG formátumot TIFF helyett (kisebb fájlok).

### Gyakori „Sérült fájl” üzenetek

**Megoldások:**

1. Másolja át újra a képeket az SD-kártyáról az integritás biztosítása érdekében.
2. Ellenőrizze az SD-kártyát hibákra.
3. Távolítsa el a sérült fájlokat a projektből.
4. Folytassa a többi kép feldolgozását.

### A rendszer túlmelegedése / fojtása

**Megoldások:**

1. Gondoskodjon megfelelő szellőzésről.
2. Tisztítsa meg a számítógép szellőzőnyílásait a portól.
3. Csökkentse a feldolgozási terhelést (használja a Free módot az Chloros+ helyett).
4. A nap hűvösebb óráiban végezze a feldolgozást.

***

## A feldolgozás befejezéséről szóló értesítés

A feldolgozás befejezésekor:

* A haladási sáv eléri a 100%-ot.
* A **„Feldolgozás befejezve”** üzenet megjelenik a hibakeresési naplóban.
* A Start gomb újra aktiválódik.
* Az összes kimeneti fájl a kamera modell alkönyvtárában található.

***

## Következő lépések

A feldolgozás befejezése után:

1. **Ellenőrizze az eredményeket** – Lásd: [A feldolgozás befejezése](finishing-the-processing.md)
2. **Ellenőrizze a kimeneti mappát** – Ellenőrizze, hogy minden fájl helyesen lett-e exportálva
3. **Tekintse át a hibakeresési naplót** – Ellenőrizze, hogy nincs-e figyelmeztetés vagy hiba
4. **Tekintse meg a feldolgozott képeket** – Használja a Képnézegetőt vagy külső szoftvert

A feldolgozott eredmények áttekintésével és használatával kapcsolatos információkat lásd: [A feldolgozás befejezése](finishing-the-processing.md).
