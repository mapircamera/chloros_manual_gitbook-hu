---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Multispektrális indexképletek

Az alábbi indexképletek az Survey3 szűrő átlagos átviteli tartományainak kombinációját használják:

<table><thead><tr><th align="center">Survey3 szűrő színe</th><th width="196.199951171875" align="center">Survey3 Szűrő neve</th><th width="159.800048828125" align="center">Áteresztési tartomány (FWHM)</th><th align="center">Átlagos transzmisszió</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB – Blue</td><td align="center">468–483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN – Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>Ezen képletek használata esetén a név végződése „\_1” vagy „\_2” lehet, ami azt jelzi, hogy melyik NIR szűrőt használták: az NIR1-et vagy az NIR2-et.

***

## EVI – Továbbfejlesztett növényzetindex

Ezt az indexet eredetileg az MODIS adatokkal való használatra fejlesztették ki, az NDVI javításaként, a magas levélfelületi indexű (LAI) területeken a növényzeti jel optimalizálásával. Leginkább olyan magas LAI értékű régiókban hasznos, ahol az NDVI érték telítődhet. A kék reflektancia tartományt használja a talaj háttérjelek korrekciójára és a légköri hatások, beleértve az aeroszol szórás csökkentésére.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Az EVI értékeknek a növényzeti pixelek esetében 0 és 1 között kell lenniük. A fényes elemek, mint például a felhők és a fehér épületek, valamint a sötét elemek, mint például a víz, rendellenes pixelértékeket eredményezhetnek egy EVI képen. Az EVI kép létrehozása előtt ki kell maszkolni a felhőket és a világos elemeket a reflexiós képből, és opcionálisan 0 és 1 közötti küszöbértéket kell beállítani a pixelértékekre.

_Hivatkozás: Huete, A. et al. „Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices.” Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Erdőborítás-index 1

Ez az index a vörös szélső sávot tartalmazó multispektrális reflektancia-képek segítségével különbözteti meg az erdő lombkoronáját más növényzetektől.

$$
FCI1 = Red * RedEdge
$$

Az erdős területek alacsonyabb FCI1 értékekkel rendelkeznek a fák alacsonyabb visszaverődése és a lombkorona árnyékainak jelenléte miatt.

_Hivatkozás: Becker, Sarah J., Craig S.T. Daughtry és Andrew L. Russ. „Robusztus erdőborítási indexek multispektrális képekhez.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 – Erdőborítás-index 2

Ez az index a vörös szélső sávot nem tartalmazó multispektrális visszaverődési képek segítségével különbözteti meg az erdő lombkoronáját más növényzetektől.

$$
FCI2 = Red * NIR
$$

Az erdős területek alacsonyabb FCI2 értékeket mutatnak a fák alacsonyabb visszaverődése és a lombkorona árnyékainak jelenléte miatt.

_Hivatkozás: Becker, Sarah J., Craig S.T. Daughtry és Andrew L. Russ. „Robust forest cover indices for multispectral images.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505–512._

***

## GEMI – Globális környezeti megfigyelési index

Ezt a nemlineáris vegetációs indexet műholdas képek alapján történő globális környezeti megfigyelésre használják, és célja a légköri hatások korrigálása. Hasonló az NDVI-hez, de kevésbé érzékeny a légköri hatásokra. A csupasz talaj befolyásolja, ezért nem ajánlott ritka vagy közepesen sűrű növényzetű területeken történő használatra.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Ahol:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Hivatkozás: Pinty, B., és M. Verstraete. GEMI: egy nemlineáris index a globális növényzet műholdas megfigyeléséhez. Vegetation 101 (1992): 15-20._

***

## GARI – Green légköri hatásokra ellenálló index

Ez az index érzékenyebb a klorofillkoncentrációk széles tartományára, és kevésbé érzékeny a légköri hatásokra, mint az NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

A gamma-állandó egy súlyozó függvény, amely a légkörben uralkodó aeroszolos körülményektől függ. Az ENVI 1,7-es értéket használ, amely Gitelson, Kaufman és Merzylak (1996, 296. oldal) által ajánlott érték.

