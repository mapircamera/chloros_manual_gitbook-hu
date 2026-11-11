# A feldolgozás nyomon követése

A feldolgozás megkezdése után a Chloros

számos lehetőséget kínál a folyamat nyomon követésére, a problémák felismerésére és az adatkészlet állapotának megértésére. Ez az oldal bemutatja, hogyan lehet nyomon követni a feldolgozást, és hogyan kell értelmezni a Chloros

által nyújtott információkat.

## A haladási sáv áttekintése

A felső fejlécben található haladási sáv valós időben mutatja a feldolgozás állapotát és a befejezettség százalékát. A haladás élőben érkezik a háttérrendszerből Server-Sent Events (SSE) segítségével, így a sáv pontosan tükrözi, hogy a feldolgozási folyamat éppen mit végez.

### Haladási sáv ingyenes módban

Chloros

+ licenc nélküli felhasználók számára:

**Kétlépcsős haladási kijelzés:**

1.**Célpont-felismerés** – Kalibrációs célpontok keresése a képeken
2. **Feldolgozás** – Korrekciók alkalmazása és exportálás**A haladási sáv a következőket jeleníti meg:**

* Átfogó befejezettségi százalék (0–100%)
* Az aktuális szakasz neve
* Egyszerű vízszintes sávos megjelenítés

### Chloros

+ haladási sáv

Chloros

+ licenccel rendelkező felhasználók számára:

**4-fokozatú haladási kijelzés:**

1.**Észlelés** – Kalibrációs célpontok keresése
2. **Elemzés** – Képek vizsgálata és a feldolgozási folyamat előkészítése
3. **Kalibrálás** – Vignettázás- és reflexiókorrekciók alkalmazása
4. **Exportálás** – A feldolgozott fájlok mentése**Interaktív funkciók:*** **Vigye az egérmutatót** a haladási sáv fölé a kibővített 4-fázisú panel megjelenítéséhez
* **Kattintson** a haladási sávra a kibővített panel rögzítéséhez/kijelöléséhez
* **Kattintson újra** a rögzítés feloldásához, és az egér elhagyásakor a panel automatikusan elrejtődik
* Minden szakasz a saját előrehaladását mutatja (0–100%)

{% hint style="info" %}
**A CLI

-vel való összhang**: egy `chloros-cli process` futtatás során ugyanaz a négy szál jelenti az „Észlelés”, „Elemzés”, „Feldolgozás” és „Exportálás” állapotokat, míg az `chloros-cli export-status` egy másik terminálról élőben mutatja a 4. szál exportálásának előrehaladását. Lásd a [CLI

-referenciát](../reference/cli-reference.md).
{% endhint %}

***

## Az egyes feldolgozási szakaszok ismertetése

{% hint style="info" %}
**Pipeline-architektúra**: Ez a 4 GUI-szakasz a [4-szálas feldolgozási pipeline-nak](../processing-architecture/processing-pipeline.md) felel meg. GPU-gyorsítással rendelkező rendszereken a 3. szál (Kalibrálás) kihasználja a [dinamikus számítási adaptációt](../processing-architecture/dynamic-compute-adaptation.md), amely az Ön konkrét hardveréhez optimalizálja a feldolgozást.
{% endhint %}

### 1. szakasz: Észlelés (célpont-észlelés)

**Mi történik:**

* A Chloros

beolvassa azokat a képeket, amelyeket a „Cél” jelölőnégyzet bejelölésével jelölt meg (ha egyik sem jelölve, akkor az összes képet)
* A számítógépes látás algoritmusai azonosítják a kalibrációs paneleket
* Minden panelből kivonják a reflexiós értékeket
* A megfelelő kalibrációs ütemezés érdekében rögzítik a célok időbélyegeit

**Időtartam:**

* Megjelölt célpontok esetén: 10–60 másodperc
* Megjelöletlen célpontok esetén: 5–30+ perc (az összes kép átvizsgálása)

**Haladási jelző:**

* Felismerés: 0% → 100%
* Beolvasott képek száma (csak a ténylegesen beolvasott képeket számolja)
* Megtalált célpontok száma

**Mire kell figyelni:**

* Ha a célpontok megfelelően vannak megjelölve, a folyamatnak gyorsan be kell fejeződnie
* Ha túl sokáig tart, előfordulhat, hogy a célpontok nincsenek megjelölve
* Ellenőrizze a hibakeresési naplóban a „Célpont megtalálva” üzeneteket

