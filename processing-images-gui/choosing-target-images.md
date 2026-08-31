# A célképek kiválasztása

Ha megjelöljük, mely képek tartalmaznak kalibrációs célpontokat, azzal pontosan megadjuk a Chloros számára, hol keresse azokat. Ha a „Cél” oszlopban legalább egy kép be van jelölve, a Chloros **kizárólag a bejelölt képeket** vizsgálja át — így a célpontok megjelölésével egyrészt felgyorsíthatja a feldolgozást, másrészt pedig elkerülheti, hogy a felmérési képeket tévesen célpontnak tekintsék.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## Miért érdemes megjelölni a célképeket?

### A megjelölés szabályozza a vizsgálatot

Ha a „Cél” oszlopban bejelöl bizonyos képeket:

* A Chloros kizárólag a bejelölt képeken keresi a célokat
* A célfelismerés sokkal gyorsabban befejeződik
* A felmérési képek nem okozhatnak téves célfelismeréseket

Ha **nincs** bejelölt kép, az Chloros visszatér a projektben található összes kép átvizsgálásához:

* A célpont-felismerő algoritmusok minden képen futnak
* Több száz vagy ezer képet vizsgálnak át feleslegesen
* A feldolgozás jelentősen tovább tart, különösen nagy adathalmazok esetén

{% hint style="success" %}
**Sebességnövelés**: Ha egy 500 képből álló adatkészletben 2–3 célképet jelöl meg, a célfelismerés ideje 30 perc felettiről 1 perc alá csökkenhet.
{% endhint %}

***

## Hogyan jelöljük meg a célképeket?

### 1. lépés: A célképek azonosítása

Nézze át az importált képeket a Fájlkezelőben, és azonosítsa, mely képek tartalmaznak kalibrációs célokat.

**Gyakori esetek:*** **Felvétel előtti cél**: A munkamenet megkezdése előtt rögzítve
* **Felvétel utáni cél**: A munkamenet befejezése után rögzítve
* **Helyszíni célok**: A felvételi területen elhelyezett célok
* **Több cél**: Munkamenetenként 2–3 célkép (ajánlott)

### 2. lépés: Ellenőrizze a **<img src="../.gitbook/assets/image (33).png" alt="" data-size="original">** céloszlopot

Minden olyan kép esetében, amely kalibrációs célt tartalmaz:

1. Keresse meg a képet a Fájlkezelő táblázatában
2. Keresse meg a **Target** oszlopot (a legjobbra lévő oszlop)
3. Jelölje be a kép **Target** oszlopában található jelölőnégyzetet
4. Ismételje meg a műveletet minden olyan kép esetében, amely célpontot tartalmaz

### 3. lépés: Ellenőrizze a kiválasztást

A feldolgozás előtt ellenőrizze még egyszer:

* [ ] Minden kalibrációs célpontot tartalmazó kép be van jelölve
* [ ] Nincs véletlenül bejelölt olyan kép, amely nem tartalmaz célpontot
* [ ] A célpontok jól láthatók a bejelölt képeken

***

## LATTICE: A célpontok opcionálisak, ha a DAQ rögzít

A LATTICE multispektrális kamerák esetében a képkockán belüli kalibrációs célpont **a két** lehetséges reflexiós referencia egyike:

* **Képkereten belüli célpont**: amikor egy megjelölt célpontkép megfelel Chloros minőségbiztosítási (QA) követelményeinek, a célpont a környező képek**abszolút reflexió-referenciájává** válik.
* **DAQ lefelé irányuló sugárzás**: ha nincs célpont (vagy a minőségellenőrzés sikertelen), a Chloros helyette a DAQ fényérzékelőjének lefelé irányuló besugárzási intenzitásából számítja ki a reflektanciát (ρ = π·L/E). Ha az `.daq` vagy a DAQ-M `.csv` felvétel lefedi a rögzített képeket, akkor**teljesen célképek nélkül** is kalibrált visszaverődést kap.

