# A célképek kiválasztása

A kalibrációs célokat tartalmazó képek megjelölése egy kulcsfontosságú lépés, amely jelentősen felgyorsítja az Chloros feldolgozási folyamatot. A célképek előzetes kiválasztásával elkerülhető, hogy az Chloros az adatkészlet minden egyes képét átvizsgálja a kalibrációs célok felkutatására.

## Miért érdemes megjelölni a célképeket?

### Feldolgozási sebesség

A célképek megjelölése nélkül az Chloros-nek:

* át kell vizsgálnia a projekt minden egyes képét
* célfelismerő algoritmusokat kell futtatnia minden képen
* szükségtelenül több száz vagy ezer képet kell ellenőriznie

**Eredmény**: A feldolgozás jelentősen hosszabb ideig tarthat, különösen nagy adatkészletek esetén.

### Megjelölt célképek esetén

Ha megjelöli a Cél oszlopot bizonyos képeknél:

* Az Chloros csak a bejelölt képeken keresi a célokat
* A célfelismerés sokkal gyorsabban befejeződik
* Az összes feldolgozási idő jelentősen csökken

{% hint style="success" %}
**Sebességnövekedés**: Ha egy 500 képet tartalmazó adatkészletben 2-3 célképet jelöl meg, a célfelismerés ideje 30+ percről 1 perc alá csökkenhet.
{% endhint %}

***

## Hogyan jelölje meg a célképeket

### 1. lépés: Azonosítsa a célképeit

Nézze át az importált képeket a Fájlböngészőben, és azonosítsa, mely képek tartalmaznak kalibrációs célokat.

**Gyakori esetek:*** **Felvétel előtti cél**: A munkamenet megkezdése előtt rögzítve
* **Felvétel utáni cél**: A munkamenet befejezése után rögzítve
* **Terepi célok**: A rögzítési területen elhelyezett célok
* **Több cél**: 2-3 célkép munkamenetenként (ajánlott)

### 2. lépés: Ellenőrizze a Cél oszlopot

Minden kalibrációs célt tartalmazó kép esetében:

1. Keresse meg a képet a Fájlkezelő táblázatában
2. Keresse meg a **Cél** oszlopot (a legjobbra lévő oszlop)
3. Jelölje be a Cél oszlopban az adott kép melletti jelölőnégyzetet
4. Ismételje meg a műveletet minden célt tartalmazó kép esetében

### 3. lépés: Ellenőrizze a kiválasztást

A feldolgozás előtt ellenőrizze még egyszer:

* [ ] Minden kalibrációs célpontot tartalmazó kép be van jelölve
* [ ] Nincs véletlenül bejelölve olyan kép, amely nem tartalmaz célpontot
* [ ] A célpontok jól láthatóak a bejelölt képeken

***

## A célképekkel kapcsolatos bevált gyakorlatok

### Útmutató a célképek rögzítéséhez

**Időzítés:**

* Rögzítse a célképeket közvetlenül a felvételi munkamenet előtt és annak során
* A DAQ fényérzékelőjével megegyező fényviszonyok mellett
* A legjobb eredmények elérése érdekében ideális esetben rögzítsen célképeket a lehető leggyakrabban. Ellenkező esetben a fényérzékelő adatait használjuk a kalibráció időbeli beállításához.

**A kamera pozíciója:**

* Tartsa a kamerát a cél felett úgy, hogy az középen legyen, és a kép közepének körülbelül 40–60%-át kitöltse.
* Tartsa a kamerát párhuzamosan/merőlegesen a cél felületéhez

**Megvilágítás:**

* Ugyanaz a környezeti megvilágítás, mint a DAQ fényérzékelőjénél
* Kerülje az árnyékokat a célfelületeken
* Ne takarja el a fényforrást a testével, járművével vagy növényzettel
* Felhős időjárás biztosítja a legkonzisztensebb eredményeket

**A cél állapota:**

* Tartsa a célpaneleket tisztán és szárazon
* Mind a 4 panelnek jól láthatónak és akadálymentesnek kell lennie
* A célokat lehetőség szerint merőlegesen/nadir irányban helyezze el a fényforráshoz képest

### Hány célkép szükséges?

**Minimum:**1 célfelvétel munkamenetenként.**Ajánlott:** 3–5 célfelvétel munkamenetenként.**A legjobb gyakorlat szerinti ütemezés:**

* 3–5 kép rögzítése röviddel azután, hogy a fényérzékelő elindította a felvételt
* A legjobb eredmények érdekében forgassa el a kamerát a felvételek között
* Opcionális: időnként a munkamenet közepén, ha a fényviszonyok folyamatosan változnak

***

## Több kamerával való munka

### Kétkamerás felállítások

Ha két MAPIR kamerát használ egyszerre (pl. Survey3W RGN + Survey3N OCN):

