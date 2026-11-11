# A DAQ fül a Chloros programban

A DAQ fül — amely a Chloros oldalsávján **Fényérzékelők** néven szerepel — a [DAQ-U, DAQ-M és DAQ-E fényérzékelők](README.md) élő vezérlőfelülete: bármilyen adatátviteli csatornán keresztül csatlakoztathat érzékelőket, valós időben figyelheti a kalibrált spektrumokat, kiszámíthatja az érzékelőpár aktuális fényvisszaverődését, és közvetlenül a projektjébe rögzítheti az `.daq` fájlokat.

A fül akkor válik elérhetővé, amikor a Chloros háttérprogram befejezi az indítást. A fül diagramjait a Chloros DAQ-szolgáltatása táplálja egy élő kapcsolaton keresztül, amely megszakadás esetén automatikusan újra csatlakozik (2–10 másodperces várakozási idő); amíg a szolgáltatás nem érhető el, az érzékelő Állapot sorában a **No Server** felirat jelenik meg.

Az elrendezés egy **érzékelő oldalsáv**(egy sor minden csatlakoztatott érzékelőhöz) és egy**diagramterület** (egy diagrammező minden érzékelőhöz vagy csoporthoz).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Érzékelő csatlakoztatása

Kattintson az oldalsáv tetején található **Érzékelő csatlakoztatása** gombra. A csatlakozási párbeszédablak megnyílik a főterületen (vagy átfedésként, ha újabb érzékelőt ad hozzá – ebben az esetben megjelenik egy Mégse gomb).

