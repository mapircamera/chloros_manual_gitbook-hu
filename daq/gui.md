# A DAQ fül az Chloros programban

A DAQ fül — amely az Chloros oldalsávon **Fényérzékelők** néven szerepel — a [DAQ-U, DAQ-M és DAQ-E fényérzékelők](README.md) élő vezérlőfelülete: bármilyen adatátviteli csatornán keresztül csatlakoztathatja az érzékelőket, valós időben figyelheti a kalibrált spektrumokat, kiszámíthatja az érzékelőpár aktuális fényvisszaverődését, és közvetlenül a projektjébe rögzítheti az `.daq` fájlokat.

A fül akkor válik elérhetővé, amikor az Chloros háttérszolgáltatás elindulása befejeződött. A fül diagramjait az Chloros DAQ-szolgáltatása táplálja egy élő kapcsolaton keresztül, amely megszakadás esetén automatikusan újra csatlakozik (2–10 másodperces várakozási idő); amíg a szolgáltatás nem érhető el, az érzékelő Állapot sorában a **Nincs szerver** felirat jelenik meg.

Az elrendezés egy **érzékelő-oldalsáv**(egy sor minden csatlakoztatott érzékelőhöz) és egy**diagramterület** (egy diagrammező minden érzékelőhöz vagy csoporthoz).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Érzékelő csatlakoztatása

Kattintson az oldalsáv tetején található **Érzékelő csatlakoztatása** gombra. A csatlakozási párbeszédablak megnyílik a főterületen (vagy átfedésként, ha újabb érzékelőt ad hozzá – ebben az esetben megjelenik egy Mégse gomb).

