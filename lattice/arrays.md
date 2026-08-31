# Többkamerás rendszerek

A LATTICE **rendszer**két vagy több LATTICE kamera egy szinkronizált egységként való összekapcsolásából áll. Az egyik kamera a**master**: ez küld egy hardveres GPIO trigger impulzust a közös szinkronizáló vonalon (alapértelmezés szerint**Line2**), így minden tag ugyanazt a pillanatot rögzíti. Az Chloros kiegészíti a PTP időszinkronizálást, élő előnézetet (kameránkénti mozaikok vagy egyetlen, összehangolt többsávos kompozíció), valamint szinkronizált felvételt – minden felvételi ciklus egy**képkockacsoportot** eredményez, amelyben minden kamera ugyanazt az időbélyeget és képkocka-azonosítót használja (a felvételi kimeneten `fid:N` néven jelenik meg).

A kamerasorok segítségével állítják elő a monokróm (M3M) kamerák a vegetációs indexeket – egy kamera egy sávot biztosít, a kamerasor pedig ezeket egy többsávos halmazba igazítja. Lásd [Monokróm kamerák és vegetációs indexek](mono-indices.md).

Három egyenértékű módszer létezik a tömb összekapcsolására, és mindegyik ugyanazt a „smart-prep” folyamatot futtatja:

| Felület | Belépési pont |
| --- | --- |
| Grafikus felület | Kamerák fül → **Mátrix csatlakoztatása** (kék gomb) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (az első sorozatszámú = vezérlő) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (az első sorozatszámú a vezérlő) |

A Smart-prep a következő lépéseket hajtja végre sorrendben: hálózati képességvizsgálat (ICMP DF ping + GVSP-vizsgálat), szinkronizációs szint kiválasztása, a keretméret automatikus csökkentése a vezetékhez való igazítás érdekében, PTP engedélyezése, kameránkénti pixelformátum automatikus kiválasztása, az automatikus expozíció beállítása az egyes kamerák mentett állapotából, valamint a GPIO-trigger konfigurálása a Line2-n.

{% hint style="info" %}
Ahhoz, hogy mindez működjön, a kameráknak elérhetőnek kell lenniük a kapcsolaton – a felismerésről, a címzésről és az első csatlakozáskor végzett kalibrációs letöltésről lásd a [Kamerák csatlakoztatása](connecting.md) című részt. Többkamerás rendszerek esetén a gazdagép hálózati kártyájának (NIC) vételi gyűrű beállításai ugyanolyan fontosak, mint a kapcsolat sebessége; a teljes tünet→megoldás táblázat az [CLI Referencia § Gazdagép hálózati kártya beállítása és hangolása](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays) részben található.
{% endhint %}

## Az Array Connect párbeszédpanel

A Kamerák fül → **Connect Array**menüpont egy háromlépéses varázslót nyit meg:**Kiválasztás → Megjelenítési mód → Beállítások**.

### 1. lépés — Vezérlő és alkamerák

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->

kiválasztása A párbeszédablak megnyitásakor azonnal átvizsgálja a hálózatot („Hálózat átvizsgálása...”), majd ellenőrzi a GPIO-trigger vezetékeit („GPIO-vezetékek ellenőrzése...”). A rendszer felépítéséhez legalább **2 kamera** szükséges.

A vezetékvizsgálat, amennyiben lehetséges, előre kitölti a szerepkör-kiválasztást, és a következő három üzenet egyikét jeleníti meg:

| Üzenet | Jelentés |
| --- | --- |
| „GPIO-főkamera észlelve — kiválasztások előre kitöltve” (zöld) | A vizsgálat megtalálta a kioldási topológiát; a főkamera és a szolga jelölőnégyzetek már be vannak jelölve. |
| „Főkamera nem észlelve – ellenőrizze a GPIO-kábelt” (narancssárga) | Egyik kamera sem érzékelt kioldási impulzust; ellenőrizze a szinkronizáló kábelezést. A szerepeket továbbra is manuálisan is kiválaszthatja. |
| „Nincs szinkronkábel: {soros számok}” (narancssárga) | A felsorolt kamerákhoz nincs csatlakoztatva szinkronkábel. |

