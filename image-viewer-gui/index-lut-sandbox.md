# Index/LUT Sandbox

Az Index/LUT Sandbox az Chloros Image Viewer oldalsávjában található interaktív munkaterület. Kiválaszthat egy képletet, hozzárendelheti a kamera csatornáit, színezheti azt színátmenettel, és beállíthatja az értéktartományt — mindezt valós időben, miközben a kép folyamatosan frissül. Az 1.2.0-s verziótól kezdve **elmentheti a létrehozott beállításokat** egy képre vagy az egész projektre vonatkozóan, újbóli feldolgozás nélkül.

## Mire szolgál a Sandbox

| Index/LUT Sandbox (interaktív)        | Projektfeldolgozás (kötegelt)       |
| -------------------------------------- | -------------------------------- |
| Egyszerre egy kép, azonnali visszajelzés  | Az egész adathalmaz egy futtatásban     |
| Kísérleti és iteratív             | Előre beállított paraméterek          |
| Valós időben renderel; csak kérésre menti  | Mindig elmenti a végleges fájlokat      |
| Tökéletes a megfelelő beállítások megtalálásához | A legjobb, ha a beállítások már véglegesek |

{% hint style="success" %}
**A szokásos munkafolyamat**: addig finomítsd a beállításokat a Sandboxban, amíg a vizualizáció nem úgy néz ki, ahogy szeretnéd, majd vagy közvetlenül a Sandboxból exportálj, vagy másold át ugyanazokat az index- és LUT-beállításokat a [Projektbeállításokba](../project-settings/project-settings.md), hogy a következő feldolgozási futtatás minden képre ráalkalmazza azokat.
{% endhint %}

***

## A Sandbox megnyitása

1. Kattintson egy képre a rácsban — az **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> fülön teljes képernyős módban nyílik meg
2. Kattintson az **Image Viewer** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> ikonra a bal oldali oldalsáv kinyitásához, ha az még nincs megnyitva
3. Válasszon ki egy többsávos réteget a jobb felső sarokban található réteg legördülő menüből — általában a **RAW (Reflectance)** a megfelelő választás, mivel a kalibrált reflexió alapján számított indexértékek képek között összehasonlíthatók

Az oldalsávon a következő elemek jelennek meg, fentről lefelé:

* a kép neve és a kamera modellje
* az **Export/Save Image(s)** gomb — akkor jelenik meg, ha az Index vagy a LUT jelölőnégyzet be van jelölve
* az **Index**és**LUT** jelölőnégyzetek
* az index konfigurációs panel
* a **Kurzorértékek** panel az értékkiolvasással, a hisztogrammal és a GSD-vezérléssel

{% hint style="warning" %}
**Monokróm kamerák esetén nem elérhető.** Egycsatornás LATTICE M3M-képen mindkét jelölőnégyzet le van tiltva, a segítő szöveg pedig így szól: _„Monokróm (M3M) érzékelők esetén nem elérhető”_ — egycsatornás kép esetén a többsávos index nincs definiálva. Az M3M kamerákból származó indexek kiszámításához egyesítsen kettőt vagy többet egy igazított többsávos képcsomagba, és használja a LATTICE index-motort.
{% endhint %}

***

## Index alkalmazása

