---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/supported-cameras
---

# Támogatott kamerák A

Chloros **minden platformon** (Windows, Linux amd64 és Linux arm64/Jetson) feldolgozza a két MAPIR kameracsalád által készített felvételeket:

* **Survey3** — Survey3W (széles látószögű) és Survey3N (keskeny látószögű) kamerák. Bemenet: `RAW+JPG`.
* **LATTICE**— M3C és M3M multispektrális kameramodulok. Bemenet: `.tif`/`.tiff` felvételek. A LATTICE kamerák**élőben is vezérelhetők** a Chloros-ból — a GUI „Cameras” fülén keresztül (Windows) vagy az `chloros-cli lattice` / a Python SDK-en keresztül (Windows és Linux) — beleértve a szinkronizált többkamerás rendszereket is. Lásd a [LATTICE útmutatót](lattice/).

A feldolgozási folyamat `.dng` bemeneti fájlokat is elfogad.

## Survey3

<table data-header-hidden><thead><tr><th width="156">Gyártó</th><th width="250">Kamera modell</th><th width="138">Szűrőmodell</th><th width="187">Képtípus</th></tr></thead><tbody><tr><td><strong>Gyártó</strong></td><td><strong>Kamera modell</strong></td><td><strong>Szűrőmodell</strong></td><td><strong>Képtípus</strong></td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RGN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>OCN</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NGB</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>RE</td><td>RAW+JPG, JPG</td></tr><tr><td>MAPIR</td><td>Survey3W, Survey3N</td><td>NIR</td><td>RAW+JPG, JPG</td></tr></tbody></table>## LATTICE

A LATTICE sorozat egy moduláris multispektrális kamerarendszer, amely a Sony IMX265 globális zárú érzékelőn (3,1 MP, 3,45 µm-es képpontok) alapul. Minden kamera modellkódként tárolja az azonosítóját:

```
<sensor>-<lens>-F<filter>        e.g.  M3C-L41-FRGN,  M3M-L87-F550
``` Az

Chloros ezt az `LATT-` előtaggal jeleníti meg (például `LATT-M3M-L41-F550`), és ez a modellszöveg vezérli az összes további lépést — az érzékelőprofil, a sávelrendezés és a kalibrálás automatikusan meghatározódik; kameránként nincs szükség semmilyen beállításra. Az objektívszám a **fokban kifejezett vízszintes látómező**: `L41` = keskeny 41°, `L87` = széles 87°.

Két érzékelőkonfiguráció létezik:

| Konfiguráció | Érzékelő      | Szűrőtípus                           | Sávok kameránként                                                        |
| ------------- | ----------- | ------------------------------------- | ----------------------------------------------------------------------- |
| **M3C**       | Bayer színes | Háromsávos sávszűrő                       | 3 spektrális sáv egyetlen expozícióból                                 |
| **M3M**       | Monokróm  | Egyetlen keskenysávú interferencia szűrő | 1 kalibrált sáv — több M3M kamera kombinálásával növényzetindexek számíthatók |

### M3C (Bayer) szűrőopciók

| Szűrő | Sávok (név @ középpont nm / FWHM nm)       |
| ------ | ---------------------------------------- |
| `FRGB` | Blue 475/30 · Green 550/30 · Red 625/30  |
| `FRGN` | Red 660/21 · Green 550/30 · NIR 850/30   |
| `FOCN` | Orange 618,4/46 · Cyan 494,1/40 · NIR 821,6/54 (mért — lásd alább) |
| `FNGB` | Blue 475/30 · Green 550/30 · NIR 850/30  |

Az `FOCN` esetében az értékek a szűrő **mért** félmaximális sávszélességei (Chloros 1.3.0 és újabb verziók; a korábbi verziók a műszaki adatlap értékeit használták: 615/21 · 490/38 · 808/14). A sávnevek változatlanok maradnak: `Orange_615`, `Cyan_490` és `NIR_808`, a mért középérték és szélesség pedig az, amit a Chloros minden kalibrált exportfájlba bejegyz.

### M3M (mono) szűrőkatalógus — 23 SKU

Az F-szám az SKU-címke; a mért sáv (amelyet minden kalibrált exporttermékre rányomtatnak) a tételenkénti szűrő-letapogatás eredménye:

| SKU    | Közép (nm, mért) | FWHM-szélek (nm) | Szélesség (nm) |
| ------ | --------------------- | --------------- | ---------- |
| F385   | 379,4                 | 367–392         | 25         |
| F405   | 403,9                 | 390–417         | 27         |
| F450   | 443,7                 | 430–458         | 28         |
| F485   | 489,7                 | 478–502         | 24         |
| F520   | 519,9                 | 504–536         | 32         |
| F550   | 548,4                 | 531–566         | 35         |
| F590   | 589,0                 | 570–608         | 38         |
| F615   | 623,8                 | 614–634         | 20         |
| F632   | 633,4                 | 616–651         | 35         |
| F650   | 651,1                 | 636–666         | 30         |
| F685   | 686,2                 | 675–698         | 23         |
| F715   | — (névleges)           | 706–724         | 18         |
| F725   | 725.2                 | 712–738         | 26         |
| F750   | 746.0                 | 729–763         | 34         |
| F780   | 775.1                 | 754–796         | 42         |
| F808   | 810.3                 | 789–832         | 43         |
| F832   | 826,1                 | 810–843         | 33         |
| F850   | 846,5                 | 828–865         | 37         |
| F880   | — (névleges)           | 867–893         | 26         |
| F905   | — (névleges)           | 892–920         | 28         |
| F940   | 940,6                 | 923–958         | 35         |
| F950   | 945,1                 | 929–961         | 32         |
| F988 † | 985,3                 | 968–1003        | 35         |

_„A sávhatárokat a félmaximum szélességének értékeiként mérik aMAPIR

tételenkénti szűrőletapogatásai alapján — ugyanazokat az értékeket, amelyeket aChloros

minden kalibrált exportba bejegyz.”_ „— (névleges)” = még nincs tételhez tartozó szkennelés; ezeknél az SKU-knél a megadott középérték az SKU-szám, a szélesség pedig a gyártó által megadott érték.

† „Az F988 reflexióját egy helyszíni reflexiós panel segítségével kalibrálják: a sáv a DAQ fényérzékelő kalibrált tartományán kívül esik, ezért a Chloros a legfrissebb panelről készült felvételt alkalmazza, és azt a panelek közötti mérések között is megtartja.” Lásd a [Kalibrációs célpontok](calibration-targets.md) című részt.

Az élő kameravezérlés, a mátrixok, a hálózati beállítások és a radiometrikus feldolgozási lánc tekintetében lásd a [LATTICE útmutatót](lattice/).