A kameratáblázat **Kamera / Soros szám / IP / Mester (rádió) / Szolga (jelölőnégyzet)** oszlopokat tartalmaz:

* Válasszon ki pontosan **egy mastert**és**egy vagy több slave-et**. Ha újra rákattint a jelenlegi master rádiógombjára, az törlődik.
* A **„Nincs szinkronkábel”** jelöléssel ellátott kamerát soha nem lehet slave-ként kiválasztani — egy trigger-vezeték nélküli slave örökké a szinkronvonalon várna, és használhatatlan képet adna. Ezt a kamerát inkább önálló kameraként csatlakoztassa.
* A már önállóan csatlakoztatott kamerák *nem* kerülnek letiltásra: a tömbhöz való csatlakozás felszabadítja az önálló munkamenetet, és újra megnyitja a kamerát a tömbön belül.

**Következő: Megjelenítési mód →**akkor válik elérhetővé, ha már kiválasztottak egy mestert és legalább egy szolga kamerát. A**Újraolvasás** gomb újra futtatja a felderítést és a vezeték-ellenőrzést.

{% hint style="warning" %}
A **Mégse** gomb le van tiltva, amíg a keresés vagy a vezetékvizsgálat folyamatban van — a vizsgálat közbeni megszakítás a LATTICE kamerák firmware-jén összeomlást okozhat az SDK kamerában. Várja meg, amíg a forgó ikon leáll.
{% endhint %}

### 2. lépés — Megjelenítési mód

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Mód | Mit kap |
| --- | --- |
| **Különálló kamerák** | Kameránként egy élő csempe, amelyek együttesen aktiválódnak, így a képkockák szinkronban maradnak. Minden kamera megtartja saját színét és beállításait. |
| **Összevont kamerák** *(alapértelmezett)* | Egyetlen élőcsempe, amely az igazított, többsávos NDVI/index kompozíciót jeleníti meg. A kamerák megosztják a sorozat színét. |

A megjelenítési mód csak az élő előnézet megjelenítését változtatja meg — a rögzítési viselkedés mindkét esetben azonos.

### 3. lépés — A rendszer beállításai és a várható eredmény

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

Amikor belép ebbe a jelenetbe, az Chloros **ajánlást**kér a háttérrendszertől, és automatikusan alkalmaz egy olyan ROI + binning kombinációt, amely illeszkedik a NIC vételi gyűrűjéhez (a binninget részesíti előnyben az ROI-kivágással szemben, mivel a binning megőrzi a teljes látómezőt). Minden elvégzett módosítás esetén az elemzés élőben újra fut, és frissíti a jobb oldali**Várható eredmény** panelt.

Bal oldali oszlop — beállítások:

| Vezérlő | Választási lehetőségek | Alapértelmezett | Megjegyzések |
| --- | --- | --- | --- |
| **ROI (látómező)** | Teljes (2048×1536) / Fele (1024×768) / Negyed (512×384) | Teljes | Érzékelő kivágás: Fele/Negyed kivágás egy kisebb területre natív képpontsűrűséggel. |
| **Binning** | 1× / 2× (összesen 2×2) / 4× (összesen 4×4) | 1× | Hardveres binning: 2×2 = teljes látómező a vezetéki költség negyedénél; 4×4 = teljes látómező 1/16-ánál. Elrejtve, ha a kamerák nem támogatják a binninget. |
| **Vezetéken továbbított kép** (kiolvasás) | — | — | A binning utáni szélesség×magasság, amelyet ténylegesen továbbítanak a vezetéken, 16-os többszörösére (minimum 64) leegyszerűsítve. |
| **Pin felbontás**| jelölőnégyzet | ki | Az Chloros általában automatikusan növeli a binninget a csatlakozáskor, amikor a tervezett sebesség**1,5 fps** alá csökken. A rögzítés megőrzi a választott képkockaméretet és elfogadja az alacsonyabb sebességet — így a túlterhelt konfigurációt automatikus sebességcsökkentés helyett kemény kapcsolatmegtagadásként kezeli. |
| **Trigger Rate** | 0,5–60 fps, lépés 0,1 | üres = auto | A master trigger-tűzsebessége. Hagyja üresen, hogy az Chloros maga számítsa ki. |
| **Hálózati sávszélesség**| 20–2000 MB/s, lépésköz 10 | üres = auto | Mennyit tud a gazdagép ténylegesen feldolgozni, MB/s-ban —**ez az az egyetlen szám, amelytől az egész tömb allokációja függ.** A hálózati adapterről automatikusan felismerve. Csökkentse, ha a tömb sérült kereteket jelent: az észlelt érték túlbecsüli az USB-adapterek és a megosztott kapcsolók teljesítményét. A módosítás után a rendszer élőben újra futtatja az előrejelzést. |

