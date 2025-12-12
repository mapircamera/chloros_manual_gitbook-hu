---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Multispektrális index képletek

Az alábbi index képletek az Survey3 szűrő átlagos átviteli tartományainak kombinációját használják:

<table><thead><tr><th align="center">Survey3 szűrő színe</th><th width="196.199951171875" align="center">Survey3 szűrő neve</th><th width="159.800048828125" align="center">Átviteli tartomány (FWHM)</th><th align="center">Átlagos átviteli tartomány</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476-512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Ezen képletek használata esetén a név végződhet „\_1” vagy „\_2” karakterekkel, amelyek azt jelzik, hogy melyik NIR szűrőt használták, azaz az NIR1 vagy az NIR2 szűrőt.

***

## EVI – Továbbfejlesztett vegetációs index

Ezt az indexet eredetileg az MODIS adatokhoz fejlesztették ki, az NDVI továbbfejlesztéseként, a magas levélterületi indexű (LAI) területeken a vegetációs jel optimalizálásával. Leginkább olyan magas LAI régiókban hasznos, ahol az NDVI telítettséget érhet el. A kék reflexiós tartományt használja a talaj háttérjelek korrekciójára és a légköri hatások, beleértve az aeroszolszórás csökkentésére.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Az EVI értékeknek 0 és 1 között kell lenniük a növényzet pixelek esetében. A felhők és fehér épületekhez hasonló világos elemek, valamint a vízhez hasonló sötét elemek rendellenes pixelértékeket eredményezhetnek egy EVI képen. Az EVI kép létrehozása előtt el kell fedni a felhőket és a fényes elemeket a visszaverődési képről, és opcionálisan a pixelértékeket 0 és 1 között kell küszöbértékkel ellátni.

_Hivatkozás: Huete, A. et al. „Az MODIS növényzetindexek radiometrikus és biofizikai teljesítményének áttekintése.” Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 – Erdőborítás-index 1

Ez az index a vörös szélső sávot is magában foglaló multispektrális visszaverődési képek segítségével különbözteti meg az erdő lombkoronáját más típusú növényzettől.

$$
FCI1 = Red * RedEdge
$$

Az erdős területek alacsonyabb FCI1 értékekkel rendelkeznek, mivel a fák visszaverődése alacsonyabb, és az erdő lombkoronájában árnyékok vannak.

_Hivatkozás: Becker, Sarah J., Craig S.T. Daughtry és Andrew L. Russ. „Robusztus erdőborítási indexek multispektrális képekhez.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 – Erdőborítási index 2

Ez az index a vörös szélű sávot nem tartalmazó multispektrális visszaverődési képek segítségével különbözteti meg az erdő lombkoronáját más típusú növényzettől.

$$
FCI2 = Red * NIR
$$

Az erdős területek alacsonyabb FCI2 értékekkel rendelkeznek, mivel a fák visszaverődése alacsonyabb, és az erdő lombkoronájában árnyékok vannak.

_Hivatkozás: Becker, Sarah J., Craig S.T. Daughtry és Andrew L. Russ. „Robusztus erdőborítási indexek multispektrális képekhez.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI – Globális környezeti monitoring index

Ezt a nemlineáris vegetációs indexet műholdas képek alapján végzett globális környezeti monitoringhoz használják, és célja a légköri hatások korrekciója. Hasonló az NDVI indexhez, de kevésbé érzékeny a légköri hatásokra. Hatással van rá a csupasz talaj, ezért ritkás vagy közepesen sűrű növényzetű területeken nem ajánlott használni.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Hol:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Hivatkozás: Pinty, B. és M. Verstraete. GEMI: nemlineáris index a globális növényzet műholdas megfigyeléséhez. Vegetation 101 (1992): 15-20._

***

## GARI - Green Légköri ellenálló index

Ez az index érzékenyebb a klorofillkoncentrációk széles tartományára és kevésbé érzékeny a légköri hatásokra, mint az NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

