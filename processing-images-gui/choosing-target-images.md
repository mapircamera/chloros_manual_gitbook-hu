# Célképek kiválasztása

A kalibrációs célokat tartalmazó képek megjelölése egy fontos lépés, amely jelentősen felgyorsítja az Chloros feldolgozási folyamatot. A célképek előzetes kiválasztásával elkerülhető, hogy az Chloros az adatkészlet minden képét beolvassa a kalibrációs célok keresése érdekében.

## Miért érdemes megjelölni a célképeket?

### Feldolgozási sebesség

A célképek megjelölése nélkül az Chloros-nek:

* át kell vizsgálnia a projektben található összes képet,
* minden képen futtatnia kell a célfelismerő algoritmusokat,
* feleslegesen több száz vagy ezer képet kell ellenőriznie.

**Eredmény**: A feldolgozás jelentősen hosszabb időt vehet igénybe, különösen nagy adatkészletek esetén.

### Megjelölt célképekkel

Ha a Cél oszlopban megjelöl bizonyos képeket:

* Az Chloros csak a megjelölt képeket vizsgálja meg célok után kutatva
* A célfelismerés sokkal gyorsabban befejeződik
* Az általános feldolgozási idő jelentősen csökken

{% hint style=&quot;success&quot; %}
**Sebességjavulás**: 500 képből álló adatkészletben 2-3 célkép megjelölésével a célfelismerés ideje 30 percről 1 perc alá csökkenhet.
{% endhint %}

***

## Hogyan jelölje meg a célképeket

### 1. lépés: Azonosítsa a célképeket

Nézze át az importált képeket a Fájlböngészőben, és azonosítsa, mely képek tartalmaznak kalibrációs célokat.

**Gyakori esetek:**

* **Felvétel előtti cél**: A munkamenet megkezdése előtt rögzített
* **Felvétel utáni cél**: A munkamenet befejezése után rögzített
* **Terepi célok**: A rögzítési területen elhelyezett célok
* **Több cél**: 2-3 célkép munkamenetenként (ajánlott)

### 2. lépés: Ellenőrizze a Cél oszlopot

Minden kalibrációs célt tartalmazó kép esetében:

1. Keresse meg a képet a Fájlböngésző táblázatában.
2. Keresse meg a **Cél** oszlopot (a legjobbra lévő oszlop).
3. Kattintson a Cél oszlopban az adott kép jelölőnégyzetére.
4. Ismételje meg az összes célt tartalmazó kép esetében.

### 3. lépés: Ellenőrizze a kiválasztást

A feldolgozás előtt ellenőrizze:

* [ ] Minden kalibrációs célt tartalmazó kép be van jelölve.
* [ ] Nincs véletlenül bejelölve olyan kép, amely nem tartalmaz célt.
* [ ] A célok jól láthatóak a bejelölt képeken.

***

## A célképek legjobb gyakorlata

### Célképek rögzítésére vonatkozó irányelvek

**Időzítés:**

* A célképeket közvetlenül a rögzítési munkamenet előtt és annak során rögzítse.
* A DAQ fényérzékelőjével megegyező fényviszonyok között.
* A legjobb eredmények elérése érdekében ideális esetben a célképeket minél gyakrabban rögzítse. Ellenkező esetben a fényérzékelő adatai alapján kerül sor a kalibrálás időbeli kiigazítására.

**Kamera pozíciója:**

* Tartsa a kamerát a cél felett úgy, hogy az középen legyen, és a kép közepének körülbelül 40-60%-át kitöltse.
* Tartsa a kamerát párhuzamosan/nadírban a cél felületével

**Világítás:**

* Ugyanaz a környezeti megvilágítás, mint a DAQ fényérzékelőjének.
* Kerülje az árnyékokat a célfelületeken.
* Ne takarja el a fényforrást a testével, járművel vagy növényzettel.
* A felhős időjárás biztosítja a legkonzisztensebb eredményeket.

**Célállapot:**

* Tartsa a célpaneleket tisztán és szárazon.
* Mind a 4 panelnek jól láthatónak és akadálymentesnek kell lennie.
* A célok lehetőség szerint merőlegesek/nadír a fényforráshoz képest.

### Hány célkép szükséges?

**Minimum:** 1 célkép munkamenetenként. **Ajánlott:** 3-5 célkép munkamenetenként.

**A legjobb gyakorlat ütemezése:**

* 3-5 kép rögzítése röviddel a fényérzékelő felvételének megkezdése után
* A legjobb eredmények elérése érdekében forgassa el a kamerát a felvételek között
* Opcionális: rendszeresen a munkamenet közepén, ha a fényviszonyok folyamatosan változnak

***

## Több kamerával való munka

### Kétkamerás beállítások

Ha két MAPIR kamerát használ egyszerre (pl. Survey3W RGN + Survey3N OCN):

