---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Multispektrális indexképletek

Az alábbi indexképletek az Survey3 szűrő átlagos átbocsátási tartományainak kombinációját használják:

<table><thead><tr><th align="center">Survey3 szűrő színe</th><th width="196.199951171875" align="center">Survey3 Szűrő neve</th><th width="159.800048828125" align="center">Áteresztési tartomány (FWHM)</th><th align="center">Átlagos transzmisszió</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB – Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN – Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN – Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN – Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN – NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR – NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>Ezen képletek használata esetén a név „\_1” vagy „\_2”-vel végződhet, ami azt jelzi, hogy melyik NIR szűrőt használták: az NIR1-et vagy az NIR2-et.

A LATTICE M3C (Bayer háromsávos) kamerák esetében ugyanaz az indexmotor használja az M3C szűrősávokat:

| M3C szűrő | 1. sáv (középső/FWHM) | 2. sáv (középső/FWHM) | 3. sáv (középső/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

A LATTICE M3M kamerák egycsatornásak (kameránként egy keskenysávú szűrő), ezért egyetlen M3M-kép alapján nem számíthatók ki többsávos indexek. Az M3M-mel történő indexszámításhoz két vagy több kamerát kell összevonni egy igazított többsávos képcsomagba, majd a LATTICE indexmotorját (`chloros-cli lattice index`, vagy a grafikus felhasználói felület élő Index Calculator funkcióját) kell használni.

***

## Hol működik az egyes indexnevek

Az Chloros **három** indexfelülettel rendelkezik, és azok előre beállított listái nem azonosak. Ebben a szakaszban ellenőrizheti, hogy egy adott név működni fog-e ott, ahol használni kívánja.

| Hol tartózkodik | Melyik lista érvényes | Szám |
| --- | --- | --- |
| Projektbeállítások → Index → Index hozzáadása (GUI) | 1. felület | 27 |
| Képmegjelenítő [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) (GUI) | 1. felület | 27 |
| `chloros-cli process --indices NDVI,NDRE` | 2. felület | 22 |
| SDK `process_folder(indices=[...])` | 2. felület | 22 |
| `chloros-cli lattice index --preset` | 3. felület | 22 (egy másik 22) |
| „Kamerák” fül – Élő indexszámító | Surface 3 | 22 (egy másik 22) |

A Surface 1 és 2 **egyszerre egy képet dolgoz fel egy kamerából**, a kameraszűrőcsatornáihoz rendelt `x`/`y`/`z`(/`a`) szimbólumhelyeket használják, amelyek az adott kamera szűrőcsatornáihoz vannak rendelve. A Surface 3 egy**igazított többsávos képhalmazon** dolgozik — több LATTICE kamera egy kockába összehangolva —, és a csatornákat kisbetűs névvel jelöli.

### 1. GUI Projektbeállítások / Képmegjelenítő sandbox legördülő menü — 27 képlet

A legördülő menü a következő sorrendben sorolja fel őket (ez a beillesztés sorrendje, nem ábécésorrend):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

A GUI-ban a kamera szűrőcsatornáit a képlet sávhelyeire húzhatja, így bármely képlet használható a kamera által támogatott bármely sávkiosztással. Az Ön által elmentett egyéni képletek e lista alatt jelennek meg.

Az **öt, kizárólag a GUI-ban elérhető** képlet – amelyek az CLI/SDK `--indices` lista nem fogadja el — a következőképpen vannak megvalósítva:

| Csak a grafikus felületen elérhető előre beállított érték | Képlet (megvalósítás szerint) | Helyek |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (négy hely) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Az egyes elemekhez tartozó leképezés az oldal alatti saját szakaszokban található (például az GARI az x=Green értéket várja, y=NIR, z=Blue, a=Red). Az GARI az egyetlen képlet az Chloros-ben, amely negyedik helyet használ.

### 2. CLI / SDK `--indices` névkiterjesztés — 22 előre beállított név

Az `chloros-cli process --indices` opció (és az SDK `indices` paraméter) a következő előre beállított neveket fogadja el:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Az ismeretlen indexneveket a rendszer figyelmen kívül hagyja.** A listán kívüli nevek (beleértve az öt, kizárólag a grafikus felületen használható képletet: `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI`, valamint a GUI-ban elmentett bármely egyéni képletet) kizárólag egy naplóbejegyzéssel kerülnek kihagyásra — a futtatás az adott index nélkül folytatódik, és a futtatás eredménye továbbra is sikeresnek minősül. A bejegyzés a következő formában jelenik meg:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

A nevek nagy- és kisbetűk közötti különbséget nem figyelembe véve, a szóközök eltávolítása után kerülnek összehasonlításra, így az `ndvi`, az `NDVI` és az ` NDVI ` ugyanazt az előre beállított értéket jelentik. Egy beállítást akkor is kihagy a rendszer, ha olyan sávra van szüksége, amelyet a fényképezőgép szűrője nem biztosít.
{% endhint %}

A megvalósított pontos képletek (az `x`/`y`/`z` szimbólumok sávhelyeket jelölnek; az alapértelmezett leképezés beállításonként szerepel):

| Előre beállított érték | Képlet (a megvalósítás szerint) | Alapértelmezett szűrő | Helyek (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Red, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)/(x/y)` | RGB | Red, Green, Blue |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | RGB | Red, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Hogyan alakul ki a sávpozíció egy előre beállított névből

Ha egy egyszerű nevet adsz meg, például `NDVI`, akkor az Chloros-nek el kell döntenie, hogy az egyes szimbólumok melyik fájl melyik csatornáját olvassák be. Ehhez ezt a táblázatot használja, amely a szűrőkódot az egyes csatornák tömbpozíciójához rendeli:

| Szűrőkód | Csatorna → tömbindex |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (az `Red` az Orange aliasaként is elfogadott, szintén 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

A beállítás **alapértelmezett szűrőjét** (a fenti „Alapértelmezett szűrő” oszlopban szereplőt) akkor használja a program, ha a projekt tartalmaz olyan képeket, amelyekre az adott szűrő van alkalmazva. Ha nincs ilyen, az Chloros átvizsgálja a projektben ténylegesen jelen lévő szűrőket az `RGN, OCN, NGB, RGB, RE, NIR` sorrendben, és kiválasztja az elsőt, amely minden csatornát biztosítani tud, amire az előre beállítottnak szüksége van. Ha egyik sem képes erre, az előre beállítást elvetik az adott futtatás során. Ezért ad ésszerű eredményt az `NDVI` kérés egy kizárólag OCN-et tartalmazó adatkészleten — az OCN Orange és NIR pozícióihoz kapcsolódik.

A LATTICE M3C modellszövegek az `F` előtaggal (`LATT-M3C-L41-FRGN`) hordozzák a szűrőt, de az előtag elhagyásra kerül, amikor a szűrőkódot leolvassák a képről, így egy FRGN kamera a fenti `RGN` soron keresztül felbontja a képet, és nincs szükség különleges kezelésre.

### 3. LATTICE indexmotor (`lattice index --preset`, élő Index Calculator) — 22 előre beállított érték

A LATTICE motor igazított többsávos képhalmazokon (élő képhalmazok vagy exportált többsávos TIFF-fájlok) működik, és kisbetűs csatornaneveket használ (`red`, `green`, `blue`, `red_edge`, `nir`). Előre beállított listája eltér a fenti kettőtől:

| Előre beállított | Képlet | Csatornák |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | piros, NIR |
| GNDVI | `(nir - green) / (nir + green)` | zöld, NIR |
| BNDVI | `(nir - blue) / (nir + blue)` | kék, NIR |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | piros\_szél, nir |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | kék, zöld, nir |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | piros, nir |
| OSAVI | `1.5 * (nir - red) / (nir + red + 0.16)` | piros, nir |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | piros, nir |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | kék, piros, nir |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | piros, nir |
| CVI | `(nir / green) - (red / green)` | piros, zöld, nir |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | piros, NIR |
| TDVI | `sqrt((nir - red) / (nir + red) + 0.5)` | vörös, NIR |
| LAI | `3.618 * ((nir - red) / (nir + 6*red - 7.5*green + 1)) - 0.118` | vörös, zöld, NIR |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | piros, zöld, kék |
| NGRDI | `(green - red) / (green + red)` | piros, zöld |
| VARI | `(green - red) / (green + red - blue)` | piros, zöld, kék |
| TGI | `green - 0.39*red - 0.61*blue` | piros, zöld, kék |
| EXG | `2*green - red - blue` | piros, zöld, kék |
| CIRE | `(nir / red_edge) - 1` | piros_szél, nir |
| CIGREEN | `(nir / green) - 1` | zöld, nir |
| NDWI | `(green - nir) / (green + nir)` | zöld, nir |

Futtassa az `chloros-cli lattice index --list-presets` parancsot a táblázat kinyomtatásához a telepített verzióból, az `--list-gradients` parancsot pedig a rendelkezésre álló színátmenetek megtekintéséhez. A csatorna szimbólumoknál a kis- és nagybetűk megkülönböztetése érvényes, és azoknak meg kell egyezniük az előre beállított nevek kisbetűs írásmódjával (pl. `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI

Ahogyan a grafikus felhasználói felületen és az CLI/SDK előre beállított listában megvalósítva, az CVI az arányok arányának képlete:

$$
CVI = {(z / y) \over (x / y)}
$$

az alapértelmezett RGB csatorna-leképezéssel: x=Red, y=Green, z=Blue. A grafikus felhasználói felületen a kamera bármely csatornáját áthúzhatja az x/y/z helyekre. Vegye figyelembe, hogy a LATTICE indexmotor `CVI` előre beállítása eltérő képletet használ, `(NIR / Green) - (Red / Green)` — az Ön által használt felülethez vonatkozóan ellenőrizze a fenti táblázatokat.

***

## ENDVI – Továbbfejlesztett normalizált növényzet-index

Ez az index az NIR és a zöld csatorna mellett a kék csatornát is felhasználja, és népszerű az NGB-szűrővel ellátott kameráknál, ahol a kék sáv helyettesíti a vöröset.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

A megvalósítás a következő szimbólumképlet: `((x+y)-(2*z))/((x+y)+(2*z))` — rendelje hozzá kamerája NIR és Green csatornáit az x/y helyekhez, az Blue csatornát pedig a z helyhez (egy NGB kamera esetében: x=NIR, y=Green, z=Blue).

***

## EVI – Továbbfejlesztett vegetációs index

Ezt az indexet eredetileg az MODIS adatokhoz fejlesztették ki, az NDVI javításaként, a magas levélfelületi indexű (LAI) területeken a növényzetjel optimalizálásával. Leginkább olyan magas LAI-értékű régiókban hasznos, ahol az NDVI értéke telítetté válhat. A kék visszaverődési tartományt használja a talajháttér-jelek korrigálására és a légköri hatások – beleértve az aeroszolos szórást is – csökkentésére.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

A vegetációs képpontok esetében az EVI értékeknek 0 és 1 között kell lenniük. A fényes elemek, például a felhők és a fehér épületek, valamint a sötét elemek, például a víz, rendellenes képpontértékeket okozhatnak egy EVI képen. Az EVI kép létrehozása előtt a felületi visszaverődési képről el kell maszkolni a felhőket és a világos elemeket, és opcionálisan a képpontértékeket 0 és 1 közötti tartományba kell korlátozni.

_Hivatkozás: Huete, A. és társai. „Áttekintés az MODIS vegetációs indexek radiometrikus és biofizikai teljesítményéről.” Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Erdőborítási index 1

_Csak GUI-ban elérhető — nem érhető el CLI/SDK `--indices` előre beállított értékként._

Ez az index a vörös szélső sávot is tartalmazó multispektrális reflexiós felvételek segítségével különbözteti meg az erdő lombkoronáját más növényzetektől.

$$
FCI1 = Red * RedEdge
$$

Az erdős területek alacsonyabb FCI1 értékeket mutatnak a fák alacsonyabb visszaverődése és a lombkorona árnyékainak jelenléte miatt.

_Hivatkozás: Becker, Sarah J., Craig S.T. Daughtry és Andrew L. Russ. „Robusztus erdőborítási indexek multispektrális képekhez.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 – Erdőborítási index 2

_Csak a grafikus felhasználói felületen érhető el — nem elérhető CLI/SDK vagy `--indices` előre beállított értékként._

Ez az index a vörös szélső sávot nem tartalmazó multispektrális reflexiós felvételek segítségével különbözteti meg az erdő lombkoronáját más növényzetektől.

$$
FCI2 = Red * NIR
$$

Az erdős területek alacsonyabb FCI2 értékeket mutatnak a fák alacsonyabb visszaverődése és a lombkorona árnyékai miatt.

_Hivatkozás: Becker, Sarah J., Craig S.T. Daughtry és Andrew L. Russ. „Robusztus erdőborítási indexek multispektrális képekhez.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – Globális környezetfigyelő index

_Csak GUI-ban elérhető — nem érhető el CLI/SDK vagy `--indices` előre beállított értékként._

Ezt a nemlineáris vegetációs indexet műholdas felvételek alapján végzett globális környezeti megfigyeléshez használják, és célja a légköri hatások korrigálása. Hasonló az NDVI-hez, de kevésbé érzékeny a légköri hatásokra. A csupasz talaj befolyásolja, ezért nem ajánlott ritka vagy közepesen sűrű növényzetű területeken történő használatra.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Ahol:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Hivatkozás: Pinty, B. és M. Verstraete. GEMI: egy nemlineáris index a globális növényzet műholdas megfigyeléséhez. Vegetation 101 (1992): 15–20._

***

## GARI – Green légköri hatásoknak ellenálló index

_Csak GUI-n — nem érhető el CLI/SDK `--indices` előre beállított értékként._

Ez az index érzékenyebb a klorofill-koncentrációk széles tartományára, és kevésbé érzékeny a légköri hatásokra, mint az NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

A gamma-állandó egy súlyozó függvény, amely a légkörben uralkodó aeroszolos körülményektől függ. Az ENVI 1,7-es értéket használ, amely Gitelson, Kaufman és Merzylak (1996, 296. oldal) által ajánlott érték.

_Hivatkozás: Gitelson, A., Y. Kaufman és M. Merzylak. „Egy Green-csatorna használata a globális növényzet távérzékelésében az EOS-MODIS segítségével.” Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – Green klorofill-index

Ezt az indexet a levelek klorofilltartalmának becslésére használják számos növényfaj esetében.

$$
GCI = {NIR \over Green} - 1
$$

A széles NIR és zöld hullámhossz-tartomány jobb előrejelzést biztosít a klorofilltartalomra vonatkozóan, miközben nagyobb érzékenységet és magasabb jel-zaj arányt tesz lehetővé.

_Hivatkozás: Gitelson, A., Y. Gritz és M. Merzlyak. „A levél klorofilltartalma és a spektrális reflexió közötti összefüggések, valamint algoritmusok a magasabb rendű növények leveleinek roncsolásmentes klorofill-értékeléséhez.” Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green levélindex

Ezt az indexet eredetileg egy RGB digitális kamerával való használatra tervezték a búza borításának mérésére, ahol a vörös, zöld és kék digitális értékek (DN-ek) 0 és 255 között mozognak.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

GLI értékek -1 és +1 között mozognak. A negatív értékek a talajt és az élettelen elemeket jelzik, míg a pozitív értékek a zöld leveleket és szárakat.

_Hivatkozás: Louhaichi, M., M. Borman és D. Johnson. „Térbeli elhelyezkedésű platform és légi felvételek a legeltetés búzára gyakorolt hatásainak dokumentálásához.” Geocarto International 16, 1. sz. (2001): 65–70._

***

## GNDVI – Green normalizált növényzet-különbség-index

Ez az index hasonló az NDVI-hez, azzal a különbséggel, hogy a vörös spektrum helyett az 540–570 nm-es zöld spektrumot méri. Ez az index érzékenyebb a klorofill-koncentrációra, mint az NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Hivatkozás: Gitelson, A. és M. Merzlyak. „A magasabb rendű növények leveleiben található klorofill-koncentráció távérzékelése.” Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – Green Optimalizált, talajhoz igazított vegetációs index

Ezt az indexet eredetileg színes-infravörös felvételek alapján fejlesztették ki a kukorica nitrogénigényének előrejelzésére. Hasonló az OSAVI-hez, de a zöld sávot pirosra cseréli.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Hivatkozás: Sripada, R. és társai: „A kukorica szezon közbeni nitrogénigényének meghatározása légi színes-infravörös felvételek segítségével.” Doktori értekezés, Észak-Karolinai Állami Egyetem, 2005._

***

## GRVI – Green arányú vegetációs index

Ez az index érzékeny az erdő lombkoronájában zajló fotoszintézis sebességére, mivel a zöld és a vörös visszaverődést erősen befolyásolják a levélpigmentek változásai.

$$
GRVI = {NIR \over Green }
$$

_Hivatkozás: Sripada, R. és munkatársai. „Légi színes-infravörös fényképezés a kukorica szezon eleji nitrogénigényének meghatározásához.” Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – Green talajkorrigált vegetációs index

Ezt az indexet eredetileg színes infravörös felvételek alapján fejlesztették ki a kukorica nitrogénigényének előrejelzésére. Hasonló az SAVI-hez, de a zöld sávot pirosra cseréli.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Hivatkozás: Sripada, R., et al. „A kukorica szezon közbeni nitrogénigényének meghatározása légi színes-infravörös felvételek segítségével.” Doktori értekezés, Észak-Karolinai Állami Egyetem, 2005._

***

## LAI – Lombozatfelületi index

Ezt az indexet a lombozatborítottság becslésére, valamint a növény növekedésének és terméshozamának előrejelzésére használják. Az ENVI a zöld LAI értéket a Boegh et al. (2002) által megadott következő empirikus képlet segítségével számítja ki:

$$
LAI = 3.618 * EVI - 0.118
$$

Ahol az EVI:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

A magas LAI értékek általában körülbelül 0 és 3,5 között mozognak. Ha azonban a jelenet felhőket és más, telített képpontokat eredményező világos elemeket tartalmaz, az LAI értékek meghaladhatják a 3,5-öt. Ideális esetben a felhőket és a fényes elemeket ki kell maszkolni a jelenetből, mielőtt létrehozná az LAI képet.

_Hivatkozás: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde és A. Thomsen. „Légi multispektrális adatok a levélfelületi index, a nitrogénkoncentráció és a fotoszintézis hatékonyságának számszerűsítéséhez a mezőgazdaságban.” Remote Sensing of Environment 81, 2–3. sz. (2002): 179–193._

***

## LCI – Levélklorofill-index

_Csak GUI-ban elérhető — nem érhető el CLI/SDK `--indices` előre beállított értékként._

Ezt az indexet a magasabb rendű növények klorofilltartalmának becslésére használják, mivel érzékeny a klorofill abszorpciója által okozott visszaverődési változásokra.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Hivatkozás: Datt, B. „Remote Sensing of Water Content in Eucalyptus Leaves.” Journal of Plant Physiology 154, 1. sz. (1999): 30–36._

***

## MNLI – Módosított nemlineáris index

Ez az index a nemlineáris index (NLI) továbbfejlesztése, amely beépíti a talajhoz igazított vegetációs indexet (SAVI) a talajháttér figyelembevétele érdekében. Az ENVI 0,5-ös értékű lombkorona-háttérkorrekciós tényezőt (_L_) használ.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Hivatkozás: Yang, Z., P. Willis és R. Mueller. „A sávarány-javított AWIFS-kép hatása a növénytermesztési osztályozás pontosságára.” A Pecora 17 Távérzékelési Szimpózium (2008) jegyzőkönyve, Denver, CO._

***

## MSAVI2 – Módosított talajkorrigált vegetációs index 2

Ez az index a Qi és munkatársai által javasolt MSAVI index egyszerűbb változata (1994) által javasolt MSAVI index egyszerűbb változata, amely továbbfejleszti a talajhoz igazított vegetációs indexet (SAVI). Csökkenti a talajból származó zajt és növeli a vegetációs jel dinamikatartományát. Az MSAVI2 egy induktív módszeren alapul, amely nem használ állandó _L_ értéket (mint az SAVI esetében) az egészséges növényzet kiemeléséhez.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Hivatkozás: Qi, J., A. Chehbouni, A. Huete, Y. Kerr és S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.” Remote Sensing of Environment 48 (1994): 119–126._

***

## MSR – Módosított egyszerű arány

Ez az index az egyszerű NIR/Red arány módosítása, amelyet a biofizikai paraméterekkel való kapcsolat linearizálására terveztek, és magasabb növényzet-sűrűségek esetén érzékenyebb, mint az NDVI.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Hivatkozás: Chen, J. „A vegetációs indexek és egy módosított egyszerű arány értékelése boreális alkalmazásokhoz.” Canadian Journal of Remote Sensing 22 (1996): 229–242._

***

## NDRE – Normalizált különbség RedEdge

Ez az index hasonló az NDVI-hez, de az NIR és az RedEdge közötti kontrasztot hasonlítja össze az Red helyett, amely gyakran hamarabb észleli a növényzetet stresszt.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalizált különbségű vegetációs index

Ez az index az egészséges, zöld növényzet mértékét jelzi. A normalizált különbség képletének és a klorofill legmagasabb abszorpciós és reflexiós tartományainak felhasználásának kombinációja miatt széles körű körülmények között is megbízható. Sűrű növényzet esetén azonban telítődhet, amikor az LAI értéke magasra emelkedik.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Ennek az indexnek az értéke -1 és 1 között mozog. A zöld növényzetre jellemző tartomány 0,2 és 0,8 között van.

_Hivatkozás: Rouse, J., R. Haas, J. Schell és D. Deering. A vegetációs rendszerek megfigyelése a Nagy-síkságon az ERTS segítségével. Harmadik ERTS szimpózium, NASA (1973): 309–317._

***

## NLI – Nemlineáris index

Ez az index abból indul ki, hogy számos vegetációs index és a felszíni biofizikai paraméterek közötti kapcsolat nemlineáris. Lineárissá teszi azokat a felszíni paraméterekkel való kapcsolatokat, amelyek általában nemlineárisak.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Hivatkozás: Goel, N. és W. Qin. „A lombkorona szerkezetének hatása a különböző vegetációs indexek, valamint az LAI és az Fpar közötti összefüggésekre: számítógépes szimuláció.” Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – Optimalizált, talajhoz igazított vegetációs index

Ez az index a talajhoz igazított vegetációs indexen (SAVI) alapul. A lombkorona-háttérkorrekciós tényezőjének standard értékeként 0,16-ot használ. Rondeaux (1996) megállapította, hogy ez az érték alacsony növényzetborítás esetén nagyobb talajváltozást biztosít, mint az SAVI, ugyanakkor 50%-nál nagyobb növényzetborítás esetén fokozott érzékenységet mutat. Ez az index leginkább olyan területeken használható, ahol a növényzet viszonylag ritka, és a talaj a lombkoronán keresztül látható.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Hivatkozás: Rondeaux, G., M. Steven és F. Baret. „Optimization of Soil-Adjusted Vegetation Indices.” Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – Renormalizált különbségű vegetációs index

Ez az index a közeli infravörös és a vörös hullámhosszok közötti különbséget, valamint az NDVI értéket használja az egészséges növényzet kiemelésére. Nem érzékeny a talaj és a napfény megfigyelési geometriájának hatásaira.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Hivatkozás: Roujean, J. és F. Breon. „A növényzet által elnyelt PAR becslése kétirányú visszaverődési mérések alapján.” Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – Talajkorrigált növényzetindex

Ez az index hasonló az NDVI-hez, de kiszűri a talajpixelek hatását. A lombozat háttérkorrekciós tényezőjét, _L_-t használja, amely a növényzet sűrűségének függvénye, és gyakran megköveteli a növényzet mennyiségének előzetes ismeretét. Huete (1988) az elsőrendű talajháttér-változások figyelembevétele érdekében _L_=0,5 optimális értéket javasol. Ez az index leginkább olyan, viszonylag ritkás növényzetű területeken használható, ahol a talaj a lombkoronán keresztül látható.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Hivatkozás: Huete, A. „A Soil-Adjusted Vegetation Index (SAVI).” Remote Sensing of Environment 25 (1988): 295–309._

***

## TDVI – Transzformált különbségű vegetációs index

Ez az index hasznos a városi környezetben található növényzetborítás figyelemmel kíséréséhez. Nem telítődik, ellentétben az NDVI és az SAVI indexekkel.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Hivatkozás: Bannari, A., H. Asalhi és P. Teillet. „Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping” In Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, 5. kötet (2002)._

***

## VARI – Látható tartományú, légköri hatásokkal szemben ellenálló index

Ez az index az ARVI-en alapul, és a légköri hatásokra alacsony érzékenységgel rendelkező jelenetekben a növényzet arányának becslésére szolgál.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Hivatkozás: Gitelson, A. és társai. „Növényzet és talajhatárok a látható spektrális tartományban: koncepció és technika a növényzet arányának távoli becsléséhez.” International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Széles dinamikatartományú vegetációs index

Ez az index hasonló az NDVI-hez, de súlyozási együtthatót (_a_) használ a közeli infravörös és a vörös jelek NDVI-hez való hozzájárulása közötti eltérés csökkentésére. Az WDRVI különösen hatékony olyan területeken, ahol a növényzet sűrűsége közepes vagy magas, és az NDVI értéke meghaladja a 0-t.6-ot meghaladja. Az NDVI értéke általában stabilizálódik, amikor a növényzetarány és a levélfelületi index (LAI) növekszik, míg az WDRVI érzékenyebb a növényzetarány szélesebb tartományára és az LAI változásaira.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

A súlyozási együttható (_a_) értéke 0,1 és 0,2 között lehet. Henebry, Viña és Gitelson (2004) a 0,2-es értéket javasolja.

_Hivatkozások_

_Gitelson, A. „Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.” Journal of Plant Physiology 161, 2. sz. (2004): 165–173._

_Henebry, G., A. Viña és A. Gitelson. „A széles dinamikatartományú vegetációs index és annak potenciális hasznossága a hiányelemzésben.” Gap Analysis Bulletin 12: 50–56._
