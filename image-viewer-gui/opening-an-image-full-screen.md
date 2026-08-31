# Kép teljes képernyős megnyitása

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>Teljes képernyőn megnyitott kép, a rétegválasztóval a jobb felső sarokban</p></figcaption></figure>

Az Chloros Képmegjelenítő egy teljes képernyős felület a képek megtekintéséhez, vizsgálatához és méréséhez. Itt olvashatja le a **valódi pixelértékeket** — csatornánkénti DN-értéket, százalékos visszaverődést vagy sugárzási intenzitást W/m²/sr/nm-ben — ahelyett, hogy a képernyőn megjelenő, kinyújtott előnézetet nézné.

## A Képmegjelenítő eléréséhez

### A Fájlkezelőből

1. Nyissa meg a **Fájlkezelő** fület <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Kattintson bármely **miniatűrre** a [képrácsban](image-grid.md)
3. A kép teljes képernyős módban megnyílik a **Képnézegető** fülön

A kép azon a terméken nyílik meg, amelyet a rács éppen megjelenített. Ha a rács beállítása `RAW (Reflectance)`, akkor az a réteg jelenik meg.

### A Képnézegető oldalsávjának megnyitása

Kattintson a bal oldali oldalsávon található **Képnézegető** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> ikonra az elemzőpanel kinyitásához. A panel felülről lefelé a következőket tartalmazza:

* a kép nevét és a kamera modelljét
* az **Exportálás/Kép(ek) mentése** gombot (csak akkor, ha index vagy LUT aktív)
* az **Index**és**LUT** jelölőnégyzetek, valamint az index konfigurációs panel — lásd [Index/LUT Sandbox](index-lut-sandbox.md)
* a **Kurzorértékek** panel: csatornánkénti leolvasás, réteghisztogram és a GSD vezérlő***

## Navigálás és nagyítás

### Képek között való böngészés

* **Következő kép**: a → gomb, vagy a**→** (jobb nyíl) billentyű
* **Előző kép**: a ← gomb, vagy a**←** (balra nyíl) gomb
* **Ugrás egy adott képre**: térjen vissza a rácsra, és kattintson a kép miniatűrjére

A nagyítás és a pásztázás a képek közötti váltás során is megmarad, így a képsorozatot végiglapozhatja úgy, hogy a kép ugyanazon részén marad.

### Nagyítás

A nagyítás az **egérkerékkel** vezérelhető, 15%-os lépésekben, a kurzorhoz rögzítve — a mutató alatti pont a mutató alatt marad. A tartományt a kép és az ablak mérete határozza meg: a „ablakhoz igazítás” beállításnál tovább nem lehet kicsinyíteni, a felső határt pedig a kép natív felbontása határozza meg.

A teljes képernyős nézőben nincsenek külön nagyítás gombok. (A rácsban a **Ctrl + `+` / `−`** billentyűkombinációval a miniatűrök mérete módosítható — ez egy másik vezérlő.)

### Képmozgatás nagyításkor

Kattintson a bal egérgombbal a képre, tartsa lenyomva, majd húzza. A képmozgatás korlátozott, így a képet nem lehet a képernyőről kihúzni.

### Pixelenkénti vizsgálat nagy nagyításnál

Amint a tényleges nagyítás meghaladja a **60×**-ot, az Chloros kiemelő keretet rajzol a kurzor alatt megjelenő egyes pixelek köré, és egy lebegő értéket jelenít meg azok mellé.

A „tényleges” nagyítás a GSD-blokk méretét veszi figyelembe: 8-as blokkméret esetén a kiemelés nem 60×-os, hanem 7,5×-os nagyításnál jelenik meg, mivel egy megjelenített pixel már 8 × 8 forráspixelt foglal el. Ha a nagyítást a küszöbérték alá csökkenti, a kiemelés eltűnik.

### Billentyűparancsok