1. Jelölje be az **Index** jelölőnégyzetet az oldalsáv tetején
2. Válassza ki a kamera szűrőjét a bal oldali legördülő menüből (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Válasszon ki egy indexképletet a jobb oldali legördülő menüből — 27 beépített képlet, valamint az Ön által elmentett egyéni képletek
4. A képlet az alábbi matematikai kifejezésként jelenik meg, minden sávhelyen egy üres körrel. **Húzzon egy színes csatornakört egy sávhelyre** a hozzárendeléshez
5. Amint a képlet által használt összes sávhely hozzárendelve van, a kép frissül, és megjeleníti az indexértékeket
6. Vigye az egérmutatót a kép fölé az értékek leolvasásához; a **Cursor Values** panel hozzáad egy indexsort a kurzor alatt lévő értékkel

Kattintson duplán egy összekapcsolt helyre annak törléséhez. A hiányos képlet a húzás közbeni normális állapot, nem hiba — a kép egyszerűen nem frissül, amíg a képlet nem teljes.

A csatornakörök színkóddal vannak jelölve: piros = Red, zöld = Green, kék = Blue, narancssárga = Orange, cián = Cyan, lila = NIR, magenta = RE. Ugyanezeket a színeket használják a csatornapontokhoz és a hisztogramgörbékhez a „Cursor Values” panelen.

### NDVI példa

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

A teljes képlet-referenciához – mindhárom előre beállított lista és az egyes nevek használati helyei – lásd a [Multispektrális indexképletek](../project-settings/multispectral-index-formulas.md) című részt.

### Az Index jelölőnégyzet bejelölve, de LUT nélkül

A kép **szürkeárnyalatos** formában jelenik meg, a két küszöbérték között kiterjesztve. Ez szándékos: az indexkép skaláris adat, és a szürkeárnyalatos megjelenítés adja vissza a leghitelesebben. Ha színt szeretne, adjon hozzá egy LUT-ot.***

## LUT-okkal (Look-Up Tables) való munka

A **Look-Up Table** (LUT) az indexértékeket színekhez rendeli: bemenet NDVI 0,65, kimenet egy adott zöld szín. Nem változtatja meg az adatokat — csak azt, ahogyan azokat értelmezzük.

### LUT hozzáadása

1. Kattintson a képlet alatt található **„+ LUT hozzáadása”** gombra (<img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line">)
2. Válasszon ki egy színátmenetet
3. Állítsa be a kivágás minimális és maximális értékét
4. Válasszon kivágási módot
5. Jelölje be az oldalsávon a **LUT** jelölőnégyzetet a rendereléshez

A LUT jelölőnégyzet addig le van tiltva, amíg ténylegesen nem konfigurálnak egy LUT-ot az indexen.

### Színátmenet kiválasztása

Vigye az egérmutatót a **színátmeneti sávra**az előre beállított listák megnyitásához — az Chloros**hét** színátmeneti beállítást tartalmaz:

| # | Színátmenet                            | Alak                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Sárga → Green (**alapértelmezett**)  | Szétterjedő — megfelel a növényzetre vonatkozó általános intuíciónak: zöld = egészséges |
| 2 | Lila → Sárga → Green             | Divergáló, jellegzetes alacsony tartománnyal                                  |
| 3 | Barna → Fehér → Blue                | Divergáló, világos középpont körül                                   |
| 4 | Fekete → Lila → Rózsaszín → Halványsárga | Szekvenciális, sötétről világosra                                           |
| 5 | Red → Sárga → Blue                 | Eltérő, egy világos középpont körül                                   |
| 6 | Lila → Blue → Green → Sárga      | Sorozatban, sötétről világosra                                           |
| 7 | Orange → Fehér → Lila             | Egy világos középpont körül divergáló                                   |

A **divergáló**színátmenet semleges színt helyez az ablak közepére, ami jól olvasható, ha a középpontnak jelentése van (küszöbérték, kiindulási dátum). A**szekvenciális** színátmenet monoton módon halad a sötétből a világosba, ami jól olvasható olyan mennyiségek esetén, amelyeknél csak a „több” és a „kevesebb” fogalmak léteznek.

Minden előre beállított színátmenet hét színállomással rendelkezik. Kattintson egy előre beállított színátmenetre, és a kép azonnal frissül (ha a LUT jelölőnégyzet be van jelölve).

### A színállomások szerkesztése

A színátmenet-sáv alatt egy sor színminta található, egy-egy minden színállomáshoz:

* **Szín módosítása**: kattintson egy színmintára a színválasztó megnyitásához (színkerék, RGB/HSV csúszkák, vagy hexadecimális kód, például `#FF0000`)
* **Állás hozzáadása**: kattintson a sor végén található**+** gombra — egy fehér állás kerül hozzáadásra
* **Állás eltávolítása**:**kattintson duplán** a színmintára
* **A szerkesztett színátmenet megőrzése**: kattintson a színátmenet-sáv melletti mentés ikonra, hogy a szerkesztett színátmenetet felvegye az előre beállított listába, így később újra kiválaszthatja

Az indexen beállított színátmenet az adott indexszel együtt kerül elmentésre a projekt beállításaiban, így a projekt bezárása és újbóli megnyitása után is megmarad.

**Kevesebb állomás**jól elkülönülő zónákat eredményez, amelyek osztályozásként értelmezhetők;**több állomás** sima, szinte fotószerű átmeneteket eredményez. Három–öt állomás prezentációs diákhoz és osztályozási térképekhez alkalmas; hat–tíz általános elemzéshez; tizenöt vagy több részletes vizsgálathoz és publikációs ábrákhoz.

### Az értéktartomány beállítása

A küszöbérték-szabályozó egy −1 és +1 közötti tartományban mozgó **kétfogantyús csúszka**, amelynek mindkét végén egy-egy szerkeszthető szövegmező található a pontos értékek megadásához, valamint egy**AUTO** gomb.

* Húzza bármelyik fogantyút, vagy írjon be egy számot a megfelelő mezőbe, majd nyomja meg az Enter billentyűt
* Az **AUTO**gomb a tartományt a kép érvényes indexértékeinek**

2. és 98. percentilisére** állítja be – ez egy jó kiindulási pont, amely figyelmen kívül hagyja a kiugró értékeket. Az Chloros az eredményt adaptív módon kerekíti: nagyon szűk tartomány esetén 4 tizedesjegyre, szűk tartomány esetén 3-ra, egyéb esetben 2-re
* Bármely kézi beállítás elsőbbséget élvez az AUTO beállítással szemben, amíg újra meg nem nyomja az AUTO gombot

Példa az NDVI ablakokra:

| Cél                                    | Min  | Max |
| --------------------------------------- | ---- | --- |
| Minden megjelenítése                         | −1,0 | 1,0 |
| Csak növényzet, talaj és víz kizárásával | 0,2  | 0,9 |
| Csak egészséges növényzet                 | 0,5  | 0,9 |
| Stressz kiemelése                        | 0,2  | 0,5 |

Az ablak szűkítése növeli a kontrasztot a vizsgált területen belül, és minden mást a tartományon kívülre tol — ahol a **Kivágási mód** határozza meg, mi történik velük.***

## Kivágási módok

Amikor egy pixel indexértéke a min/max ablakon kívülre esik, a Kivágási mód határozza meg, hogyan kerül megrajzolásra.

| Legördülő menü címke                  | Tárolt érték      | A tartományon kívüli pixelek a következőképpen kerülnek megrajzolásra                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Minimum és Maximum** (alapértelmezett) | `clip`            | A színátmenet legközelebbi végszíne — a minimum alatti értékek az első színt, a maximum feletti értékek az utolsót veszik fel |
| **Átlátszó háttér**      | `transparent`     | Teljesen átlátszó (valódi alfa)                                                                                                  |
| **Indexelt háttér**| `indexColor`      | Szürkeárnyalatos, a kép**teljes** indextartományára kiterjesztve, így a tartományon kívüli struktúra is szürkében látható                |
| **Eredeti háttér**         | `backgroundColor` | Maga az alatta lévő kép, így a színréteg a valódi jelenet tetején helyezkedik el                                                |

| Mód                       | Legalkalmasabb                               | Megjelenés                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Minimális és maximális**      | Teljes adatmegjelenítés, tudományos elemzés | Minden pixel színes                      |
| **Átlátszó háttér** | GIS-rétegek, értéktartományok elkülönítése   | Szín az ablakon belül, semmi kívül |
| **Index háttér**       | Kiemelés az adatkontextus megőrzése mellett    | Színes belül, szürke kívül               |
| **Eredeti háttér**    | Jelentések és prezentációk              | Színes belül, fénykép kívül         |

{% hint style="info" %}
**Az adat nélküli pixelek minden üzemmódban mindig átlátszóak.** Azokat a pixeleket, amelyek indexe nem véges (0/0 osztás), vagy pontosan −1,0 vagy +1,0 (telítettségi jelzők, amikor az egyik sáv értéke nulla, a másiké pedig nem), adat nélkülinek tekintjük, nem pedig szélsőséges értéknek. Ezzel a túlexponált fényes részek és a sötét árnyékok nem kerülnek be a színskálába, hanem nem kerülnek ábrázolásra a képkocka legszélsőségesebb értékeként. Ugyanez a szabály határozza meg, hogy mely pixelek táplálják az AUTO küszöbértékeket és az index-hisztogramot, így mindhárom érték egyezik.
{% endhint %}

Az átlátszóság megmarad, ha az exportot PNG formátumban írják ki. JPG formátumban ez nem ábrázolható.

***

## Értékek leolvasása a beállítás közben

A konfigurációs panel alatt található **Kérőjel-értékek** panel a Sandbox mérőeszköze:

* Vigye a kurzort a kép fölé, és olvassa le a csatornánkénti forrásértékeket, valamint az indexértéket a saját sorában
* Kapcsolja be a hisztogram felett található **INDEX** gombot, hogy megtekintse az indexértékek eloszlását a képkockában, ahol a két klipküszöbérték narancssárga szaggatott vonalakkal, a kurzor értéke pedig fehér vonallal jelenik meg — ez a leggyorsabb módja annak, hogy kiválasszon egy olyan ablakot, amely ténylegesen tartalmazza az adatait
* Kapcsolja be a **CURSOR** gombot, hogy a mutató alatt lévő értékeknél jelzővonalak jelenjenek meg
* Nagyítson 60×-nál nagyobb mértékben (kevesebbre, ha GSD-blokkméret van beállítva), hogy az egyes megjelenített pixelek lebegő értékkel legyenek kiemelve

Gyakorlati eljárás:

1. Jegyezze fel az értékeket az egészséges növényzet, a stresszhatásnak kitett növényzet, a csupasz talaj és a víz felett
2. Nézze meg, hol helyezkednek el ezek a csoportok az indexhisztogramon
3. Állítsa be a min/max értékeket úgy, hogy azok az Ön számára fontos csoportot keretezzék
4. Válasszon egy kivágási módot – az _Eredeti háttér_ lehetőség megőrzi a jelenet láthatóságát a kivágott rész körül

***

## Exportálás a Sandboxból

A fentiak mind élő előnézetnek számítanak, amíg el nem menti őket. Az oldalsáv tetején található **Kép(ek) exportálása/mentése** gomb megnyit egy ablakot, amely az oldalsáv fölé csúszik (nem fedi el a képet, így továbbra is láthatja, miről dönt).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Beállítások

| Beállítás                          | Hatás                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Alkalmazás az aktuális képre**      | Pontosan a megjelenített képet menti el ezekkel a beállításokkal                                                                                                |
| **Alkalmazás az összes projektképre** | Ugyanazt a konfigurációt futtatja le a projekt minden képén. Azokat a képeket, amelyek nem tartalmazzák az indexhez szükséges sávokat, a program kihagyja, és nem tekinti hibásnak |
| **Index/LUT gradiens sáv**      | Exportonként egy külön jelmagyarázó képet is létrehoz, az értéktartomány feliratozásával                                                                     |
| **Index hisztogram**             | Exportonként egy külön hisztogramképet is létrehoz, amelyen láthatóak az adatok minimális és maximális értékei, valamint a kivágási küszöbértékek                                               |

Ha a kép fül **GSD blokkmérete** 1 felett van, a panel ezt jelzi, mielőtt véglegesítené: az exportálás azt menti el, amit éppen lát, beleértve a blokkátlagolást is. Ha teljes felbontást szeretne, először állítsa vissza a GSD vezérlőt 1-re.

### Hová kerülnek a fájlok

Az **Export**gombra való minden kattintás egy**új, soha nem újrahasznosított mappát** hoz létre:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Példák: `Sandbox_Exports/NDVI_LUT_001/`, majd a következő futtatásnál `Sandbox_Exports/NDVI_LUT_002/`. A számozás a lemezen már meglévő elemek átvizsgálásával történik, így az újraindítások után is megmarad, és a kézzel törölt mappákat sem érinti. Soha semmi nem kerül felülírásra — a Sandbox lényege éppen az, hogy az egyes kísérleteket az előzővel hasonlítsa össze.

A mappában, képek szerint:

| Fájl                                                   | Tartalom                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | A renderelt kép, pixelről pixelre pontosan az, amit a néző látott |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | A színátmeneti sáv kiegészítő fájlja, ha kérték                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | Az indexhisztogram kiegészítő fájlja, ha kérték                  |

A két kiegészítő kép mindig **teljes felbontásban** kerül rögzítésre, még akkor is, ha a fő kép blokkátlagolt: a blokkméret megegyezik a kijelző felbontásával, és mindkét kiegészítő kép a valódi, pixelenkénti indexértékeket tartalmazza. Ezenkívül többet jelenítenek meg, mint a képernyőn látható változatok — mindkettő feltünteti a kiterjesztési ablakot _és_ a valós adatok minimumát és maximumát, így a mentett jelmagyarázat hónapokkal később is olvasható, anélkül, hogy a projektet megnyitnánk.

### Előrehaladás és eredmények

A teljes projekt exportálása percek alatt lezajlik, így a futtatás nem blokkolja a rendszert, hanem egy élő előrehaladási csatornán keresztül jelenti vissza az állapotot:

* Egy előrehaladási sáv az `current / total` kódot és az írás alatt álló fájlt jeleníti meg
* Befejezéskor az ablak jelzi, hány képet exportáltak, hányat hagytak ki, valamint a kimeneti mappa elérési útját
* A kihagyott képek az ok megjelölésével kerülnek felsorolásra (legfeljebb öt jelenik meg, utána egy „+N további” sor). A leggyakoribb ok az, hogy egy réteg nem rendelkezik az indexhez szükséges csatornákkal
* Ha a projektben **egyetlen** kép sem tudja használni az indexet, a futtatás hibaüzenetet jelez, ahelyett, hogy üres mappát hagyna hátra

Egyszerre csak egy sandbox-export futhat. Ha az egyik futás közben megpróbálunk egy másodikat elindítani, a rendszer egy egyértelmű üzenettel elutasítja a műveletet, ahelyett, hogy két futtatás versengjen ugyanazon a projektfájlért.

### A rács felveszi a futtatást

Minden befejezett futtatás saját gombként jelenik meg az [képrács](image-grid.md) eszköztáron, `<IndexName> <Index|LUT> <NNN>` felirattal. Így lehet összehasonlítani a futtatásokat: végezzen két exportálást különböző gradiensekkel vagy küszöbértékekkel, majd váltson a rács két gombja között.

***

## Egyéni indexképletek (Chloros+)

{% hint style="info" %}
**Hol hozhatók létre**: a Sandbox oldalsávjában, vagy a feldolgozás előtt a**Projektbeállítások** menüben. Mindkettő ugyanabba a projekt szintű listába ír.
{% endhint %}

1. Nyissa meg az egyéni képlet-kalkulátort az indexképlet legördülő menüből (ehhez be kell jelentkezni egy megfelelő Chloros+ előfizetéssel)
2. Írja be a képletet a **sáv-hely szimbólumok** használatával: `x`, `y`, `z`, `a`, `b`, `c` szimbólumokat – ezek nem sávnevek
3. Rendelkezésre álló operátorok: `+`, `-`, `*`, `/`, `^` és `()` csoportosításhoz
4. Elérhető függvények: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Nevezd el és mentsd el — ez megjelenik a képlet legördülő menü alján, és a csatornakörök húzásával rendelheted hozzá a helyeket, pontosan úgy, mint egy beépített előre beállított értéknél

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**Az egyéni képletek kizárólag a grafikus felületen használhatók.** Az CLI/SDK `--indices` opció kibővíti a 22 beépített előre beállított nevet, és minden mást – beleértve az egyéni képleteket is – figyelmen kívül hagy. Egyéni képlet tömeges feldolgozásához konfigurálja azt a Projektbeállításokban, majd futtassa a feldolgozást, vagy használja a Sandbox „Alkalmazás az összes projektképre” exportfunkcióját.
{% endhint %}

***

## Hibaelhárítás

### „Ennek a rétegnek nincsenek meg azok a csatornái, amelyekre ennek az indexnek szüksége van”

A képlet olyan csatornahelyzetet olvas be, amely az aktuális rétegben nem található meg — például háromrészes indexet egy egy- vagy kétcsatornás fájlban. Váltson többsávos rétegre (reflektancia vagy debayered), vagy válasszon olyan indexet, amely megfelel a fényképezőgép szűrőjének.

### „Nem sikerült elérni a képfeldolgozó háttérrendszert”

A háttérrendszer nem válaszol. Ellenőrizze a Napló fület; ha a háttérprogram újraindul, a Sandbox magától helyreáll, amint az újra elérhetővé válik.

### A kép nem változott, amikor egy kört húztam

A képlet még nem teljes. A hiányos képletet a rendszer normál húzás közbeni állapotként kezeli – semmi nem kerül renderelésre, és hibaüzenet sem jelenik meg. Töltse ki a képlet által használt összes mezőt.

### Az egész kép egy színű

A klipablak valószínűleg jóval az adatokon kívül van. Nyomja meg az **AUTO**gombot, hogy a 2. vagy 98. percentilishez igazítsa, vagy kapcsolja be az**INDEX** hisztogramot, hogy lássa, hol helyezkednek el valójában az adatok.

### Az exportált színek nem egyeznek azzal, amit láttam

Egyezniük kellene – az exportálási útvonal szándékosan tükrözi az élő előnézetet, beleértve a kivágási mód alfa-értékét is, és a blokkátlagolást pontosan úgy alkalmazzák a színezés _után_, ahogyan azt a néző is teszi. Ha eltérnek, ellenőrizze, hogy a GSD blokkméret nem változott-e a megtekintés és az exportálás között.

***

## Következő lépések

* [**Képrétegek**](image-layers.md) — melyik rétegen futtasson indexet, és mit jelentenek az értékek
* [**Kép teljes képernyős megnyitása**](opening-an-image-full-screen.md) — a kurzor kijelzése, a hisztogram és a GSD-vezérlés részletesen
* [**Multispektrális indexképletek**](../project-settings/multispectral-index-formulas.md) — minden előre beállított érték, minden felületen
* [**Projektbeállítások**](../project-settings/project-settings.md) — a kiválasztott beállítások rögzítése egy feldolgozási futtatásba
