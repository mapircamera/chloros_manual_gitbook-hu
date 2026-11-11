# Többkamerás rendszerek

A LATTICE **rendszer**két vagy több LATTICE kamera összekapcsolásából áll, amelyek egy szinkronizált egységet alkotnak. Az egyik kamera a**master**: ez küld egy hardveres GPIO-trigger impulzust a közös szinkronvonalon (alapértelmezés szerint**Line2**), így minden tag ugyanabban a pillanatban exponál. A Chloros PTP időszinkronizálást, élő előnézetet (kameránkénti mozaikok vagy egyetlen, igazított többsávos kompozit) és szinkronizált felvételt biztosít — minden felvételi ciklus egy **képkockacsoportot** eredményez, amelyben minden kamera ugyanazt az időbélyeget és képkocka-azonosítót használja (amelyet a rögzítési kimeneten `fid:N` formában jelenítenek meg).

A tömbök segítségével állítják elő a monokróm (M3M) kamerák a vegetációs indexeket – egy kamera egy sávot ad hozzá, a tömb pedig ezeket egy többsávos halmazba igazítja. Lásd: [Mono kamerák és vegetációs indexek](mono-indices.md).

Három egyenértékű módszer létezik a tömb összekapcsolására, és mindegyik ugyanazt a „smart-prep” folyamatot futtatja:

| Felület | Belépési pont |
| --- | --- |
| GUI | Kamerák fül → **Tömb összekapcsolása** (kék gomb) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (első sorozatszám = master) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (az első sorozatszámú a master) |

