# Fájlok hozzáadása egy projekthez

Miután létrehozott vagy megnyitott egy projektet az Chloros programban, a következő lépés a multispektrális képek hozzáadása a feldolgozás megkezdéséhez. A Fájlkezelő<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> fül segítségével könnyen importálhatja a képeket és kezelheti az adatkészletet.

## A Fájlkezelő elérés

1. Nyisson meg vagy hozzon létre egy projektet az Chloros programban
2. Kattintson a **Fájlkezelő** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> ikonra a bal oldali sávban
3. A Fájlkezelő panelen megjelenik a projekt fájllistája

{% hint style="info" %}
**Támogatott fájltípusok**: Az Chloros támogatja az MAPIR, Survey3W és Survey3N kamerák RAW+JPG és JPG képfájljait. Csak a RAW+JPG formátum használata ajánlott.
{% endhint %}

***

## Képek hozzáadása a projekthez

Két fő módja van a képek hozzáadásának a projekthez:

### 1. módszer: Fájlok hozzáadása

Használja ezt az opciót egyedi képfájlok vagy egy kis fájlválogatás importálásához.

1. Kattintson a **&quot;Fájlok hozzáadása&quot;** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> gombra a Fájlkezelő panel tetején
2. Keresse meg a képeket tartalmazó mappát
3. Válasszon ki egy vagy több képfájlt (tartsa lenyomva a **Ctrl** billentyűt több fájl kiválasztásához)
4. Kattintson a **„Megnyitás”** gombra a kiválasztott fájlok importálásához

### 2. módszer: Mappa hozzáadása

Ezzel az opcióval egyszerre importálhatja egy mappa összes képét.

1. Kattintson a **„Mappa hozzáadása”** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> gombra a Fájlkezelő panel tetején
2. Keresse meg és válassza ki a felvételi munkamenet képeit tartalmazó mappát
3. Kattintson a **„Mappa kiválasztása”** gombra a mappában található összes támogatott kép importálásához***

## A Fájlkezelő táblázat megértése

A képek importálása után azok a következő oszlopokat tartalmazó táblázatban jelennek meg:

### Fájlnév

* A fényképezőgép eredeti fájlneve
* Megtartja a fényképezőgép elnevezési konvencióját (pl. IMG\_0001.RAW)

### Időbélyeg

* A kép rögzítésének dátuma és időpontja
* A kép EXIF metaadataiból kivont adat
* PPK szinkronizáláshoz és kalibrációs célpont felismeréshez használatos

### Fényképezőgép modell

* Automatikusan felismert kamera- és szűrőkonfiguráció
* Példák: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* A megfelelő feldolgozási profilok alkalmazásához használatos

### Cél oszlop (jelölőnégyzet)

* Jelölje be ezt a jelölőnégyzetet azoknál a képeknél, amelyek kalibrációs célpontokat tartalmaznak
* Jelentősen felgyorsítja a célpontok felismerését a feldolgozás során
* A részleteket lásd a [Célképek kiválasztása](choosing-target-images.md) részben

### Képadatok megtekintése

A táblázat feletti jobb felső sarokban található kapcsoló gombra kattintva a kiválasztott kép adatai megjelennek a képrács területén.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Fájlok kezelése a projektben

### Fájlok eltávolítása

A nem kívánt képek eltávolításához a projektből:

1. Válasszon ki egy vagy több képet a Fájlböngésző táblázatban
2. Kattintson a **„Kijelöltek eltávolítása”** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> gombra
3. Erősítse meg az eltávolítást (a fájlok nem kerülnek törlésre a lemezről, csak a projektből kerülnek eltávolításra)

### Rendezés és szűrés

* **Rendezés oszlop szerint**: Kattintson bármelyik oszlopfejlécre a képek rendezéséhez
* **Időbélyeg szerinti rendezés**: Hasznos a felvételek időrendi sorrendbe rendezéséhez
* **Kamera modell szűrő**: Csoportosítsa a képeket kameratípus szerint, ha több kamerát használ***

## Képelőnézet

### Teljes kép megtekintése

Kattintson bármelyik képminiatűrre a Fájlkezelőben, hogy az megjelenjen a fő előnézeti területen:

1. A kép megjelenik a középső előnézeti panelen
2. Használja a nagyítási vezérlőket a kép részleteinek megtekintéséhez
3. A nyílgombokkal navigálhat a képek között

### Gyors navigáció

