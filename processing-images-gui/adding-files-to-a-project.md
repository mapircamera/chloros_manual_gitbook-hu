# Fájlok hozzáadása egy projekthez

Miután létrehozott vagy megnyitott egy projektet az Chloros programban, a következő lépés a multispektrális képek hozzáadása a feldolgozás megkezdéséhez. A Fájlböngésző<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> fül segítségével könnyedén importálhat képeket és kezelheti adatkészletét.

## A Fájlböngésző elérés

1. Nyisson meg vagy hozzon létre egy projektet az Chloros programban
2. Kattintson a **Fájlböngésző** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> ikonra a bal oldali sávban.
3. A Fájlböngésző panel megjeleníti a projekt fájljainak listáját.

{% tipp style=&quot;info&quot; %}
**Támogatott fájltípusok**: Az Chloros támogatja az MAPIR Survey3W és Survey3N kamerák RAW+JPG és JPG képfájljait. Csak a RAW+JPG formátumot ajánljuk.
{% endhint %}

***

## Képek hozzáadása a projekthez

Két fő módszer létezik a képek hozzáadására a projekthez:

### 1. módszer: Fájlok hozzáadása

Ezzel az opcióval egyedi képfájlokat vagy egy kis fájlválogatást importálhat.

1. Kattintson a **„Fájlok hozzáadása”** gombra a Fájlkezelő panel tetején.
2. Keresse meg a képeket tartalmazó mappát.
3. Válasszon ki egy vagy több képfájlt (több fájl kiválasztásához tartsa lenyomva a **Ctrl** billentyűt).
4. Kattintson az **„Megnyitás”** gombra a kiválasztott fájlok importálásához.

### 2. módszer: Mappa hozzáadása

Ezzel az opcióval egyszerre importálhatja az összes képet egy mappából.

1. Kattintson a **„Mappa hozzáadása”** gombra a Fájlböngésző panel tetején.
2. Keresse meg és válassza ki a rögzített képeket tartalmazó mappát.
3. Kattintson a **„Mappa kiválasztása”** gombra az összes támogatott kép importálásához a mappából.

***

## A Fájlböngésző táblázat megértése

A képek importálása után azok a következő oszlopokkal rendelkező táblázatban jelennek meg:

### Miniatűr

* Az egyes képek kicsinyített előnézete.
* Kattintson a miniatűrre a teljes kép megtekintéséhez a fő előnézeti területen.

### Fájlnév

* A fényképezőgép eredeti fájlneve.
* Megőrzi a fényképezőgép névadási konvencióját (pl. IMG\_0001.RAW).

### Időbélyeg

* A kép rögzítésének dátuma és időpontja.
* A kép EXIF metaadataiból kivonva.
* PPK szinkronizáláshoz és kalibrációs célpontok észleléséhez használatos

### Kamera modell

* Automatikusan észlelt kamera és szűrő konfiguráció
* Példák: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* A megfelelő feldolgozási profilok alkalmazásához használatos

### Cél oszlop (jelölőnégyzet)

* Jelölje be ezt a jelölőnégyzetet a kalibrációs célokat tartalmazó képeknél
* Jelentősen felgyorsítja a célok észlelését a feldolgozás során
* A részleteket lásd a [Célképek kiválasztása](choosing-target-images.md) című részben.

***

## Fájlok kezelése a projektben

### Fájlok eltávolítása

A nem kívánt képek eltávolítása a projektből:

1. Válasszon ki egy vagy több képet a Fájlböngésző táblázatban
2. Kattintson a **„Kiválasztottak eltávolítása”** gombra
3. Erősítse meg az eltávolítást (a fájlok nem kerülnek törlésre a lemezről, csak eltávolításra a projektből)

### Rendezés és szűrés

* **Oszlop szerinti rendezés**: Kattintson bármely oszlopfejlécre a képek rendezéséhez
* **Időbélyeg szerinti rendezés**: Hasznos a kronológiai felvételi sorozatok rendezéséhez.
* **Fényképezőgép-modell szűrő**: Több fényképezőgép használata esetén csoportosítsa a képeket fényképezőgép-típus szerint.

***

## Képek előnézete

### Teljes kép megtekintése

Kattintson bármelyik kép miniatűrjére a Fájlböngészőben, hogy azt a fő előnézeti területen megjelenítse:

1. A kép a középső előnézeti panelen jelenik meg.
2. A nagyítás vezérlőivel megtekintheti a kép részleteit.
3. A nyílgombokkal navigálhat a képek között

### Gyors navigáció