Ez az automatikus viselkedés az alapértelmezett. A CLI / SDK oldalon ez az `--reflectance-source auto` beállításnak felel meg; kényszerítheti az `target` (szigorú — nincs DAQ-helyettesítés) vagy az `daq` (DAQ-domináns) beállítást is. Lásd a [CLI Referenciát](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**LATTICE célgeometriák**: az Survey3-hez használt klasszikus panelérzékelés mellett a LATTICE feldolgozás támogatja az**ArUco-jelölt célokat**, a**fix ROI-célokat**és a**csíkos célokat**is, amelyek projektként konfigurálhatók. Egységenkénti**mért** célreflektancia-letapogatások adhatók meg sorozatszámmal (CLI: `--target-reflectance-dir`, cél-egységenként egy `<serial>.csv`), a névleges T3/T4P spektrumok pedig tartalékként szolgálnak.

{% hint style="info" %}
**F988 modul**: Az F988 reflektanciáját a helyszíni reflektancia-panel segítségével kalibrálják: mivel a sáv a DAQ fényérzékelő kalibrált tartományán kívül esik, ezért az Chloros a legfrissebb panelrögzítést alkalmazza, és azt a panel-megfigyelések között megőrzi. Ha egy F988 modult kizárólag DAQ-adatok alapján dolgoznak fel, az Chloros elutasítja a DAQ-alapú reflektanciát az adott sáv esetében (kihagyási ok: `dls-uncalibrated-band-988`) — a panel-alapú munkafolyamat a támogatott módszer.
{% endhint %}

***

## A célképek készítésének bevált gyakorlata

### Útmutató a célképek rögzítéséhez

**Időzítés:**

* Rögzítse a célképeket közvetlenül a rögzítési munkamenet előtt és annak során is
* Ugyanazokban a fényviszonyok között, mint a DAQ fényérzékelője
* A legjobb eredmények elérése érdekében ideális esetben a lehető leggyakrabban készítsen célképeket. Ellenkező esetben a fényérzékelő adatait használjuk a kalibráció időbeli kiigazításához.

**A kamera pozíciója:**

* Tartsa a kamerát a cél felett úgy, hogy az középen legyen, és a kép közepének körülbelül 40–60%-át kitöltse.
* Tartsa a kamerát párhuzamosan vagy merőlegesen a célfelületre

**Megvilágítás:**

* Ugyanaz a környezeti megvilágítás, mint a DAQ fényérzékelőjénél
* Kerülje az árnyékok kialakulását a célfelületeken
* Ne takarja el a fényforrást a testével, járművével vagy növényzettel
* Felhős időjárás esetén a legkonzisztensebb eredmények érhetők el

**A célállapot:**

* Tartsa a célpaneleket tisztán és szárazon
* A cél összes paneljének (pl. egy T4-es esetében mind a 4-nek) jól láthatónak és akadálymentesnek kell lennie
* Ha lehetséges, a célok legyenek merőlegesek/nadir irányban a fényforráshoz képest

### Hány célkép szükséges?

**Minimum:**1 célfelvétel munkamenetenként.**Ajánlott:** 3–5 célfelvétel munkamenetenként.**Bevált gyakorlat szerinti ütemterv:**

* 3–5 felvétel röviddel azután, hogy a fényérzékelő elindította a felvételt
* A legjobb eredmények érdekében forgassa el a kamerát a felvételek között
* Opcionális: a munkamenet közepén is rendszeresen, ha a fényviszonyok folyamatosan változnak

***

## Több kamerával való munka

### Kétkamerás felállítások

Ha két MAPIR kamerát használ egyszerre (pl.: Survey3W RGN + Survey3N OCN):

1. Rögzítse a célképeket **mindkét kamerával** egyszerre
2. Használja **ugyanazt a fizikai céltárgyat** mindkét kamerához
3. Jelölje meg a célképeket **mindkét kameratípus** számára a Fájlkezelőben
4. A Chloros minden kamera kalibrálásához a megfelelő céltárgyakat fogja használni

### Kameramodell oszlop

A **Kameramodell** oszlop segít azonosítani, hogy melyik kép melyik kamerából származik:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* stb.

Ezzel az oszloppal ellenőrizheti, hogy a projektben minden kameratípushoz megjelölte-e a célpontokat.

***

## Célpont-felismerési beállítások

### A felismerési érzékenység beállítása

Ha a Chloros nem ismeri fel megfelelően a célpontjait, állítsa be ezeket a beállításokat a [Projektbeállítások](adjusting-project-settings.md) menüpontban:**Minimális kalibrációs mintafelület (px):*** **Alapértelmezett**: 25 képpont
* **Növelje**, ha kis méretű tárgyaknál téves észlelések jelentkeznek
* **Csökkentse**, ha a célpontok nem kerülnek észlelésre**Minimális célpont-csoportosítás (0–100):*** **Alapértelmezett**: 60
* **Növelje**, ha a célpontok több észlelésre oszlanak szét
* **Csökkentse**, ha a színváltozással rendelkező célpontok nem kerülnek teljes mértékben észlelésre

{% hint style="info" %}
**A CLI-hez kapcsolódó tipp**: Az `chloros-cli process` ugyanazokat a beállítókat fogadja el (`--min-target-size`, `--target-clustering`), és az `--target`/`--targets` jelzője a teljes bemeneti mappát „csak célpanel”ként jelöli meg. Lásd a [CLI Referenciát](../reference/cli-reference.md).
{% endhint %}

***

## Gyakori célkép-problémák

### Probléma: Nem észlelt célpontok

**Lehetséges okok:**

* A célképek nincsenek megjelölve a Fájlkezelőben
* A cél túl kicsi a képkockán (a kép kevesebb mint 30%-a)
* Rossz megvilágítás (árnyékok, tükröződés)
* Túl szigorú célfelismerési beállítások

**Megoldások:**

1. Ellenőrizze, hogy a „Cél” oszlop be van-e jelölve a megfelelő képeknél
2. Ellenőrizze a célképek minőségét az előnézetben
3. Ha a minőség gyenge, rögzítse újra a célokat
4. Szükség esetén módosítsa a célfelismerési beállításokat

### Probléma: Hamis célfelismerések

**Lehetséges okok:**

* Fehér épületek, járművek vagy talajborítás, amelyeket tévesen célként azonosít a rendszer
* Világos foltok a növényzetben
* Túl alacsony felismerési érzékenység

**Megoldások:**

1. Csak a tényleges célképeket jelölje meg — csak a bejelölt képeket vizsgálja meg a rendszer
2. Növelje a minimális kalibrációs minta területét
3. Növelje a minimális célcsoportosítási értéket
4. Győződjön meg arról, hogy a célképeken csak a cél látható (minimális háttérzavar)

***

## Ellenőrzési ellenőrzőlista

A feldolgozás megkezdése előtt ellenőrizze a célkép-kiválasztást:

* [ ] Minden munkamenetben legalább 1 célkép megjelölve (vagy LATTICE esetén egy, a munkamenetet lefedő `.daq`/`.csv` felvétel)
* [ ] Az összes célképnél be van jelölve a cél oszlop jelölőnégyzete
* [ ] A célképek a felméréssel azonos időkereten belül készültek
* [ ] A célok kattintás után az előnézetben jól láthatók
* [ ] Minden célképen láthatók az összes kalibrációs panelek
* [ ] Nincsenek árnyékok vagy akadályok a célokon
* [ ] Kétkamerás rendszer esetén: A célpontok mindkét kameratípusnál meg vannak jelölve

***

## Célpont nélküli feldolgozás

### LATTICE: DAQ-felvétellel

Ha egy DAQ fényérzékelő rögzítette a lefelé irányuló sugárzási intenzitást a LATTICE-felvételek készítése közben, nincs szükség célpontra:

1. Importálja az `.daq` (vagy DAQ-M `.csv`) fájlt a képekkel együtt
2. Hagyja bejelölés nélkül a „Célpont” oszlopot
3. A visszaverődési tényezőt a DAQ lefelé irányuló sugárzási referencia alapján automatikusan kiszámítja a rendszer
4. A sugárzáshoz soha nincs szükség célpontra vagy DAQ-ra — ez kizárólag a kamera gyári radiometrikus kalibrációjából származik

### Feldolgozás referencia nélkül

Célpontok és DAQ nélkül is elvégezhető a feldolgozás:

1. Hagyja bejelölés nélkül az összes „Célpont” oszlop jelölőnégyzetét
2. **Kapcsolja ki** a „Reflektancia-kalibrálás / fehéregyensúly” opciót a Projektbeállításokban — így a célpont-felismerés teljesen kihagyásra kerül
3. A vignettakorrekció továbbra is alkalmazásra kerül
4. A kimenet nem lesz kalibrálva az abszolút reflektancia tekintetében (a LATTICE multispektrális verzió továbbra is exportálja a debayerezett, előnézeti és sugárzási termékeket)

{% hint style="warning" %}
**Survey3is tudományos munkához nem ajánlott**: Reflektancia-kalibrálás nélkül a Survey3 képpontértékei csak relatív fényerőt jelentenek, nem pedig tudományos reflektanciaméréseket. Pontos, megismételhető eredmények érdekében használjon kalibrációs célpontokat (vagy a LATTICE esetében egy DAQ fényérzékelőt).
{% endhint %}

***

## Következő lépések

Miután megjelölte a célképeit:

1. **Ellenőrizze a beállításait** – Lásd [A projekt beállításainak módosítása](adjusting-project-settings.md)
2. **Indítsa el a feldolgozást** – Lásd: [A feldolgozás elindítása](starting-the-processing.md)
3. **Kövesse nyomon a folyamatot** – Lásd: [A feldolgozás nyomon követése](monitoring-the-processing.md)

A kalibrációs célpontokról további információkat a [Kalibrációs célpontok](../calibration-targets.md) című részben talál.