_Hivatkozás: Gitelson, A., Y. Kaufman és M. Merzylak. „Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.” Remote Sensing of Environment 58 (1996): 289–298._

***

## GCI – Green klorofill-index

Ezt az indexet a levelek klorofilltartalmának becslésére használják számos növényfaj esetében.

$$
GCI = {NIR \over Green} - 1
$$

A széles NIR és zöld hullámhosszúságok jobb előrejelzést nyújtanak a klorofilltartalomra vonatkozóan, miközben nagyobb érzékenységet és magasabb jel-zaj arányt tesznek lehetővé.

_Hivatkozás: Gitelson, A., Y. Gritz és M. Merzlyak. „A levélklorofill-tartalom és a spektrális visszaverődés közötti összefüggések, valamint algoritmusok a magasabb rendű növények leveleinek roncsolásmentes klorofill-értékeléséhez.” Journal of Plant Physiology 160 (2003): 271–282._

***

## GLI – Green levélindex

Ezt az indexet eredetileg digitális RGB kamerával való használatra tervezték a búza borításának mérésére, ahol a vörös, zöld és kék digitális számok (DN-ek) 0 és 255 között mozognak.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Az GLI értékek -1 és +1 között mozognak. A negatív értékek a talajt és az élettelen elemeket jelzik, míg a pozitív értékek a zöld leveleket és szárakat jelzik.

_Hivatkozás: Louhaichi, M., M. Borman és D. Johnson. „Térbeli elhelyezkedésű platform és légi felvételek a legeltetés búzára gyakorolt hatásainak dokumentálásához.” Geocarto International 16, 1. sz. (2001): 65–70._

***

## GNDVI – Green normalizált növényzetindex

Ez az index hasonló az NDVI-hez, azzal a különbséggel, hogy a vörös spektrum helyett az 540–570 nm-es zöld spektrumot méri. Ez az index érzékenyebb a klorofill-koncentrációra, mint az NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Hivatkozás: Gitelson, A. és M. Merzlyak. „A magasabb rendű növények leveleiben található klorofill-koncentráció távérzékelése.” Advances in Space Research 22 (1998): 689–692._

***

## GOSAVI – Green Optimalizált, talajhoz igazított vegetációs index

Ezt az indexet eredetileg színes infravörös fényképezéssel tervezték a kukorica nitrogénigényének előrejelzésére. Hasonló az OSAVI-hez, de a zöld sávot pirosra cseréli.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Hivatkozás: Sripada, R. et al. „A kukorica szezon közbeni nitrogénigényének meghatározása légi színes-infravörös fényképezés segítségével.” Doktori értekezés, Észak-Karolinai Állami Egyetem, 2005._

***

## GRVI – Green arányú vegetációs index

Ez az index érzékeny az erdő lombkoronájában zajló fotoszintézis mértékére, mivel a zöld és a vörös visszaverődést erősen befolyásolják a levélpigmentek változásai.

$$
GRVI = {NIR \over Green }
$$

_Hivatkozás: Sripada, R. és társai. „Légi színes infravörös fényképezés a kukorica szezon eleji nitrogénigényének meghatározásához.” Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Talajhoz igazított vegetációs index

Ezt az indexet eredetileg színes infravörös fényképezéssel tervezték a kukorica nitrogénigényének előrejelzésére. Hasonló az SAVI-hez, de a zöld sávot pirosra cseréli.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Hivatkozás: Sripada, R. et al. „A kukorica szezonközi nitrogénigényének meghatározása légi színes-infravörös fényképezés segítségével.” Doktori disszertáció, Észak-Karolinai Állami Egyetem, 2005._

***

## LAI – Lombozatfelületi index

Ezt az indexet a lombozatborítás becslésére, valamint a növénytermesztés növekedésének és hozamának előrejelzésére használják. Az ENVI a zöld LAI-et a Boegh et al (2002) által megadott következő empirikus képlet segítségével számítja ki:

$$
LAI = 3.618 * EVI - 0.118
$$

Ahol EVI:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

