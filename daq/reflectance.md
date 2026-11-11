# Reflektancia-mérési munkafolyamatok

A DAQ fényérzékelő a radiometrikus képeket reflektanciává alakítja. Két különböző munkafolyamat létezik:

1. **Egyérzékelős** — egy DAQ méri a lefelé irányuló sugárzási intenzitást, miközben a kamera felvételt készít, és a Chloros a kamera által mért sugárzási értéket elosztja ezzel a referenciaértékkel.
2. **Kétérzékelős** — két DAQ-érzékelő, amelyek közül az egyik az eget, a másik pedig egy tárgyat figyel, kamera használata nélkül élő spektrális reflektancia-görbét állít elő.

## Egyérzékelős + kamera (lefelé irányuló referencia)

A DAQ lefelé irányuló fényérzékelőként (DLS) működik: a kamera méri a felfelé irányuló sugárzási intenzitást **L**(W/m²/sr/nm), a DAQ a lefelé irányuló besugárzási intenzitást**E** (W/m²/nm), és a Chloros a sávonkénti visszaverődést a következőképpen számítja ki:

> ρ = π · L / E

A DAQ-leolvasás időbélyege mindig **egyezik az expozíció időbélyegével** — ezért osztja meg a DAQ és a kamerák a PTP-szabvány szerinti órájukat (lásd [DAQ-E hálózat és időszinkronizálás](ethernet-ptp.md)). Külső munkákhoz illessze be a Sunshine koszinusz-korlátozót, és adja meg helyesen; a korlátozó megadása közvetlenül skálázza az E értéket (lásd [Cap Profiles & Calibrated Range](caps-and-range.md)). Kvantitatív munkák esetén vegye figyelembe a műszer jellemzőit: a kvantitatív besugárzási erősséget legalább 15 másodpercnyi mérési értékek átlagából kell kiszámítani.

### Élő rögzítés

Kapcsolja össze a DAQ-ot egy kamerával a Kamerák fülön: minden kamera beállítási paneljén található egy **Fényérzékelő** legördülő menü, amely felsorolja a Fényérzékelők fülről minden csatlakoztatott DAQ-ot (DAQ-U/M/E); szinkronizált rendszer esetén a rendszer egészére vonatkozó fényérzékelő-kiválasztás minden tagra átterjed (az egyes kamerák továbbra is felülírhatják ezt). A társítás után az érzékelő spektrumai a kamera DLS-helyére kerülnek, a reflexiós értékek exportja pedig az illesztett mérési értékkel osztva jelenik meg. Egy rendszerben a saját érzékelővel rendelkező tag a saját mérési értékét használja – az élő kompozitban szereplő sávjaihoz, az exportált adataihoz és az automatikus expozícióhoz –, míg a többi tag a rendszerét használja.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->


Két tudnivaló viselkedés:

* **Nincs DAQ-kapcsolat → a reflexancia elutasításra kerül, nem hamisítják.** Az Chloros elutasítja a reflexancia-eredményt, és rögzíti az elhagyás okát, ahelyett, hogy csendben egy alacsonyabb értéket adna vissza.
* **A felhasznált mérési érték megmarad.** Minden reflektancia-képkockához a ténylegesen alkalmazott DAQ-mérési érték `.daq` fájlként kerül mentésre a felvételek `daq/` mappájába, így a felvétel később újra feldolgozható ([Felvétel és a .daq formátum](recording.md)). A mellette található `.daq.cameras.json` fájl megnevezi az általa leírt kamerákat, és az újrafeldolgozás során a rendszer kizárólag azokhoz a kamerákhoz tartozó képekkel párosítja össze.

### A rögzített képanyag feldolgozása

A repülés utáni feldolgozáshoz rögzítsen egy `.daq` fájlt a munkamenet során, és tárolja azt a képek mellett — a feldolgozási folyamat automatikusan meghatározza az időbélyeggel egyező lefelé irányuló sugárzást, és az első használatkor letölti a hiányzó gyári kalibrációt a MAPIR felhőjéből. A GUI-felvételek automatikusan hozzáadódnak a nyitott projekthez, amikor leállnak.

A reflexiós referencia a feldolgozáskor választható – `--reflectance-source` az `chloros-cli process`-en, vagy a GUI Projektbeállítások menüpontjában található reflexiós forrás beállítás:

| Érték | Viselkedés |
| --- | --- |
| `auto` (alapértelmezett) | A minőségbiztosításon átment, a képkockán belüli kalibrációs célpont az abszolút referencia; a DAQ-lefelé irányuló sugárzás (ρ = π·L/E) a tartalék megoldás. Az a célpont, amely jelzéssel (leesett panel vagy meghibásodott sáv) felel meg a minőségbiztosításnak, nem kerül automatikusan előnyben részesítésre: a célpont, a célpont+DAQ és a kizárólag DAQ láncokat a kihagyott paneleken mért pontszámuk sorrendjében próbálják ki. |
| `daq` | DAQ-döntő |
| `target` | Szigorú, kereten belüli cél; nincs DAQ-helyettesítés |