* **Előző kép**: Kattintson a balra mutató nyílra, vagy nyomja meg a ← gombot
* **Következő kép**: Kattintson a jobb nyílra, vagy nyomja meg a → gombot
* **Nagyítás/Kicsinyítés**: Használja az egérgörgőt vagy a nagyító gombokat
* **Panorámázás**: Nagyítás esetén kattintson a képre, és húzza az egérrel***

## Duplikált fájlok kezelése

Az Chloros automatikusan felismeri és figyelmen kívül hagyja a duplikált fájlokat:

* Az azonos fájlnevekkel rendelkező fájlokat kihagyja
* Megakadályozza a véletlen kettős feldolgozást
* Figyelmeztető üzenet jelenik meg, ha duplikátumokat észlel

{% hint style="warning" %}
**Fontos**: Ne nevezze át és ne módosítsa az eredeti képfájlokat az importálás előtt. Az Chloros a megfelelő feldolgozáshoz az eredeti fájlnevekre és metaadatokra támaszkodik.
{% endhint %}

***

## Vegyes kamerás adatkészletek

Ha a projekt több MAPIR kamerából származó képeket tartalmaz:

1. Az Chloros automatikusan felismeri az egyes kameramodelleket
2. Minden kameratípust a megfelelő kalibrációs profillal dolgoz fel
3. A Fájlkezelő a Kamera modell oszlopban jeleníti meg a kamera modelljét
4. A feldolgozás minden kameratípusra a megfelelő beállításokat alkalmazza

**Példa**: Survey3W RGN + Survey3N OCN kettős kamera beállítás***

## Bevált gyakorlatok

### Rendezés az importálás előtt

* A kalibrációs célképeket tartsa ugyanabban a mappában, mint a felmérési képeket
* Tartsa meg a kamera/SD-kártya eredeti mappaszerkezetét
* Ne keverje össze különböző munkamenetek adatkészleteit egy projektben

### Fájlnevezés

* Tartsa meg az eredeti kamerafájlneveket (IMG\_0001.RAW stb.)
* Ne nevezze át a fájlokat az importálás előtt
* Az eredeti nevek fontos metaadatokat tartalmaznak

### Kalibrációs célképek

* Minden munkamenethez mindig csatoljon 1-2 kalibrációs célképet
* A felvételi munkamenet előtt és után is készítsen felvételeket a célpontokról
* Helyezze a célpontokat ugyanolyan fényviszonyok közé, mint a felvételi területet
* A feldolgozás felgyorsítása érdekében jelölje meg a célképeket a Cél jelölőnégyzet segítségével

***

## Gyakori problémák és megoldások

### A képek nem jelennek meg az importálás után

**Lehetséges okok:**

* A fájlformátum nem támogatott (csak RAW+JPG és JPG az MAPIR kamerákból)
* A képek nem MAPIR kamerákból származnak (lásd [Támogatott kamerák](../supported-cameras.md))
* A fájl sérült vagy az SD-kártyáról nem sikerült teljes mértékben átmásolni

**Megoldás**: Ellenőrizze a fájlformátum és a fényképezőgép modelljének kompatibilitását

### A fényképezőgép modellje nem került felismerésre

**Lehetséges okok:**

* Módosított EXIF metaadatok
* A képeket külső szoftverrel szerkesztették
* A fájlátvitel nem sikerült teljes mértékben

**Megoldás**: Importálja újra az eredeti, módosítatlan fájlokat a fényképezőgépből/SD-kártyáról

### Hiányzó időbélyegek

**Lehetséges okok:**

* A fényképezőgép órája nincs megfelelően beállítva
* Az EXIF-adatokat külső szoftver eltávolította

**Megoldás**: Ellenőrizze, hogy a fényképezés során a fényképezőgép időbeállításai helyesek voltak-e***

## Következő lépések

A fájlok importálása után:

1. **Ellenőrizze a fájllistát** – Győződjön meg arról, hogy minden kép megfelelően betöltődött
2. **Ellenőrizze a fényképezőgép-modelleket** – Ellenőrizze, hogy a fényképezőgép felismerése helyes-e
3. **Jelölje meg a célképeket** – Lásd: [Célképek kiválasztása](choosing-target-images.md)
4. **Állítsa be a beállításokat** – Konfigurálja a feldolgozási opciókat a [Projektbeállítások](adjusting-project-settings.md) menüpontban
5. **Indítsa el a feldolgozást** – Lásd: [A feldolgozás elindítása](starting-the-processing.md)

A projektkonfigurációval kapcsolatos részletes információkat a [Projektbeállítások módosítása](adjusting-project-settings.md) című részben találja.
