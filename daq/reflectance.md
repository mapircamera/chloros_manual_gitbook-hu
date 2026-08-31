# Reflektancia-feldolgozási folyamatok

A DAQ fényérzékelő a radiometrikus képeket reflektanciává alakítja. Két különböző feldolgozási folyamat létezik:

1. **Egyérzékelős** — egy DAQ méri a lefelé irányuló sugárzási intenzitást, miközben egy kamera rögzíti a képet, és az Chloros a kamera által mért sugárzási értéket elosztja ezzel a referenciaértékkel.
2. **Kétérzékelős** — két DAQ-érzékelő, amelyek közül az egyik az eget, a másik pedig egy objektumot figyel, kamerát nem igénylő, valós idejű spektrális reflexiós görbét állít elő.

## Egyérzékelős + kamera (lefelé irányuló referencia)

A DAQ lefelé irányuló fényérzékelőként (DLS) működik: a kamera méri a felfelé irányuló sugárzási intenzitást **L**(W/m²/sr/nm), a DAQ méri a lefelé irányuló sugárzási intenzitást**E** (W/m²/nm), és az Chloros a következőképpen számítja ki a sávonkénti visszaverődést:

> ρ = π · L / E

A DAQ-leolvasás időbélyege mindig **egyezik az expozíció időbélyegével** — ezért a DAQ és a kamerák egy PTP-szabályozott órát használnak (lásd [DAQ-E hálózat és időszinkronizálás](ethernet-ptp.md)). Külső munkavégzéshez illessze fel a Sunshine koszinusz sapkát, és adja meg helyesen; a sapka megadása közvetlenül skálázza az E értéket (lásd [Sapkaprofilok és kalibrált tartomány](caps-and-range.md)). Kvantitatív munkavégzés esetén vegye figyelembe a műszer jellemzőit: a kvantitatív besugárzási erősséget legalább 15 másodpercnyi mérési értékek átlagából kell kiszámítani.

### Élő rögzítés

Kössük össze a DAQ-ot egy kamerával a Kamerák fülön: minden kamera beállítási paneljén található egy **Fényérzékelő** legördülő menü, amely a Fényérzékelők fülről felsorolja az összes csatlakoztatott DAQ-ot (DAQ-U/M/E); szinkronizált rendszer esetén a rendszer egészére vonatkozó fényérzékelő-kiválasztás minden tagra átterjed (az egyes kamerák továbbra is felülírhatják ezt). A társítás után az érzékelő spektrumai a kamera DLS-helyére kerülnek, a reflexiós értékek pedig az illesztett mérési értékkel osztva kerülnek exportálásra.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->

Két tudnivaló viselkedés:

* **Nincs DAQ társítva → a reflexió elutasításra kerül, nem hamisítják meg.** Az Chloros elutasítja a reflexiós eredményt, és rögzíti az elhagyás okát, ahelyett, hogy csendben egy alacsonyabb értéket adna vissza.
* **A felhasznált mérési érték megmarad.** Minden reflektancia-képkockához a ténylegesen alkalmazott DAQ-érték `.daq` kiegészítő fájlként kerül a kép mellé, így a felvétel később újra feldolgozható ([Felvétel és a .daq formátum](recording.md)).

### A rögzített képek feldolgozása

A repülés utáni feldolgozáshoz rögzítsen egy `.daq` fájlt a munkamenet során, és tárolja azt a képek mellett — a feldolgozási folyamat automatikusan meghatározza az időbélyeggel egyező lefelé irányuló sugárzást, és az első használatkor letölti az esetleg hiányzó gyári kalibrációt az MAPIR felhőjéből. A GUI-felvételek a leállás után automatikusan hozzáadódnak a megnyitott projekthez.

A reflexiós referencia a feldolgozáskor választható – az `--reflectance-source` az `chloros-cli process`-en, vagy a GUI Projektbeállítások menüpontjában található reflexiós forrás beállítás:

| Érték | Viselkedés |
| --- | --- |
| `auto` (alapértelmezett) | A minőségbiztosításon átesett, a képkockán belüli kalibrációs célpont az abszolút referencia; a DAQ lefelé irányuló sugárzás (ρ = π·L/E) a tartalék |
| `daq` | DAQ-alapú |
| `target` | Szigorú, a keretben lévő cél; nincs DAQ-helyettesítés |

A célfolyamatokról lásd a [Kalibrációs célpontok](../calibration-targets.md) részt, valamint a [LATTICE fejezetet](../lattice/README.md) fejezetet, valamint a [CLI Referencia](../reference/cli-reference.md) című részt a teljes feldolgozási folyamat ismertetése érdekében. Az exportált reflexiós pixelek olvasásakor használja a megadott skálát (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) – lásd a [Kimeneti képformátumok](../output-image-formats.md) részt.