### 2. szakasz: Elemzés

**Mi történik:**

* A képek EXIF-metaadatainak olvasása (időbélyegek, expozíciós beállítások)
* A kalibrációs stratégia meghatározása a célpontok időbélyegei és a rendelkezésre álló DAQ lefelé irányuló adatok alapján
* A képfeldolgozási sor rendezése
* A párhuzamos feldolgozást végző munkások előkészítése (csak Chloros

+)

**Időtartam:** 5–30 másodperc**Haladási jelző:**

* Elemzés: 0% → 100%
* Gyors szakasz, általában gyorsan befejeződik

**Mire kell figyelni:**

* Az előrehaladásnak szünetek nélkül, egyenletesen kell történnie
* A hiányzó metaadatokra vonatkozó figyelmeztetések a hibakeresési naplóban jelennek meg

### 3. szakasz: Kalibrálás

**Mi történik:*** **Debayering**: A RAW Bayer-minta 3 csatornára történő konvertálása (a LATTICE monokróm modulok esetében kihagyásra kerül, megjegyzéssel)
* **Vignettakorrekció**: Az objektív szélén jelentkező sötétedés eltávolítása
* **Fényvisszaverődési kalibrálás**: Normalizálás a célértékek és/vagy a DAQ-lefelé irányuló jel alapján
* **Indexszámítás**: Multispektrális indexek kiszámítása
* Minden kép feldolgozása a teljes feldolgozási folyamaton keresztül

**Időtartam:** A teljes feldolgozási idő nagy része (60–80%)**Haladási jelző:**

* Kalibrálás: 0% → 100%
* Jelenleg feldolgozás alatt álló kép
* Feldolgozott képek / Összes kép

**Feldolgozási mód:*** **Szabad mód**: Egyszerre egy képet dolgoz fel egymás után
* **„Chloros

+” mód**: Hardverhez alkalmazkodó munkavállalói csoportot futtat – 1–4 párhuzamos munkavállaló GPU-rendszereken (a VRAM-tól függően), egy munkavállaló fizikai magonként (mínusz egy) kizárólag CPU-val rendelkező rendszereken. Lásd [Dinamikus számítási adaptáció](../processing-architecture/dynamic-compute-adaptation.md)
* **GPU-gyorsítás**: Jelentősen felgyorsítja ezt a szakaszt**Mire kell figyelni:**

* A képek számának egyenletes növekedése
* Ellenőrizze a hibakeresési naplót a képek feldolgozásának befejezéséről szóló üzenetekért
* Figyelmeztetések a képminőséggel vagy kalibrációs problémákkal kapcsolatban

### 4. szakasz: Exportálás

**Mi történik:**

* A feldolgozott képek a kiválasztott formátumban lemezre írásra kerülnek, amint elkészülnek
* **LATTICE**: minden képkocka szétosztásra kerül az összes engedélyezett termékre (debayered / preview / radiance / reflectance)
* Multispektrális indexképek exportálása LUT-színekkel
* Az `<project>/<camera>/<format>/<Product>_Images/` kimeneti fa létrehozása — az exportált fájlok megtartják a forrásfájl nevét; a mappa azonosítja a terméket

**Időtartam:** a teljes feldolgozási idő 10–20%-a**Haladási jelző:**

* Exportálás: 0% → 100%
* Fájlok írása folyamatban
* Exportformátum és célhely

**Mire kell figyelni:**

* Lemezterület-figyelmeztetések
* Fájlírási hibák
* Az összes konfigurált kimenet befejezése

***

## Hibaelhárítási napló fül

A Hibaelhárítási napló részletes információkat nyújt a feldolgozás előrehaladásáról és az esetleges problémákról. A háttérrendszer indításakor megjelenő üzenetek is megjelennek a naplókonzolban, így a napló teljes képet ad a folyamatról, még akkor is, ha később nyitja meg.

### A hibakeresési napló eléréséhez

1. Kattintson a bal oldali oldalsávon található **Hibakeresési napló** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

ikonra
2. Megnyílik a naplópanel, amelyen a feldolgozás valós idejű üzenetei jelennek meg
3. Az oldal automatikusan görgetődik, hogy a legfrissebb üzenetek jelenjenek meg

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written), with the 1.3.0 bug icon (Report a bug) visible at the top of the tab -->