1. Készítse el a célképeket **mindkét kamerával** egyszerre.
2. Használja **ugyanazt a fizikai célt** mindkét kamerához.
3. Jelölje meg a célképeket **mindkét kameratípushoz** a Fájlböngészőben.
4. Az Chloros minden kamera kalibrálásához a megfelelő célokat fogja használni.

### Kamera modell oszlop

A **Kamera modell** oszlop segít azonosítani, hogy melyik képek melyik kamerából származnak:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* stb.

Ezzel az oszloppal ellenőrizheti, hogy a projektben minden kameratípushoz megjelölte-e a célpontokat.

***

## Célpont-felismerési beállítások

### A felismerés érzékenységének beállítása

Ha az Chloros nem ismeri fel helyesen a célpontokat, állítsa be ezeket a beállításokat a [Projektbeállítások](adjusting-project-settings.md) menüpontban:

**Minimális kalibrációs mintaterület:**

* **Alapértelmezett**: 25 pixel
* **Növelje**, ha kis tárgyaknál téves észlelések történnek.
* **Csökkentse**, ha a célpontok nem kerülnek észlelésre.

**Minimális célpont-csoportosítás:**

* **Alapértelmezett**: 60
* **Növelje**, ha a célpontok több felismerésre oszlanak
* **Csökkentse**, ha a színváltozású célpontok nem kerülnek teljes mértékben felismerésre

***

## Gyakori célkép-problémák

### Probléma: Nincs felismert célpont

**Lehetséges okok:**

* A célképek nincsenek megjelölve a Fájlkezelőben
* A célpont túl kicsi a keretben (a kép &lt; 30%-a)
* Rossz megvilágítás (árnyékok, tükröződés)
* A célfelismerési beállítások túl szigorúak

**Megoldások:**

1. Ellenőrizze, hogy a Cél oszlopban a megfelelő képek vannak-e bejelölve.
2. Ellenőrizze a célképek minőségét az előnézetben.
3. Ha a minőség rossz, készítsen új felvételeket a célokról.
4. Szükség esetén módosítsa a célfelismerési beállításokat.

### Probléma: Hamis célfelismerések

**Lehetséges okok:**

* Fehér épületek, járművek vagy talajtakaró tévesen célpontnak minősülnek
* Világos foltok a növényzetben
* Túl alacsony felismerési érzékenység

**Megoldások:**

1. Csak a tényleges célképeket jelölje meg, hogy korlátozza a felismerés hatókörét
2. Növelje a minimális kalibrációs mintaterületet
3. Növelje a minimális célcsoportosítási értéket
4. Győződjön meg arról, hogy a célképek csak a célpontot mutatják (minimális háttérzavar)

***

## Ellenőrzési ellenőrzőlista

A feldolgozás megkezdése előtt ellenőrizze a célképek kiválasztását:

* [ ] Legalább 1 célkép megjelölése munkamenetenként
* [ ] A cél oszlop jelölőnégyzetei be vannak jelölve az összes célképnél
* [ ] A célképek a felméréssel azonos időintervallumban készültek
* [ ] A célok kattintáskor jól láthatóak az előnézetben
* [ ] Mind a 4 kalibrációs panel látható az egyes célképeken
* [ ] Nincsenek árnyékok vagy akadályok a célpontokon
* [ ] Kettős kamera esetén: mindkét kameratípushoz megjelölt célpontok

***

## Célpont nélküli feldolgozás

### Kalibrációs célpontok nélküli feldolgozás

Bár tudományos munkához nem ajánlott, célpontok nélkül is elvégezhető a feldolgozás:

1. Hagyja bejelölés nélkül az összes célpont oszlop jelölőnégyzetét
2. **Kapcsolja ki** a „Reflektancia kalibrálás” opciót a Projektbeállítások menüben
3. A vignettázás korrekció továbbra is alkalmazásra kerül.
4. A kimenet nem lesz kalibrálva az abszolút reflektancia tekintetében.

{% hint style=&quot;warning&quot; %}
**Nem ajánlott**: Reflektancia-kalibrálás nélkül a pixelértékek csak a relatív fényerőt jelzik, nem pedig tudományos reflektancia-méréseket. A pontos, megismételhető eredmények érdekében használjon kalibrációs célokat.
{% endhint %}

***

## Következő lépések

Miután megjelölte a célképeket:

1. **Ellenőrizze a beállításokat** – Lásd: [A projektbeállítások módosítása](adjusting-project-settings.md)
2. **Indítsa el a feldolgozást** - Lásd: [A feldolgozás elindítása](starting-the-processing.md)
3. **Figyelje a folyamatot** - Lásd: [A feldolgozás figyelése](monitoring-the-processing.md)

A kalibrációs célokról további információkat a [Kalibrációs célok](../calibration-targets.md) című részben talál.