Jobb oldali oszlop — **Előrejelzett eredmény**:

* **Szinkronizálási szint** — „Egyidejű rögzítés” (zöld), „Egyidejű rögzítés (FTD-elcsúszott kibocsátás)” (zöld), „Elcsúszott rögzítés (100 ms eltolódás)” (sárga) vagy „Túl nagy konfiguráció” (piros).
* **fps-előrejelzés** — tartományként jelenik meg („halvány → erős”), mivel a szinkronizált tömb sebességét a leglassabb kamera expozíciós ideje korlátozza.
* **NIC-sor** — kapcsolati sebesség és folyamatos átviteli sebesség („NIC {mbps} Mbps · folyamatos {N} MB/s”).
* **Sim-emit burst ellenőrzés** — képes-e a gazdagép NIC-je egyidejűleg fogadni egy burstot az összes kamerától („Sim-emit burst: X MB · NIC-gyűrű használható: Y MB ✓/✗”).
* **Vezeték-kapacitás ellenőrzés** — állandósult összesített igény az ütközésmentes vezeték-felső határhoz viszonyítva („Vezeték-kapacitás: {igény} MB/s, amelyet {n} kamera igényel · felső határ {felső határ} MB/s ✓/✗ túlterhelés”).
* **„Maximális kameraszám ezen a vezetéken: {n} — a kameránkénti sávszélesség-alsó határ határozza meg, így a binning nem növeli.”** — akkor jelenik meg, ha közel jársz a kameraszám-felső határhoz (vagy túllépted azt).
* **„Ezekkel a beállításokkal KÉPKOCKÁK FOGNAK KIESNI.”**— piros figyelmeztetés a háttérrendszer indoklásával, valamint a blokkoló tényezők listájával és kék**javítási javaslatokkal** („Hogy ez a rendszer beférjen a hálózatba” / „Az egyidejű rögzítés engedélyezéséhez”).

Az **Alkalmazás és csatlakozás** gomb addig nem használható, amíg nincs előrejelzés, és a gomb felirata megmondja, miért utasítja el a műveletet:

| Gomb felirata | Jelentés | Mi segít valójában |
| --- | --- | --- |
| „Elemzés folyamatban...” | Az elemzés még folyamatban van. | Várjon. |
| **„Túl sok kamera van ehhez a hálózathoz”**| A rendszer túlterheli a hálózatot (az aggregációs ellenőrzés sikertelen). | Kevesebb kamera, végpontok közötti jumbo keretek, vagy gyorsabb hálózati kártya.**A kisebb ROI NEM segít** — lásd alább. |
| **„Csökkentse a ROI-t az engedélyezéshez”** | Ezekkel a beállításokkal a keretek kiesnének (a burst/ring ellenőrzés sikertelen). | Csökkentse a ROI-t, növelje a binninget, vagy javítsa ki a hálózati kártya vételi gyűrűjét. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

A csatlakozás során megjelenhet egy zöld **kalibrációs letöltési panel** sorozatszámonkénti haladási sávval: amikor egy kamerát először csatlakoztatnak egy gépre, az Chloros a kamerából GigE-n keresztül letölti a ~3,8 MB-os gyári kalibrációs csomagot (kameránként körülbelül 70 másodperc). A gyorsítótárba mentett kameráknál ez a panel soha nem jelenik meg. Lásd [Kamerák csatlakoztatása](connecting.md).

## Sávszélesség: hány kamera fér el