### A DAQ kalibrált tartományán kívüli sávok

A DAQ radiometrikusan kalibrált tartománya ~374–974 nm. Az Chloros elutasítja a DAQ-alapú reflexiót minden olyan kamerasáv esetében, amelynek spektrális súlyának kevesebb mint a fele esik ebbe a tartományba, és az `dls-uncalibrated-band-<nm>` kihagyási okot jelenti. A forgalomban lévő SKU-k közül ez csak az F988-at érinti: az F988 reflektanciáját egy helyszíni reflektancia-panel segítségével kalibrálják; mivel a sáv a DAQ fényérzékelőjének kalibrált tartományán kívül esik, az Chloros a legutóbbi panelrögzítést alkalmazza, és azt a panel-megfigyelések között megőrzi. Ha egy F988 kamerát kizárólag DAQ-módban üzemeltetnek, az Chloros elutasítja a DAQ-alapú reflektanciát az adott sávban, az `dls-uncalibrated-band-988` kihagyási okkal – a panel-munkafolyamat a támogatott módszer.

## Kettős érzékelő (környezeti + tárgy)

Két DAQ-érzékelő – bármilyen párosításban, bármilyen hordozón – kamerát nem igénylő élő reflexiós spektrumot ad: az egyik érzékelő az ég felé néz (**Környezeti fényforrás**), a másik a tárgy felé (**Tárgy-szkenner**), és az Chloros hullámhosszonként kiszámítja:

> R(λ) = tárgy(λ) / környezeti(λ)

(nulla, ha a környezeti fény ≤ 0).

### A grafikus felhasználói felületen

Miután mindkét érzékelőt csatlakoztatta a Light Sensors (Fényérzékelők) fülön, nyissa meg az érzékelő-hozzáadási felületet (a rácsnézetben egy diagramlapkán található „+” gomb), és válassza a **Combine Ambient + Object**(Környezeti fény + tárgy kombinálása) lehetőséget. Válassza ki a két érzékelőt a „Környezeti fényforrás” és az „Objektum-szkenner” legördülő menükből, majd kattintson a „Létrehozás” gombra. A csoport önálló diagramként és egy zöld**REF** jelöléssel ellátott oldalsáv-sorban jelenik meg.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->

A reflexiós diagram (listanézet) alatt egy élő **vegetációs index táblázat** számítja ki az indexeket a görbéből a kék 450 / zöld 550 / vörös 670 / NIR 800 nm sávközpontok felhasználásával. Az abszolút skálát semlegesítő arányalapú indexek (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) mindig megjelennek; azok az indexek, amelyekhez abszolút reflexió szükséges (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) csak akkor jelennek meg, ha mindkét érzékelő teljesítménykalibrált modell.

### Látszólagos vs. relatív – a címkézési szabály

Az Chloros a kettős érzékelős kimenetet annak megfelelően címkézi, amit az érzékelőpár ténylegesen képes biztosítani:

| Érzékelőpár | Címke |
| --- | --- |
| Mindkét érzékelő kalibrálva — gyári csomag betöltve | **Látszólagos visszaverődés** |
| Bármelyik érzékelő nem kalibrálva | **Relatív reflexió** |

Mindhárom modell radiometrikus: amint egy érzékelő gyári kalibrációs csomagja betöltődik, spektrumai abszolút W/m²/nm értékűek lesznek, így a kalibrált érzékelőpár aránya egy abszolút látszólagos reflexiót ad – ezt nem a transzport határozza meg. Ha egy érzékelő továbbra is nyers számlálási adatokat továbbít (nincs elérhető csomag), az eredmény relatív görbévé alacsonyodik (a spektrális alakzat továbbra is érvényes). Mindkét érzékelőnek helyesen megadott korlátprofilokkal kell rendelkeznie ([Korlátprofilok és kalibrált tartomány](caps-and-range.md)).

### Az Python-ből

Az összevont SDK felületen nincs külön kettős érzékelős hívás: nyisson meg két munkamenetet az `chloros_sdk.connect_daq_sensor()`-szel, és saját maga állapítsa meg az `latest()` spektrumok arányát, ugyanazt a címkézési konvenciót alkalmazva. (Kétérzékelős rögzítő eszköz létezik az MAPIR belső, közvetlen hardveres felületén is, amelyet a [CLI Referencia](../reference/cli-reference.md) című dokumentumban a teljesség kedvéért felsorolják – ez nem része a szállított CLI-nek; a fenti grafikus felhasználói felületen bemutatott munkafolyamat a támogatott élő eljárás.)
