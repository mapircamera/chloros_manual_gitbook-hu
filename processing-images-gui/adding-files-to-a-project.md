# Fájlok hozzáadása egy projekthez

Miután létrehozott vagy megnyitott egy projektet az Chloros programban, a következő lépés a multispektrális képek hozzáadása a feldolgozás megkezdéséhez. A „Fájlkezelő” (<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">) fül segítségével könnyedén importálhatja a képeket és kezelheti az adatkészletét.

## A Fájlkezelő megnyitása

1. Nyisson meg vagy hozzon létre egy projektet az Chloros programban
2. Kattintson a bal oldali oldalsávon található **Fájlkezelő** (<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">) ikonra
3. A Fájlkezelő panelen megjelenik a projekt fájllistája

{% hint style="info" %}
**Támogatott fájltípusok**:

* **Survey3W / Survey3N**: RAW+JPG párok és JPG képek (RAW+JPG ajánlott)
* **LATTICE**: `.tif` / `.tiff` felvételek — az Chloros kameravezérlővel vagy egy LATTICE hub segítségével rögzítve
* **Fényérzékelő-adatok**: `.daq` felvételek (DAQ-U/M/E) és DAQ-M `.csv` lefelé irányuló napfény-naplók — a képekkel együtt importálva a reflexiós kalibráláshoz
{% endhint %}

***

## Képek hozzáadása a projekthez

Két fő módszer létezik a képek hozzáadására a projekthez:

### 1. módszer: Fájlok hozzáadása

Ezzel az opcióval importálhat egyes képfájlokat vagy egy kis fájlválogatást.

1. Kattintson a **„Fájlok hozzáadása”** gombra (<img src="../.gitbook/assets/image (3).png" alt="" data-size="line">) a Fájlkezelő panel tetején
2. Keresse meg a képeket tartalmazó mappát
3. Válasszon ki egy vagy több képfájlt (tartsa lenyomva a **Ctrl** billentyűt több fájl kijelöléséhez)
4. Kattintson a **„Megnyitás”** gombra a kijelölt fájlok importálásához

### 2. módszer: Mappa hozzáadása

Ezzel az opcióval egyszerre importálhatja egy mappa összes képét. Egy párbeszédablakban **több mappát** is kiválaszthat.

1. Kattintson a Fájlkezelő panel tetején található **„Mappa hozzáadása”** gombra (<img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line">)
2. Keresse meg és válassza ki a felvételi munkamenet képeit tartalmazó mappát vagy mappákat
3. Kattintson a **„Mappa kiválasztása”** gombra az összes támogatott kép importálásához

{% hint style="info" %}
**A betöltés sikertelen fájlokról a rendszer értesítést küld.** Ha egy mappa olyan fájlokat tartalmaz, amelyeket az Chloros felismer, de nem tud betölteni, a rendszer figyelmeztető üzenetet jelenít meg – a képek nem tűnnek el észrevétlenül a rácsból.
{% endhint %}

***

## LATTICE-felvételi mappák importálása

A LATTICE-felvételeket **exportálási szintenként egy-egy almappában** mentik – például `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` – a gyökérkönyvtárban található, ehhez tartozó `.daq` lefelé irányuló fájllal együtt:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Állítsa be a „Mappa hozzáadása” opciót a felvételek gyökérkönyvtárára** (a fenti `output/`). Ha a kiválasztott mappa maga nem tartalmaz képeket, de almappákkal rendelkezik, az Chloros automatikusan belép azokba — a szint almappáit és a gyökérmappát (`.daq`) egy lépésben veszi fel.**A felvételek importálásának módja:*** Minden felvétel **egyetlen kép**ként kerül importálásra, felvételenként csoportosítva (nem szintenként egy-egy bejegyzésként). Ugyanazon felvétel többi szintje az adott kép nézetmódjaiként jelenik meg.
* **A feldolgozás mindig a nyers képkockából indul.** A többi szint megtekinthető, de csak az `raw` kerül be a feldolgozási folyamatba — egy már feldolgozott termék újrafeldolgozása a korrekciók kétszeri alkalmazását eredményezné, ezért az Chloros elutasításra kerül. Egy újraimportált export soha nem foglalhatja el egy felvétel nyers fájl helyét.
* Egy felvételi mappa **nyers fájlok nélkül** is menthető és normálisan megjelenik, de a feldolgozás kihagyja, és ezt a naplóban is jelzi. (Az CLI jelző `--input-level` ebben az esetben kényszerítheti a belépési pontot — lásd [az CLI Referenciát](../reference/cli-reference.md#what-a-captures-folder-looks-like).)**A LATTICE hub munkamenetek** ugyanúgy importálhatók: az „Add Folder” gombbal jelölje ki a hubból másolt munkamenetmappát (amely tartalmazza az `raw/` és az `previews/` fájlokat), valamint az esetleges DAQ-M `.csv` lefelé irányuló naplófájlokat. Ha a kamera vagy a DAQ kalibrációja még nincs tárolva a gépen, az Chloros az importáláskor automatikusan letölti azt a sorozatszám alapján (ehhez egyszeri internetkapcsolatra van szükség).***

## A fájlkezelő táblázat megértése

A képek importálása után egy táblázatban jelennek meg, amely a következő oszlopokat tartalmazza:

### Fájlnév

* A fényképezőgép eredeti fájlneve
* Megőrzi a fényképezőgép névadási konvencióját (pl. IMG\_0001.RAW vagy capture\_20260816\_101500\_SN213800234\_raw.tif)

### Időbélyeg

* A kép rögzítésének dátuma és időpontja
* A kép EXIF-metadatából nyert információ
* Fényérzékelő-illesztéshez, PPK-szinkronizáláshoz és kalibrációs célpontok ütemezéséhez használatos

### Fényképezőgép-modell

* Automatikusan felismert fényképezőgép- és szűrőkonfiguráció
* Survey3 példák: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* LATTICE példák: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* A helyes feldolgozási profilok alkalmazásához használatos

### Céloszlop (jelölőnégyzet)

* Jelölje be ezt a jelölőnégyzetet azoknál a képeknél, amelyek kalibrációs célokat tartalmaznak
* Ha legalább egy kép be van jelölve, **csak a bejelölt képeket vizsgálja** a rendszer célok után
* Részletekért lásd: [Célképek kiválasztása](choosing-target-images.md)

### Képek metaadatainak megtekintése

A táblázat feletti jobb felső sarokban található kapcsoló gombra kattintva a kép rácsos területén megjelennek a kiválasztott kép metaadatai.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## A projektben található fényérzékelő-fájlok

* Az `.daq` és `.csv` fájlok megjelennek a Fájlkezelő listájában, de nem kattintható képek — ezek a reflexiós kalibráláshoz szükséges lefelé irányuló besugárzási intenzitást biztosítják.
* Minden importált `.daq`/`.csv` fájl felkerül a **Projektbeállítások → DAQ fényérzékelő** menüpontba, ahol ellenőrizheti az egyes fájlokra érvényes diffúzor-korrekciót. Lásd: [A projektbeállítások módosítása](adjusting-project-settings.md).
* A **Fényérzékelők** fülön készített felvételek automatikusan hozzáadódnak a megnyitott projekthez – nincs szükség kézi importálásra.***

## Fájlok kezelése a projektben

### Fájlok eltávolítása

A nem kívánt képek eltávolításához a projektből:

1. Válasszon ki egy vagy több képet a Fájlkezelő táblázatban
2. Kattintson a **„Kijelöltek eltávolítása”** gombra <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Erősítse meg az eltávolítást (a fájlok nem kerülnek törlésre a lemezről, csak eltávolításra kerülnek a projektből)

### Rendezés és szűrés

* **Oszlop szerinti rendezés**: Kattintson bármely oszlopfejlécre a képek rendezéséhez
* **Időbélyeg szerinti rendezés**: Hasznos a felvételek időrend szerinti csoportosításához
* **Kamera modell szerinti szűrés**: Csoportosítsa a képeket kameratípus szerint, ha több kamerát használ***

## Képelőnézet

### Teljes kép megtekintése

Kattintson bármelyik képminiatűrre a Fájlkezelőben, hogy az megjelenjen a fő előnézeti területen:

1. A kép megjelenik a középső előnézeti panelen
2. A nagyítási vezérlőkkel megvizsgálhatja a kép részleteit
3. A nyílgombokkal navigálhat a képek között

### Gyors navigáció

* **Előző kép**: Kattintson a bal nyílra, vagy nyomja meg a ← gombot
* **Következő kép**: Kattintson a jobb nyílra, vagy nyomja meg a → gombot
* **Nagyítás/kicsinyítés**: Használja az egérgörgőt vagy a nagyítási gombokat
* **Képmozgatás**: Nagyítás esetén kattintson a képre, és húzza el***

## Ismétlődő fájlok kezelése

Az Chloros automatikusan felismeri és figyelmen kívül hagyja az ismétlődő fájlokat:

* Az azonos fájlnévvel rendelkező fájlokat kihagyja
* Megakadályozza a véletlen kétszeri feldolgozást
* Figyelmeztető üzenet jelenik meg, ha ismétlődő fájlokat észlel

{% hint style="warning" %}
**Fontos**: Az importálás előtt ne nevezze át és ne módosítsa az eredeti képfájlokat. Az Chloros a megfelelő feldolgozáshoz az eredeti fájlnevekre és metaadatokra támaszkodik.
{% endhint %}

***

## Vegyes kamerás adatkészletek

Ha a projektje több MAPIR kamerából származó képeket tartalmaz:

1. Az Chloros automatikusan felismeri az egyes kameramodelleket — Survey3, LATTICE vagy ezek keverékét
2. Minden kameratípust a hozzá tartozó kalibrációs profillal dolgoznak fel
3. A Fájlkezelő a „Kameramodell” oszlopban jeleníti meg a kameramodellt
4. A feldolgozás során minden kamera saját kimeneti mappafát kap

**Példahelyzetek**: Survey3W RGN + Survey3N OCN kétkamerás konfiguráció, vagy egy LATTICE-rendszer, amely egy RGB vezérlőegységet és több keskenysávú modult tartalmaz***

## Bevált gyakorlatok

### Rendezés az importálás előtt

* A kalibrációs célképeket ugyanabban a mappában tárolja, mint a felmérési képeket
* Minden felvételi munkamenet `.daq` / `.csv` fényérzékelő-fájljait tárolja az adott munkamenet képeivel együtt
* Tartsa meg a kamera/SD-kártya/hub eredeti mappaszerkezetét
* Ne keverje össze különböző felvételi munkamenetek adatkészleteit egy projektben

### Fájlnevezés

* Tartsa meg az eredeti kamerafájlneveket (IMG\_0001.RAW, capture\_..., stb.)
* Ne nevezze át a fájlokat az importálás előtt
* Az eredeti nevek fontos metaadatokat tartalmaznak

### Kalibrációs célképek

* Minden munkamenethez mindig csatoljon 1–2 kalibrációs célképet (Survey3; a LATTICE esetében ez helyettesíthető egy DAQ-felvétellel — lásd [Célképek kiválasztása](choosing-target-images.md))
* A felvételi munkamenet előtt és után is készítsen felvételeket a kalibrációs célképről
* Helyezze a célképeket ugyanolyan fényviszonyok közé, mint a felvételi területet
* Jelölje meg a célképeket a „Cél” jelölőnégyzet bejelölésével

***

## Gyakori problémák és megoldások

### A képek nem jelennek meg az importálás után

**Lehetséges okok:**

* A fájlformátum nem támogatott (lásd a támogatott típusok listáját az oldal tetején)
* A képek nem MAPIR kamerákkal készültek (lásd [Támogatott kamerák](../supported-cameras.md))
* A fájl sérült, vagy az SD-kártyáról nem sikerült teljes mértékben átmásolni

**Megoldás**: Ellenőrizze a fájlformátum és a fényképezőgép-modell kompatibilitását, és nézze meg a fájlbetöltési figyelmeztetést azoknál a fájloknál, amelyeknél hiba történt

### A fényképezőgép-modell nem került felismerésre

**Lehetséges okok:**

* Módosított EXIF-metaadatok
* Külső szoftverrel szerkesztett képek
* Hiányos fájlátvitel

**Megoldás**: Importálja újra az eredeti, módosítatlan fájlokat a fényképezőgépről vagy az SD-kártyáról

### Hiányzó időbélyegek

**Lehetséges okok:**

* A fényképezőgép órája nincs megfelelően beállítva
* A külső szoftver eltávolította az EXIF-adatokat

**Megoldás**: Ellenőrizze, hogy a fényképezés során a fényképezőgép időbeállításai helyesek voltak-e

### A projekt újbóli megnyitásakor hiányzó fájlokról szóló jelentés

Ha a forrásfájlokat a projekt legutóbbi megnyitása óta áthelyezték vagy törölték, az Chloros kód jelzi, **mely** fájlok hiányoznak, ahelyett, hogy üres rácsot nyitna meg. Állítsa vissza a fájlokat az eredeti helyükre, vagy távolítsa el a hiányzó bejegyzéseket, majd importálja újra őket.***

## Következő lépések

A fájlok importálása után:

1. **Ellenőrizze a fájllistát** – Győződjön meg arról, hogy minden kép helyesen töltődött be
2. **Ellenőrizze a fényképezőgép-modelleket** – Ellenőrizze, hogy a fényképezőgépek felismerése helyes-e
3. **Jelölje meg a célképeket** – Lásd: [Célképek kiválasztása](choosing-target-images.md)
4. **Állítsa be a beállításokat** – Konfigurálja a feldolgozási opciókat a [Projektbeállítások](adjusting-project-settings.md) menüpontban
5. **Indítsa el a feldolgozást** – Lásd: [A feldolgozás elindítása](starting-the-processing.md)

A projektkonfigurációval kapcsolatos részletes információkat a [Projektbeállítások módosítása](adjusting-project-settings.md) című részben találja.