Hogy egy tömb mennyit bír el, az a hálózati kapcsolat tulajdonsága, nem az Chloros-é, így a tervezési adatok a hardver kézikönyvben találhatók: **[Tömb sávszélesség-tervezés](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Mit tesz ezekkel az Chloros: a csatlakozási párbeszédablak hálózati vizsgálatot futtat, kiszámítja a elérhető képkockasebességet, és kiválasztja a megfelelő szintet. Ha a rendszer túlterheli a vezetéket, a rendszer nem dobja el csendben a csomagokat, hanem megtagadja a csatlakozást — lásd a fent leírt, a várható eredményt bemutató panelt.

## Amikor keretek hiányoznak

Egy kamera két teljesen különböző okból hiányozhat egy közzétett csoportból,
és ezek ellentétes megoldásokat igényelnek. Az Chloros külön számolja őket, ahelyett, hogy egyetlen
„hiányos” számot jelentené, amelyik egyik okot sem nevezi meg:

| Mi történt | Mit jelent | Hol kell keresni |
| --- | --- | --- |
| **Sérült**— a keret megérkezett, de szerkezetileg hibás volt | GVSP csomagvesztés a hálózati útvonalon | A**vezetékkapacitás**, a hálózati kártya vételi gyűrűje, a jumbo keretek, a kapcsoló |
| **Soha nem érkezett meg**— egyáltalán nem érkezett keret | A kamera nem indult el, vagy semmi sem hagyta el | Az**M8 szinkronkábel**, a szinkronvonal, hogy minden tag be van-e kapcsolva |

A felosztást 10 másodpercenként újraértékelik, amíg a rendszer adatot továbbít. 5 % felett
a rendszer mindkét számot megnevezve naplózza, és minden sérült puffert kameránként az első
előforduláskor jelent, majd percenként összesíti, így a hosszú munkamenet is olvasható marad.

**A sérült képkockák, amelyeknél a „soha nem érkezett” érték nulla, azt jelentik, hogy az indítás és a kábeles szinkronizálás tökéletes**,
és minden elveszett képkocka a hálózati útvonalon keletkezik. A megoldás a **Wire Budget** érték csökkentése és
az újracsatlakozás.

{% hint style="warning" %}
**A triggerfrekvencia csökkentése nem segít a sérült képkockák esetén.** A kamera csomag
ütemezése egyszer, a csatlakozáskor kerül beállításra. A triggerfrekvencia csökkentése azt változtatja meg, hogy milyen gyakran történik egy burst,
nem pedig azt, hogy milyen gyorsan jut el maga a burst a vezetékre. Egy mért, 4 kamerás rendszeren a
triggerfrekvencia 5-szeres csökkentése nem változtatott semmit, míg a vezetéki költségkeret 240-ről
200 MB/s-ra csökkentése ugyanazon rendszer esetében a sérült keretek arányát 10,4 %-ról nullára csökkentette.
{% endhint %}

A futó rendszer nem tudja újratervezni magát – válasszuk le és csatlakoztassuk újra, hogy a kapcsolódási idő
beállító az új sávszélesség-keretnek megfelelően működhessen.

### Az USB-hálózati adapterek sebessége 200 MB/s-ra van korlátozva

Egy USB-Ethernet-adapter az *Ethernet* kapcsolati sebességét hirdeti, de amit ténylegesen
fenntartani tud, azt az USB-busz és az illesztőprogramja korlátozza. Egy USB 10GbE dongle-t korábban
körülbelül 1000 MB/s átviteli sebességgel írtak jóvá – egy olyan értéket, amelyet soha senki nem mért meg –, és ha
négy kamerát ehhez a fantom tartalékhoz igazítottak, a képkockák 6–18%-a sérült meg, miközben a rendszer
továbbra is megfelelő célképkockasebességet jelentett. Az USB-n keresztül csatlakozó adapterek sebessége jelenleg
**200 MB/s**-ra van korlátozva. A korlát abszolút érték, nem pedig százalékos, mivel a korlátot a
busz jelenti: egy USB 1 GbE-adapter körülbelül 80 MB/s sebességet ér el, és ez nem érinti a korlátozást.

Ha a gazdagép valóban gyorsabb a korlátnál, emelje meg a **Wire Budget** értéket, hogy ezt jelezze.

## PTP időszinkronizálás

A képkockák *szinkronizálása* a hardveres triggerből származik; a **PTP** (IEEE 1588 PTPv2) minden eszközön összehasonlítható *időbélyegeket* biztosít. Ez alapértelmezés szerint engedélyezve van az array csatlakozásakor:

* Az **Chloros gazdagép háttérrendszere futtatja a PTP grandmastert**. A LATTICE kamerák és a DAQ-E fényérzékelők a 0. tartományban ehhez igazodnak, így a képek időbélyegei és a DAQ spektrumok egy órajelre esnek (~1 ms).
* Az `--no-ptp` (CLI) letiltja ezt a funkciót a laboratóriumi munkákhoz — ilyenkor a kamerák közötti időbélyegek **nem** összehasonlíthatók.
* Az CLI segítségével ellenőrizhető a szinkronizálás állapota:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Maga a „Kamerák” fül nem tartalmaz PTP-jelzőt; a kameránkénti szinkronizálási adatok között ott található az írásvédett **Szerep**(Master/Slave), a**Szinkronizálási vonal** és a tömb „Képességek” szintje. A DAQ-E PTP-állapota a „Fényérzékelők” fül érzékelő-részletei között látható.

## Az élő tömbnézet

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

A fő megjelenítési területen két elrendezés közül választhat (a felső sávban kapcsolható): **rácsnézet**(minden csempe egy cella; a rács lakatjának feloldása után húzással átrendezhető) és**listanézet**(a tömbök teljes szélességben a tetején, alattuk egy aktív kamera). A**Feed Zoom** csúszka méretezi a lapokat; 200 px-nél kisebb cellaszélesség esetén a név/fps feliratok automatikusan elrejtődnek.**Különálló mód** esetén kameránként egy lap jelenik meg. Minden lapon megjelenik:

* a kamera neve (bal felső sarokban),
* egy **fps-kijelzés** (jobb felső sarokban) — ez a háttérrendszer által jelentett kamera *valódi képfelvételi gyakorisága*, nem pedig az előnézeti lekérdezési gyakoriság (az élő előnézet 30 fps-re van korlátozva a képfelvételi gyakoriságtól függetlenül),
* egy állapotjelző pont — zöld (adatfolyam) / sárga (betöltés) / piros (hiba),
* egy **elavult képkockát jelző forgó ikon**, ha 2 másodpercig nem érkezett új képkocka — ez normális jelenség ~5 másodpercig bármely csatlakozás/leválás után, amíg a háttérrendszer újraelosztja a sávszélességet a kamerák között.**Kombinált mód**esetén egyetlen összetett csempe jelenik meg: a háttérrendszer elvégzi a debayeringet, méretezést, igazítást, zajszűrést, sávonkénti sugárzási értékre történő konverziót (plusz DLS-reflektancia, ha fényérzékelő van hozzárendelve), kiértékeli a tömb indexkifejezését, alkalmazza a LUT-ot, és az eredményt MJPEG formátumban továbbítja. Amíg az első igazított képkocka meg nem jelenik, a lapka jelzi az állapotát: „Tömb előkészítése…”, „Igazítás kalibrálása…”, „Az első képkockára várakozás…” vagy – ha az automatikus igazítás újrapróbálkozási ideje (~30 másodperc) lejárt – „Igazítás szükséges” felirattal, egy**Igazítás kalibrálása** gombbal.

Hasznos tudnivalók a kombinált üzemmódról:

* A kompozit a **fő**kamera képkockájához van igazítva. Az AE-ROI célzás és a pontmérés a kompoziton a master esetében pontos, a slave-ek esetében pedig hozzávetőleges; használja a**Split View** (tömbbeállítások → „Show member cameras”) funkciót a kameránkénti, pixel-pontos csempékhez anélkül, hogy további kamerakapcsolatokat kellene megnyitnia.
* A **Rétegek megjelenítése**(tömbbeállítások; alapértelmezés szerint ki van kapcsolva) lehetővé teszi az előtér- és háttérréteg kiválasztását — bármely taggép vagy**Index** lehet az. Ha az előtér = Index, a LUT Min/Max határain kívüli pixelek a háttérréteget jelenítik meg.
* A **Render felbontás** (alapértelmezés szerint 720p) beállítja az élő közvetítés magasságát *és* a mentett kompozit exportméretét. A kameránkénti képek mindig teljes felbontásban kerülnek exportálásra.
* Az igazítás munkamenetenként kerül kiszámításra, és soha nem kerül mentésre — az RMS-maradékértékekről és az Újrakalibrálás gombról lásd a tömbbeállítások panelen található igazítási részt.

## Felvétel: megfigyelés vs. elemzés

A rendszer felvételi felületei egyértelműen két csoportra oszthatók: **megfigyelési minőségű**(azt rögzíti, amit lát) és**elemzési minőségű** (nyers adatokat rögzít, későbbi kalibrálással):

| Munkafolyamat | Minőségi szint | Mit ment el | Felhasználói felület | CLI |
| --- | --- | --- | --- | --- |
| **Rögzítés**(állóképek) | Elemzés | Egy szinkronizált képkockacsoport minden átfutásonként; kameránkénti fájlok minden kiválasztott exportszinten (nyers/debayered/sugárzás/fényvisszaverődés/előnézet/index) + `.daq` sidecar |****Minden rögzítése** gomb + Rögzítési beállítások | `lattice array-capture` |
| **Indexvideó rögzítése** | Megfigyelés | A megjelenített élő kombinált indexkompozíció — 8 bites, előnézeti felbontás, beépített LUT; az élő stream megnyitása szükséges | ● Indexvideó rögzítése (kombinált tömbök) | `lattice array-record` |
| **Nyers sorozat → videó létrehozása**| Elemzés | Nyers érzékelő képkockák teljes rögzítési sebességgel + manifeszt + `.daq`, majd offline rekonstrukció kalibrált sugárzási / visszaverődési / index videóvá, a DAQ-leolvasásokhoz időben illesztve | ⦿ Nyers sorozat rögzítése →**Videó létrehozása** | `lattice array-burst` → `lattice array-build-video` |

Általános szabály: ha a pixelek *méréseket* szolgáltatnak, használjon rögzítést vagy sorozatfelvételt (elemzési minőségű); ha csak *meg szeretné nézni vagy bemutatni*, amit a szenzor látott, rögzítse az indexvideót (megfigyelési minőségű).

### Rögzítési beállítások (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

A **Capture All** melletti fogaskerék megnyitja a Rögzítési beállítások panelt (ehhez nyitott projekt szükséges — a rögzítések abba kerülnek mentésre):

* **Rögzítési mód**:**Egyedi**(egy átfutás) /**Folyamatos**(egymást követő; rögzítések számával, alapértelmezés szerint 1, vagy időtartammal, alapértelmezés szerint 10 másodperccel korlátozva) /**Intervallum** (időfelvétel: N rögzítés X-es időközönként, összesen Y-ig, alapértelmezés szerint 1 rögzítés 5 másodpercenként 1 percig).
* **Exporttípusok kameránként**: Raw, Debayered, Radiance, Reflectance, Preview, Index — alapértelmezés szerint minden alkalmazható be van kapcsolva. A Radiance/Reflectance opciók el vannak rejtve az RGB-szűrős kamerák esetében;**a Reflectance csak akkor jelenik meg, ha a kamerának van DAQ fényérzékelője** (saját vagy a sorozatból örökölt); Az Indexhez konfigurált indexkifejezés szükséges.
* **Aligned**(tömbönként, alapértelmezés szerint**be**): a tagok exportjait a tömb igazítási profiljához igazítja, így az exportok pixel-regisztráltak lesznek. A Raw mindig igazítatlan marad, de a transzformációt a metaadatokban hordozza.
* **Fastest Capture** (kapcsoló): csak nyers adat + a hozzárendelt DAQ-leolvasás + az ingyenes kombinált index-kompozit, kihagyva a kalibrációs számításokat a rögzítéskor a maximális sebesség elérése érdekében — a sugárzás/reflektancia/index később újrakalkulálható a mentett `.daq` fájlból.
* A kijelölések a projekttel együtt megmaradnak. A rejtett vagy szüneteltetett kamerákat a rendszer kihagyja.

Az ezzel egyenértékű CLI (ugyanaz a háttér végpont, ugyanaz a szemantika):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

TIFF A felvételek tömörítése `deflate` (veszteségmentes, alapértelmezett) vagy `none` — a teljes jelzőtáblák, a rögzítési mappa felépítése és az újrafeldolgozási szabályok a [CLI Referencia](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess) alatt találhatók.

## DAQ fényérzékelő párosítása

A fényvisszaverődés- és megvilágítás-korrigált előnézetekhez lefelé irányuló fényadatokra van szükség egy DAQ-érzékelőtől (amelyet a **Fényérzékelők** fülön kell csatlakoztatni):

* Az oldalsáv **sorozat-sora**egy**„DAQ · be/ki” gombot** jelenít meg — *be* állapotban van, ha be van állítva egy sorozat-szintű fényérzékelő **vagy** bármelyik tagkamera rendelkezik saját érzékelővel; a segítő szöveg pontosan felsorolja, melyik érzékelő melyik kamerát látja el adatokkal.
* A tömb szintű beállításokat a tömbbeállításokban lehet megadni → **Környezeti fényérzékelő**→**Fényérzékelő** legördülő menüben. A kiválasztás a projekttel együtt megmarad, átterjed minden tagkamerára, és az egyes kamerák továbbra is felülírhatják saját érzékelőjükkel.
* Az alatta lévő állapotsor jelzi az aktuális állapotot: **Ki**→ „Várakozás az első spektrumra…” →**„Aktív — a rendszer összes kamerája megvilágítás-korrigált”** → vagy, ha az elmúlt 3 másodpercben nem érkezett új spektrum, egy elavult értesítés — továbbra is az utolsó leolvasási értéket használja a rendszer (a leolvasási értékek soha nem járnak le a rögzítési útvonalon).

Érzékelő hozzárendelése esetén: elérhetővé válik a „Reflectance” (Fényvisszaverődés) exporttípus, az élő előnézetek megvilágítás-korrigáltak, a prediktív automatikus expozíció felhasználhatja a spektrumot, és minden fényvisszaverődés-rögzítés a ténylegesen használt DAQ-leolvasási értéket **`.daq` sidecar** a kép mellé, így a rögzítés később újra feldolgozható.

## `array-connect` CLI beállítások

| Jelző | Alapértelmezett | Leírás |
| --- | --- | --- |
| `--serials SN1,SN2,…` | az összes LATTICE kamera automatikus felismerése (legalább 2 szükséges) | **Az első soros csatlakozó a MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO szinkronizáló vonal. |
| `--target-fps F` | automatikus | A MASTER kioldási frekvenciája. |
| `--binning {1,2,4}` | automatikus | Hardveres binning. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | Szinkronizációs szintválasztó szakértői felülírása. |
| `--wire-ceiling-mbps MB_PER_S` | automatikusan felismert | Gazdagép vezeték-kapacitása MB/s-ban — a **Wire Budget** mező CLI formátuma. Csökkentse ezt az értéket, ha a tömb sérült kereteket jelent. A projekttel együtt mentésre kerül, így egy későbbi újracsatlakozáskor visszaáll. |
| `--no-recommend` | ki | A hálózati elemzés lépésének kihagyása. |
| `--no-ptp` | ki | A PTP letiltása (ebben az esetben a kamerák közötti időbélyegek nem összehasonlíthatók). |

Az `lattice array-list`, `array-status` és `array-disconnect` parancsok a tartós munkamenetet kezelik. A teljes alparancs-referencia, beleértve az igazítást (`align-calibrate` / `align-apply`) és a hálózati eszközöket is tartalmazó teljes alparancs-referencia a [CLI Referencia § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice) oldalon található; az SDK megfelelői (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) a [SDK hivatkozásban](../reference/sdk-reference.md) találhatók. Az Python-től kezdve a vezeték-keret az `connect_array(..., wire_ceiling_mbps=120)`, az élő, sérült és soha meg nem érkezett adatok felosztása pedig az [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames) található.