### A naplóüzenetek értelmezése A

Chloros

-napló sorai előtt zárójelben szereplő címkék jelzik az alrendszer nevét – például `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. A legfontosabb, amit tudni kell, az a **futásösszefoglaló**, amely minden futás végén megjelenik (beleértve a leállított futásokat is):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

További `[RUN-SUMMARY]` segítő sorok következnek, ha valamire magyarázatot kell adni — például egy olyan futtatás esetén, amely nem eredményezett semmit, vagy egy olyan kamera, amelynek kért termékét alkalmazhatatlannak minősítve kihagyták. Az `[EXPORT-CHECK]` sorok kameránkénti kihagyásokat magyarázzák (pl. miért nem kapott egy RGB

-kamera sugárzási terméket).

Az üzenetek általános súlyossági szintjei (az alábbi példák szemléltető jellegűek, nem szó szerinti idézetek):

#### Információs üzenetek (fehér/szürke)

Normál feldolgozási frissítések: a feldolgozás megkezdődött, célpontok észlelve (panelek számával együtt), képek szerinti kalibrációs előrehaladás, fájlok exportálva, feldolgozás befejezve.

#### Figyelmeztető üzenetek (sárga)

Nem kritikus problémák, amelyek nem állítják le a feldolgozást — pl. hiányzó GPS-adatok egy képkockában, nagy időbélyeg-különbség a célképek között, vagy alacsony kontraszt egy kalibrációs panelen.

**Tennivaló:** A feldolgozás után ellenőrizze a figyelmeztetéseket, de ne szakítsa meg a folyamatot

#### Hibaüzenetek (pRed

)

Kritikus problémák, amelyek a feldolgozás meghiúsulásához vezethetnek – pl. teli lemez, sérült képfájl, vagy célpontok hiánya a reflexiós kalibrálás kérésének idején.

**Tennivaló:** Állítsa le a feldolgozást, szüntesse meg a hibát, indítsa újra

### Gyakori naplóbejegyzések

| Helyzet                             | Jelentés                                       | Szükséges intézkedés                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Célpont észlelve a \[filename] fájlban        | Kalibrációs célpont sikeresen megtalálva         | Nincs – normális                                         |
| Képenkénti haladási sávok              | Aktuális haladási állapot frissítése                       | Nincs – normális                                         |
| Nem találtak célpontokat                      | Nem észleltek kalibrációs célpontokat               | Jelölje meg a célképeket, vagy tiltsa le a reflexiós kalibrációt |
| Nem elegendő lemezterület               | Nincs elegendő tárhely a kimenethez                 | Szabadítson fel lemezterületet                                    |
| A sérült fájl kihagyása               | A képfájl sérült                         | Másolja át újra a fájlt az SD-kártyáról                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | A nyers képkocka nélküli felvétel nem feldolgozható | Végezzen új felvételt nyers képkockával, vagy használja a CLI

-t `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | A futtatás nem eredményezett képtermékeket — hibaüzenet jelent meg tippekkel | Olvassa el a tippsorokat; ellenőrizze, mit hagyott ki a rendszer és miért |
| `[PROCESS] refused: The project folder is too deep for this computer…` | A Windows

hosszú elérési útjai ki vannak kapcsolva, és a projektmappa túl mély a 260 karakteres elérési útkorlát miatt; semmi sem került feldolgozásra | Használjon rövidebb projektmappát (az üzenet megadja a leghosszabb megengedett nevet), vagy kapcsolja be a Windows

hosszú elérési útjait, és indítsa újra a Chloros

-t — lásd [A feldolgozás elindítása](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer) |
| `[unmix-clamp-mask] …` | Egy LATTICE-termék clamp maszkja nem írható; maga a termék rendben van, és a következő üzenetet jeleníti meg: `Chloros:UnmixClampMask=unavailable` | Ellenőrizze a lemezterületet és a mappa jogosultságait, majd ha szüksége van a maszkra, futtassa újra a feldolgozást |

### Naplóadatok másolása

A hibaelhárítás vagy a támogatás céljából a napló másolásához:

1. Nyissa meg a Debug Log panelt
2. Kattintson a **Másolás**gombra (a napló a vágólapra kerül), vagy a**Letöltés** gombra, hogy szöveges fájlként mentse
3. Illessze be egy szöveges fájlba vagy e-mailbe
4. Szükség esetén küldje el a MAPIR

ügyfélszolgálatnak

### Hiba jelentése a Naplófülről

Újdonság az 1.3.0-s verzióban: a **Hiba jelentése**ikon a**Hibakeresési napló** fül tetején közvetlenül a MAPIR

-hez küldi a jelentést.

1. Kattintson a **Hibakeresési napló** fül tetején található hiba ikonra
2. Írja le a problémát: mit csinált, mire számított, és mi történt helyette (kötelező)
3. Opcionálisan jelölje be az **Adjon hozzá néhány képet (legfeljebb 10)**lehetőséget, és válasszon ki képeket a megnyitott projektből, vagy használja a**Tallózás...** gombot bármely más képfájlhoz
4. Kattintson a **Küldés** gombra

A jelentés tartalmazza a leírását, az esetlegesen csatolt képeket, a Chloros

-naplókat és a rendszer adatait (Chloros

-verzió, operációs rendszer, GPU és projektbeállítások). A jelentés elküldéséhez be kell jelentkeznie a Chloros

ba (**Felhasználó** fül). A küldés közben bezárhatja az ablakot; amikor befejeződik, a naplóba egy hivatkozási szám kerül beírásra, például `Bug report sent. Reference: BR-…` — ezt idézze meg, amikor kapcsolatba lép a MAPIR

-rel. Ha a jelentés nem tölthető fel (például internetkapcsolat hiánya miatt), a Chloros

elmenti azt a számítógépére, és megjeleníti az elérési utat, így e-mailben elküldheti az support@mapir.camera címre.

***

## Rendszererőforrások figyelése

### CPU-használat

**Szabad mód:**

* 1 CPU-mag ~100%-os kihasználtsággal
* A többi mag tétlen vagy rendelkezésre áll
* A rendszer továbbra is reagál

**„Chloros

+” párhuzamos mód:**

* Több mag magas kihasználtsággal – a számuk a [Dinamikus számítási adaptáció](../processing-architecture/dynamic-compute-adaptation.md) által választott stratégiától függ
* A rendszer reagálóképessége romolhat

**Figyeléshez:**

* Windows

Feladatkezelő (Ctrl+Shift+Esc)
* Teljesítmény fül → CPU szakasz
* Keresse meg a „Chloros

” vagy „chloros-backend” folyamatokat

### Memória (RAM) használat

**Jellemző használat:**

* Kis projektek (&lt; 100 kép): 2–4 GB
* Közepes projektek (100–500 kép): 4–8 GB
* Nagy projektek (500+ kép): 8–16 GB
* A „Chloros

+” párhuzamos üzemmód több RAM-ot igényel

**Ha kevés a memória:**

* Feldolgozzon kisebb adagokat
* Zárja be az egyéb alkalmazásokat
* Növelje a RAM-kapacitást, ha rendszeresen nagy adathalmazokat dolgoz fel

### GPU-használat (Chloros

+ CUDA-val)

Ha a GPU-gyorsítás engedélyezve van:

* Az NVIDIA GPU kihasználtsága magas (60–90%)
* Nő a VRAM-használat (4 GB feletti VRAM szükséges; 7 GB feletti a párhuzamos Texture Aware debayeringhez)
* A kalibrálási szakasz jelentősen gyorsabb

**Figyelemmel kísérés:**

* NVIDIA tálcaikon
* Feladatkezelő → Teljesítmény → GPU
* GPU-Z vagy hasonló felügyeleti eszköz

### Lemez I/O

**Mire számíthat:**

* Magas lemezolvasási terhelés az elemzési szakaszban
* Magas lemezírási terhelés az exportálási szakaszban
* Az SSD jelentősen gyorsabb, mint a HDD

**Teljesítményt növelő tippek:**

* Ha lehetséges, használjon SSD-t a projektmappa tárolásához
* Kerülje a hálózati meghajtók használatát nagy adathalmazok esetén
* Győződjön meg arról, hogy a lemez nem közelíti meg a kapacitáshatárt (ez befolyásolja az írási sebességet)

***

## Problémák észlelése a feldolgozás során

### Figyelmeztető jelek