A Smart-prep a következőket hajtja végre: hálózati képességvizsgálat (ICMP DF ping + GVSP-vizsgálat), szinkronizációs szint kiválasztása, képkockasebesség-becslés a kért képkockaméretre, a tömörített 12-bites átvitelre vonatkozó döntés (lásd [Tömörített 12-bites átvitel](#packed-12-bit-transport)), PTP engedélyezését, kameránkénti pixelformátum automatikus kiválasztását, az egyes kamerák mentett állapotából származó automatikus expozíció-beállítást, valamint a GPIO-trigger konfigurálását a Line2-n.

**Alapértelmezés szerint teljes érzékelő (újdonság az 1.3.0-ban).** A Connect megtartja a kért képkockaméretet — a teljes 2048×1536-os érzékelőt, binning és kivágás nélkül, hacsak másképp nem dönt — és hagyja, hogy a képkockasebesség a gazdagép és a hálózat által biztosított értékre csökkenjen. Soha nem végez binninget vagy kivágást saját kezdeményezésére (az 1.2.0-s verzió 1,5 fps alatt automatikusan binninget hajtott végre). A binning vagy egy kisebb ROI kiválasztása rajtad múlik: a csatlakozási párbeszédpanelen válassz egyet, és a „Projected Outcome” (Várható eredmény) mező megmutatja az abból adódó képkockasebességet; az `array-connect` parancs kinyomtatja a gyorsabb lehetőségeket a várható sebességekkel együtt.

{% hint style="info" %}
Ahhoz, hogy mindez működjön, a kameráknak elérhetőnek kell lenniük a kapcsolaton — lásd [Kamerák csatlakoztatása](connecting.md) című részben található információk a felismerésről, a címzésről és az első csatlakozáskor végzett kalibrációs letöltésről. Többkamerás rendszerek esetén a gazdagép hálózati kártyájának (NIC) vételi gyűrű beállításai ugyanolyan fontosak, mint a kapcsolat sebessége; a teljes tünet→megoldás táblázat a [CLI-referencia § Gazdagép hálózati kártyájának beállítása és hangolása](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays) című részben található.
{% endhint %}

## Az Array Connect párbeszédpanel

A Kamerák fül → **Connect Array**menüpont egy háromlépéses varázslót nyit meg:**Kiválasztás → Megjelenítési mód → Beállítások**.

### 1. lépés — A master és a slave kamerák kiválasztása



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->

A párbeszédablak megnyitásakor azonnal átvizsgálja a hálózatot („Hálózat átvizsgálása...”), majd ellenőrzi a szinkronkábeleket („Szinkronkábelek ellenőrzése...”), hogy meghatározza, mely kamerák csatlakoznak az M8 szinkronvonalhoz. Egy tömb létrehozásához legalább **2 kamera** szükséges.

Az M8 szinkronkábelek egy csomópontban futnak össze, így **bármelyik bekötött kamera lehet master**. A vizsgálat előre kitölti a szerepeket — az RGB (`-FRGB`) kamerát főkameraként, ha csatlakoztatva van, egyébként az első csatlakoztatott kamerát, a többi csatlakoztatott kamerát pedig alkameraként —, és a három felirat egyikét jeleníti meg:

| Felirat | Jelentés |
| --- | --- |
| „Szinkronkábel található {n} kamerán. Bármelyikük lehet master; az RGB-kamera (vagy egyébként az első) előre kiválasztva van.” (zöld) | A szonda megtalálta a kamerákat a szinkronvonalon; a master rádió és a slave jelölőnégyzetek már be vannak jelölve. |
| „Nem észlelt főkamerát – ellenőrizze a GPIO-kábelt” (narancssárga) | Egyik kamera sem érzékelte a kioldó impulzust; ellenőrizze a szinkronkábel-összeköttetést. A szerepkörök továbbra is előre be vannak jelölve (RGB-kamera, egyébként a lista első kamerája, mint főkamera), és ezeket módosíthatja. |
| „Nincs szinkronkábel: {soros portok}” (narancssárga) | A felsorolt kamerákhoz nincs csatlakoztatva szinkronkábel. |

A kamera táblázat oszlopai: **Kamera / Soros / IP / Fő (rádió) / Alárendelt (jelölőnégyzet)**. A sorok**kamera sorrendben** vannak felsorolva — az M3C (Bayer) kamerák az M3M (mono) kamerák előtt; az M3C-k között az FRGB, FRGN, FOCN, majd az FNGB; az M3M-ek szűrőhullámhossz szerint, a legalacsonyabbtól kezdve (F450 az F808 előtt). Az oldalsávon található kameralista és a rács alapértelmezett elrendezése ugyanazt a sorrendet követi.

* Válasszon pontosan **egy mastert**és**egy vagy több slave-et**. Ha újra rákattint az aktuális master rádiógombjára, az törlődik. Egy másik master kiválasztása egy kattintással történik: az új master elhagyja a slave-eket, és az előző master slave-vé válik (hacsak nincs szinkronkábelje).
* A **„Nincs szinkronkábel”** jelöléssel ellátott kamerát soha nem lehet alkameraként kiválasztani — egy trigger-vezeték nélküli alkamera örökké a szinkronvonalon várna, és használhatatlan képet adna. Ehelyett csatlakoztassa azt a kamerát önálló kameraként.
* A már önállóan csatlakoztatott kamerák *nem* kerülnek letiltásra: a tömbhöz való csatlakozás feloldja az önálló munkamenetet, és újra megnyitja a kamerát a tömbön belül.

**Következő: Megjelenítési mód →**akkor válik elérhetővé, ha már kiválasztottak egy mestert és legalább egy szolga kamerát. A**Újraszkennelés** újra futtatja a felderítést és a vezeték-ellenőrzést.

{% hint style="warning" %}
A **Mégse** gomb le van tiltva, amíg a keresés vagy a vezetékellenőrzés folyamatban van — a vezetékellenőrzés közbeni megszakítás összeomlást okozhat a LATTICE kamerák firmware-jén futó kamera-SDK-ben. Várja meg, amíg a forgó ikon leáll.
{% endhint %}

### 2. lépés — Megjelenítési mód

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->



| Mód | Mit kap |
| --- | --- |
| **Különálló kamerák** | Kameránként egy élő csempe, mindegyik egyszerre indul el, így a képkockák szinkronban maradnak. Minden kamera megtartja saját színét és beállításait. |
| **Összevont kamerák** *(alapértelmezett)* | Egyetlen csempe, amely az összehangolt, többsávos NDVI/index kompozíciót jeleníti meg. A kamerák megosztják a sorozat színét. |

A megjelenítési mód csak az élő előnézet megjelenítését változtatja meg — a rögzítési viselkedés mindkét esetben azonos.

### 3. lépés — Sorozatbeállítások és a várható eredmény

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->



Ebbe a jelenetbe lépve a Chloros **ajánlást**kér a háttérrendszertől, és azt alkalmazza: a**teljes érzékelő – binning és ROI-kivágás nélkül – amennyiben bármely képkockasebesség képes ezt kezelni**, függetlenül attól, hogy a gazdagép és a vezeték milyen sebességet biztosít. Csak akkor, ha egyetlen sebesség sem képes a teljes érzékelőt kezelni, az ajánlás kicsinyíti a képkockát, és ebben az esetben a binninget részesíti előnyben az ROI-kivágással szemben, mivel a binning megőrzi a teljes látómezőt. Ettől kezdve a felbontást cseréled el a képkockasebességért. Minden végzett módosítás után az elemzés élőben újra fut, és frissíti a jobb oldali**Várható eredmény** panelt.

Bal oldali oszlop — beállítások:

| Vezérlő | Választási lehetőségek | Alapértelmezett | Megjegyzések |
| --- | --- | --- | --- |
| **ROI (látómező)** | Teljes (2048×1536) / Fele (1024×768) / Negyed (512×384) | Teljes | Érzékelő kivágás: Fele/negyed kivágás egy kisebb területre natív képpontsűrűséggel. |
| **Binning** | 1× / 2× (összeg 2×2) / 4× (összeg 4×4) | 1× | Hardveres binning: 2×2 = teljes látómező a vezetékterhelés negyedénél; 4×4 = teljes látómező 1/16-ánál. Rejtve, ha a kamerák nem támogatják a binninget. |
| **Vezetéken továbbított kép** (kiolvasás) | — | — | A binning utáni szélesség×magasság, amelyet ténylegesen továbbítanak a vezetéken, 16-os többszörösére leigazítva (minimum 64). |
| **Triggerfrekvencia** | 0,5–60 fps, lépésenként 0,1 | üres = automatikus | A master trigger-tűzgyakorisága. Hagyja üresen, hogy a Chloros számítsa ki. |
| **Vezeték-kapacitás**| 20–2000 MB/s, lépésenként 10 | üres = automatikus | Mennyit tud a gazdagép ténylegesen feldolgozni, MB/s-ban —**az az egyetlen szám, amelytől az egész tömb allokációja függ.** A hálózati adapterről automatikusan felismerve. Csökkentse, ha a tömb sérült kereteket jelent: az észlelt érték túlbecsüli az USB-adapterek és a megosztott kapcsolók teljesítményét. A módosítás után a rendszer élőben újra futtatja a prognózist. A mező az utolsó tömb csatlakozásakor használt sávszélességgel nyílik meg (a projekttel együtt mentve), így az egyszer kijavított érték megmarad. |

Az 1.2.0-s verzióból származó **Pin felbontás** jelölőnégyzet eltűnt: a csatlakozás mindig megtartja a választott keretméretet, és helyette a képkockasebességet csökkenti.

Jobb oldali oszlop — **Előre jelzett eredmény**:

* **Szinkronizálási szint** — „Egyidejű rögzítés” (zöld), „Egyidejű rögzítés (FTD-elcsúszott kibocsátás)” (zöld), „Elcsúszott rögzítés (100 ms eltolódás)” (sárga) vagy „Túl nagy konfiguráció” (piros). A Chloros választja ki a szintet: egyidejű rögzítést, ha a vezeték olyan sebességgel osztható meg, amelyhez a kamerák tempója igazodhat (a képkockasebesség csökken, ahogy az egyes kamerák részesedése csökken), és eltolt rögzítést csak akkor, ha ez nem lehetséges — így a rendszer bármilyen hálózaton csatlakozik, a hálózat által megengedett képkockasebességgel.
* **fps-előrejelzés** — az a képkockasebesség, amelyen a rendszer várhatóan fog működni. Egyidejű rögzítés esetén ez egy szám: először a sebesség jön, az automatikus expozíció kitölti az általa megadott időtartamot, majd növeli az erősítést, így egy sötét jelenet nem csökkenti azt (lásd [Hogyan osztja el az automatikus expozíció az expozíciós időt és az erősítést](camera-settings.md)). Lépcsőzetes rögzítés esetén ez tartományként jelenik meg („sötét → világos”), mivel sötét jelenetben az egyes kamerák expozíciója meghosszabbítja a lépcsőzést.
* **NIC-vonal** — kapcsolat sebessége és tartós átviteli sebesség („NIC {mbps} Mbps · tartós {N} MB/s”).
* **Sim-emit burst ellenőrzés** — képes-e a gazdagép hálózati kártyájának gyűrűje egyidejűleg fogadni az összes kamerától érkező burst-ot („Sim-emit burst: X MB · NIC gyűrű használható: Y MB ✓/✗”).
* **Vezeték-kapacitás ellenőrzés** — állandó állapotú összesített igény az ütközésmentes vezeték-felső határhoz viszonyítva („Vezeték-kapacitás: {igény} MB/s, amelyet {n} kamera igényel · felső határ {felső határ} MB/s ✓/✗ túlterhelés”).
* **„Maximális kameraszám ezen a vezetéken: {n} — a kameránkénti sávszélesség alsó határa határozza meg, így a csoportosítás nem növeli.”** — akkor jelenik meg, ha közel van (vagy túllépi) a kameraszám felső határát.
* **„Ezekkel a beállításokkal KÉPEK VESZÍTENEK EL."**— piros figyelmeztetés a háttérrendszer indoklásával, valamint egy blokkolók listájával és kék**javítási javaslatokkal** („Ahhoz, hogy ez a tömb beférjen a hálózatra” / „Az egyidejű rögzítés feloldásához”).

Az **Alkalmazás és csatlakozás** gomb addig nem elérhető, amíg nincs előrejelzés, és a címkéje megmutatja, miért utasítja el:

| Gomb felirata | Jelentés | Mi segít ténylegesen |
| --- | --- | --- |
| „Elemzés...” | Az elemzés még folyamatban van. | Várjon. |
| **„Túl sok kamera van ehhez a hálózathoz”**| A hálózat még egy kamerát sem képes egyszerre továbbítani (az összesítési ellenőrzés sikertelen): sokkal több kamera van, mint amennyit a hálózat képes kezelni, vagy más kamerák is streamelnek rajta. | Kevesebb kamera, végpontok közötti jumbo keretek, vagy gyorsabb hálózati kártya.**Egy kisebb ROI NEM segít** — lásd alább. |
| **„Csökkentse a ROI-t az engedélyezéshez”** | Ezekkel a beállításokkal a képkockák kiesnének (burst/ring ellenőrzés sikertelen). | Csökkentse a ROI-t, növelje a binninget, vagy javítsa ki a hálózati kártya vételi gyűrűjét. |



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

A csatlakozás során megjelenhet egy zöld **kalibrációs letöltési panel**, soronkénti haladási sávval: amikor egy kamerát először csatlakoztatnak egy gépre, a Chloros letölti a kamerából a ~3,8 MB-os gyári kalibrációs csomagot GigE-n keresztül (kameránként körülbelül 70 másodperc). A gyorsítótárban tárolt kameráknál ez a panel soha nem jelenik meg; hatnál több kamera esetén a sorok görgethetők. Lásd [Kamerák csatlakoztatása](connecting.md).

### Tömörített 12 bites adatátvitel

A 12 bites formátumok (`BayerRG12`, `Mono12`) általában 16 bites szavakként kerülnek továbbításra. **A**tömörített**átvitel ugyanazt a 12 bitet 1,5 bájtban küldi el pixelenként, és a gazdagépen bontja ki őket, így minden képkocka továbbítása 25 %-kal rövidebb időt vesz igénybe — teljes érzékelőterhelés mellett ez másodpercenként körülbelül egyharmaddal több képkockát jelent.**A mentett adatok megegyeznek.** Az Array Connect automatikusan dönt erről (`auto`): csak akkor tömörít, ha az előnyös, és a gazdagép rendelkezik a kicsomagoláshoz szükséges szabad CPU-kapacitással. A grafikus felületen nincs erre vonatkozó kapcsoló; a parancssorból (CLI) használja az `array-connect --packed {auto,on,off}` (alapértelmezett: `auto`). A 8-bites és 16-bites formátumokat soha nem tömöríti.

## Sávszélesség: hány kamera fér el

Az, hogy egy rendszer mennyit bír el, a hálózat tulajdonsága, nem a Chlorosé, ezért a tervezési adatok a hardver kézikönyvben találhatók: **[Array Bandwidth Planning](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

Chloros hogyan kezeli ezeket: a csatlakozási párbeszédablak hálózati vizsgálatot futtat, kiszámítja az elérhető képkockasebességet, és kiválasztja a megfelelő szintet. A tömb bármilyen hálózaton csatlakozik: lassú kapcsolaton minden kamera a vezeték rá eső részéhez igazodik, és a képkockasebesség csökken, ahelyett, hogy a tömböt elutasítanák vagy a képkockáit kicsinyítenék. Csak azokat a tömböket utasítja el a rendszer, amelyeket a vonal egyszerre még egy kamerával sem képes kiszolgálni (**Túl sok kamera ehhez a hálózathoz**), ahelyett, hogy csendben eldobná a csomagokat — lásd a fent leírt, a várható eredményeket bemutató panelt.

## Ha képkockák hiányoznak

Egy kamera két teljesen különböző okból hiányozhat egy közzétett csoportból,
és ezek ellentétes megoldást igényelnek. A Chloros külön számolja őket, ahelyett, hogy egyetlen
„hiányos” számot jelentené, amelyik egyik okot sem nevezi meg:

| Mi történt | Mit jelent | Hol kell keresni |
| --- | --- | --- |
| **Sérült**— a képkocka megérkezett, de szerkezetileg hibás volt | GVSP-csomagvesztés a hálózati útvonalon | A**vezeték kapacitása**, a hálózati kártya vételi gyűrűje, a jumbo-képkockák, a kapcsoló |
| **Soha nem érkezett meg**— egyáltalán nem érkezett keret | A kamera nem indult el, vagy semmi sem jött ki belőle | Az**M8 szinkronkábel**, a szinkronvonal, hogy minden tag be van-e kapcsolva |

A felosztást 10 másodpercenként újraértékelik, amíg a rendszer streamel. 5 % felett
mindkét szám megnevezésével naplózzák, és minden sérült puffert az első alkalommal jelentenek
kameránként, majd percenként összesítik, így a hosszú munkamenetek is olvashatók maradnak.

**A sérült képkockák, ahol a „soha nem érkezett” érték nulla, azt jelentik, hogy a kioldás és a kábeles szinkronizálás tökéletes**,
és minden elveszett képkocka a hálózati útvonalon keletkezett. A megoldás a **Wire Budget** érték csökkentése és
az újracsatlakozás.

{% hint style="warning" %}
**A trigger-frekvencia csökkentése nem segít a sérült képkockák esetén.** A kamera csomag-
ütemezése egyszer, a csatlakozáskor kerül beállításra. A trigger-frekvencia csökkentése azt változtatja meg, hogy milyen gyakran történik egy burst,
nem pedig azt, hogy milyen gyorsan jut el maga a burst a vezetékre. Egy mért, 4 kamerából álló rendszerben a
trigger rate 5-szeres csökkentése nem változtatott semmit, míg a wire budget 240-ről
200 MB/s-ra csökkentése ugyanazon rendszer esetében a sérült képkockák arányát 10,4 %-ról nullára csökkentette.
{% endhint %}

A futó rendszer nem tudja újratervezni magát — válassza le, majd csatlakoztassa újra, hogy a csatlakozási idő
beállító az új sávszélesség-keretnek megfelelően működhessen.

### Az USB-hálózati adapterek korlátozottak

Egy USB-Ethernet-adapter hirdeti az *Ethernet* kapcsolati sebességét, de amit ténylegesen
fenntarthat, azt az USB-busz és az illesztőprogram határozza meg. Egy USB 10GbE dongle-t korábban
körülbelül 1000 MB/s átviteli sebességgel – egy olyan értéket, amelyet még soha senki sem mért meg –, és ha
négy kamerát ehhez a fantomszerű tartalékhoz igazítottak, a képkockák 6–18 %-a sérült meg, miközben a rendszer
továbbra is megfelelő célképkockasebességet jelentett. Az USB-n keresztül csatlakoztatott adapterek átviteli sebessége jelenleg
**200 MB/s**-ra van korlátozva Windows és NVIDIA Jetson rendszereken, **400 MB/s**-ra pedig Linux x86_64 rendszereken. A korlát abszolút érték, nem pedig százalékos, mivel a korlátot a
busz jelenti: egy USB 1 GbE adapter körülbelül 80 MB/s sebességet ér el, és ez nem érinti.

Ha a gazdagéped valóban gyorsabb a korlátnál, emeld meg a **Wire Budget** értéket, hogy ezt jelezd.

## PTP időszinkronizálás

A képkockák *szinkronizálását* a hardveres trigger biztosítja; a **PTP** (IEEE 1588 PTPv2) minden eszközön összehasonlítható *időbélyegeket* biztosít. Alapértelmezés szerint a tömb csatlakoztatásakor engedélyezve van:

* A **Chloros gazdagép háttérrendszere futtatja a PTP grandmastert**. A LATTICE kamerák és a DAQ-E fényérzékelők a 0. tartományban ehhez igazodnak, így a képek időbélyegei és a DAQ spektrumok egy órajelre (~1 ms) esnek.
* Az `--no-ptp` (CLI) parancs kikapcsolja a funkciót asztali munkához — ilyenkor a kamerák közötti időbélyegek **nem** összehasonlíthatók.
* Ellenőrizze a szinkronizálás állapotát a CLI-en:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

Maga a „Cameras” (Kamerák) fül nem tartalmaz PTP-jelzőt; a kameránkénti szinkronizációs adatok között ott található az írásvédett **Role**(Szerep) (Master/Slave), a**Sync Line** (Szinkronizációs vonal) és a tömb „Capabilities” (Képességek) szintje. A DAQ-E PTP-állapota a „Light Sensors” (Fényérzékelők) fül érzékelő-részletei között jelenik meg.

## Az élő tömbnézet

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

A fő megjelenítési terület két elrendezést kínál (a felső sávban váltható): **rácsnézet**(minden csempe egy cella; a rács lakatjának feloldása után húzással átrendezhető) és**lista nézet**(teljes szélességű sorok: egy**összevont**rendszer egy sort foglal el – ez az összetett csempe –, míg egy**különálló**rendszerben**minden tag saját sort kap**, saját élő képpel és saját kalibrációs cél táblázattal; a rácsok alatt egy önálló kamera — az aktív). A**Feed Zoom** csúszka méretezi a lapokat; 200 px-nél kisebb cellaszélesség esetén a név/fps feliratok automatikusan elrejtődnek. A tagkamerák az oldalsávban és a rács alapértelmezett elrendezésében a kamerák sorrendjében jelennek meg (M3C az M3M előtt, FRGB elsőként, monokrom kamerák hullámhossz szerint); a rács feloldása és az áthúzás továbbra is felülírja a rács sorrendjét.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



**A különálló mód** kameránként egy-egy mozaikot jelenít meg. Minden mozaik feliratozza:

* a kamera nevét (bal felső sarokban),
* egy **fps-kijelzést** (jobb felső sarokban) — ez a háttérrendszer által jelentett kamera *tényleges képfelvételi sebessége*, nem pedig az előnézeti lekérdezési sebesség (az élő előnézet 30 fps-re van korlátozva, függetlenül a képfelvételi sebességtől),
* egy állapotpont — zöld (adatfolyam) / sárga (betöltés) / piros (hiba),
* egy **elavult képkockát jelző forgó ikon**, ha 2 másodpercig nem érkezett friss képkocka — ez normális jelenség ~5 másodpercig bármely csatlakozás/leválás után, amíg a háttérrendszer újra egyensúlyba hozza a sávszélességet a kamerák között.**Kombinált mód**: egyetlen összetett csempét jelenít meg: a háttérrendszer elvégzi a debayeringet, méretezést, igazítást, zajszűrést, sávonkénti sugárzási értékre történő konvertálást (plusz DLS-reflektancia, ha fényérzékelő van hozzárendelve), kiértékeli a tömb indexkifejezését, alkalmazza a LUT-ot, majd az eredményt MJPEG formátumban továbbítja. Amíg az első igazított képkocka meg nem jelenik, a mozaik jelzi az aktuális állapotát: „Tömb előkészítése…”, „Igazítás kalibrálása…”, „Az első képkockára vár…”, vagy – ha a csempe automatikus igazítási próbálkozásainak ideje (kb. egy perc) lejárt – „Igazítás szükséges” felirattal és egy**Igazítás kalibrálása**gombbal.

A mátrixcsempe fps-kijelzése szintén jelzi a stream állapotát:**„Beállítások alkalmazása…”**, amíg a beállítások módosítása (formátum, binning, ROI) újraindítja a kamerákat,**„Leállt”**, amikor a mátrix aktív, de már nem továbbít képkockacsoportokat, és**„Leállt”**, ha leállt az adatfolyam – ebben az esetben csatlakoztassa újra a sorozatot. Ha a spektrális szétválasztásnak a kompozit képen látható sávot a képpontok jelentős részén (25 % vagy több) le kellett korlátoznia, a csempe a következő üzenetet jeleníti meg:**„Alacsony megbízhatóságú sávadatok · N%"** feliratot jeleníti meg: az adott sávra épülő indexértékek megbízhatatlanok (általában egy sáv túl kevés fényt kapott).

Hasznos tudnivalók a kombinált üzemmódról:

* A kompozíció a **fő**kamera képkockájához van igazítva. Az AE-ROI célzás és a pontmérés a kompozíción a fő kameránál pontos, a szolga kameráknál pedig hozzávetőleges; a**Split View** (tömbbeállítások → „Show member cameras”) használatával pixelpontos, kameránkénti csempéket kaphat anélkül, hogy további kamerakapcsolatokat kellene megnyitnia.
* **A**Display Layers**(tömbbeállítások; alapértelmezés szerint ki van kapcsolva) lehetővé teszi az előtér- és háttérréteg kiválasztását — bármely tagkamera vagy**Index**. Ha az előtér = Index, a LUT Min/Max határain kívül eső pixelek a háttérréteget jelenítik meg.
* A **Render felbontás** (alapértelmezés szerint 720p) beállítja az élő közvetítés magasságát *és* a mentett kompozit export méretét. A kameránkénti képek mindig teljes felbontásban kerülnek exportálásra.
* Az igazítás munkamenetenként kerül kiszámításra, és soha nem kerül mentésre. A Chloros **minden tömböt automatikusan igazít a csatlakozás után, mindkét megjelenítési módban**, miután a tagok automatikus expozíciója stabilizálódott (ez akár 5 percig is eltarthat). A sorozatbeállítások panelen található**Igazítás**szakaszban láthatók az RMS-maradékértékek és az**Igazítás újrakalibrálása** gomb — még azoknál a különálló sorozatoknál is, amelyeknek nincs összetett mozaikjuk.

## Felvétel: megfigyelés vs. elemzés

A sorozat rögzítési felületei egyértelműen két csoportra oszthatók: **megfigyelési minőségű**(a látható kép rögzítése) és**elemzési minőségű** (nyers adatok rögzítése, későbbi kalibrálás):

| Munkafolyamat | Minőség | Mit ment el | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Rögzítés**(állóképek) | Elemzés | Egy szinkronizált képkockacsoport minden átfutásonként; kameránkénti fájlok minden kiválasztott exportszinten (nyers/debayered/sugárzás/fényvisszaverődés/előnézet/index) + `.daq` sidecar |**Minden rögzítése** gomb + Rögzítési beállítások | `lattice array-capture` |
| **Indexvideó rögzítése** | Megfigyelés | A megjelenített élő kombinált indexkompozíció — 8 bites, előnézeti felbontás, beépített LUT. CLI-ből, nyitott GUI-nézet nélkül, a háttérprogram rendereli a tömb NDVI-jét (vagy `--expr`-et), miután a tömb összehangolásra került; ha a felvétel nem ír ki egyetlen képkockát sem, a művelet sikertelen | ● Indexvideó rögzítése (kombinált tömbök) | `lattice array-record` |
| **Nyers sorozat → videó létrehozása**| Elemzés | Nyers érzékelő-képkockák teljes rögzítési sebességgel + manifeszt + `.daq`, majd offline rekonstrukció kalibrált sugárzási / visszaverődési / index videóvá, a DAQ-leolvasásokhoz időben illesztve | ⦿ Nyers sorozat rögzítése →**Videó létrehozása** | `lattice array-burst` → `lattice array-build-video` |

Általános szabály: ha a pixelek *méréseket* szolgáltatnak, használjon rögzítést vagy sorozatfelvételt (elemzési minőségű); ha csak *meg szeretné nézni vagy bemutatni*, amit a szenzor látott, rögzítse az indexvideót (megfigyelési minőségű).

### Rögzítési beállítások (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->



A **Capture All** melletti fogaskerék megnyitja a Rögzítési beállítások panelt (ehhez nyitott projekt szükséges — a rögzítések abba kerülnek mentésre):

* **Rögzítési mód**:**Egyedi**(egy átfutás) /**Folyamatos**(egymást követő felvételek; rögzítési számmal (alapértelmezés: 1) vagy időtartammal (alapértelmezés: 10 másodperc) korlátozva) /**Intervallum** (időraffozás: N felvétel X időközönként, összesen Y-ig; alapértelmezés szerint 1 felvétel 5 másodpercenként 1 percig).
* **Exporttípusok kameránként**: Raw, Debayered, Radiance, Reflectance, Preview, Index — alapértelmezés szerint minden alkalmazható be van kapcsolva. A Radiance/Reflectance opciók el vannak rejtve az RGB-szűrős kameráknál; **a Reflectance csak akkor jelenik meg, ha a kamerának van DAQ fényérzékelője** (saját vagy a tömbből örökölt); az Indexhez konfigurált indexkifejezés szükséges.
* **Igazítás**(tömbönként, alapértelmezés szerint**be**): a tagfájlok exportját a tömb igazítási profiljához igazítja, így az exportált fájlok pixel-igazítottak lesznek. A nyers adatok mindig torzításmentesek maradnak, de a transzformációt a metaadatokban hordozzák. A tömb tagfájljait az egyes kamerák natív tájolásában mentik (a Tükör/Forgatás beállításokat a fájl rögzíti, de nem alkalmazza).
* **Leggyorsabb rögzítés** (kapcsoló): csak nyers adat + a hozzárendelt DAQ-érték + az ingyenes kombinált index-kompozit, kihagyva a kalibrációs számításokat a rögzítéskor a maximális sebesség elérése érdekében — a sugárzási erősséget/visszaverődést/indexet később kell újjáépíteni a mentett `.daq` fájlból.
* A kiválasztások a projekttel együtt megmaradnak. A rejtett vagy szüneteltetett kamerákat a rendszer kihagyja.
* **Folyamatos** mód a kameracsoportoknál az Chloros-háttérrendszerben fut (újdonság az 1.3.0-s verzióban): ha a listán szereplő összes kamera egy kameracsoporthoz tartozik, a háttérrendszer maga futtatja a rögzítési ciklust, egy csoportot írva, miközben kiválasztja a következőt, rögzítésenkénti kérés nélkül. A meghajtó telítődése leállítja a futást („Leállítva: meghajtó teli. N mentve, F sikertelen”). A magas rögzítési sebességekkel kapcsolatos Microsoft Defender-megjegyzést lásd a [Rögzítési beállítások és módok](capture.md#capture-modes) című részben.

Az ehhez tartozó CLI-parancs (ugyanaz a háttérrendszer-végpont, ugyanaz a szemantika):

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

A rögzítések TIFF-tömörítése: `deflate` (veszteségmentes, alapértelmezett) vagy `none` — a teljes jelzőtáblák, a rögzítési mappa felépítése és az újrafeldolgozási szabályok a [CLI-referenciában](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess) találhatók.

## DAQ fényérzékelő párosítása

A reflexió- és megvilágításkorrigált előnézetekhez lefelé irányuló fényadatokra van szükség egy DAQ-érzékelőtől (amelyet a **Fényérzékelők** fülön kell csatlakoztatni):

* Az oldalsáv **tömb sorában**megjelenik egy**„DAQ · be/ki” gomb** — *be* állapotban van, ha tömb szintű fényérzékelő van beállítva **vagy** bármelyik tagkamera rendelkezik saját érzékelővel; a gomb felirata pontosan felsorolja, melyik érzékelő melyik kamerát táplálja.
* A beállítás a tömb egészére vonatkozik: a tömbbeállítások → **Környezeti fényérzékelő**→**Fényérzékelő** legördülő menüben adható meg. A kiválasztás a projekttel együtt megmarad, átterjed minden tagkamerára, és az egyes kamerák továbbra is felülírhatják saját érzékelőjükkel.
* Az alatta lévő állapotsor jelzi az aktuális állapotot: **Ki**→ „Az első spektrumra várakozás…” →**„Aktív — a rendszer összes kamerájának megvilágítását korrigálták”** → vagy, ha az elmúlt 3 másodpercben nem érkezett friss spektrum, egy elavult értesítés — továbbra is az utolsó leolvasott értéket használja a rendszer (a leolvasott értékek soha nem járnak le a rögzítési útvonalon).

Érzékelő hozzárendelése esetén: elérhetővé válik a „Reflectance” (Fényvisszaverődés) exporttípus, az élő előnézetek megvilágítás-korrigáltak, a prediktív automatikus expozíció felhasználhatja a spektrumot, és minden fényvisszaverődés-felvétel a ténylegesen használt DAQ-leolvasást **`.daq` fájl** néven írja a felvételek `daq/` mappába, így a rögzítés később újra feldolgozható. Mellette egy `.daq.cameras.json` fájl megnevezi azokat a kamerákat, amelyek képkockáit az adott mérési érték leírja; újrafeldolgozáskor a Chloros a mérési értéket kizárólag az említett kamerák képeihez rendeli hozzá.

## `array-connect` CLI-opciók

| Opció | Alapértelmezett | Leírás |
| --- | --- | --- |
| `--serials SN1,SN2,…` | az összes LATTICE kamera automatikus felismerése (legalább 2 szükséges) | **Az első sorozatszám a MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | GPIO szinkronizáló vonal. |
| `--target-fps F` | auto | A MASTER trigger tűzgyakorisága. |
| `--binning {1,2,4}` | auto | Hardveres binning. Az „auto” beállítás esetén a teljes érzékelő (binning nélkül) használata érvényes, ha bármely képkockasebesség ezt támogatja; a „connect never” beállítás soha nem alkalmaz binninget, csak a sebesség fenntartása érdekében. |
| `--packed {auto,on,off}` | `auto` | [Tömörített 12 bites adatátvitel](#packed-12-bit-transport): Az `auto` csak akkor csomagol, ha ez előnyös, és a gazdagép képes a kicsomagolásra. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | auto | A szinkronizációs szint-választó szakértői felülírása. |
| `--wire-ceiling-mbps MB_PER_S` | automatikusan felismerve | A gazdagép vezeték-kapacitása MB/s-ban — a **Wire Budget** mező CLI-formátuma. Csökkentse ezt az értéket, ha a tömb sérült kereteket jelent. A projekttel együtt mentésre kerül, így egy későbbi újracsatlakozáskor visszaáll. |
| `--no-recommend` | ki | A hálózati elemzési lépés kihagyása. |
| `--no-ptp` | ki | A PTP letiltása (ebben az esetben a kamerák közötti időbélyegek nem összehasonlíthatók). |

Az `lattice array-list`, `array-status` és `array-disconnect` parancsok a tartós munkamenetet kezelik. A teljes alparancs-referencia, beleértve az igazítást (`align-calibrate` / `align-apply`) és a hálózati eszközöket, a [CLI-referencia § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); az SDK-megfelelők (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) a [SDK-referenciában](../reference/sdk-reference.md) találhatók. Python ból a vezetékes adatforgalmi keret az `connect_array(..., wire_ceiling_mbps=120)`, az élő, sérült és soha meg nem érkezett adatok felosztása pedig az [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames) alatt található.