| Gomb                             | Hol       | Művelet                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Teljes képernyő | Következő kép                          |
| **←**                           | Teljes képernyő | Előző kép                      |
| **Ctrl + R**                    | Teljes képernyő | Az index/LUT sandbox visszaállítása         |
| **Ctrl + `+`**/**Ctrl + `=`** | Rács        | Nagyobb miniatűrök (gombnyomásonként 4 px)  |
| **Ctrl + `−`**                  | Rács        | Kisebb miniatűrök (4 px gombnyomásonként) |***

## A kurzor értékei

Ha a kurzort a kép fölé viszi, a **Kurzorértékek** panel megjeleníti az alatta lévő összes csatorna értékét.

{% hint style="success" %}
**Ezek a fájl tényleges értékei.** A képernyőn látható vászon egy 8 bites, kinyújtott előnézet, amely nem tudja ezeket az értékeket megadni, ezért az Chloros a tényleges termékfájlból vesz mintát a kijelzéshez. Ezért jeleníti meg egy 12 bites nyers képkocka a 255 feletti értékeket, és ezért jeleníti meg egy float32 sugárzási réteg a fizikai egységeket.
{% endhint %}

### Az oszlopok jelentése

A panel a megtekintett réteghez igazodik:

| Megtekintett réteg              | Megjelenített oszlopok    | Megjegyzések                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Reflektancia                        | **DN**és**%** | A százalékot a fájl saját skálája alapján számítja ki — lásd alább                                      |
| Sugárzás                           | **W/m²/sr/nm**   | Float fizikai értékek; nincs DN oszlop, mert a DN itt értelmetlen                           |
| Nyers / Debayered / előnézet / JPG    | **DN**           | Egész számú digitális értékek                                                                         |
| 32 bites százalékos visszaverődési értékek exportja | csak **%**       | A tárolt lebegőpontos érték nem DN, ezért egész számra kerekítve értelmetlen `0` vagy `1` érték jelenne meg |

Minden sor a fényképezőgép szűrőjének csatornájának nevével van jelölve — `Red / Green / NIR` az RGN-hez, `Orange / Cyan / NIR` az OCN esetében, `NIR / Green / Blue` az NGB esetében, `Red / Green / Blue` az RGB esetében, valamint az RE, NIR és a mono M3M kamerák esetében az egycsatornás név. Minden címke egy színes pontot tartalmaz, amely megegyezik az indexképlet-szerkesztőben használt csatornakörökkel.

A mentett **index- és LUT**-képek különleges esetet jelentenek: spektrális sávok helyett színképi komponenseket tartalmaznak, ezért soraik `Red / Green / Blue` (vagy egycsatornás indexfájl esetén `Index`) jelöléssel szerepelnek, a kamera szűrőnevei helyett nevei helyett.

Amikor egy index aktív a sandboxban, a csatornák alatt megjelenik egy további sor, amely a kurzor helyén lévő **indexértéket** mutatja, az index nevével és egy fehér ponttal, amely megegyezik a hisztogramon található jelölőjével.

### A reflexió százalékos értéke az egyes fájlok saját skáláját használja

{% hint style="warning" %}
**Ne feltételezzük, hogy 65535 = 100%.** Az Chloros a fényvisszaverődést különböző skálákon tárolja attól függően, hogy melyik kamera állította elő, és a néző fájlonként meghatározza a helyes értéket.
{% endhint %}

| Forrás                  | A 1,0-es reflektanciának megfelelő DN | Azonosítás módja                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | Minden LATTICE-reflektancia-exportba be van írva az `Chloros:PixelScale=32768` XMP-címke. A 2×-es tartalék lehetővé teszi, hogy a fájl 1,0 feletti ρ-értékeket is tartalmazzon csúszás nélkül |
| **Survey3**|**65535**                      | Nincs Chloros XMP méretarány-címke — az Survey3 kalibráció a ρ × dtype-max értéket írja be, és 1,0-nál levágja                                                               |

A nézőprogram, az index/LUT sandbox és az index-export egyaránt ugyanazon egyetlen implementáción keresztül határozza meg a skálát, így a kurzor helyén leolvasott érték megegyezik azzal az értékkel, amelyet az index-számítás is használt.

Két fontos következmény:

* Egy **32-bites százalékos**TIFF a DN/65535-öt float típusúként tárolja, míg egy**8 bites** PNG/JPG export a DN × 255/65535 értéket tárolja — a néző mindkettőt visszaalakítja, mielőtt százalékot jelenít meg.
* Egy eset nem állítható vissza: egy **8-bites forrásból származó felvétel**8-bites TIFF** exportja nem kerül átméretezésre, hanem 0–255-re van korlátozva, és szándékosan nem tartalmaz méretarány-címkét. Ezeknél a fájloknál a panel csak a DN-értéket jeleníti meg, százalékos oszlop nélkül. Ez a valós helyzet, nem pedig hiba.***

## A réteg hisztogramja

A kurzor sorai alatt látható a megtekintett réteg élő hisztogramja, **256 osztással**. Alapértelmezés szerint egy kombinált görbét rajzol, súlyozott `(R + 2G + B) / 4` – ugyanazt a mérési teret használja, mint a LATTICE kamera hisztogramjai. Az**RGB** bekapcsolásával a görbe helyébe csatornánkénti görbék lépnek a csatornák színeiben, amelyek additív keveréssel jelennek meg, így az átfedések is jól olvashatók maradnak. A monokróm rétegeknél mindig az egyetlen görbe jelenik meg.

A vízszintes tengely a réteg saját mértékegységében van megadva:

| Réteg       | Tengely mértékegysége  | Tengely maximuma                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Fényvisszaverődés | százalék    | 125% — a termék tartaléka lehetővé teszi a 1,0 feletti ρ értéket           |
| Sugárzás    | W/m²/sr/nm | A képkocka saját csúcsértéke, két számjegyre felfelé kerekítve |
| 8 bites adat  | DN         | 255                                                        |
| 12 bites adat | DN         | 4095                                                       |
| 16 bites adat | DN         | 65535                                                      |

Ha a tengely DN-ben van, és az említett három felső határ egyikén áll meg, az Chloros azt is tudja, hogy milyen bitmélységű az éppen megtekintett kép.

A hisztogram felett három gomb található:

| Gomb     | Alapértelmezett | Hatás                                                                                                                                                                                                                                                                                   |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **KURZOR** | Be      | A fenti sorokban feltüntetett pontos értékeken jelölővonalakat rajzol a hisztogramra, így láthatja, hogy a kurzor alatt lévő pixel hol helyezkedik el a képkocka eloszlásában. RGB módban csatornánként egy jelölő jelenik meg, saját színnel; egyébként egyetlen fehér jelölő látható a kombinált értéknél |
| **INDEX**| Be      | Csak akkor jelenik meg, ha index aktív. Átváltja a hisztogramot a forrás sávokról az**indexérték-eloszlásra**, ahol a két vágási küszöbérték narancssárga szaggatott vonalakkal, a kurzor indexértéke pedig fehér vonallal jelenik meg                                                          |
| **RGB**| Ki     | Átvált a kombinált görbéről a csatornánkénti görbékre. Monokróm érzékelő esetén ez a gomb**MONO** feliratot jelenít meg, és letiltva van — csak egy csatorna jeleníthető meg                                                                                                                                  |

A hisztogramot a **látható blokkok** alapján számítja ki, nem a mögöttük lévő forráspixelek alapján: a GSD-blokkméret megváltoztatásával az eloszlás újraszámításra kerül, így a hisztogram, a kurzorjelölő és a megjelenített kép mindig egyezik.***

## GSD-blokkméret

A panel alján található a **GSD (px)**vezérlő: egy számmező, egy**1-től 256-ig**terjedő csúszka és egy**RESET** gomb.

Ez a beállítás a _megjelenített_ képet durvítja azzal, hogy egy N × N-es forrásképpont-blokk átlagát egy megjelenített képpontba számítja. Az `1` a natív felbontás.

* Ez hatással van **a teljes képernyős nézetre, a rácsos miniatűrökre, a kurzor kijelzésére és mindkét hisztogramra** – minden, ami a képet mutatja, ugyanazon az alapfelbontáson alapul.
* Ez **csak a megjelenítésre vonatkozik**. A feldolgozás és az exportálás nem változik. Az egyetlen kivétel szándékos: az [Index/LUT Sandbox](index-lut-sandbox.md) exportálás elmenti azt, amit éppen nézel, így átveszi az aktuális blokkméretet, és az exportáló panel figyelmeztet, ha a blokkméret meghaladja az 1-et.
* Az érték **projektenként** kerül tárolásra `viewer_display.gsd_bin` néven az `project.json` fájlban, így a program bezárása és újbóli megnyitása után is megmarad.
* A kurzor kijelzése a blokkot jelzi, nem a forráspixelt, ha a blokkméret 1 felett van — a megjelenített érték a kurzor alatt lévő blokk átlaga.

{% hint style="info" %}
**Miért „blokkméret”, és nem centiméter/pixel?** A cm/px értékhez szükség van a talajszint feletti magasságra. Egyetlen képkocka EXIF-adatai a tengerszint feletti GPS-magasságot tartalmazzák, nem pedig a célzott terep feletti magasságot, így az Chloros nem jeleníti meg azt a földfelszíni távolságot, amelyet nem tud pontosan kiszámítani. A forráspixelekben megadott blokkméret ugyanazt a tartalékértéket jelenti, amelyet az MAPIR felhőeszközök is használnak, amikor a földfelszíni mintavételi távolság ismeretlen.
{% endhint %}

***

## Megtekinthető képformátumok

A néző jobb felső sarkában található réteg legördülő menü az aktuális kép minden változatát felsorolja. Hogy mely bejegyzések jelennek meg, a kamerától és a feldolgozott tartalomtól függ — a teljes listát és a legördülő menü működését lásd az [Képrétegek](image-layers.md) alatt.

### Survey3

* **JPG** — a kamera saját előnézeti fájlja
* **RAW (Eredeti)** — az `.RAW` forrásfájl, megjelenítés céljából debayerezve, korrekciók nélkül
* **RAW (Cél)** — olyan képkocka, amelyről megállapítást nyert, hogy kalibrációs célt tartalmaz
* **RAW (Fényvisszaverődés)** — a kalibrált fényvisszaverődési termék (65535 = ρ 1,0)
* **Vignette Corrected**/**Sensor Response** — a nem kalibrált tartalék termék
* **White Balanced** — a fehér egyensúlyra beállított termék
* **RAW (`<INDEX>` Index)**és**`<INDEX>` LUT** — kiszámított indexképek