**A folyamat leáll (5 percnél hosszabb ideig nincs változás):**

* Ellenőrizze a hibajelentést (Debug Log)
* Ellenőrizze a rendelkezésre álló lemezterületet
* A Feladatkezelőben ellenőrizze, hogy a Chloros

fut-e

**Gyakran megjelenő hibaüzenetek:**

* Állítsa le a feldolgozást, és vizsgálja meg a hibákat
* Gyakori okok: lemezterület, sérült fájlok, memóriaproblémák
* Lásd az alábbi Hibaelhárítás részt

**A rendszer nem reagál:**

* A Chloros

párhuzamos módja túl sok erőforrást igényel
* Fontolja meg az egyidejű feladatok számának csökkentését vagy a hardver frissítését
* A szabad mód kevésbé erőforrás-igényes

### Mikor kell leállítani a feldolgozást

Állítsa le a feldolgozást, ha a következőket észleli:

* ❌ „A lemez megtelt” vagy „A fájl nem írható” hibák
* ❌ Ismétlődő képfájl-sérülési hibák
* ❌ A rendszer teljesen lefagyott (nem reagál)
* ❌ Rájött, hogy helytelen beállításokat konfigurált
* ❌ Helytelen képek lettek importálva

**A leállítás módja:**

1. Kattintson a**Leállítás gombra** (a Start gomb helyett) — egyszer elég
2. A sávon a „Leállítás...” felirat jelenik meg, amíg a feldolgozás alatt álló kép befejeződik, majd a futtatás leállított állapotban ér véget
3. A már exportált termékek a lemezen maradnak; a napló őszintén rögzíti az `[RUN-SUMMARY]` kóddal, hogy mi fejeződött be
4. Javítsa ki a problémákat, majd indítsa újra — a futtatás az elejétől kezdődik

***

## Hibaelhárítás a feldolgozás során

### A feldolgozás nagyon lassú

**Lehetséges okok:**

* Megjelölés nélküli célképek (az összes kép beolvasása)
* HDD tároló SSD helyett
* Elégtelen rendszererőforrások
* Túl sok index van beállítva
* Hálózati meghajtóhoz való hozzáférés

**Megoldások:**

1. Ha a folyamat éppen most indult és az „Észlelés” szakaszban van: Állítsa le, jelölje meg a célokat, indítsa újra
2. A jövőben: Használjon SSD-t, csökkentse az indexek számát, frissítse a hardvert
3. Nagy adathalmazok kötegelt feldolgozásához fontolja meg a parancssori felület (CLI

) használatát

### „Lemezterület” figyelmeztetések

**Megoldások:**

1. Azonnal szabadítson fel lemezterületet
2. Helyezze át a projektet egy nagyobb tárhellyel rendelkező meghajtóra
3. Csökkentse az exportálandó indexek számát
4. Tiltsa le a felesleges LATTICE exporttermékeket (Projektbeállítások → Feldolgozás)
5. Használjon JPG formátumot a TIFF

helyett (kisebb fájlok)

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
3. Csökkentse a feldolgozási terhelést (használja a „Free” módot a „Chloros

+” helyett)
4. A feldolgozást a nap hűvösebb óráiban végezze

***

## Értesítés a feldolgozás befejezéséről

A feldolgozás befejezésekor:

* A haladási sáv eléri a 100%-ot
* Az `[RUN-SUMMARY]` sorok megjelennek a hibakeresési naplóban a végső számadatokkal
* A Start gomb újra aktiválódik
* Az összes kimeneti fájl a projekt kameránkénti kimeneti mappájában található: `<project>/<camera>/<format>/<Product>_Images/`

***

## Következő lépések

A feldolgozás befejezése után:

1. **Eredmények áttekintése** – Lásd: [A feldolgozás befejezése](finishing-the-processing.md)
2. **A kimeneti mappa ellenőrzése** – Ellenőrizze, hogy minden fájl helyesen került-e exportálásra
3. **A hibakeresési napló áttekintése** – Ellenőrizze, hogy vannak-e figyelmeztetések vagy hibák
4. **A feldolgozott képek előnézete** – Használja a Képnézegetőt vagy külső szoftvert

A feldolgozott eredmények áttekintésével és használatával kapcsolatos információkért lásd: [A feldolgozás befejezése](finishing-the-processing.md).