A magas LAI értékek általában körülbelül 0 és 3,5 között mozognak. Ha azonban a felvételen felhők és más, telített pixeleket eredményező fényes elemek találhatók, az LAI értékek meghaladhatják a 3,5-öt. Ideális esetben a felvételről el kell távolítani a felhőket és a fényes elemeket, mielőtt LAI képet készítene.

_Hivatkozás: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde és A. Thomsen. „Légi multispektrális adatok a levélfelületi index, a nitrogénkoncentráció és a fotoszintézis hatékonyságának számszerűsítéséhez a mezőgazdaságban.” Remote Sensing of Environment 81, 2-3. sz. (2002): 179-193._

***

## LCI – Levélklorofill-index

Ezt az indexet a magasabb rendű növények klorofilltartalmának becslésére használják, mivel érzékeny a klorofill abszorpciója által okozott visszaverődés változásaira.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Hivatkozás: Datt, B. „Az eukaliptuszlevelek víztartalmának távérzékelése.” Journal of Plant Physiology 154, 1. sz. (1999): 30–36._

***

## MNLI – Módosított nemlineáris index

Ez az index a nemlineáris index (NLI) továbbfejlesztése, amely beépíti a talajhoz igazított vegetációs indexet (SAVI) a talajháttér figyelembevétele érdekében. Az ENVI 0,5-ös értékű lombkorona-háttérkorrekciós tényezőt (_L_) használ.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Hivatkozás: Yang, Z., P. Willis és R. Mueller. „Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.” Proceedings of the Pecora 17 Remote Sensing Symposium (2008), Denver, CO._

***

## MSAVI2 – Módosított talajhoz igazított vegetációs index 2

Ez az index a Qi és társai (1994) által javasolt MSAVI index egyszerűbb változata, amely továbbfejlesztése a talajhoz igazított vegetációs indexnek (SAVI). Csökkenti a talajzajt és növeli a vegetációs jel dinamikatartományát. Az MSAVI2 egy induktív módszeren alapul, amely nem használ állandó _L_ értéket (mint az SAVI) az egészséges növényzet kiemelésére.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Hivatkozás: Qi, J., A. Chehbouni, A. Huete, Y. Kerr és S. Sorooshian. „A Modified Soil Adjusted Vegetation Index.” Remote Sensing of Environment 48 (1994): 119–126._

***

## NDRE – Normalizált különbség RedEdge

Ez az index hasonló az NDVI-hez, de az Red helyett az NIR és az RedEdge közötti kontrasztot hasonlítja össze, ami gyakran hamarabb észleli a növényzet stresszét.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Normalizált növényzet-különbség index

Ez az index az egészséges, zöld növényzet mértékét jelzi. A normalizált különbség képletének és a klorofill legmagasabb abszorpciós és reflexiós tartományainak felhasználásának kombinációja miatt széles körű körülmények között megbízható. Sűrű növényzet esetén azonban telítődhet, amikor az LAI értéke magas lesz.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

Ennek az indexnek az értéke -1 és 1 között mozog. A zöld növényzetre jellemző tartomány 0,2 és 0,8 között van.

_Hivatkozás: Rouse, J., R. Haas, J. Schell és D. Deering. Monitoring Vegetation Systems in the Great Plains with ERTS. Third ERTS Symposium, NASA (1973): 309-317._

***

## NLI – Nemlineáris index

Ez az index abból indul ki, hogy számos vegetációs index és a felszíni biofizikai paraméterek közötti kapcsolat nemlineáris. Lineárissá teszi azokat a felszíni paraméterekkel való kapcsolatokat, amelyek általában nemlineárisak.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Hivatkozás: Goel, N. és W. Qin. „A lombkorona szerkezetének hatása a különböző vegetációs indexek és az LAI, valamint az Fpar közötti kapcsolatokra: számítógépes szimuláció.” Remote Sensing Reviews 10 (1994): 309–347._

***

## OSAVI – Optimalizált talajkorrigált vegetációs index