A gamma-állandó egy súlyozó függvény, amely a légkörben található aeroszolok állapotától függ. Az ENVI 1,7-es értéket használ, amely Gitelson, Kaufman és Merzylak (1996, 296. oldal) által ajánlott érték.

_Hivatkozás: Gitelson, A., Y. Kaufman és M. Merzylak. „A Green csatorna használata a globális növényzet távérzékelésében az EOS-MODIS segítségével.” Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI – Green klorofillindex

Ez az index a levelek klorofilltartalmának becslésére szolgál számos növényfaj esetében.

$$
GCI = {NIR \over Green} - 1
$$

A széles NIR és zöld hullámhosszok jobb előrejelzést tesznek lehetővé a klorofilltartalomra vonatkozóan, miközben nagyobb érzékenységet és magasabb jel-zaj arányt biztosítanak.

_Hivatkozás: Gitelson, A., Y. Gritz és M. Merzlyak. „A levél klorofilltartalma és a spektrális visszaverődés közötti kapcsolatok, valamint algoritmusok a magasabb rendű növények leveleinek roncsolásmentes klorofilltartalom-meghatározásához.” Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI – Green Levélindex

Ezt az indexet eredetileg digitális RGB kamerával való használatra tervezték a búza borításának mérésére, ahol a piros, zöld és kék digitális számok (DN-ek) 0 és 255 között mozognak.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Az GLI értékek -1 és +1 között mozognak. A negatív értékek a talajt és az élettelen elemeket jelölik, míg a pozitív értékek a zöld leveleket és szárakat jelölik.

_Hivatkozás: Louhaichi, M., M. Borman és D. Johnson. „Térbeli elhelyezkedésű platform és légi felvételek a legeltetés búzára gyakorolt hatásának dokumentálására.” Geocarto International 16, 1. szám (2001): 65-70._

***

## GNDVI - Green Normalizált növényzetindex

Ez az index hasonló az NDVI-hez, azzal a különbséggel, hogy a vörös spektrum helyett az 540–570 nm-es zöld spektrumot méri. Ez az index érzékenyebb a klorofillkoncentrációra, mint az NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Hivatkozás: Gitelson, A. és M. Merzlyak. „A klorofillkoncentráció távérzékelése magasabb rendű növények leveleiben.” Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Optimalizált talajhoz igazított vegetációs index

Ezt az indexet eredetileg színes infravörös fényképezéssel tervezték a kukorica nitrogénigényének előrejelzésére. Hasonló az OSAVI-hez, de a zöld sávot pirosra cseréli.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Hivatkozás: Sripada, R., et al. „A kukorica szezonbeli nitrogénigényének meghatározása légi színes infravörös fényképezéssel.” Ph.D. disszertáció, Észak-Karolina Állami Egyetem, 2005._

***

## GRVI - Green arányos vegetációs index

Ez az index érzékeny az erdő lombkoronájának fotoszintézisére, mivel a zöld és piros visszaverődést erősen befolyásolják a levélpigmentek változásai.

$$
GRVI = {NIR \over Green }
$$

_Hivatkozás: Sripada, R. et al. „Légi színes infravörös fényképezés a kukorica korai nitrogénigényének meghatározásához.” Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Talajhoz igazított vegetációs index

Ezt az indexet eredetileg színes infravörös fényképezéssel tervezték a kukorica nitrogénigényének előrejelzésére. Hasonló az SAVI-hez, de a zöld sávot pirosra cseréli.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Hivatkozás: Sripada, R., et al. „A kukorica szezonbeli nitrogénigényének meghatározása légi színes infravörös fényképezés segítségével.” Ph.D. disszertáció, Észak-Karolina Állami Egyetem, 2005._

***

## LAI – Levélterület-index

Ez az index a lombborítás becslésére, valamint a növény növekedésének és hozamának előrejelzésére szolgál. Az ENVI a zöld LAI értéket Boegh et al (2002) következő empirikus képlete alapján számítja ki:

$$
LAI = 3.618 * EVI - 0.118
$$

Ahol EVI:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

A magas LAI értékek általában 0 és 3,5 között mozognak. Ha azonban a kép felhőket és más fényes elemeket tartalmaz, amelyek telített pixeleket eredményeznek, az LAI értékek meghaladhatják a 3,5-öt. Ideális esetben a felhőket és a fényes elemeket el kell fedni a képről, mielőtt LAI képet készít.

_Hivatkozás: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde és A. Thomsen. „Légi multispektrális adatok a levélterület-index, a nitrogénkoncentráció és a fotoszintézis hatékonyságának számszerűsítéséhez a mezőgazdaságban.” Remote Sensing of Environment 81, no. 2-3 (2002): 179-193._

***

## LCI – Levélklorofill-index

Ez az index a magasabb rendű növények klorofilltartalmának becslésére szolgál, és érzékeny a klorofill abszorpciója által okozott visszaverődés változásaira.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Hivatkozás: Datt, B. „Az eukaliptuszlevelek víztartalmának távérzékelése.” Journal of Plant Physiology 154, 1. sz. (1999): 30-36._

***

## MNLI – Módosított nemlineáris index

Ez az index a nemlineáris index (NLI) továbbfejlesztése, amely magában foglalja a talajhoz igazított vegetációs indexet (SAVI) a talaj háttérének figyelembevétele érdekében. Az ENVI 0,5-ös értékű lombkorona-háttér-igazítási tényezőt (_L_) használ.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Hivatkozás: Yang, Z., P. Willis és R. Mueller. „A sávarány-javított AWIFS-kép hatása a növénykultúrák osztályozásának pontosságára.” A Pecora 17 távérzékelési szimpózium (2008) jegyzőkönyve, Denver, CO._

***

## MSAVI2 – Módosított talajhoz igazított vegetációs index 2

Ez az index a Qi és társai (1994) által javasolt MSAVI index egyszerűbb változata, amely javítja a talajhoz igazított vegetációs indexet (SAVI). Csökkenti a talaj zaját és növeli a vegetációs jel dinamikus tartományát. Az MSAVI2 egy induktív módszeren alapul, amely nem használ állandó _L_ értéket (mint az SAVI) az egészséges vegetáció kiemeléséhez.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Hivatkozás: Qi, J., A. Chehbouni, A. Huete, Y. Kerr és S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.” Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE – Normalizált különbség RedEdge

Ez az index hasonló az NDVI-hez, de az NIR és az RedEdge közötti kontrasztot hasonlítja össze az Red helyett, ami gyakran hamarabb észleli a növényzet stresszét.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalizált növényzetindex

Ez az index az egészséges, zöld növényzet mértékét jelzi. A normalizált különbség képlete és a klorofill legmagasabb abszorpciós és reflexiós tartományainak kombinációja széles körű feltételek mellett is megbízhatóvá teszi. Sűrű növényzet esetén azonban telítetté válhat, amikor az LAI magas értéket ér el.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Az index értéke -1 és 1 között mozog. A zöld növényzet esetében a szokásos tartomány 0,2 és 0,8 között van.

_Hivatkozás: Rouse, J., R. Haas, J. Schell és D. Deering. A növényzet rendszereinek figyelemmel kísérése a Nagy Síkságon az ERTS segítségével. Harmadik ERTS szimpózium, NASA (1973): 309-317._

***

## NLI – nemlineáris index

Ez az index feltételezi, hogy sok növényzetindex és a felszíni biofizikai paraméterek közötti kapcsolat nemlineáris. Lineárissá teszi a felszíni paraméterekkel való kapcsolatokat, amelyek általában nemlineárisak.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Hivatkozás: Goel, N. és W. Qin. „A lombkorona szerkezetének hatása a különböző vegetációs indexek és az LAI és Fpar közötti kapcsolatokra: számítógépes szimuláció.” Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI – Optimalizált talajhoz igazított vegetációs index