| Vezérlőelem | Viselkedés |
| --- | --- |
| **Eszköztípus** | `DAQ-U (USB)` (alapértelmezett), `DAQ-M (Bluetooth)` vagy `DAQ-E (Ethernet)`. A váltás újraindítja a keresést az újonnan kiválasztott átviteli módra. |
| **Port / BLE-eszköz / Gazdagépnév / IP** | A felderített eszközöket `device - description` néven sorolja fel; az első, érzékelőként felismert bejegyzés automatikusan kiválasztásra kerül. A keresés során a következőket jeleníti meg: `Scanning...` (USB), `Scanning (N)...` 8 másodperces visszaszámlálással (BLE) vagy `Discovering ethernet sensors (N)...` 5 másodperces visszaszámlálással (Ethernet). Üres eredmények esetén a következőket jeleníti meg: `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Frissítés** | Azonnal újra beolvassa a kiválasztott átviteli módot (BLE/Ethernet-beolvasás közben letiltva). |
| **Érzékelő csatlakoztatása** | A készülék kiválasztása után aktiválódik; a kapcsolat létrehozása közben a kijelzés `Connecting...`-re változik. |

A felderítés csak **amíg a csatlakozási párbeszédablak a képernyőn látható**, és 15 másodpercenként ismétlődik, kizárólag a kiválasztott átviteli módra vonatkozóan — a lap megnyitása önmagában nem indítja el a szkennelést. Hiba esetén a párbeszédablak a következő üzenetet jeleníti meg: *„A csatlakozás nem sikerült. Próbálja meg kihúzni, majd újra bedugni az érzékelőt, majd kattintson újra a Csatlakozás gombra.”*

Az oldalsáv automatikusan megnyílik, amikor az első érzékelő csatlakozik.

{% hint style="info" %}
**A DAQ-E nem jelenik meg?**A DAQ-E-nek nincs állapotjelző LED-je — ellenőrizze a PoE/kapcsolatjelzőt azon a kapcsolón vagy injektorporton, amelyhez csatlakoztatva van, és a bekapcsolás után várjon néhány másodpercet, amíg elindul. A Chloros-gépnek ugyanabban a broadcast-tartományban kell lennie (az mDNS nem halad át a routereken). A telepítő**Chloros Backend (UDP-In)** tűzfalszabálya engedélyezi az mDNS-t (UDP 5353), a DAQ-E adatokat (UDP 5002) és a PTP-t (UDP 319/320); egy harmadik féltől származó tűzfal esetében ugyanezeket a portokat kell engedélyezni. Egy LAN-on található két DAQ-E egységet a rendszer külön-külön ismer fel, mindkettőt a saját `daq-e-<id>.local` gazdagépnév alatt.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>Az „Eszköztípus” menüpont a DAQ-U (USB), DAQ-M (Bluetooth) és DAQ-E (Ethernet) lehetőségeket kínálja</figcaption></figure>

***

## Az érzékelők oldalsávja

Minden csatlakoztatott érzékelő egy sort kap (plusz egy sor minden Ambient+Object csoportra). A sorok húzással átrendezhetők, és sorrendjük a diagramlapok sorrendjét is megváltoztatja. Kattintson egy sorra, hogy az adott érzékelő/csoport legyen az aktív diagram a listanézetben.

| Elem | Jelentés |
| --- | --- |
| Színes bal oldali szegély | Az érzékelő grafikonjának színe. |
| Átviteli jelvény | `DAQ-U` / `DAQ-M` / `DAQ-E`, vagy zöld `REF` jelvény az Ambient+Object fényvisszaverődési csoport esetében. |
| Eszköz neve | Alapértelmezés szerint az érzékelő sorozatszáma (a kalibráláshoz, az `.daq` fájlnevekhez és az importáláshoz szükséges állandó azonosító); az egyedi nevek projektenként megmaradnak. |
| **Kalibrálva** jelölő (zöld) | Akkor jelenik meg, ha az érzékelő gyári kalibrációs csomagja betöltődött, azaz a spektrumok valódi W/m²/nm értékeket mutatnak. |
| **Frissítés elérhető** jelölő (borostyánszínű, csak DAQ-E esetén) | A futó firmware régebbi, mint a jelenlegi Chloros-verzióhoz mellékelt kép. A frissítés során valós idejű haladást jelenít meg (`Flashing… N%`, `Restarting sensor…`, majd `Updated X → Y` vagy `Failed`). |
| Szem | Be- és kikapcsolja az érzékelő láthatóságát a grafikonon. |
| Fogaskerék | Megnyitja az érzékelőnkénti beállítások modális ablakát (lent). |
| ✕ (piros) | Leválasztja az érzékelőt, vagy eltávolít egy „Környezet+Objektum” csoportot. |

A sorok felett két gomb található:

* **Érzékelő csatlakoztatása** — megnyitja a csatlakozási párbeszédablakot (foglalt állapotban `Connecting...`-re változik a felirat).
* **Minden rögzítése / Minden leállítása**— elindítja vagy leállítja a `.daq` felvételt**minden**csatlakoztatott érzékelőn. Ehhez legalább egy érzékelőre**és egy megnyitott projektre** van szükség (eszköztipp: „Nyisson meg egy projektet a felvételhez”); a gomb pirosra vált, amíg bármelyik felvétel fut.

Üres állapotban a „Nincs csatlakoztatott érzékelő” felirat látható.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Érzékelőnkénti beállítások (a fogaskerék-ablak)

Az érzékelősoron található fogaskerék-ikonnal nyitható meg. A tartalom sorrendben:

* **Információs sorok** — Eszköztípus (DAQ-U/M/E), Csatlakozás (`Serial (USB)` / `Bluetooth` / `Ethernet`), port (COM-port, BLE-cím vagy gazdagép) és soros port.
* **Kalibrációs jelentés: Letöltés** — letölti az eszköz NIST-nyomon követhető kalibrációs tanúsítványát (PDF), és megnyitja azt a PDF-nézőben. A sorozatszám ismeretében érhető el; a tanúsítványt az első csatlakozáskor a rendszer gyorsítótárba menti.
* **Eszköznév** — a ceruzára kattintva átnevezhető; projektenként megmarad.
* **Grafikonvonal színe** — színminta; projektenként megmarad.
* **Integrációs idő (ms)**— csúszka + szám,**1–500 ms**, alapértelmezett**32 ms**. Az AE bekapcsolt állapotában le van tiltva.
* **Képkockák átlaga**— csúszka + szám,**1–50 képkocka**, alapértelmezett**20**.
* **AE: BE/KI**— automatikus expozíció kapcsoló;**alapértelmezés szerint BE** a csatlakozáskor. Kapcsolja ki, ha az integrációs időt manuálisan szeretné beállítani.
* **Streamelés leállítása / Streamelés indítása** — az élő stream szüneteltetése vagy folytatás.
* **Felvétel / Felvétel leállítása** — érzékelőnkénti `.daq` felvétel (nyitott projektre van szükség).
* **Cap** — a cap-korrekciós profil (következő szakasz).
* **Élő információs sorok** — Integrációs idő (ms), FPS, Minták, Felvétel (piros `REC` vagy `Off`) és Állapot (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Csak DAQ-E: hálózat, firmware és PTP sorok

* **Hostname / IP** — a készülék aktuális címe.
* **Firmware**— az aktuális firmware-verzió, valamint egy műveleti mező: egy**Frissítés a \<version\>

** gomb jelenik meg, ha ez a Chloros-verzió egy újabb DAQ-E firmware-képet tartalmaz. A frissítés körülbelül 30 másodperc alatt történik a hálózaton keresztül; az érzékelő automatikusan újraindul és újra csatlakozik, és egy megszakadt átvitel esetén az aktuális firmware sértetlen marad. A folyamat élőben követhető (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), és a mezőben az `Up to date` érték jelenik meg, ha az aktuális.
* **PTP-szinkronizálás** — az élő PTP-állapot (visszatér az `unknown` értékre). A DAQ-E 1.2.0-ás vagy újabb firmware-verziója kizárólag szolgaóraként vesz részt az IEEE 1588 PTPv2-ben; a Chloros-gazdagép háttérrendszere a PTP grandmaster, és minden LAN-on lévő DAQ-E és LATTICE kamera a 0. tartományban szolgaállapotban van hozzá, így az időbélyegek eltérése nagyjából 1 ms-on belül marad.
* **Hálózat**— az a cím, amelyen az egység fut, és hogy hogyan szerezte meg (DHCP, link-local vagy statikus), valamint egy mentett beállítás, amely újraindítás után is érvényben marad. A**Módosítás…**gomb megnyit egy űrlapot:**Automatikus (DHCP)**vagy**Statikus IP**, IP-címmel, alhálózati maszkkal és opcionális átjáróval. Az**Alkalmazás és újraindítás** gomb elmenti a beállításokat, újraindítja az érzékelőt és újra csatlakoztatja azt. A számítógép hálózatán kívüli statikus címet a rendszer elutasítja, mivel a Chloros az újraindítás után nem tudná elérni az érzékelőt; ekkor megjelenik egy jelölőnégyzet, amely lehetővé teszi a cím használatát olyan számítógépek esetében, amelyek átcímkézésre kerülnek, vagy átirányításra kerülnek az adott hálózatra. A **Stream at power-on**beállítás határozza meg, hogy az érzékelő a bekapcsolás után azonnal megkezdi-e az adatfolyam-átvitelt, még mielőtt bármely szoftver csatlakozna hozzá. Az**Érzékelő újraindítása** gomb újraindítja az érzékelőt, és az aktuális beállításokkal újra csatlakoztatja. Az érzékelő rögzítése közben az újraindítás nem lehetséges. Újdonság a 1.3.0-s verzióban.

Egy „Környezet+Objektum” csoport esetében a fogaskerék-modál csak a csoport forrásérzékelőit, az eszköz nevét és a grafikon vonalának színét jeleníti meg.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->



### Fedél kiválasztása

A **Fedél** legördülő menü megadja a Chloros-nak, hogy melyik fizikai fedél van felszerelve az érzékelő diffúzorára, és minden spektrumra alkalmazza az adott fedél gyárilag mért korrekciós profilját. A választási lehetőségek a modelltől függenek:

| Modell | Fedélválasztási lehetőségek |
| --- | --- |
| DAQ-U | Nincs (csupasz érzékelő), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (koszinusz-korrektor) |
| DAQ-M | Nincs (csupasz érzékelő), Sunshine (koszinuszkorrektor) |
| DAQ-E | Nincs (csupasz érzékelő), FOV 15°, FOV 45°, FOV 90°, Sunshine (koszinuszkorrektor) |

**Minden modell alapértelmezett beállítása a „Napfény” (koszinuszkorrektor)** — a MAPIR minden DAQ-t a „Napfény” fedéllel felszerelve szállít, és ez a szabványos kültéri konfiguráció: 180°-os félgömb alakú látómező, ahol a koszinusz-hiba 60°-ig ≤ ±4 %, 70°-ig pedig ≤ ±4,5 % (kb. 15°-os napmagasság alatt nem ajánlott), tervezés szerinti csillapítással (~12×). A választás a projektben megmarad.

{% hint style="warning" %}
**A fedél kiválasztásának meg kell egyeznie a fizikailag felszerelt fedéllel.**Sem az érzékelő, sem a szoftver nem képes felismerni, hogy melyik fedél van felszerelve. A kiválasztás hatással van mind az élő korrekcióra, mind az minden `.daq` fájlba beírt jelölésre — a Sunshine fedél ~12-szeres csillapításával egy be nem jelentett fedélcsere nagyjából ezzel a tényezővel tévesen korrigálja a spektrumokat. (Ugyanazon fedél eltávolítása és újbóli felszerelése körülbelül 1,5 %-os eltérést eredményez.) A**None (csupasz érzékelő)** lehetőséget csak akkor válassza, ha a fedél fizikailag eltávolítva van; egy DAQ-E esetében a „None” beállítás még mindig a gyári geometriai profilt alkalmazza a süllyesztett üvegdiffúzorra – ez nem egy „no-op” művelet –, és a fedél nélküli DAQ-E egy laboratóriumi konfiguráció, nem pedig támogatott terepi konfiguráció.
{% endhint %}

{% hint style="info" %}
Frissítés egy korábbi kézikönyvből: az 1.1.0-s verzióban szereplő, böngészőoldali „Sunshine Diffuser Installed” kapcsoló eltűnt. A fedél kezelése mostantól szenzoronkénti fedélprofil formájában történik, amelyet szerveroldalon alkalmaznak.
{% endhint %}

***

## A diagramterület

Egy rögzített felső sáv tartalmaz egy **lista ⇄ rács nézet közötti váltót**és egy**diagram nagyítás** csúszkát (csempe mérete 200–2000 px). A nézet automatikusan rácsra vált, ha több diagramcsoport létezik, és vissza listára, ha egy vagy kevesebb van. A nézetmód és a diagramméret projektenként megmarad.

Az egyes érzékelők **spektrumdiagramja** a következőket mutatja:

* **X-tengely** — Hullámhossz (nm). Az érzékelő rácsa 340–1010 nm-es tartományban 5 nm-es lépésekben (135 pont) van felosztva, a megjelenítéshez 1 nm-re interpolálva.
* **Y-tengely**— kalibrált érzékelő esetén (zöld**Kalibrált**jelölő),**Teljesítmény (W/m²/nm)**— radiometrikusan kalibrált spektrális besugárzás mindhárom hordozón — a csúcsértékből automatikusan kiválasztott SI-előtaggal (m/µ/n). Azok az érzékelők, amelyek kalibrációs csomagja nincs betöltve, nyers mérési értékeiket**Intenzitás (impulzusok)** néven, méretezés nélkül ábrázolják. Ha egy diagram mindkét típust ábrázolja, a teljesítmény a bal tengelyen, a számlálások pedig a jobb oldali tengelyen jelennek meg.
* Egyetlen görbe alatt szivárványos spektrális kitöltés látható; egy diagramon több érzékelő színes vonalakként jelenik meg, halványított kitöltéssel.
* **Egérmutatóval áthúzás**— függőleges kurzor a hullámhosszal és az érzékelőnkénti értékkel (kalibrált érzékelő esetén W/m²/nm, egyéb esetben teljes számlálási érték);**húzással** nagyítható (nagyításkor megjelenik egy kicsinyítő gomb).
* Egy **+** gomb (csak rácsnézetben) érzékelő hozzáadásához ehhez a diagramhoz vagy csoport létrehozásához (lásd alább).
* A készülék neve középen, a tetején, és egy forgó ikon az első képkocka megjelenéséig.

A **telítettség** nem jelenik meg magán a diagramon: egy telített érzékelő piros `SATURATED` állapotszöveget és egy piros `Saturated: Yes` sort jelenít meg az élő adattáblázatban. Az integrációs idő csökkentésével vagy az AE újraengedélyezésével törölhető.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->



***

## Élő adattábla (lista nézet)

A diagram alatt, lista nézetben, 500 ms-enként frissül:

* **Minden modell**: Fény színminta (sRGB a CIE XYZ-ből), Telített (Igen/Nem), CIE 1931 X/Y/Z, Színkoordináták x/y, CIE u′/v′, CCT (K), CRI (Ra), domináns hullámhossz (nm), csúcshullámhossz (nm), gerjesztési tisztaság, Duv, CIE L\*/a\*/b\* és Munsell H/V/C.
* **Kizárólag kalibrált érzékelők**(bármelyik DAQ-U / DAQ-M / DAQ-E, miután betöltötték a gyári kalibrációs csomagot — ezt a zöld**Kalibrált** jelölés jelzi az érzékelő sorában): Teljes teljesítmény (W/m²), fotopikus lux (lx), skotopikus lux (lx), S/P arány, PPFD, valamint PPFD Red /Green /Blue (µmol/m²/s), továbbá az opikus besugárzási erősségek — S-kúp, melanopikus, rodopikus, M-kúp, L-kúp (mind W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Reflektancia-csoportok (Környezeti + Objektum)

Két összekapcsolt érzékelő kombinálható egy élő reflexiókijelzővé — kamera nélkül:

1. Rácsnézetben kattintson a **+**gombra egy diagramlapkán, majd válassza a**Környezeti + Objektum kombinálása** lehetőséget.
2. Válasszon ki egy **Környezeti fényforrás**érzékelőt és egy**Objektum-szkennert**(két különböző érzékelőt), majd kattintson a**Létrehozás** gombra. A

Chloros a két élő adatfolyamból hullámhosszonként kiszámítja az R(λ) = objektum(λ) / környezeti(λ) értéket (0, ha a környezeti ≤ 0). A csoport címkéje az érzékelők kalibrációs osztályától függ:

* Mindkét érzékelő kalibrálva (csomag betöltve) → **„Látszólagos reflexió”**.
* Bármelyik érzékelő nem kalibrálva → **„Relatív visszaverődés”**.

A csoport zöld `REF` sor formájában jelenik meg az oldalsávban és saját diagramján (szivárványos kitöltés, az értékek 4 tizedesjegyig láthatók az egérmutatóval, húzással nagyítható).

A **+**menüben elérhető az**Új érzékelő hozzáadása** opció is, három elhelyezési lehetőséggel: *Új érzékelő összevonása* (ehhez a diagramhoz csatlakoztatja), *Meglévő érzékelő ide helyezése* vagy *Új érzékelő megtekintése* (saját diagram).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->



### Növényzetindex-táblázat

Lista nézetben a növényzetindex-táblázat a reflektancia-csoport diagramja alatt helyezkedik el, amelyet a sávközpontok élő reflektanciájából számítanak ki: **kék 450 / zöld 550 / vörös 670 / NIR 800 nm** sávközpontok élő reflektanciájából számított (értékek 4 tizedesjegyig, `---`, ha nem számítható; az index nevére mutatva megjelenik a teljes neve):

* **Mindig megjelenik** (skála-invariáns, bármilyen érzékelő-kombináció esetén): NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR.
* **Csak akkor, ha mindkét érzékelő teljesítménykalibrálva van** (mindkét csomag betöltve): EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## `.daq` fájlok rögzítése

* A rögzítéshez **nyitott projekt** szükséges — ellenkező esetben mind az „Összes rögzítése” (oldalsáv), mind az egyes érzékelőkhöz tartozó „Rögzítés” gomb le van tiltva.
* A fájlok az **`<project folder>/light_sensor/`** mappába kerülnek; a fájlnevek tartalmazzák az érzékelő azonosítóját és az időbélyeget, az eszköz nevét pedig a felvételhez tárolja a rendszer.
* Amikor a felvétel leáll (Stop, Stop All, vagy a felvétel közbeni kapcsolatmegszakadás), a kész `.daq` **automatikusan hozzáadódik a nyitott projekthez** — a projekt fájllistájában megjelenik manuális hozzáadás nélkül, készen arra, hogy [reflektancia-feldolgozáshoz](README.md) szolgáljon lefelé irányuló sugárzási adatként.
* Felvétel közben egy piros `REC` jelző jelenik meg a beállítási ablak élő soraiban.

A kvantitatív besugárzási értékekhez legalább 15 másodpercnyi adatot kell átlagolni — ez a műszer jellemzője, nem pedig hiba.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->



***

## Többérzékelős elrendezések és a projekt állapotának megőrzése

* Több érzékelőt egyesíthet egy grafikonon (közös tengelyek), külön grafikonokat tarthat fenn (automatikus rácsos elrendezés), érzékelőket mozgathat a grafikonok között, sorokat/lapokat húzással rendezhet át, valamint a szem ikonra kattintva elrejtheti az egyes érzékelőket.
* Chloros projektként tárolja: az eszközneveket, a grafikon színeit, a diagram méretét, a nézet módot, valamint az egyes érzékelők beállításait (integrációs idő, képkockák átlagolása, AE állapot, sapka kiválasztása).
* **A projekt újbóli megnyitásakor az érzékelők automatikusan újra csatlakoznak** a címük alapján — COM-port a DAQ-U esetében, BLE-eszköz a DAQ-M esetében, mDNS-gazdagépnév a DAQ-E esetében (még akkor is feloldódik, ha a készülék IP-címe megváltozott) —, és újraalkalmazza az egyes érzékelők mentett sapkaprofilját, képkockák átlagolását, AE-állapotát és a kézi integrációs időt.***

## Kamera párosítása (DLS)

Nincs mit párosítani. Ellentétben a drónos DLS-munkafolyamatokkal, amelyek előre összekapcsolják a fényérzékelőt a kamerával, a Chloros a DAQ-adatokat utólag illeszti a képekhez: az importálás/feldolgozás során az `.daq` mérési eredményeit interpolálják az egyes felvételek expozíciós időbélyegéhez. Bármely csatlakoztatott érzékelővel rögzíthet (az `.daq` automatikusan bekerül a projektbe), és a reflexiófeldolgozás idő szerint megtalálja a megfelelő mérési értékeket — a lefelé irányuló adatok felhasználásáról lásd a [DAQ fényérzékelők](README.md) részt. **Élő**LATTICE-felvételek esetén válassza ki az érzékelőt a Kamerák fül**Fényérzékelő** legördülő menüjéből; ekkor minden felvétel elmenti a felhasznált mérési értéket, amely kizárólag az adott felvétel kameráihoz van rendelve (lásd [Reflektancia-munkafolyamatok](reflectance.md)).</version\>
