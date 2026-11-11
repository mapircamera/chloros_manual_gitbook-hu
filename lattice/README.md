# LATTICE kamerák

A LATTICE a MAPIR moduláris multispektrális kamerarendszere mezőgazdasági és tudományos képalkotáshoz. Minden LATTICE kamera a Sony IMX265 globális zárú érzékelőn (**3,1 MP, 3,45 µm-es képpontok**) alapul, és**GigE Vision** eszközként csatlakozik Etherneten keresztül. A

Chloros 1.3.0 három felületről vezérli a LATTICE kamerákat élőben — felismerés, élő előnézet, felvétel és szinkronizált többkamerás rendszerek —:

| Felület    | Hol                                                          | Platformok                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| Grafikus felület | **Kamerák** fül a Chloros oldalsávjában                         | Windows 10/11 x64                                        |
| CLI        | `chloros-cli lattice` parancscsalád                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Hardvert keres?**A kameramodulok, objektívek, szűrők és sávok, keretek és rögzítések, kábelek, PoE és trigger-vezetékek leírását a [**LATTICE felhasználói kézikönyv**](https://mapir.gitbook.io/lattice-camera) tartalmazza. Ez a fejezet a kamerák Chloros-ból történő vezérlését tárgyalja.

A LATTICE felvételei szabványos `.tif`/`.tiff` fájlok, és a Chloros mindig a nyers felvételből kezdi feldolgozni őket. A teljes parancs- és API-felületet a [CLI-referencia](../reference/cli-reference.md) és az [SDK-referencia](../reference/sdk-reference.md) tartalmazza.

## Két érzékelőkonfiguráció

| Konfiguráció | Érzékelő       | Szűrő                                | Mit ad vissza egy kamera                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayer-színes | háromsávos sávszűrő                |**Három kalibrált sáv egyetlen expozícióból**                 |
| **M3M**| Monokróm   | egyetlen keskenysávú interferencia szűrő |**Egy kalibrált sáv**; több M3M kamera kombinálásával indexek állíthatók elő |

Mivel az M3M kamera egyetlen szűrő mögött monokróm, minden sáv saját expozíciót kap. Az M3C kamera egyetlen érzékelő-expozícióval lefedi mindhárom sávját.

## Modellkódok és névadás

Minden kamera a GenICam `DeviceUserID` formátumban tárolja azonosítóját modellkódként:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

A Chloros ezt az `LATT-` előtaggal jeleníti meg (például `LATT-M3M-L87-F450`). Ugyanez az `LATT-…` karakterlánc kerül beírásra minden export EXIF `Model` címkéjébe, és a feldolgozott projektekben a kamera kimeneti mappájának nevéként is használatos.

| Komponens | Értékek                                                   | Jelentés                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Érzékelő    | `M3C` / `M3M`                                            | Bayer színes / monokróm                                                                          |
| Objektív | `L41` / `L87`                                            | A szám a **fokban kifejezett vízszintes látómező**: L41 = keskeny (41°), L87 = széles (87°)    |
| Szűrő    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) vagy `F<nm>` (M3M) | Lásd [Szűrők és spektrális sávok](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

A modellszöveg határozza meg az összes további lépést: az Chloros az `DeviceUserID` + `DeviceSerialNumber` értékekből határozza meg az érzékelő profilját, a sávok elrendezését és a gyári kalibrálást. Kameránként nincs semmi beállítani való — lásd [Kamerák csatlakoztatása](connecting.md).

## Szűrők és sávok

A sávközpontok, az FWHM-értékek és a teljes 23 SKU-ból álló M3M-katalógus a termék specifikációi közé tartoznak, ezért a hardver kézikönyvben találhatók: [**Szűrők és spektrális sávok**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Ami a szoftver oldalán fontos: a modellláncban szereplő szűrőkód határozza meg, hogy aChlorosmely termékeket tud létrehozni. Az RGB-szűrős kamerák (`FRGB`) kizárólag debayered és előnézeti termékeket adnak ki — a sávonkénti sugárzás és visszaverődés szélessávú érzékelő esetében nem releváns, ezért a Chloros kihagyja őket, és ezt jelzi is. Minden egyéb szűrő a teljes sugárzás → visszaverődés → index láncot eredményezi.

## A radiometrikus kalibrálás áttekintése

Minden LATTICE kamerát a gyárban egyénileg kalibrálnak egy NIST-nyomon követhető lánc alapján, és minden kamerához egyedi tanúsítványt mellékelnek. Hogy ez mit foglal magában, hogyan mérik, és milyen pontosságot lehet megadni, az a hardver kézikönyvben található: [**Gyári radiometrikus kalibrálás**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

A szoftver oldalán az a fontos, hogy az Chloros a kamera csatlakozásakor a megfelelő kalibrálást állítsa be, és az alkalmazott együtthatókat minden exportáláskor rögzítse — lásd [Kamerák csatlakoztatása](connecting.md).

## Ebben a fejezetben

* [Kamerák csatlakoztatása](connecting.md) — automatikus felismerés, a grafikus felhasználói felület csatlakozási párbeszédablaka, a CLI/SDK megfelelői, valamint az, hogy a kamera csatlakozásakor hogyan kerül meghatározásra a gyári kalibrálás (kamerán belüli csomag vs. felhő).
* [Kamera beállítások](camera-settings.md) — a Kamerák fülön található összes vezérlőelem: a kameralista, a kameránkénti beállítások panelje és a tömbbeállítások panelje (beleértve az ROI-szerkesztőt és az igazítást).
* [Felvételi beállítások és módok](capture.md) — „Minden felvétel”, „Egyedi” / „Folyamatos” / „Intervallum” felvétel, exporttípusok és a felvételek mappa felépítése.
* [Többkamerás rendszerek](arrays.md) — a rendszercsatlakozási varázsló, a szinkronkábel-érzékelő, a sávszélesség, a PTP és az élő rendszer-nézet.
* [Mono kamerák és vegetációs indexek](mono-indices.md) — M3M kamerák, valamint az igazítás → rétegezés → indexelés munkafolyamat.

A teljes parancssor a [CLI-referenciában](../reference/cli-reference.md) és az [SDK-referenciában](../reference/sdk-reference.md) található.
