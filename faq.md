---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# GYIK

<details>

<summary>Feldolgozhatok-e az MAPIR márkától eltérő kamerák képeit az Chloros szoftverrel?</summary>

Nem, az Chloros csak az MAPIR kamerák képeinek feldolgozását támogatja. További információkért kérjük, tekintse meg a [támogatott kameramodellek](supported-cameras.md) listáját. Az MAPIR Cloud szolgáltatáson keresztül más kamerák képeinek feldolgozását is kínáljuk, a teljes listát [itt](https://mapir.gitbook.io/mapir-cloud/supported-cameras) találja.

</details>

<details>

<summary>Kalibrálhatom a képeimet a fényvisszaverődés tekintetében kalibrációs célpont nélkül?</summary>

Nem. Ha a nem célképek rögzítésekor nem készít képet a kalibrációs célpontról, akkor nem tudja a kép pixelértékeit egy ismert reflektancia százalékhoz rendelni. Ha emellett nem csatolja az MAPIR fényérzékelő naplóját sem, akkor a környezeti fény spektruma nem kerül mérésre, és a reflektancia eredmények nem lesznek pontosak.

</details>

<details>

<summary>Szerkeszthetem a képeimet az Chloros-ben történő feldolgozás előtt?</summary>

Nem. Az Chloros feltételezi, hogy a bemeneti adatokat nem módosították. Ne változtassa meg a fájlneveket.

</details>

<details>

<summary>Beállíthatom az MAPIR és Survey3 kameráimat automatikus expozícióra, és feldolgozhatom a képeket az Chloros programban?</summary>

Nem. Az Survey3 képadat-készleteknek fix/zárt expozícióval kell rendelkezniük, tehát nem lehet automatikus zársebesség vagy automatikus ISO. Ugyanazon kameramodell összes képének azonos zársebességgel és ISO-val (expozícióval) kell rendelkeznie.

</details>

<details>

<summary>Az Chloros képes ortomosaikus képeket feldolgozni vagy elemezni?</summary>

Nem. Csak az egyes MAPIR kameraképek támogatottak, az ortomozikus térképhez hasonló összerakott képek nem.

</details>

<details>

<summary>Hogyan gyorsíthatom fel az Chloros célfelismerési lépését?</summary>

A fájlkezelő táblázatban a jobb oldali oszlopban a célképek előzetes kijelölésével az Chloros-nek megadhatja, hogy csak azokban a képekben keressen kalibrációs célpontokat, ami jelentősen felgyorsítja a feldolgozást.

</details>

<details>

<summary>Ha feltöltöm a képeimet <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">az</a> <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR Cloud</a>ra, akkor a feltöltés előtt feldolgoznom kell őket az Chloros-ben?</summary>

Ha feltöltést tervez az online feldolgozó platformunkra [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), ne szerkessze a képeket a feltöltés előtt. A Cloud elvégzi ugyanazokat a feldolgozási lépéseket, és még többet is.

</details>

<details>

<summary>Az MAPIR valaha is támogatni fogja az X funkciót? Nagyon szeretném, ha az MAPIR kínálná az X funkciót.</summary>

Mindig szívesen fogadjuk termékeinkkel kapcsolatos visszajelzéseket. Ha problémát tapasztal termékeinkkel kapcsolatban, vagy javaslata van azok fejlesztésére, kérjük, [LÉPJEN VELÜNK KAPCSOLATBA](https://www.mapir.camera/community/contact), hogy megoszthassa velünk gondolatait. Kutatási és fejlesztési munkánk nagy részét az ügyfeleink legfontosabb igényeinek figyelembevétele irányítja.

</details>

<details>

<summary>Az Chloros elérhető az Linux-hez?</summary>

Igen! Az Chloros 1.1.0 támogatja az Linux amd64 (x86_64) és arm64 (NVIDIA Jetson JetPack 6) architektúrákat az `.deb` csomagok révén. Az CLI és az Python SDK teljes mértékben támogatott az Linux-en. Az Linux-hez nincs grafikus felhasználói felület — minden interakció az [CLI](CLI.md) vagy az [Python SDK](api-python-sdk.md) segítségével történik. A részleteket lásd az [Linux Áttekintés](linux/linux-overview.md) című részben.

</details>

<details>

<summary>Futtathatom az Chloros-et az NVIDIA Jetsonon?</summary>

Igen! Az Chloros 1.1.0 támogatja az NVIDIA Jetson platformokat, beleértve a Jetson Nano, Orin Nano, Orin NX és AGX Orin modelleket, amelyek JetPack 6 rendszert futtatnak. Az Chloros automatikusan felismeri a Jetson modellt, és optimalizálja a feldolgozási stratégiát. A beállítási és telepítési utasításokat az [NVIDIA Jetson útmutatóban](linux/nvidia-jetson-guide.md) találja.

</details>

<details>

<summary>Az Chloros automatikusan optimalizálja a hardveremet?</summary>

Igen! Az Chloros 1.1.0 tartalmazza a [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) funkciót, amely automatikusan felismeri a CPU-t, a GPU-t, a RAM-ot és (Jetson esetén) a hőérzékelőket. Ezután kiválasztja az optimális feldolgozási stratégiát – az `GPU_PARALLEL`-től a nagy memóriájú rendszereken, az `GPU_SINGLE`-en a korlátozott eszközökön, egészen az `CPU_PARALLEL`-ig az NVIDIA GPU nélküli rendszereken. Nincs szükség kézi konfigurálásra.

</details>

<details>

<summary>Mi az a 4-szálas feldolgozási folyamat?</summary>

Az Chloros 1.1.0 egy 4-szálas csővezetékes architektúrát használ az Chloros+ felhasználók számára: Az 1. szál (Detection) betölti a képeket és felismeri a kalibrációs célpontokat, a 2. szál (Calibration) kiszámítja a reflexiós kalibrációt, a 3. szál (Processing) elvégzi a GPU-val gyorsított debayeringet és az indexszámítást, a 4. szál (Export) pedig írja a kimeneti fájlokat. A maximális átviteli sebesség elérése érdekében több kép is lehet egyszerre különböző szálakban. A részleteket lásd a [Feldolgozási folyamat](processing-architecture/processing-pipeline.md) alatt.

</details>

<details>

<summary>Hogyan futtathatok diagnosztikát az Chloros telepítésemen?</summary>

Az `selftest` parancs segítségével 7 rendszerdiagnosztikát futtathat, beleértve a verzióellenőrzést, a portok elérhetőségét, a háttérprogram indítását, az API kapcsolódást, a rendszerinformációkat, a zajszűrő modelleket és a CUDA elérhetőségét:

```bash
chloros-cli selftest
```

Ez különösen hasznos az Linux/Jetson rendszereken a GPU és a CUDA beállítások ellenőrzéséhez.

</details>