| Vezérlő | Viselkedés |
| --- | --- |
| **Eszköztípus** | `DAQ-U (USB)` (alapértelmezett), `DAQ-M (Bluetooth)` vagy `DAQ-E (Ethernet)`. A váltás újraindítja a frissen kiválasztott átviteli mód keresését. |
| **Port / BLE-eszköz / Gazdagépnév / IP** | A felismert eszközöket `device - description` néven sorolja fel; az érzékelőként felismert első bejegyzés automatikusan kiválasztásra kerül. A keresés során a következőket jeleníti meg: `Scanning...` (USB), `Scanning (N)...` 8 másodperces visszaszámlálással (BLE) vagy `Discovering ethernet sensors (N)...` 5 másodperces visszaszámlálással (Ethernet). Üres eredmények esetén a következőket jeleníti meg: `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Frissítés** | Azonnal újra beolvassa a kiválasztott átviteli módot (BLE/Ethernet-beolvasás közben letiltva). |
| **Csatlakozás** | A készülék kiválasztása után engedélyezhető; a kapcsolat létrehozása közben a címke `Connecting...`-re változik. |

A felderítés csak **amíg a csatlakozási párbeszédablak a képernyőn látható**, és 15 másodpercenként ismétlődik kizárólag a kiválasztott átviteli módra vonatkozóan — a fül megnyitása önmagában nem indítja el a szkennelést. Hiba esetén a párbeszédablak a következő üzenetet jeleníti meg: *„A csatlakozás nem sikerült. Próbálja meg kihúzni, majd újra bedugni az érzékelőt, majd kattintson újra a Csatlakozás gombra.”*

Az oldalsáv automatikusan megnyílik, amikor az első érzékelő csatlakozik.

{% hint style="info" %}
**A DAQ-E nem jelenik meg?** A DAQ-E-nek nincs állapot-LED-je — ellenőrizze a PoE/kapcsolatjelzőt azon a kapcsolón vagy injektorporton, amelyhez csatlakoztatva van, és a bekapcsolás után várjon néhány másodpercet, amíg elindul. Az Chloros készüléknek ugyanazon a broadcast-tartományon kell lennie (az mDNS nem halad át a routereken). Az Windows-en fogadja el a Defender tűzfal felugró ablakát, amikor az Chloros először köti össze a multicast-socketeit (mDNS UDP 5353, DAQ-E adat UDP 5002, PTP UDP 319/320). Egy LAN-on található két DAQ-E egységet a rendszer külön-külön észleli, mindkettőt a saját `daq-e-<id>.local` gazdagépnév alatt.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Az „Eszköztípus” menüpont a következőket kínálja: DAQ-U (USB), DAQ-M (Bluetooth) és DAQ-E (Ethernet)</figcaption></figure>***

## Az érzékelők oldalsávja

Minden csatlakoztatott érzékelő egy sort kap (plusz egy sort minden Ambient+Object csoportonként). A sorok húzással átrendezhetők, és sorrendjük a diagramlapok sorrendjét is megváltoztatja. Kattintson egy sorra, hogy az adott érzékelő/csoport legyen az aktív diagram a listanézetben.

| Elem | Jelentés |
| --- | --- |
| Színes bal oldali szegély | Az érzékelő grafikonjának színe. |
| Transzport jelvény | `DAQ-U` / `DAQ-M` / `DAQ-E`, vagy zöld `REF` jelvény az Ambient+Object visszaverődési csoport esetében. |
| Eszköz neve | Alapértelmezés szerint az érzékelő sorozatszáma (a kalibráláshoz, az `.daq` fájlnevekhez és az importáláshoz szükséges állandó azonosítója); az egyedi nevek projektenként megmaradnak. |
| **Kalibrálva** jelölő (zöld) | Akkor jelenik meg, amikor az érzékelő gyári kalibrációs csomagja betöltődött, azaz a spektrumok valódi W/m²/nm értékeket mutatnak. |
| **Frissítés elérhető** jelölő (borostyánszínű, csak DAQ-E esetén) | A futó firmware régebbi, mint az ehhez az Chloros buildhez mellékelt kép. A frissítés során valós idejű haladást jelenít meg (`Flashing… N%`, `Restarting sensor…`, majd `Updated X → Y` vagy `Failed`). |
| Szem | Be- és kikapcsolja az érzékelő láthatóságát a diagramon. |
| Fogaskerék | Megnyitja az érzékelőnkénti beállítások modált (lent). |
| ✕ (piros) | Leválasztja az érzékelőt, vagy eltávolít egy Ambient+Object csoportot. |

A sorok felett két gomb található:

* **Érzékelő csatlakoztatása** — megnyitja a csatlakozási párbeszédablakot (foglalt állapotban `Connecting...`-re változik a neve).
* **Minden rögzítése / Minden leállítása**— elindítja vagy leállítja a**minden**csatlakoztatott érzékelőn történő `.daq` felvételt. Ehhez legalább egy érzékelőre**és egy megnyitott projektre** van szükség (eszköztipp: „Nyisson meg egy projektet a rögzítéshez”); a gomb pirosra vált, amíg bármelyik rögzítés fut.

Üres állapotban a „Nincs csatlakoztatott érzékelő” felirat látható.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Érzékelőnkénti beállítások (a fogaskerék-ablak)

Az érzékelősoron található fogaskerék-ikonnal nyitható meg. A tartalom sorrendben:

* **Információs sorok** — Eszköztípus (DAQ-U/M/E), Csatlakozás (`Serial (USB)` / `Bluetooth` / `Ethernet`), port (COM-port, BLE-cím vagy gazdagép) és soros port.
* **Kalibrációs jelentés: Letöltés** — letölti az egység NIST-nyomon követhető kalibrációs tanúsítványát (PDF) és megnyitja azt a PDF-megjelenítőben. Akkor érhető el, ha a sorozatszám ismert; a tanúsítványt az első csatlakozáskor tárolja a rendszer.
* **Eszköznév** — kattintson a ceruzára az átnevezéshez; projektként marad meg.
* **Grafikonvonal színe** — színminta; projektként marad meg.
* **Integrációs idő (ms)**— csúszka + szám,**1–500 ms**, alapértelmezett**32 ms**. Az AE bekapcsolt állapotában le van tiltva.
* **Képkockák átlaga**— csúszka + szám,**1–50 képkocka**, alapértelmezett**20**.
* **AE: BE/KI**— automatikus expozíció kapcsoló;**alapértelmezés szerint BE** a csatlakozáskor. Kapcsolja ki az integrációs idő kézi beállításához.
* **Streaming leállítása / Streaming elindítása** — az élő közvetítés szüneteltetése vagy folytatás.
* **Felvétel / Felvétel leállítása** — érzékelőnkénti `.daq` felvétel (nyitott projekt szükséges).
* **Cap** — a cap-korrekciós profil (következő szakasz).
* **Élő információs sorok** — Integrációs idő (ms), FPS, Minták, Felvétel (piros `REC` vagy `Off`) és Állapot (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Csak DAQ-E: hálózati, firmware- és PTP-sorok

* **Hostname / IP** — a készülék aktuális címe.
* **Firmware**— az aktuális firmware-verzió, valamint egy műveleti mező: egy**Update to \<version\>

** gomb jelenik meg, ha ez az Chloros verzió egy újabb DAQ-E firmware-képet tartalmaz. A frissítés körülbelül 30 másodperc alatt történik a hálózaton keresztül; az érzékelő automatikusan újraindul és újra csatlakozik, és egy megszakadt átvitel esetén az aktuális firmware változatlan marad. A folyamat élőben követhető (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), és a mezőben az `Up to date` érték jelenik meg, ha az aktuális.
* **PTP szinkronizálás** — az élő PTP állapot (visszatér az `unknown` értékre). A DAQ-E v1.2.0+ firmware kizárólag szolgaóraként vesz részt az IEEE 1588 PTPv2-ben; az Chloros gazdagép háttérrendszere a PTP grandmaster, és minden LAN-on lévő DAQ-E és LATTICE kamera a 0-s tartományban szolgaállapotban van hozzá, az időbélyegek eltérése nagyjából 1 ms-on belül marad.

Egy „Ambient+Object” csoport esetében a felszerelés mód ablak csak a csoport forrásérzékelőit, az eszköz nevét és a grafikon vonalának színét jeleníti meg.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->

### Fedél kiválasztása

A **Fedél** legördülő menü megadja az Chloros-nek, hogy melyik fizikai fedél van felszerelve az érzékelő diffúzorára, és az adott fedél gyárilag mért korrekciós profilját alkalmazza minden spektrumra. A választási lehetőségek a modelltől függenek:

| Modell | Fedélválasztási lehetőségek |
| --- | --- |
| DAQ-U | Nincs (csupasz érzékelő), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (koszinusz-korrektor) |
| DAQ-M | Nincs (csupasz érzékelő), Sunshine (koszinusz-korrektor) |
| DAQ-E | Nincs (csupasz érzékelő), FOV 15°, FOV 45°, FOV 90°, Sunshine (koszinusz-korrektor) |

**Minden modell alapértelmezett beállítása a Napfény (koszinusz-korrektor)** — Az MAPIR minden DAQ-ot a Sunshine sapkával felszerelve szállít, és ez a szabványos kültéri konfiguráció: 180°-os félgömb alakú látómező, ahol a koszinusz-hiba 60°-ig ≤ ±4 %, 70°-ig pedig ≤ ±4,5 % (kb. 15°-os napmagasság alatt nem ajánlott), tervezésből adódóan csillapított (~12×). A választás a projektben megmarad.

{% hint style="warning" %}
**A sapka kiválasztásának meg kell egyeznie a fizikai sapkával.**Sem az érzékelő, sem a szoftver nem képes felismerni, hogy melyik sapka van felszerelve. A kiválasztás hatással van mind az élő korrekcióra, mind az minden `.daq` fájlba beírt jelölésre — a Sunshine kupak ~12×-os csillapításával egy be nem jelentett kupakcseréje nagyjából ezzel a tényezővel tévesen korrigálja a spektrumokat. (Ugyanazon fedél eltávolítása és visszaszerelése körülbelül 1,5 %-os ismétlődést eredményez.) Csak akkor válassza a**None (csupasz érzékelő)** lehetőséget, ha a fedelet fizikailag eltávolította; a DAQ-E esetében a „None” beállítás még mindig a gyári geometriai profilt alkalmazza a süllyesztett üvegdiffúzorra – ez nem egy „no-op” művelet –, és a csupasz DAQ-E egy laboratóriumi konfiguráció, nem pedig támogatott terepi konfiguráció.
{% endhint %}

{% hint style="info" %}
Frissítés egy korábbi kézikönyvből: a 1.1.0-s verzióban szereplő, böngészőoldali „Sunshine Diffuser Installed” kapcsoló eltűnt. A fedél kezelése mostantól az érzékelőnkénti fedélprofilon alapul, amelyet szerveroldalon alkalmaznak.
{% endhint %}

***

## A diagramterület

Egy rögzített felső sáv tartalmaz egy **lista ⇄ rács nézet közötti váltót**és egy**Diagram nagyítás** csúszkát (lapka mérete 200–2000 px). A nézet automatikusan rácsnézetre vált, ha több diagramcsoport létezik, és vissza listanézetre, ha egy vagy kevesebb van. A nézetmód és a diagramméret projektenként megmarad.

Az egyes érzékelők **spektrumdiagramja** a következőket mutatja:

* **X-tengely** — Hullámhossz (nm). Az érzékelő rácsa 340–1010 nm tartományban 5 nm-es lépésekben (135 pont) van felosztva, a megjelenítéshez 1 nm-re interpolálva.
* **Y-tengely** — Teljesítmény (W/m²), a csúcsérték alapján automatikusan kiválasztott SI-előtaggal (m/µ/n). A spektrumok radiometrikusan kalibrált spektrális besugárzási erősséget (W/m²/nm) jelentenek mindhárom adatátviteli mód esetében.
* Egyetlen görbe alatt szivárványos spektrális kitöltés látható; egy grafikonon több érzékelő színes vonalakként jelenik meg, tompított kitöltéssel.
* **Egérmutatóval áthúzás**— függőleges kurzor a hullámhosszal és az érzékelőnkénti értékkel;**húzás** a nagyításhoz (nagyításkor megjelenik egy kicsinyítő gomb).
* Egy **+** gomb (csak rácsnézetben) érzékelő hozzáadásához ehhez a diagramhoz vagy csoport létrehozásához (lent).
* Az eszköz neve középen, felül, és egy forgó ikon, amíg meg nem érkezik az első képkocka.

A **telítettség** nem jelenik meg magán a grafikonon: a telített érzékelő piros `SATURATED` állapotszöveget és egy piros `Saturated: Yes` sort jelenít meg az élő adattáblázatban. Az integrációs idő csökkentésével vagy az AE újraengedélyezésével törölhető.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->

***

## Élő adattábla (lista nézet)

A diagram alatt, lista nézetben, 500 ms-enként frissül:

* **Minden modell**: Fény színminta (sRGB a CIE XYZ-ből), Telített (Igen/Nem), CIE 1931 X/Y/Z, Színkoordináták x/y, CIE u′/v′, CCT (K), CRI (Ra), Domináns hullámhossz (nm), csúcshullámhossz (nm), gerjesztési tisztaság, Duv, CIE L\*/a\*/b\* és Munsell H/V/C.
* **Kizárólag kalibrált érzékelők**(bármelyik DAQ-U / DAQ-M / DAQ-E, miután betöltötték a gyári kalibrációs csomagot – ezt a zöld**Kalibrált** jelölés jelzi az érzékelő sorában): Teljes teljesítmény (W/m²), fotopikus lux (lx), skotopikus lux (lx), S/P arány, PPFD és PPFD Red/Green/Blue (µmol/m²/s), valamint az opikus sugárzási intenzitások — S-kúp, melanopikus, rodopikus, M-kúp, L-kúp (mind W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Fényvisszaverődési csoportok (Környezeti + Objektum)

Két összekapcsolt érzékelő kombinálható egy élő fényvisszaverődési kijelzővé — kamera használata nélkül:

1. Rácsnézetben kattintson a **+**gombra egy diagramlapkán, majd válassza a**Környezeti + Objektum összevonása** lehetőséget.
2. Válasszon ki egy **Környezeti fényforrás**érzékelőt és egy**Objektum-szkennert**(két különböző érzékelőt), majd kattintson a**Létrehozás** gombra.

Az Chloros a két élő adatfolyamból hullámhosszonként kiszámítja az R(λ) = objektum(λ) / környezeti(λ) értéket (0, ha a környezeti ≤ 0). A csoport címkéje az érzékelők kalibrációs osztályát követi:

* Mindkét érzékelő kalibrálva (csomag betöltve) → **„Látszólagos visszaverődés”**.
* Bármelyik érzékelő nem kalibrálva → **„Relatív reflexió”**.

A csoport zöld `REF` sor formájában jelenik meg az oldalsávban és saját diagramján (szivárványos háttér, az értékek 4 tizedesjegyig láthatók az egérmutatóval, húzással nagyítható).

A **+**menü a**Új érzékelő hozzáadása** lehetőséget is kínálja három elhelyezési lehetőséggel: *Új érzékelő egyesítése* (ehhez a diagramhoz csatlakoztatja), *Meglévő érzékelő ide helyezése* vagy *Új érzékelő megtekintése* (saját diagram).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->

### Növényzetindex-táblázat

Lista nézetben a vegetációs index táblázat a reflektancia-csoport diagramja alatt helyezkedik el, amelyet a sávközpontok **kék 450 / zöld 550 / vörös 670 / NIR 800 nm** (értékek 4 tizedesjegyig, `---`, ha nem számítható ki; az index nevére mutatva megjelenik a teljes neve):

* **Mindig megjelenik** (skálától független, bármilyen érzékelő-kombináció esetén): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Csak akkor, ha mindkét érzékelő teljesítménykalibrálva van** (mindkét csomag betöltve): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## `.daq` fájlok rögzítése

* A rögzítéshez **nyitott projekt** szükséges — ellenkező esetben mind az „Összes rögzítése” (oldalsáv), mind az egyes érzékelőkhöz tartozó „Rögzítés” gomb le van tiltva.
* A fájlok az **`<project folder>/light_sensor/`** mappába kerülnek; a fájlnevek tartalmazzák az érzékelő azonosítóját és az időbélyeget, az eszköz neve pedig a felvételhez kerül tárolásra.
* Amikor a felvétel leáll (Stop, Stop All, vagy a felvétel közbeni kapcsolatmegszakadás), a kész `.daq` fájl **automatikusan hozzáadódik a nyitott projekthez** — manuális hozzáadás nélkül megjelenik a projekt fájllistájában, készen arra, hogy [reflektancia-feldolgozás](README.md).
* Felvétel közben egy piros `REC` jelző jelenik meg a beállítási ablak élő soraiban.

A sugárzási intenzitás mennyiségi értékeinek kiszámításához legalább 15 másodpercnyi adatot kell átlagolni — ez a műszer jellemzője, nem hiba.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->

***

## Többszenzoros elrendezések és a projekt állandósága

* Több érzékelőt egyesíthet egy grafikonon (közös tengelyek), külön grafikonokat tarthat fenn (automatikus rácselrendezés), érzékelőket mozgathat a grafikonok között, sorokat/lapokat húzással rendezhet át, valamint a szem ikonra kattintva elrejtheti az egyes érzékelőket.
* Projektként az Chloros megőrzi: az eszközneveket, a grafikon színeit, a diagram méretét, a nézet módját és az egyes érzékelők beállításait (integrációs idő, képkockák átlagolása, AE állapot, sapka kiválasztása).
* **A projekt újbóli megnyitásakor az érzékelők automatikusan újra csatlakoznak** a címük alapján — COM-port a DAQ-U esetében, BLE-eszköz a DAQ-M esetében, mDNS-gazdagépnév a DAQ-E esetében (akkor is feloldódik, ha az egység IP-címe megváltozott) —, és újraalkalmazza az egyes érzékelők mentett sapkaprofilját, képkockák átlagolását, AE-állapotát és a kézi integrációs időt.***

## Kamera párosítása (DLS)

Nincs mit párosítani. Ellentétben a drónos DLS-munkafolyamatokkal, amelyek előre összekapcsolják a fényérzékelőt a kamerával, az Chloros a DAQ-adatokat utólag illeszti a képekhez: importáláskor/feldolgozáskor az `.daq` méréseit interpolálják az egyes felvételek expozíciós időbélyegéhez. Bármely csatlakoztatott érzékelővel rögzíthet (az `.daq` automatikusan bekerül a projektbe), és a reflexiós feldolgozás idő alapján megtalálja a megfelelő mérési értékeket — a lefelé irányuló adatok felhasználásáról lásd a [DAQ fényérzékelők](README.md) című részt.</version\>