1. Rögzítse a célképeket **mindkét kamerával** egyszerre
2. Használja **ugyanazt a fizikai célt** mindkét kamerához
3. Jelölje meg a célképeket **mindkét kameratípus** számára a Fájlkezelőben
4. Az Chloros minden kamera kalibrálásához a megfelelő célokat fogja használni

### Kamera modell oszlop

A **Kamera modell** oszlop segít azonosítani, hogy melyik kép melyik kamerából származik:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* stb.

Használja ezt az oszlopot annak ellenőrzésére, hogy a projektben minden kameratípushoz megjelölte-e a célpontokat.

***

## Célpont-felismerési beállítások

### A felismerési érzékenység beállítása

Ha az Chloros nem ismeri fel helyesen a célpontokat, állítsa be ezeket a beállításokat a [Projektbeállítások](adjusting-project-settings.md) menüpontban:**Minimális kalibrációs minta terület:*** **Alapértelmezett**: 25 pixel
* **Növelje**, ha kis méretű tárgyaknál téves észlelések történnek
* **Csökkentse**, ha a célpontok nem kerülnek észlelésre**Minimális célpont-csoportosítás:*** **Alapértelmezett**: 60
* **Növelje**, ha a célpontok több észlelésre oszlanak
* **Csökkentse**, ha a színváltozással rendelkező célpontok nem kerülnek teljes mértékben felismerésre***

## Gyakori célpontkép-problémák

### Probléma: Nincs felismert célpont

**Lehetséges okok:**

* A célpontképek nincsenek megjelölve a Fájlkezelőben
* A célpont túl kicsi a képen (&lt; a kép 30%-a)
* Rossz megvilágítás (árnyékok, tükröződés)
* Túl szigorú célpont-felismerési beállítások

**Megoldások:**

1. Ellenőrizze, hogy a Cél oszlop be van-e jelölve a megfelelő képeknél
2. Ellenőrizze a célképek minőségét az előnézetben
3. Ha a minőség gyenge, rögzítse újra a célokat
4. Szükség esetén módosítsa a célfelismerési beállításokat

### Probléma: Hamis célfelismerések

**Lehetséges okok:**

* Fehér épületek, járművek vagy talajtakaró tévesen célként azonosítva
* Világos foltok a növényzetben
* Túl alacsony felismerési érzékenység

**Megoldások:**

1. Csak a tényleges célképeket jelölje meg az észlelési tartomány korlátozása érdekében
2. Növelje a minimális kalibrációs minta területét
3. Növelje a minimális célcsoportosítási értéket
4. Győződjön meg arról, hogy a célképeken csak a cél látható (minimális háttérzavar)

***

## Ellenőrzési ellenőrzőlista

A feldolgozás megkezdése előtt ellenőrizze a célképek kiválasztását:

* [ ] Legalább 1 célkép megjelölve munkamenetenként
* [ ] Az összes célképnél be van jelölve a Cél oszlop jelölőnégyzete
* [ ] A célképek a felméréssel azonos időkereten belül készültek
* [ ] A célok kattintáskor jól láthatók az előnézetben
* [ ] Mind a 4 kalibrációs panel látható minden célképen
* [ ] Nincsenek árnyékok vagy akadályok a célokon
* [ ] Kétkamerás rendszer esetén: A célok mindkét kameratípusnál meg vannak jelölve

***

## Cél nélküli feldolgozás

### Feldolgozás kalibrációs célpontok nélkül

Bár tudományos munkához nem ajánlott, célpontok nélkül is feldolgozhatja az adatokat:

1. Hagyja bejelölés nélkül az összes Cél oszlop jelölőnégyzetét
2. **Kapcsolja ki** a „Reflektancia kalibráció” opciót a Projektbeállításokban
3. A vignettakorrekció továbbra is alkalmazásra kerül
4. A kimenet nem lesz kalibrálva az abszolút reflektancia tekintetében

{% hint style="warning" %}
**Nem ajánlott**: Reflektancia-kalibrálás nélkül a képpontértékek csak relatív fényerőt jelentenek, nem pedig tudományos reflektanciaméréseket. Pontos, megismételhető eredmények érdekében használjon kalibrációs célpontokat.
{% endhint %}

***

## Következő lépések

Miután megjelölte a célképeket:

1. **Ellenőrizze a beállításokat** – Lásd: [A projekt beállításainak módosítása](adjusting-project-settings.md)
2. **Indítsa el a feldolgozást** – Lásd: [A feldolgozás elindítása](starting-the-processing.md)
3. **Kövesse nyomon a folyamatot** – Lásd: [A feldolgozás nyomon követése](monitoring-the-processing.md)

A kalibrációs célpontokról további információkat a [Kalibrációs célpontok](../calibration-targets.md) című részben talál.