### LATTICE

A LATTICE-felvételek ugyanazt a legördülő menüt használják, a feldolgozási folyamat szintjeinek neveivel:

| Réteg                 | Mit tartalmaz                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (Eredeti)**    | A rögzített forrás RAW-kép                                     |
| **RAW (Debayered)**   | A lineáris, debayered kép                                           |
| **RAW (Előnézet)**     | A kijelzőn megjelenő előnézet — hamisszínű kiterjesztés multispektrális kamerákhoz |
| **Fehér egyensúly**    | A kijelzőn megjelenő előnézet az RGB master kamerákhoz (fehér egyensúly + gamma)   |
| **RAW (sugárzás)**    | Float32 spektrális sugárzás W/m²/sr/nm-ben                              |
| **RAW (visszaverődés)** | uint16 visszaverődés, 32768 = ρ 1,0                                    |

A sugárzás és a visszaverődés kizárólag multispektrális adatok: az RGB főkamera nem rendelkezik sávonkénti radiometriával, ezért ezek a rétegek nem készülnek el hozzá.

***

## Index és LUT alkalmazás

A multispektrális indexek és szín-LUT-ok az oldalsávról alkalmazhatók:

1. Nyissa meg az **Image Viewer** (<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">) oldalsávot
2. Jelölje be az **Index** lehetőséget
3. Válassza ki a kamera szűrőjét és egy indexképletet, majd húzza a csatornaköröket a képlet helyeire
4. Adjon hozzá egy LUT-ot, és válasszon gradienst, küszöbértékeket és egy kivágási módot
5. Olvassa le az értékeket a kurzor helyén, és mentse el az eredményt az **Export/Save Image(s)** gombbal mentse el az eredményt.

A teljes útmutatót az [Index/LUT Sandbox](index-lut-sandbox.md) oldalon találja.

***

## Hibaelhárítás

### A kép nem nyílik meg

**Lehetséges okok**: a fájlt az importálás után áthelyezték vagy törölték; a termék soha nem került írásra; nincs elegendő memória egy nagyon nagy képhez.**Mit kell tenni**:

1. Ellenőrizze, hogy a réteg fájlja még mindig létezik-e a projekt kimeneti fájlrendszerében
2. Nyissa meg a fájlt egy külső nézőprogramban, hogy megbizonyosodjon arról, hogy sértetlen
3. Zárja be az egyéb alkalmazásokat a memória felszabadítása érdekében

### A kép fekete, fehér vagy rendkívül színes

**Lehetséges okok**: a kijelző-nyújtásnak nincs mit feldolgoznia (szinte állandó képkocka); egy szokatlan értékeket tartalmazó float32 réteg; olyan index, amely nem eredményezett érvényes adatokat.**Mit kell tenni**:

1. Olvassa le a kurzorértékeket — ha minden csatorna nulla vagy annak közelében van, a probléma az adatokban van, nem a kijelzőben
2. Ellenőrizze a hisztogramot: egyetlen kiugrás az egyik végén azt jelzi, hogy a képkocka levágott vagy üres
3. Ellenőrizze a réteget létrehozó futtatás feldolgozási naplója

### Az értékek helytelennek tűnnek

**Lehetséges okok**: más rétegen van, mint gondolná; százalékot hasonlít össze nyers DN-értékkel; ugyanazt az osztót használva hasonlít össze egy LATTICE fájlt egy Survey3 fájllal.**Mit kell tenni**:

1. Ellenőrizze a legördülő menüben a kiválasztott réteget — a panel mértékegységei a réteget követik
2. A reflexió esetében használja a **%** oszlopot, ahelyett, hogy magad osztanád el a DN-t; ha osztanod kell, használd az adott fájl `Chloros:PixelScale` értékét (32768 a LATTICE esetében, ha nincs megadva, akkor 65535 az Survey3 esetében)
3. Állítsa vissza a GSD blokkméretet 1-re — 1 felett a blokkátlagot olvassa ki, nem a pixelt
4. Ellenőrizze, hogy a reflexiós kalibrálás valóban lefutott-e az adott képkockán; egy nem kalibrált tartaléktermék (Sensor Response / Vignette Corrected) nem reflexiós érték

***

## Következő lépések

* [**Képrétegek**](image-layers.md) — minden réteg neve (ha létezik), és az egyes értékek jelentése
* [**Index/LUT Sandbox**](index-lut-sandbox.md) — index-vizualizációk létrehozása, finomhangolása és exportálása
* [**Térképjelölők**](map-markers.md) — ugyanaz a képsorozat térképen
* [**Multispektrális indexképletek**](../project-settings/multispectral-index-formulas.md) — az indexre vonatkozó hivatkozás

A feldolgozási munkafolyamatot lásd a [Képek feldolgozása (GUI)](../processing-images-gui/adding-files-to-a-project.md) című részben.