Ez az index a talajhoz igazított vegetációs indexen (SAVI) alapul. A lombkorona háttér-igazítási tényezőjének standard értékeként 0,16-ot használ. Rondeaux (1996) megállapította, hogy ez az érték alacsony növényzetfedés esetén nagyobb talajváltozást biztosít, mint az SAVI, miközben 50% feletti növényzetfedés esetén nagyobb érzékenységet mutat. Ez az index leginkább olyan területeken használható, ahol a növényzet viszonylag ritka, és a talaj a lombkorona alatt látható.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Hivatkozás: Rondeaux, G., M. Steven és F. Baret. „Optimization of Soil-Adjusted Vegetation Indices” (Talajhoz igazított vegetációs indexek optimalizálása). Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI – Renormalizált különbség vegetációs index

Ez az index a közeli infravörös és a vörös hullámhosszok közötti különbséget használja az NDVI-szel együtt, hogy kiemelje az egészséges növényzetet. Nem érzékeny a talaj és a nap megfigyelési geometriájának hatására.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Hivatkozás: Roujean, J. és F. Breon. „A növényzet által elnyelt PAR becslése kétirányú visszaverődésmérésekből.” Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI – Talajjal korrigált növényzetindex

Ez az index hasonló az NDVI-hez, de elnyomja a talajpixelek hatását. A lombkorona háttérkorrekciós tényezőjét, _L_ használja, amely a növényzet sűrűségének függvénye, és gyakran előzetes ismeretet igényel a növényzet mennyiségéről. Huete (1988) az _L_=0,5 optimális értéket javasolja az elsőrendű talajháttér-változások figyelembevételére. Ez az index leginkább olyan területeken használható, ahol a növényzet viszonylag ritka, és a talaj a lombkorona alatt látható.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Hivatkozás: Huete, A. „Talajhoz igazított vegetációs index (SAVI).” Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI – Transzformált különbség vegetációs index

Ez az index hasznos a városi környezet vegetációs borításának figyelemmel kíséréséhez. Nem telítődik, mint az NDVI és az SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Hivatkozás: Bannari, A., H. Asalhi és P. Teillet. „Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping” (Transzformált növényzetindex (TDVI) a növényzet lefedettségének feltérképezéséhez) In Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, 5. kötet (2002)._

***

## VARI – Látható légköri ellenállási index

Ez az index az ARVI-en alapul, és a légköri hatásokra kevéssé érzékeny növényzet arányának becslésére szolgál.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Hivatkozás: Gitelson, A. et al. „Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Széles dinamikatartományú vegetációs index

Ez az index hasonló az NDVI-hez, de súlyozási együtthatót (_a_) használ a közeli infravörös és vörös jelek NDVI-hez való hozzájárulásának eltérésének csökkentésére. Az WDRVI különösen hatékony olyan jelenetekben, ahol a növényzet sűrűsége közepes vagy magas, és az NDVI meghaladja a 0,6-ot. Az NDVI általában kiegyenlítődik, amikor a növényzet aránya és a levélterületi index (LAI) növekszik, míg az WDRVI érzékenyebb a növényzet arányának szélesebb tartományára és az LAI változásaira.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

A súlyozási együttható (_a_) értéke 0,1 és 0,2 között lehet. Henebry, Viña és Gitelson (2004) 0,2 értéket javasolnak.

_Hivatkozások_

_Gitelson, A. „Széles dinamikatartományú vegetációs index a vegetáció biofizikai jellemzőinek távoli számszerűsítéséhez.” Journal of Plant Physiology 161, 2. szám (2004): 165–173._

_Henebry, G., A. Viña és A. Gitelson. „A széles dinamikus tartományú vegetációs index és annak potenciális hasznossága a hiányelemzésben.” Gap Analysis Bulletin 12: 50-56._