Ez az index a talajkorrigált vegetációs indexen (SAVI) alapul. A lombkorona háttérkorrekciós tényezőjének standard értékeként 0,16-ot használ. Rondeaux (1996) megállapította, hogy ez az érték alacsony növényzetborítás esetén nagyobb talajváltozást biztosít, mint az SAVI, miközben 50%-nál nagyobb növényzetborítás esetén fokozott érzékenységet mutat. Ez az index leginkább olyan területeken használható, ahol a növényzet viszonylag ritka, és a talaj látható a lombkoronán keresztül.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Hivatkozás: Rondeaux, G., M. Steven és F. Baret. „Optimization of Soil-Adjusted Vegetation Indices.” Remote Sensing of Environment 55 (1996): 95–107._

***

## RDVI – Renormalizált növényzet-különbség index

Ez az index a közeli infravörös és a vörös hullámhosszok közötti különbséget használja az NDVI-szel együtt az egészséges növényzet kiemelésére. Nem érzékeny a talaj és a nap megfigyelési geometriájának hatására.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Hivatkozás: Roujean, J. és F. Breon. „A növényzet által elnyelt PAR becslése kétirányú visszaverődési mérések alapján.” Remote Sensing of Environment 51 (1995): 375–384._

***

## SAVI – Talajkorrigált növényzetindex

Ez az index hasonló az NDVI-hez, de elnyomja a talajpixelek hatását. A lombozat háttérkorrekciós tényezőjét, _L_-t használja, amely a növényzet sűrűségének függvénye, és gyakran előzetes ismeretet igényel a növényzet mennyiségéről. Huete (1988) az _L_=0,5 értéket javasolja az elsőrendű talajháttér-változások figyelembevételéhez. Ez az index leginkább olyan területeken használható, ahol a növényzet viszonylag ritka, és a talaj látható a lombozat alatt.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Hivatkozás: Huete, A. „A Soil-Adjusted Vegetation Index (SAVI).” Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI – Transformed Difference Vegetation Index

Ez az index hasznos a városi környezet növényzetborításának figyelemmel kíséréséhez. Nem telítődik, mint az NDVI és az SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Hivatkozás: Bannari, A., H. Asalhi és P. Teillet. „Transformed Difference Vegetation Index (TDVI) for Vegetation Cover Mapping” In Proceedings of the Geoscience and Remote Sensing Symposium, IGARSS &#x27;02, IEEE International, 5. kötet (2002)._

***

## VARI – Látható, légköri hatásoknak ellenálló index

Ez az index az ARVI-en alapul, és a légköri hatásokra alacsony érzékenységgel rendelkező jelenetekben a növényzet arányának becslésére szolgál.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Hivatkozás: Gitelson, A. et al. „Vegetation and Soil Lines in Visible Spectral Space: A Concept and Technique for Remote Estimation of Vegetation Fraction.” International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI – Széles dinamikatartományú növényzetindex

Ez az index hasonló az NDVI-hez, de súlyozási együtthatót (_a_) használ, hogy csökkentse a közeli infravörös és a vörös jelek hozzájárulásának eltérését az NDVI-hez. Az WDRVI különösen hatékony olyan jelenetekben, ahol a növényzet sűrűsége közepes vagy magas, amikor az NDVI meghaladja a 0,6-ot. Az NDVI értéke általában kiegyenlítődik, amikor a növényzet aránya és a lombozatindex (LAI) növekszik, míg az WDRVI érzékenyebb a növényzet arányának szélesebb tartományára és az LAI változásaira.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

A súlyozási együttható (_a_) értéke 0,1 és 0,2 között lehet. Henebry, Viña és Gitelson (2004) a 0,2-es értéket javasolja.

_Hivatkozások_

_Gitelson, A. „Wide Dynamic Range Vegetation Index for Remote Quantification of Biophysical Characteristics of Vegetation.” Journal of Plant Physiology 161, No. 2 (2004): 165-173._

_Henebry, G., A. Viña és A. Gitelson. „The Wide Dynamic Range Vegetation Index and its Potential Utility for Gap Analysis.” Gap Analysis Bulletin 12: 50–56._