A célfolyamatokról lásd a [Kalibrációs célok](../calibration-targets.md) részt, valamint a [LATTICE fejezetet](../lattice/README.md), valamint a [CLI-referenciát](../reference/cli-reference.md) a teljes feldolgozási folyamat megismeréséhez. Az exportált reflexiós pixelek olvasásakor használja a megjelölt skálát (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) — lásd a [Kimeneti képformátumok](../output-image-formats.md) részt.

### A DAQ kalibrált tartományán kívüli sávok

A DAQ radiometrikusan kalibrált tartománya ~374–974 nm. A Chloros elutasítja a DAQ-alapú reflektanciát minden olyan kamerasáv esetében, amelynek spektrális súlyának kevesebb mint a fele esik ebbe a tartományba, és az `dls-uncalibrated-band-<nm>` kihagyási okot jelenti. A forgalomban lévő termékek közül ez csak az F988-at érinti: az F988 reflektanciáját egy helyszíni reflektancia-panel segítségével kalibrálják; mivel a sáv a DAQ fényérzékelőjének kalibrált tartományán kívül esik, a Chloros a legutóbbi panelről készült felvételt alkalmazza, és azt a panel-megfigyelések között megőrzi. Ha egy F988 kamerát kizárólag DAQ-módban üzemeltetnek, a Chloros elutasítja az adott sáv DAQ-alapú reflexióját, az `dls-uncalibrated-band-988` kihagyási ok megadásával — a panel-alapú munkafolyamat a támogatott módszer.

## Kettős érzékelő (környezeti + tárgy)

Két DAQ-érzékelő – bármely páros, bármilyen hordozón – kamerát nem igénylő élő reflexiós spektrumot ad: az egyik érzékelő az ég felé néz (**Környezeti fényforrás**), a másik a tárgy felé (**Tárgy-szkenner**), és aChloros
hullámhosszonként kiszámítja:

> R(λ) = tárgy(λ) / környezeti(λ)

(nulla, ha a környezeti fény ≤ 0).

### A grafikus felhasználói felületen

Miután mindkét érzékelőt csatlakoztatta a Fényérzékelők fülön, nyissa meg az érzékelő-hozzáadási felületet (a rácsnézetben egy diagramlapkán található „+” gomb), és válassza a **Környezeti + tárgy kombinálása**lehetőséget. Válassza ki a két érzékelőt a**Környezeti fényforrás**és az**Objektum-szkenner**legördülő menükből, majd kattintson a**Létrehozás**gombra. A csoport saját diagramként jelenik meg, valamint egy zöld**REF** jelöléssel ellátott oldalsáv-sorban.



<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->
A reflexiós diagram (listanézet) alatt egy élő **vegetációs index táblázat** számítja ki az indexeket a görbéből, a kék 450 / zöld 550 / vörös 670 / NIR 800 nm sávközpontok felhasználásával. Az abszolút skálát semlegesítő arányalapú indexek (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) mindig megjelennek; az abszolút reflektanciát igénylő indexek (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) csak akkor jelennek meg, ha mindkét érzékelő teljesítménykalibrált modell.

### Látszólagos vs. relatív — a címkézési szabály A

Chloros a kettős érzékelős kimenetet annak alapján címkézi, amit az érzékelőpár ténylegesen igényelhet:

| Érzékelőpár | Címke |
| --- | --- |
| Mindkét érzékelő kalibrálva — gyári csomag betöltve | **Látszólagos reflexió** |
| Bármelyik érzékelő nem kalibrálva | **Relatív reflektancia** |

Mindhárom modell radiometrikus: amint egy érzékelő gyári kalibrációs csomagja betöltődik, spektrumai abszolút W/m²/nm értékeket mutatnak, így a kalibrált érzékelőpár aránya egy abszolút látszólagos reflektanciát ad — ezt nem a transzport határozza meg. Ha egy érzékelő továbbra is nyers számlálási adatokat továbbít (nincs elérhető csomag), az eredmény relatív görbévé alacsonyodik (a spektrális alakzat továbbra is érvényes). Mindkét érzékelőnek helyesen deklarált korlátprofilokkal kell rendelkeznie ([Korlátprofilok és kalibrált tartomány](caps-and-range.md)).

### Python-ből

A közös SDK felületen nincs külön kétérzékelős hívás: nyisson meg két munkamenetet az `chloros_sdk.connect_daq_sensor()`-szel, és saját maga számítsa ki az `latest()` spektrumok arányát, ugyanazt a címkézési konvenciót alkalmazva. (Kétérzékelős rögzítő eszköz létezik a MAPIR belső, közvetlen hardveres felületén is, amelyet a teljesség kedvéért felsorolunk a [CLI-referenciában](../reference/cli-reference.md) — ez nem része a szállított CLI-nek; a fenti GUI-munkafolyamat a támogatott élő útvonal.)
