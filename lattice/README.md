# LATTICE kamerák

A LATTICE az MAPIR moduláris multispektrális kamerarendszere, amelyet mezőgazdasági és tudományos képalkotásra fejlesztettek ki. Minden LATTICE kamera a Sony IMX265 globális zárú érzékelőn (**3,1 MP, 3,45 µm-es képpontok**) alapul, és**GigE Vision** eszközként csatlakozik Etherneten keresztül.

Az Chloros 1.2.0 három felületről vezérli a LATTICE kamerákat élőben – felismerés, élő előnézet, felvétel és szinkronizált többkamerás rendszerek –:

| Felület    | Hol                                                          | Platformok                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| Grafikus felhasználói felület        | **Kamerák** fül az Chloros oldalsávban                         | Windows 10/11 x64                                        |
| CLI        | `chloros-cli lattice` parancscsalád                           | Windows 10/11 x64, Linux x86_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Hardvert keres?**A kameramodulok, objektívek, szűrők és sávok, keretek és rögzítések, kábelek, PoE és trigger-vezetékek leírását a [**LATTICE felhasználói kézikönyv**](https://mapir.gitbook.io/lattice-camera) tartalmazza. Ez a fejezet az Chloros segítségével történő kameravezetést tárgyalja.

A LATTICE-felvételek szabványos `.tif`/`.tiff` fájlok, és az Chloros mindig a nyers felvételből indul ki a feldolgozásuk során. A teljes parancsokhoz és a felülethez lásd az [CLI referencia](../reference/cli-reference.md) és az [SDK referencia](../reference/sdk-reference.md) című dokumentumokban találhatók a teljes parancsok és az API felület.

## Két érzékelő-konfiguráció

| Konfiguráció | Érzékelő       | Szűrő                                | Mit ad vissza egy kamera                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Bayer-színes | háromsávos sávszűrő                |**Három kalibrált sáv egyetlen expozícióból**                 |
| **M3M**| Monokróm   | egyetlen keskenysávú interferencia-szűrő |**Egy kalibrált sáv**; indexekhez több M3M kamerát kombináljon |

Mivel az M3M kamera egyetlen szűrő mögött monokróm, minden sáv saját expozíciót kap. Az M3C kamera egyetlen érzékelő-expozícióval lefedi mindhárom sávját.

## Modellkódok és névadás

Minden kamera a GenICam-protokollban az `DeviceUserID` formátumú modellkódként tárolja azonosítóját:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros az `LATT-` előtaggal jeleníti meg (például `LATT-M3M-L87-F450`). Ugyanez az `LATT-…` karakterlánc kerül beírásra minden export EXIF `Model` címkéjébe, és a feldolgozott projektekben a kamera kimeneti mappájának nevéként szolgál.

| Komponens | Értékek                                                   | Jelentés                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Érzékelő    | `M3C` / `M3M`                                            | Bayer színes / monokróm                                                                          |
| Objektív      | `L41` / `L87`                                            | A szám a **fokban kifejezett vízszintes látómező**: L41 = keskeny (41°), L87 = széles (87°)    |
| Szűrő    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) vagy `F<nm>` (M3M) | Lásd [Szűrők és spektrális sávok](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

A modellkód határozza meg az összes további lépést: az Chloros az `DeviceUserID` + `DeviceSerialNumber` értékekből határozza meg az érzékelő profilját, a sávok elrendezését és a gyári kalibrálást. Kameránként nincs semmi beállítani való – lásd [Kamerák csatlakoztatása](connecting.md).

## Szűrők és sávok

A sávközpontok, az FWHM sávhatárok és a teljes, 23 SKU-ból álló M3M-katalógus a termék specifikációi, ezért a hardver kézikönyvben találhatók: [**Szűrők és spektrális sávok**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

Ami a szoftver oldalán fontos: a modellszövegben szereplő szűrőkód határozza meg, hogy az Chloros mely termékeket képes létrehozni. Az RGB szűrővel rendelkező kamerák (`FRGB`) kizárólag debayered és előnézeti termékeket bocsátanak ki — a sávonkénti sugárzás és visszaverődés szélessávú érzékelő esetében nem releváns, ezért az Chloros kihagyja őket, és erről tájékoztatást is ad. Minden egyéb szűrő a teljes sugárzás → visszaverődés → index láncot adja ki.

## A radiometrikus kalibrálás áttekintése

Minden LATTICE kamerát a gyárban egyedileg kalibrálnak egy NIST-nyomon követhető lánc alapján, és kameránkénti tanúsítvánnyal szállítják. Hogy ez mit foglal magában, hogyan mérik, és milyen pontosságot lehet megadni, azt a hardver kézikönyv tartalmazza: [**Gyári radiometrikus kalibrálás**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

A szoftver tekintetében az a fontos, hogy az Chloros a kamera csatlakozásakor meghatározza a megfelelő kalibrálást, és az alkalmazott együtthatókat minden exportáláskor rögzíti – lásd [Kamerák csatlakoztatása](connecting.md).

## Ebben a fejezetben

* [Kamerák csatlakoztatása](connecting.md) — automatikus felismerés, a grafikus felhasználói felület csatlakozási párbeszédablaka, az CLI/SDK megfelelői, valamint a gyári kalibrálás meghatározásának módja (kamerán tárolt csomag vs. felhő) a kamera csatlakozásakor.

A LATTICE-hez kapcsolódó további témák — kamerabeállítások és élő vezérlés, rögzítési módok, többkamerás rendszerek, valamint mono (M3M) feldolgozás és indexek — a kézikönyv saját fejezeteiben kerülnek tárgyalásra, a teljes parancsfelület pedig az [CLI Referencia](../reference/cli-reference.md) és a [SDK Referencia](../reference/sdk-reference.md) című dokumentumokban található.
