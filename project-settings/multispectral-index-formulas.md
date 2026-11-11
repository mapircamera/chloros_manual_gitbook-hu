---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Multispektrális indexképletek

Az alábbi indexképletek a Survey3 szűrő átlagos átlátszási tartományainak kombinációját használják:

<table><thead><tr><th align="center">Survey3 szűrő színe</th><th width="196.199951171875" align="center">Survey3 szűrő neve</th><th width="159.800048828125" align="center">Áteresztési tartomány (FWHM)</th><th align="center">Átlagos transzmisszió</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re-RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Ezen képletek használata esetén a név „\_1” vagy „\_2” végződéssel zárulhat, ami azt jelzi, hogy melyik NIR-szűrőt használták: az NIR1-et vagy a NIR2-t.

A LATTICE M3C (Bayer háromsávos) kamerák esetében ugyanaz az indexmotor használja az M3C szűrősávokat:

| M3C szűrő | 1. sáv (közép/FWHM) | 2. sáv (közép/FWHM) | 3. sáv (közép/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

A LATTICE M3M kamerák egycsatornásak (kameránként egy keskenysávú szűrő), ezért egyetlen M3M-kép alapján nem számíthatók ki többsávos indexek. Az M3M-mel történő indexszámításhoz egyesítsen két vagy több kamerát egy igazított többsávos képcsomagba, és használja a LATTICE indexmotorját (`chloros-cli lattice index`, vagy a grafikus felhasználói felület élő Index Calculator funkcióját).

***

## Hol működik az egyes indexnevek

A Chloros-nak **három** indexfelülete van, és azok előre beállított listái nem azonosak. Ebben a szakaszban ellenőrizheti, hogy egy név működni fog-e ott, ahol használni kívánja.

| Hol tartózkodik | Melyik lista érvényes | Szám |
| --- | --- | --- |
| Projektbeállítások → Index → Index hozzáadása (GUI) | 1. felület | 27 |
| Képmegjelenítő [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) (GUI) | 1. felület | 27 |
| `chloros-cli process --indices NDVI,NDRE` | 2. felület | 22 |
| SDK `process_folder(indices=[...])` | 2. felület | 22 |
| `chloros-cli lattice index --preset` | 3. felület | 22 (egy másik 22) |
| Kamerák fül – élő indexszámító | 3. felület | 22 (egy másik 22) |

A Surface 1 és 2 **egyszerre egy képet dolgoz fel egy kamerából**, az `x`/`y`/`z`(/`a`) szimbólumhelyeket használva, amelyek az adott kamera szűrőcsatornáihoz vannak rendelve. A 3. felület**igazított többsávos képcsomaggal** dolgozik — több, egy kockába összehangolt LATTICE-kamera —, és a csatornákat kisbetűs névvel jelöli.

### 1. GUI Projektbeállítások / Képnézegető sandbox legördülő menü — 27 képlet

A legördülő menü a következő sorrendben sorolja fel őket (ez a beillesztés sorrendje, nem ábécésorrend):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

A GUI-ban a kamera szűrőcsatornáit a képlet sávhelyeire húzhatja, így bármely képlet használható a kamera által támogatott bármely sávkiosztással. Az Ön által elmentett egyéni képletek e lista alá kerülnek.

Az **öt, kizárólag a GUI-ban elérhető** képlet — amelyeket az CLI/SDK `--indices` lista nem fogad el — a következőképpen van megvalósítva:

| Kizárólag GUI-ban elérhető előre beállított érték | Képlet (megvalósítás szerint) | Helyek |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (négy hely) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

Az egyes elemekre vonatkozó leképezés az oldal alján található saját szakaszokban található (például a GARI esetében x=Green, y=NIR, z=Blue, a=Red). A GARI az egyetlen olyan képlet a Chloros-ban, amely negyedik helyet is használ.

### 2. CLI / SDK `--indices` névkiterjesztés — 22 előre beállított érték

Az `chloros-cli process --indices` opció (és az SDK `indices` paraméter) a következő előre beállított neveket fogadja el:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Az ismeretlen indexneveket a rendszer figyelmen kívül hagyja.** A listán kívüli nevek (beleértve az öt, kizárólag a grafikus felhasználói felületen használható képletet: `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI`, valamint a GUI-ban elmentett bármely egyéni képlet) kizárásra kerül, csupán egy naplóbejegyzéssel jelzve — a futtatás az adott index nélkül folytatódik, és a futtatás eredménye továbbra is sikeresnek minősül. A bejegyzés a következőképpen jelenik meg:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

A nevek nagy- és kisbetűk közötti különbséget nem figyelembe véve, a szóközök eltávolítása után kerülnek összehasonlításra, így az `ndvi`, az `NDVI` és az ` NDVI ` ugyanazt az előre beállított értéket jelenti. Egy beállítást akkor is kihagy a rendszer, ha olyan sávra van szüksége, amelyet a fényképezőgép szűrője nem biztosít.
{% endhint %}

A megvalósított pontos képletek (az `x`/`y`/`z` szimbólumok sávhelyeket jelölnek; az alapértelmezett leképezés beállításonként szerepel):

| Előre beállított érték | Képlet (megvalósított formában) | Alapértelmezett szűrő | Helyek (x, y, z) |
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
| CVI | `(z/y)*(x/y)` | RGN | Red, Green, NIR |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | NGB | NIR, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Hogyan alakulnak ki a sávpozíciók egy előre beállított névből

Ha egy egyszerű nevet ad meg, például `NDVI`, a Chloros-nak el kell döntenie, hogy az egyes szimbólumok melyik fájl melyik csatornáját olvassák be. Ehhez ezt a táblázatot használja, amely a szűrőkódot az egyes csatornák tömbpozíciójához rendeli:

| Szűrőkód | Csatorna → tömbindex |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (az `Red` az Orange (szintén 0) aliasaként elfogadott) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

Az előre beállított profil **alapértelmezett szűrőjét** (a fenti „Alapértelmezett szűrő” oszlopot) akkor használja a program, ha a projekt tartalmaz olyan képeket, amelyeknél ez a szűrő van beállítva. Ha nem, akkor a Chloros a projektben ténylegesen jelen lévő szűrőket `RGN, OCN, NGB, RGB, RE, NIR` sorrendben vizsgálja át, és kiválasztja az elsőt, amely minden, az előre beállított szűrőhöz szükséges csatornát biztosítani tud. Ha egyik sem képes erre, akkor az előre beállított szűrőt elvetik az adott futtatás során. Ezért ad még mindig értelmezhető eredményt az `NDVI`, ha azt kizárólag OCN-adatállományra kérik – ez ugyanis az OCN Orange és NIR pozícióihoz kapcsolódik.

A LATTICE M3C modell karakterláncai az `F` előtaggal hordozzák a szűrőt (`LATT-M3C-L41-FRGN`), de az előtag elhagyásra kerül, amikor a szűrőkódot leolvassák a képről, így egy FRGN kamera a fenti `RGN` soron keresztül oldja meg a feladatot, és nincs szükség különleges kezelésre.

### 3. LATTICE indexmotor (`lattice index --preset`, élő Index Calculator) — 22 előre beállított érték

A LATTICE-motor igazított többsávos képhalmazokon (élő képtömbökön vagy exportált többsávos TIFF-fájlokon) működik, és kisbetűs csatornaneveket használ (`red`, `green`, `blue`, `red_edge`, `nir`). Előre beállított listája eltér a fenti kettőtől:

| Előre beállított | Képlet | Csatornák |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | vörös, NIR |
| GNDVI | `(nir - green) / (nir + green)` | zöld, NIR |
| BNDVI | `(nir - blue) / (nir + blue)` | kék, NIR |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | vörös_szél, NIR |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | kék, zöld, NIR |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | vörös, NIR |
| OSAVI | `(nir - red) / (nir + red + 0.16)` | piros, nir |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | piros, nir |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | kék, piros, nir |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | piros, NIR |
| CVI | `nir * red / green**2` | piros, zöld, NIR |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | piros, NIR |
| TDVI | `1.5 * (nir - red) / sqrt(nir**2 + red + 0.5)` | piros, NIR |
| LAI | `3.618 * (2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)) - 0.118` | kék, vörös, NIR |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | vörös, zöld, kék |
| NGRDI | `(green - red) / (green + red)` | piros, zöld |
| VARI | `(green - red) / (green + red - blue)` | piros, zöld, kék |
| TGI | `green - 0.39*red - 0.61*blue` | piros, zöld, kék |
| EXG | `2*green - red - blue` | piros, zöld, kék |
| CIRE | `(nir / red_edge) - 1` | piros\_szél, nir |
| CIGREEN | `(nir / green) - 1` | zöld, nir |
| NDWI | `(green - nir) / (green + nir)` | zöld, nir |

Futtassa az `chloros-cli lattice index --list-presets` parancsot a táblázat kinyomtatásához a telepített verzióból, az elérhető színátmenetekhez pedig az `--list-gradients` parancsot. A csatorna szimbólumoknál a kis- és nagybetűk megkülönböztetése fontos, és azoknak meg kell egyezniük az előre beállított kisbetűs nevekkel (pl. `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI – klorofill-vegetációs index

Ez az index a lombkorona sűrűségének széles tartományában érzékeny a levelek klorofilltartalmára. Az NIR, a vörös és a zöld sávokat használja.

$$
CVI = NIR * {Red \over Green^{2}}
$$

A grafikus felhasználói felületen (GUI) és a parancssori felületen (CLI)/SDK előre beállított listájában ez a szimbólumképlet: `(z/y)*(x/y)`, ahol x=Red, y=Green, z=NIR (egy RGN kamera sávjai); a LATTICE indexmotor előre beállított értéke `nir * red / green**2`. Minden felület ugyanazt az indexet számítja ki.

_Hivatkozás: Vincini, M., E. Frazzi és P. D'Alessio. „A Broad-Band Leaf Chlorophyll Vegetation Index at the Canopy Scale.” Precision Agriculture 9 (2008): 303–319._

***

## ENDVI – Továbbfejlesztett normalizált különbségű vegetációs index

Ez az index az NIR- és a zöld csatorna mellett a kék csatornát is felhasználja, és népszerű az NGB-szűrős kameráknál, ahol a kék sáv helyettesíti a vöröset.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

A megvalósításhoz az `((x+y)-(2*z))/((x+y)+(2*z))` szimbólumképletet kell használni – rendelje hozzá kamerája NIR és Green csatornáit az x/y helyekhez, a Blue-t pedig a z-hez (NGB kamerák esetében: x=NIR, y=Green, z=Blue).

***

## EVI – Továbbfejlesztett vegetációs index

Ezt az indexet eredetileg a MODIS-adatokhoz fejlesztették ki az NDVI továbbfejlesztéseként, a magas levélfelületi indexű (LAI) területeken a növényzeti jel optimalizálásával. Leginkább olyan magas LAI-értékű régiókban hasznos, ahol az NDVI értéke telítetté válhat. A kék reflektancia-tartományt használja a talaj háttérjelek korrigálására és a légköri hatások – beleértve az aeroszol-szórást is – csökkentésére.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Az EVI-értékeknek a növényzeti képpontok esetében 0 és 1 között kell lenniük. A fényes elemek, például a felhők és a fehér épületek, valamint a sötét elemek, például a víz, rendellenes képpontértékeket eredményezhetnek egy EVI-képen. Az EVI-kép létrehozása előtt a reflektancia-képről el kell távolítani a felhőket és a világos elemeket, és opcionálisan a képpontértékeket 0 és 1 közötti küszöbértékre kell korlátozni.

_Hivatkozás: Huete, A. és társai. „A MODIS növényzeti indexek radiometrikus és biofizikai teljesítményének áttekintése.” Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Erdőborítási index 1

_Csak GUI-ban elérhető — CLI-ként vagy SDK-ként nem elérhető `--indices` előre beállított érték._

Ez az index a vörös szélső sávot is tartalmazó multispektrális reflektancia-képek segítségével különbözteti meg az erdő lombkoronáját más növényzet-típusoktól.

$$
FCI1 = Red * RedEdge
$$

Az erdős területek alacsonyabb FCI1-értékeket mutatnak a fák alacsonyabb visszaverődése és a lombkorona árnyékai miatt.

_Hivatkozás: Becker, Sarah J., Craig S.T. Daughtry és Andrew L. Russ. „Robusztus erdőborítás-indexek multispektrális képekhez.” Photogrammetric Engineering & Remote Sensing 84.8 (2018): 505–512._

***

## FCI2 – Erdőborítás-index 2

_Csak GUI-n elérhető — CLI-ként vagy SDK-ként nem elérhető `--indices` előre beállított érték._

Ez az index a vörös szélső sávot nem tartalmazó multispektrális visszaverődési képek segítségével különbözteti meg az erdő lombkoronáját más növényzetfajtáktól.

$$
FCI2 = Red * NIR
$$

Az erdős területek alacsonyabb FCI2-értékeket mutatnak a fák alacsonyabb visszaverődése és a lombkorona árnyékainak jelenléte miatt.

_Hivatkozás: Becker, Sarah J., Craig S.T. Daughtry és Andrew L. Russ. „Robusztus erdőborítási indexek multispektrális képekhez.” Photogrammetric Engineering & Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – Globális környezetfigyelő index

_Csak GUI-ban elérhető — CLI-ként vagy SDK-ként nem elérhető `--indices` előre beállított érték._

Ezt a nemlineáris vegetációs indexet globális környezeti megfigyelésre használják műholdas felvételek alapján, és célja a légköri hatások korrigálása. Hasonló az NDVI-hez, de kevésbé érzékeny a légköri hatásokra. A csupasz talaj befolyásolja, ezért ritka vagy közepesen sűrű növényzetű területeken nem ajánlott a használata.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Ahol:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Hivatkozás: Pinty, B. és M. Verstraete. „GEMI: a Non-Linear Index to Monitor Global Vegetation From Satellites”. Vegetation 101 (1992): 15–20._

***

## GARI – Green Atmospherically Resistant Index (zöld, légköri hatásoknak ellenálló index)

_Csak GUI-ban elérhető – CLI-ként vagy SDK-ként nem elérhető `--indices` előre beállított érték._

Ez az index érzékenyebb a klorofill-koncentrációk széles tartományára, és kevésbé érzékeny a légköri hatásokra, mint az NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

A gamma-állandó egy súlyozó függvény, amely a légkörben uralkodó aeroszol-viszonyoktól függ. Az ENVI 1,7-es értéket használ, amely Gitelson, Kaufman és Merzylak (1996, 296. oldal) által ajánlott érték.

_Hivatkozás: Gitelson, A., Y. Kaufman és M. Merzylak. „Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.” Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – zöld klorofill-index (Green Chlorophyll Index)

Ezt az indexet a levelek klorofilltartalmának becslésére használják számos növényfaj esetében.

$$
GCI = {NIR \over Green} - 1
$$

A széles NIR- és zöld hullámhossz-tartomány jobb előrejelzést biztosít a klorofilltartalomra vonatkozóan, miközben nagyobb érzékenységet és magasabb jel-zaj arányt tesz lehetővé.

_Hivatkozás: Gitelson, A., Y. Gritz és M. Merzlyak. „A levélklorofill-tartalom és a spektrális visszaverődés közötti összefüggések, valamint algoritmusok a magasabb rendű növények leveleinek roncsolásmentes klorofill-értékeléséhez.” Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – zöldlevél-index (Green Leaf Index)

Ezt az indexet eredetileg digitális RGB-kamerával való használatra tervezték a búza borításának mérésére, ahol a vörös, zöld és kék digitális értékek (DN-ek) 0 és 255 között mozognak.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

A GLI-értékek -1 és +1 között mozognak. A negatív értékek a talajt és az élettelen elemeket jelzik, míg a pozitív értékek a zöld leveleket és szárakat jelzik.

_Hivatkozás: Louhaichi, M., M. Borman és D. Johnson. „Térbeli helymeghatározású platform és légi felvételek a legeltetés búzára gyakorolt hatásainak dokumentálásához.” Geocarto International 16, 1. sz. (2001): 65–70._

***

## GNDVI – Green Normalized Difference Vegetation Index

Ez az index hasonló az NDVI-hez, azzal a különbséggel, hogy a vörös spektrum helyett az 540–570 nm-es zöld spektrumot méri. Ez az index érzékenyebb a klorofill-koncentrációra, mint az NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Hivatkozás: Gitelson, A. és M. Merzlyak. „A magasabb rendű növények leveleiben található klorofill-koncentráció távérzékelése.” Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – Zöldre optimalizált, talajhoz igazított vegetációs index (Green Optimized Soil Adjusted Vegetation Index)

Ezt az indexet eredetileg színes infravörös felvételek alapján fejlesztették ki a kukorica nitrogénigényének előrejelzésére. Hasonló az OSAVI-hez, de a vörös sávot zöld sávval helyettesíti.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Hivatkozás: Sripada, R. és társai. „A kukorica szezonközi nitrogénigényének meghatározása légi színes-infravörös fényképezés segítségével.” Doktori értekezés, Észak-Karolinai Állami Egyetem, 2005._

***

## GRVI – zöld arányú vegetációs index (Green Ratio Vegetation Index)

Ez az index érzékeny az erdő lombkoronájában zajló fotoszintézis sebességére, mivel a zöld és a vörös visszaverődést erősen befolyásolják a levélpigmentek változásai.

$$
GRVI = {NIR \over Green }
$$

_Hivatkozás: Sripada, R. és társai. „Légi színes infravörös felvételek a kukorica szezon eleji nitrogénigényének meghatározásához.” Agronomy Journal 98 (2006): 968–977._

***

## GSAVI – Zöld talajra korrigált vegetációs index (Green Soil Adjusted Vegetation Index)

Ezt az indexet eredetileg színes infravörös fényképezéssel fejlesztették ki a kukorica nitrogénigényének előrejelzésére. Hasonló a SAVI-hez, de a vörös sávot zöld sávra cseréli.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Hivatkozás: Sripada, R. és munkatársai. „A kukorica szezon közbeni nitrogénigényének meghatározása légi színes infravörös fényképezés segítségével.” Doktori értekezés, Észak-Karolinai Állami Egyetem, 2005._

***

## LAI – levélfelületi index

Ezt az indexet a lombozatborítás becslésére, valamint a növénytermesztés növekedésének és hozamának előrejelzésére használják. Az ENVI a zöld LAI-t a Boegh és társai (2002) által megadott következő empirikus képlet segítségével számítja ki:

$$
LAI = 3.618 * EVI - 0.118
$$

Ahol az EVI:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

A magas LAI-értékek általában körülbelül 0 és 3,5 között mozognak. Ha azonban a felvételen felhők vagy más, telített képpontokat eredményező fényes elemek jelennek meg, az LAI-értékek meghaladhatják a 3,5-öt. Ideális esetben az LAI-kép létrehozása előtt ki kell maszkolni a felhőket és a fényes elemeket a felvételből.

_Hivatkozás: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde és A. Thomsen. „Légi multispektrális adatok a levélfelületi index, a nitrogénkoncentráció és a fotoszintézis hatékonyságának számszerűsítéséhez a mezőgazdaságban.” Remote Sensing of Environment 81, 2–3. sz. (2002): 179–193._

***

## LCI – levélklorofill-index

_Csak GUI-ban elérhető — CLI-ként vagy SDK-ként nem elérhető `--indices` előre beállított érték._

Ezt az indexet a magasabb rendű növények klorofilltartalmának becslésére használják, mivel érzékeny a klorofill abszorpciója által okozott visszaverődési változásokra.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Hivatkozás: Datt, B. „Remote Sensing of Water Content in Eucalyptus Leaves.” Journal of Plant Physiology 154, 1. sz. (1999): 30–36._

***

## MNLI – Módosított nemlineáris index

Ez az index a nemlineáris index (NLI) továbbfejlesztése, amely a talajháttér figyelembevétele érdekében beépíti a talajhoz igazított vegetációs indexet (SAVI). Az ENVI 0,5-ös lombkorona-háttérkorrekciós tényezőt (_L_) használ.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Hivatkozás: Yang, Z., P. Willis és R. Mueller. „A sávarány-javított AWIFS-kép hatása a növénytermesztési osztályozás pontosságára.” A Pecora 17 távérzékelési szimpózium (2008) jegyzőkönyve, Denver, CO._

***

## MSAVI2 – Módosított talajkorrigált vegetációs index 2

Ez az index a Qi és munkatársai által javasolt MSAVI-index egyszerűbb változata (1994) által javasolt MSAVI-index egyszerűbb változata, amely továbbfejleszti a talajhoz igazított vegetációs indexet (SAVI). Csökkenti a talajból származó zajt és növeli a vegetációs jel dinamikatartományát. Az MSAVI2 egy induktív módszeren alapul, amely nem használ állandó _L_ értéket (mint a SAVI) az egészséges vegetáció kiemelésére.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Hivatkozás: Qi, J., A. Chehbouni, A. Huete, Y. Kerr és S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.” Remote Sensing of Environment 48 (1994): 119–126._

***

## MSR – Módosított egyszerű arány

Ez az index az egyszerű NIR/Red arány módosítása, amelyet a biofizikai paraméterekkel való kapcsolat linearizálására terveztek, és magasabb növényzet-sűrűség esetén érzékenyebb, mint az NDVI.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Hivatkozás: Chen, J. „A vegetációs indexek és egy módosított egyszerű arány értékelése boreális alkalmazásokhoz.” Canadian Journal of Remote Sensing 22 (1996): 229–242._

***

## NDRE – Normalizált különbség RedEdge

Ez az index hasonló az NDVI-hez, de a Red helyett a NIR és a RedEdge közötti kontrasztot hasonlítja össze, ami gyakran hamarabb jelzi a növényzet stresszét.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalizált különbségű vegetációs index

Ez az index az egészséges, zöld növényzet mértékét jelzi. A normalizált különbség képletének és a klorofill legmagasabb abszorpciós és reflexiós tartományainak felhasználásának kombinációja miatt széles körű körülmények között is megbízható. Ugyanakkor sűrű növényzet esetén, amikor az LAI értéke magasra emelkedik, telítődhet.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Az index értéke -1 és 1 között mozog. A zöld növényzet esetében a szokásos tartomány 0,2 és 0,8 között van.

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

Ez az index a talajhoz igazított vegetációs indexen (SAVI) alapul. A lombkorona háttérkorrekciós tényezőjének standard értékeként 0,16-ot használ. Rondeaux (1996) megállapította, hogy ez az érték alacsony növényborítás esetén nagyobb talajváltozást biztosít, mint a SAVI, miközben 50%-nál nagyobb növényborítás esetén fokozott érzékenységet mutat. Ez az index leginkább olyan, viszonylag ritkás növényzetű területeken használható, ahol a talaj látható a lombkoronán keresztül.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Hivatkozás: Rondeaux, G., M. Steven és F. Baret. „Optimization of Soil-Adjusted Vegetation Indices.” Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – Renormalizált különbségű vegetációs index

Ez az index a közeli infravörös és a vörös hullámhosszok közötti különbséget, valamint az NDVI-t használja az egészséges növényzet kiemelésére. Nem érzékeny a talaj és a nap megfigyelési geometriájának hatásaira.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Hivatkozás: Roujean, J. és F. Breon. „A növényzet által elnyelt PAR becslése kétirányú reflexiós mérések alapján.” Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – talajkorrigált vegetációs index

Ez az index hasonló az NDVI-hez, de kiszűri a talajpixelek hatását. A lombkorona háttérkorrekciós tényezőjét, az _L_-t használja, amely a vegetáció sűrűségének függvénye, és gyakran megköveteli a vegetáció mennyiségének előzetes ismeretét. Huete (1988) az _L_=0,5 értéket javasolja az elsőrendű talajháttér-változások figyelembevételéhez. Ez az index leginkább olyan területeken használható, ahol a növényzet viszonylag ritka, és a talaj a lombkoronán keresztül látható.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Hivatkozás: Huete, A. „A Soil-Adjusted Vegetation Index (SAVI).” Remote Sensing of Environment 25 (1988): 295–309._

***

## TDVI – Transformed Difference Vegetation Index

Ez az index hasznos a városi környezetben található növényzetborítás figyelemmel kísérésére. Nem telítődik, ellentétben az NDVI-vel és a SAVI-vel.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Hivatkozás: Bannari, A., H. Asalhi és P. Teillet. „Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping” In Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS '02, IEEE International, 5. kötet (2002)._

***

## VARI – Látható, légköri hatásoknak ellenálló index

Ez az index az ARVI-n alapul, és a légköri hatásokra alacsony érzékenységgel rendelkező jelenetekben a növényzet arányának becslésére szolgál.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Hivatkozás: Gitelson, A., és társai. „Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction.” International Journal of Remote Sensing 23 (2002): 2537–2562._

***

## WDRVI – Széles dinamikatartományú növényzetindex

Ez az index hasonló az NDVI-hez, de súlyozási együtthatót (_a_) alkalmaz, hogy csökkentse a közeli infravörös és a vörös jelek NDVI-hez való hozzájárulása közötti eltérést. A WDRVI különösen hatékony olyan területeken, ahol közepes vagy magas a növényzet sűrűsége, és az NDVI értéke meghaladja a 0,6-ot. Az NDVI értéke általában eléri a platót, amikor a növényzet aránya és a levélfelületi index (LAI) növekszik, míg a WDRVI érzékenyebb a növényzet arányának szélesebb tartományára és az LAI változásaira.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

A súlyozási együttható (_a_) értéke 0,1 és 0,2 között lehet. Henebry, Viña és Gitelson (2004) a 0,2-es értéket javasolják.

_Hivatkozások_

_Gitelson, A. „Széles dinamikatartományú vegetációs index a vegetáció biofizikai jellemzőinek távoli számszerűsítéséhez.” Journal of Plant Physiology 161, 2. sz. (2004): 165–173._

_Henebry, G., A. Viña és A. Gitelson. „A széles dinamikatartományú vegetációs index és annak potenciális hasznossága a hiányelemzésben.” Gap Analysis Bulletin 12: 50–56._
