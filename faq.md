---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# GYIK

<details>

<summary>Feldolgozhatok-e az MAPIR márkájú kameráktól származó képeket az Chloros segítségével?</summary>

Nem, az Chloros csak az MAPIR kamerák – az Survey3 és a LATTICE családok – képeinek feldolgozását támogatja. További információkért kérjük, tekintse meg a [támogatott kameramodellek](supported-cameras.md) listáját. Az MAPIR Cloud szolgáltatáson keresztül más kamerák feldolgozását is kínáljuk, a teljes listát [itt](https://mapir.gitbook.io/mapir-cloud/supported-cameras) találja.

</details>

<details>

<summary>Támogatja-e az Chloros a LATTICE kamerákat?</summary>

Igen. Az Chloros 1.2.0 teljes körűen támogatja a LATTICE M3C és M3M kameramodulokat: **élő vezérlés**— felismerés, csatlakozás, előnézet és felvétel a felhasználói felület „Kamerák” füléről, az `chloros-cli lattice`-en vagy az Python SDK-en keresztül, beleértve a PTP időszinkronizálással ellátott, szinkronizált többkamerás rendszereket is — valamint a rögzített képek**teljes radiometrikus feldolgozása** (nyers → debayered → sugárzás → visszaverődés → index). Lásd a [Támogatott kamerák](supported-cameras.md) listát és a [LATTICE útmutatót](lattice/README.md).

</details>

<details>

<summary>Kalibrálhatom-e a képeimet reflektancia szempontjából kalibrációs célpont nélkül?</summary>

**Survey3:** Nem. Ha a nem célpontról készült felvételek rögzítésekor nem készít felvételt a kalibrációs célpontról is, akkor nem tudja a kép pixelértékeit egy ismert reflektancia-százalékhoz rendelni. Ha az MAPIR fényérzékelő naplófájlját sem csatolja, akkor a környezeti fény spektruma nem kerül mérésre, és a reflexió eredményei nem lesznek pontosak.**LATTICE:** Igen. A fényvisszaverődést a panel helyett egy DAQ fényérzékelő által mért lefelé irányuló besugárzási intenzitáshoz is lehet viszonyítani (ρ = π·L/E). Ha a képkockában *van* egy minőségbiztosítási (QA) követelményeknek megfelelő célpont, az alapértelmezés szerint abszolút referencia lesz (`--reflectance-source auto`). Egy kivétel: „Az F988 fényvisszaverődést a jelenetben lévő fényvisszaverő panel segítségével kalibrálják: a sáv a DAQ fényérzékelő kalibrált tartományán kívül esik, ezért az Chloros a legutóbbi panelről készült felvételt alkalmazza, és azt a panel-megfigyelések között megőrzi.” Lásd [Kalibrációs célpontok](calibration-targets.md).

</details>

<details>

<summary>Szükségem van DAQ fényérzékelőre?</summary>

A sugárzáshoz nem: a LATTICE sugárzási exportadatok az egyes kamerák gyári radiometrikus kalibrációjából származnak, és sem DAQ-érzékelőre, sem célpontra nincs szükségük. A **reflektancia**esetében szükség van a környezeti fényre vonatkozó referenciára — ez lehet egy DAQ fényérzékelő lefelé irányuló mérése vagy egy a képkockában elhelyezett kalibrációs célpont. A DAQ-érzékelő segítségével**anélkül**állíthat elő kalibrált reflektanciát,**hogy bármilyen panelt elhelyezne a jelenetben**. A rögzített `.daq` fájlok időbélyeg alapján automatikusan hozzá vannak rendelve a felvételeihez. Lásd a [Kalibrációs célpontokat](calibration-targets.md) és az [CLI-referenciát](reference/cli-reference.md).

</details>

<details>

<summary>Használhatom az Chloros-et egy AI-asszisztenssel (Claude, ChatGPT stb.)?</summary>

Igen — ez a kézikönyv és az CLI/SDK kifejezetten erre a célra készült:

* A teljes kézikönyv-tartalomjegyzék az `https://mapir.gitbook.io/chloros/llms.txt` címen érhető el, hogy a mesterséges intelligencia-asszisztensek minden oldalt megtalálhassanak.
* Minden oldal nyers Markdown-kódja elérhető a kisbetűs oldal URL alatt, az `.md` kiegészítéssel (például `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* Az [CLI Referencia](reference/cli-reference.md) és [SDK hivatkozás](reference/sdk-reference.md) az LLM-ek számára készült: pontos jelzők, alapértelmezett értékek, kilépési szémantika és másolható-beilleszthető parancsok.

Az [AI-asszisztensek](ai-assistants.md) című cikkben olvashatod el, hogyan állíthatod be az asszisztensedet az Chloros-re.

</details>

<details>

<summary>Hová kerülnek a feldolgozott kimeneti fájlok?</summary>

A kimeneti fájlok a projektmappa alatt kerülnek elmentésre, kameránként, majd fájlformátum szerint csoportosítva:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **kamera-mappa** — `LATT-<sensor>-<lens>-F<filter>` a LATTICE-hez, `<model>_<filter>` (pl. `Survey3N_RGN`) az Survey3-hez
* **formátum-mappa** — `tiff16`, `tiff8`, `png8`, `jpg8` vagy `tiff32`
* **termékmappák** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (mindig az `tiff32` alatt), `<INDEX>_Index_Images/`**Az exportált fájlok megtartják a forrásfájl nevét — a mappanév azonosítja a terméket, nem pedig a fájlnév kiterjesztése.**Az CLI használatával a projektmappa a bemeneti mappa mellé jön létre, hacsak nem adja meg az `-o` parancsot. Ne feledje, hogy egy olyan `chloros-cli process` futtatás, amely termékeket kért, de egyet sem írt ki, az `Processing finished but wrote no image products.` kódot nyomtatja ki, és**nem nulla értékkel lép ki**, így a szkriptek felismerhetik. Lásd a [Kimeneti képformátumok](output-image-formats.md) és az [CLI Referencia](reference/cli-reference.md) című részeket.

</details>

<details>

<summary>Szerkeszthetem a képeimet az Chloros-ben történő feldolgozás előtt?</summary>

Nem. Az Chloros feltételezi, hogy a bemeneti adatokat nem módosították. Ne változtassa meg a fájlneveket.

</details>

<details>

<summary>Beállíthatom-e az MAPIR és az Survey3 kameráimat automatikus expozícióra, és feldolgozhatom-e a képeket az Chloros programban?</summary>

Nem. Az Survey3 képkészleteknél rögzített/lezárt expozícióra van szükség, tehát nem használható automatikus zársebesség vagy automatikus ISO. Ugyanazon kameramodell összes képének azonos zársebességgel és ISO-értékkel (expozícióval) kell rendelkeznie.

A LATTICE kamerákra ez a korlátozás nem vonatkozik: az Chloros valós időben szabályozza az expozíciót (Smart AE), és minden felvétel rögzíti a ténylegesen használt expozíciót és erősítést, amit a radiometrikus feldolgozási folyamat figyelembe vesz.

</details>

<details>

<summary>Az Chloros képes ortomozzaikus képek feldolgozására vagy elemzésére?</summary>

Nem. Csak az egyes MAPIR kamerák által készített képeket támogatja, az ortomozzaikus térképhez hasonló összeillesztett képeket nem.

</details>

<details>

<summary>Hogyan gyorsíthatom fel az Chloros célpont-felismerési lépését?</summary>

A fájlkezelő táblázatban a jobb oldali oszlopban a célképek előzetes kijelölése arra utasítja az Chloros-et, hogy csak azokban a képekben keresse a kalibrációs célpontokat, ami jelentősen felgyorsítja a feldolgozást.

</details>

<details>

<summary>Ha feltöltöm a képeimet <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">az</a> <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR </a>Cloudra, feldolgoznom kell őket az Chloros-ben a feltöltés előtt?</summary>

Ha feltöltést tervez az online feldolgozó platformunkra [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), ne szerkessze a képeket a feltöltés előtt. A Cloud elvégzi ugyanazokat a feldolgozási lépéseket, sőt még többet is.

</details>

<details>

<summary>Az MAPIR valaha is támogatni fogja az X funkciót? Nagyon szeretném, ha az MAPIR kínálná az X funkciót.</summary>

Mindig szívesen fogadjuk a termékeinkkel kapcsolatos visszajelzéseket. Ha problémát tapasztal a termékeinkkel kapcsolatban, vagy javaslata van azok fejlesztésére, kérjük, [LÉPJEN VELÜNK KAPCSOLATBA](https://www.mapir.camera/community/contact), hogy megoszthassa velünk gondolatait. Kutatási és fejlesztési munkánk nagy részét az ügyfeleink legfontosabb igényeinek figyelembevétele irányítja.

</details>

<details>

<summary>Az Chloros elérhető az Linux rendszerhez?</summary>

Igen! Az Chloros 1.2.0 támogatja az Linux amd64 (x86_64) és arm64 (NVIDIA Jetson JetPack 6) architektúrákat az `.deb` csomagok révén. Az CLI és az Python SDK verziók teljes mértékben támogatottak az Linux rendszeren, beleértve az élő LATTICE kamera- és DAQ-érzékelő-vezérlést is. Az Linux-hez nincs grafikus felhasználói felület — minden interakció a [CLI](CLI.md) vagy az [Python SDK](api-python-sdk.md) segítségével történik. A részleteket lásd az [Linux Áttekintés](linux/linux-overview.md) című dokumentumban.

</details>

<details>

<summary>Futtathatom az Chloros-et az NVIDIA Jetsonon?</summary>

Igen! Az Chloros támogatja az NVIDIA Jetson platformokat, beleértve a Jetson Nano, Orin Nano, Orin NX és AGX Orin modelleket, amelyek JetPack 6 rendszert futtatnak. Az Chloros automatikusan felismeri a Jetson-modelljét, és optimalizálja a feldolgozási stratégiát. A beállítási és telepítési utasításokat az [NVIDIA Jetson útmutató](linux/nvidia-jetson-guide.md) tartalmazza.

</details>

<details>

<summary>Az Chloros automatikusan optimalizálja a hardveremet?</summary>

Igen! Az Chloros tartalmazza a [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) funkciót, amely automatikusan felismeri a CPU-t, a GPU-t, a RAM-ot és (Jetson esetén) a hőérzékelőket. Ezt követően kiválasztja az optimális feldolgozási stratégiát – az `GPU_PARALLEL`-től a nagy memóriájú rendszereken, az `GPU_SINGLE`-en át a korlátozott erőforrású eszközökön, egészen az `CPU_PARALLEL`-ig az NVIDIA GPU nélküli rendszereken. Nincs szükség kézi konfigurálásra.

</details>

<details>

<summary>Mi az a 4-szálas feldolgozási folyamat?</summary>

Az Chloros 4-szálas csővezetékes architektúrát használ az Chloros+ felhasználók számára: Az 1. szál (Detection) betölti a képeket és felismeri a kalibrációs célpontokat, a 2. szál (Calibration) kiszámítja a reflexiós kalibrációt, a 3. szál (Processing) elvégzi a GPU-val gyorsított debayeringet és az indexszámításokat, míg a 4. szál (Export) írja a kimeneti fájlokat. A maximális átviteli sebesség elérése érdekében több kép is egyidejűleg feldolgozható különböző szálakon. A részleteket lásd a [Feldolgozási folyamat](processing-architecture/processing-pipeline.md) című részben.

</details>

<details>

<summary>Hogyan futtathatok diagnosztikát az Chloros telepítésemen?</summary>

Az `selftest` parancs segítségével futtathat egy 7 lépésből álló alapvető tesztet: verzió, portok elérhetősége, háttérprogram indítása, API kapcsolódás (`/api/test`), rendszerinformációk (`/api/system-info` — GPU/CUDA/PyTorch), zajszűrő modell jelenléte, valamint a CUDA és a zajszűrő készenléte:

```bash
chloros-cli selftest
```

Ez különösen hasznos az Linux/Jetson rendszereken a GPU és a CUDA beállításainak ellenőrzéséhez.

</details>