* **Előző kép**: Kattintson a bal nyílra, vagy nyomja meg a ← gombot
* **Következő kép**: Kattintson a jobb nyílra, vagy nyomja meg a → gombot
* **Nagyítás/kicsinyítés**: Használja az egér görgőjét vagy a nagyító gombokat
* **Panoráma**: Nagyítás esetén kattintson a képre, és húzza el

***

## Duplikált fájlok kezelése

Az Chloros automatikusan felismeri és figyelmen kívül hagyja az ismétlődő fájlokat:

* Az azonos fájlnévvel rendelkező fájlokat kihagyja.
* Megakadályozza a véletlen kettős feldolgozást.
* Figyelmeztető üzenet jelenik meg, ha ismétlődő fájlokat észlel.

{% hint style=&quot;warning&quot; %}
**Fontos**: Az importálás előtt ne nevezze át és ne módosítsa az eredeti képfájlokat. Chloros a megfelelő feldolgozáshoz az eredeti fájlnevekre és metaadatokra támaszkodik.
{% endhint %}

***

## Vegyes kameraadatkészletek

Ha a projekt több MAPIR kamerából származó képeket tartalmaz:

1. Az Chloros automatikusan felismeri az egyes kameramodelleket.
2. Az egyes kameratípusokat a megfelelő kalibrációs profillal dolgozza fel.
3. A Fájlböngésző a Kameramodell oszlopban jeleníti meg a kameramodellt.
4. A feldolgozás minden kameratípusra a megfelelő beállításokat alkalmazza.

**Példa**: Survey3W RGN + Survey3N OCN kettős kamera beállítás

***

## Bevált gyakorlatok

### Importálás előtti szervezés

* A kalibrációs célképeket tartsa ugyanabban a mappában, mint a felmérési képeket.
* Tartsa meg a kamera/SD-kártya eredeti mappaszerkezetét.
* Ne keverje össze a különböző munkamenetek adatait egy projektben.

### Fájlnevezés

* Tartsa meg a kamera eredeti fájlneveit (IMG\_0001.RAW stb.).
* Ne nevezze át a fájlokat importálás előtt.
* Az eredeti nevek fontos metaadatokat tartalmaznak.

### Kalibrációs célképek

* Minden munkamenetben mindig 1-2 kalibrációs célképet vegyen fel.
* A felvételek előtt és után rögzítse a célokat.
* Helyezze a célokat ugyanolyan fényviszonyok közé, mint a felvételi területet.
* A feldolgozás gyorsítása érdekében jelölje meg a célképeket a Cél jelölőnégyzet segítségével.

***

## Gyakori problémák és megoldások

### A képek nem jelennek meg az importálás után

**Lehetséges okok:**

* A fájlformátum nem támogatott (csak RAW+JPG és JPG fájlok MAPIR kamerákból)
* A képek nem MAPIR kamerákból származnak (lásd [Támogatott kamerák](../supported-cameras.md))
* A fájl sérült vagy az SD-kártyáról nem sikerült teljes mértékben átmásolni

**Megoldás**: Ellenőrizze a fájlformátum és a fényképezőgép modell kompatibilitását.

### A fényképezőgép modellje nem ismerhető fel

**Lehetséges okok:**

* Módosított EXIF metaadatok
* Külső szoftverrel szerkesztett képek
* Hiányos fájlátvitel

**Megoldás**: Importálja újra az eredeti, módosítatlan fájlokat a fényképezőgépből/SD-kártyáról.

### Hiányzó időbélyegek

**Lehetséges okok:**

* A kamera órája nincs megfelelően beállítva
* Az EXIF-adatok külső szoftver által törölve

**Megoldás**: Ellenőrizze, hogy a kamera időbeállításai a felvétel készítése során megfelelőek voltak-e.

***

## Következő lépések

A fájlok importálása után:

1. **Ellenőrizze a fájllistát** – Győződjön meg arról, hogy minden kép megfelelően töltődött be.
2. **Ellenőrizze a kamera modelleket** – Ellenőrizze a kamera megfelelő felismerését.
3. **Jelölje meg a célképeket** – Lásd: [Célképek kiválasztása](choosing-target-images.md)
4. **Állítsa be a beállításokat** – Konfigurálja a feldolgozási opciókat a [Projektbeállítások](adjusting-project-settings.md) menüpontban
5. **Indítsa el a feldolgozást** – Lásd: [A feldolgozás elindítása](starting-the-processing.md)

A projekt konfigurációjával kapcsolatos részletes információkat a [Projektbeállítások módosítása](adjusting-project-settings.md) című részben talál.
