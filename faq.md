---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# GYIK

<details>

<summary>Feldolgozhatok-e a Chloros programmal olyan kamerákból származó képeket, amelyek nem a MAPIR márkájúak?</summary>

Nem, a Chloros kizárólag a MAPIR kamerák – a Survey3 és a LATTICE termékcsaládok – képeinek feldolgozását támogatja. További információkért kérjük, tekintse meg a [támogatott kameramodellek](supported-cameras.md) listáját. A MAPIR Cloud szolgáltatás keretében más kamerák képeinek feldolgozását is kínáljuk, a teljes listát [itt](https://mapir.gitbook.io/mapir-cloud/supported-cameras) találja.

</details>

<details>

<summary>Támogatja-e a Chloros a LATTICE kamerákat?</summary>

Igen. A Chloros 1.3.0 teljes körűen támogatja a LATTICE M3C és M3M kameramodulokat: **élő vezérlés**— felismerés, csatlakozás, előnézet és felvétel a GUI „Kamerák” fülén, az `chloros-cli lattice` oldalon vagy a Python SDK segítségével, beleértve a PTP időszinkronizálással ellátott, szinkronizált többkamerás rendszereket is — valamint a felvételek**teljes radiometrikus feldolgozása** (nyers → debayered → sugárzás → visszaverődés → index). Lásd a [Támogatott kamerák](supported-cameras.md) részt és a [LATTICE útmutatót](lattice/README.md).

</details>

<details>

<summary>Kalibrálhatom-e a képeimet reflektancia szempontjából kalibrációs célpont nélkül?</summary>

**Survey3

:** Nem. Ha a nem kalibrációs célpontról készült képek rögzítésekor nem készít képet a kalibrációs célpontról is, akkor nem fogja tudni a kép pixelértékeit egy ismert reflektancia-százalékhoz rendelni. Ha a MAPIR

fényérzékelő naplófájlját sem csatolja, akkor a környezeti fény spektruma nem kerül mérésre, és a reflexió eredményei nem lesznek pontosak.

**LATTICE:** Igen. A reflexió a panel helyett egy DAQ fényérzékelővel mért lefelé irányuló besugárzási intenzitáshoz viszonyítható (ρ = π·L/E). Ha egy minőségbiztosítási (QA) ellenőrzésen átment, a képkockán belüli célpont *jelen van*, az alapértelmezés szerint abszolút referenciává válik (`--reflectance-source auto`), kivéve, ha egy jelzővel halad át, amely esetben a célpont és a DAQ-láncok a mért pontszám alapján versenyeznek egymással. Egy kivétel: „Az F988 fényvisszaverődést egy a jelenetben lévő fényvisszaverő panel segítségével kalibrálják: a sáv a DAQ fényérzékelő kalibrált tartományán kívül esik, ezért az Chloros

a legutóbbi panelről készült felvételt alkalmazza, és azt a panel megfigyelései között megőrzi.” Lásd [Kalibrációs célpontok](calibration-targets.md).

</details>

<details>

<summary>Szükségem van DAQ fényérzékelőre?</summary>

A sugárzáshoz nem: a LATTICE sugárzási exportjai az egyes kamerák gyári radiometrikus kalibrációjából származnak, és sem DAQ-érzékelőre, sem célpontra nincs szükségük. A **reflektancia**esetében szükség van a környezeti fényre vonatkozó referenciára – akár egy DAQ fényérzékelő lefelé irányuló mérésére, akár egy képkockán belüli kalibrációs célpontra. A DAQ-érzékelő segítségével**anélkül állíthat elő kalibrált reflektanciát, hogy bármilyen panelt elhelyezne a jelenetben**. A rögzített `.daq` fájlok időbélyeg alapján automatikusan illeszkednek a felvételeihez. Lásd a [Kalibrációs célpontokat](calibration-targets.md) és a [CLI

-referenciát](reference/cli-reference.md).

</details>

<details>

<summary>Használhatom-e a Chloros-t egy AI-asszisztenssel (Claude, ChatGPT stb.)?</summary>

Igen — ez a kézikönyv és a CLI/SDK is erre lett kialakítva:

* A teljes kézikönyv tartalomjegyzéke az `https://mapir.gitbook.io/chloros/llms.txt` címen érhető el, hogy az AI-asszisztensek minden oldalt megtalálhassanak.
* Minden oldal nyers Markdown-kódja elérhető a kisbetűs oldal-URL-jén, amelyhez az `.md` kiegészítés tartozik (például `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* A [CLI-referencia](reference/cli-reference.md) és az [SDK-referencia](reference/sdk-reference.md) nagy nyelvi modellek (LLM) számára készült: pontos paraméterek, alapértelmezett értékek, kilépési szabályok és másolható-beilleszthető parancsok.

Az [AI-asszisztensek](ai-assistants.md) című részben olvashatja el, hogyan állíthatja be asszisztensét a Chloros használatára.

</details>

<details>

<summary>Hová kerülnek a feldolgozott kimeneti fájlok?</summary>

A termékek a projektmappa alatt kerülnek elmentésre, kameránként, majd fájlformátum szerint csoportosítva:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **kamera-mappa** — `LATT-<sensor>-<lens>-F<filter>` a LATTICE esetében, `<model>_<filter>` (pl. `Survey3N_RGN`) a Survey3 esetében
* **formátum-mappa** — `tiff16`, `tiff8`, `png8`, `jpg8` vagy `tiff32`
* **termékmappák** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (mindig az `tiff32` alatt), `<INDEX>_Index_Images/`
* **`clamp_masks/`** — kizárólag LATTICE multispektrális: a termékmappák mellett található pixelenkénti maszkok, lásd alább**Az exportált fájlok megtartják a forrásfájl nevét — a terméket a mappa azonosítja, nem a fájlnév kiterjesztése.**CLI használata esetén a projektmappa az alapértelmezett projektmappa alatt jön létre (`Documents/MAPIR/Chloros Projects`, hacsak nem módosította), kivéve, ha megadja az `-o` paramétert; a relatív `-o` a parancs futtatásának helyét jelentő mappához viszonyítva értendő. Ne feledje, hogy egy olyan `chloros-cli process` futtatás, amely termékeket kért, de egyet sem írt ki, az `Processing finished but wrote no image products.` üzenetet írja ki, és**nem nulla kóddal lép ki**, így a szkriptek felismerhetik. Lásd a [Kimeneti képformátumok](output-image-formats.md) és a [CLI-referencia](reference/cli-reference.md) című szakaszokat.

</details>

<details>

<summary>Mik azok a clamp_masks mappák és a *_clampmask.tif fájlok?</summary>

Újdonság a 1.3.0-s verzióban. A multispektrális LATTICE M3C kamera sugárzási és visszaverődési termékei 3×3 sávos szétválasztást hajtanak végre, és amennyiben egy sávban negatív értéket eredményez, az Chloros ezt az értéket nullára vágja. Az a termék, amelyben legalább egy pixel levágásra került, egy egycsatornás `uint8` maszkot kap, amely jelzi, mely pixelek és sávok kerültek levágásra (0. bit = a termék R csatornája, 1. bit = G, 2. bit = B). A maszkokat egy `clamp_masks/` mappában mentik **a** termékmappák mellett — például `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif` — és a termék `Chloros:UnmixClampMask` XMP-címkéje megnevezi a maszkját (vagy `none`-et jelöl).

Ezek nem a jelenet képei: a Chloros saját importálója kihagyja őket, és mivel soha nem helyezkednek el termékmappában, a termékmappát változatlan formában átadhatja a Pix4D-nek vagy a Metashape-nek. Lásd [Clamp maszkok](output-image-formats.md#clamp-masks).

</details>

<details>

<summary>Miért jelzi a Chloros, hogy „A projektmappa túl mély ehhez a számítógéphez”?</summary>

Ez csak a Windowsra vonatkozik. A Windows alapértelmezés szerint kikapcsolja a hosszú elérési utakat, így egyetlen fájl elérési útja sem lehet hosszabb 260 karakternél. A Chloros-termékek a projektmappa alatt körülbelül 130 karakterrel helyezkednek el, ezért a Chloros a feldolgozás előtt elutasítja a mélyebb projektmappákat – ahelyett, hogy futna, de semmit sem írna, ahogyan az a 1.3.0-ás verzió előtt előfordulhatott. Az üzenet megadja a leghosszabb, még elfogadható projektmappát.

A problémát a következő módokon orvosolhatja:

* Használjon rövidebb projektmappát (a Projektbeállítások menüben található **Munkakönyvtár** mezőt, vagy az CLI esetében az `-o` / `chloros-cli set-project-folder` parancsot).
* Kapcsolja be a Windows hosszú elérési útjait, és indítsa újra a Chloros-t. A Chloros telepítőjének **Windows hosszú elérési útjainak engedélyezése** opciója (alapértelmezés szerint be van jelölve) ezt végzi el; futtassa újra a telepítőt, ha kikapcsolta.

Lásd: [A feldolgozás elindítása](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).

</details>

<details>

<summary>Szerkeszthetem a képeimet a feldolgozás előtt a Chlorosban?</summary>

Nem. A Chloros feltételezi, hogy a bemeneti adatokat nem módosították. Ne változtassa meg a fájlneveket.

</details>

<details>

<summary>Beállíthatom-e a MAPIR Survey3 kameráimat automatikus expozícióra, és feldolgozhatom-e a képeket a Chloros-ban?</summary>

Nem. A Survey3-képadatkészleteknek rögzített/lezárt expozícióval kell rendelkezniük, tehát nem lehet automatikus zársebesség vagy automatikus ISO. Ugyanazon kameramodell összes képének azonos zársebességgel és ISO-értékkel (expozícióval) kell rendelkeznie.

A LATTICE kamerákra ez a korlátozás nem vonatkozik: a Chloros valós időben szabályozza az expozíciót (Smart AE), és minden felvétel rögzíti a ténylegesen használt expozíciót és erősítést, amit a radiometrikus feldolgozási folyamat figyelembe vesz.

</details>

<details>

<summary>Chloros feldolgozhatja vagy elemezheti az ortomozzaikus képeket?</summary>

Nem. Csak az egyes MAPIR-kamerákkal készült képeket támogatja, az ortomozzaikus térképhez hasonló, összerakott képeket nem.

</details>

<details>

<summary>Hogyan gyorsíthatom fel aChloros célpont-felismerési lépését?</summary>

A fájlkezelő táblázatában a jobb oldali oszlopban a célképek előzetes kijelölésével megadhatja a Chloros-nak, hogy csak azokban a képekben keressen kalibrációs célpontokat, ami jelentősen felgyorsítja a feldolgozást.

</details>

<details>

<summary>Ha feltöltöm a képeimet <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">a</a> <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">MAPIR </a>Cloudra, feldolgoznom kell őket a Chlorosban a feltöltés előtt?</summary>

Ha feltöltést tervez az online feldolgozó platformunkra [MAPIR

Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), ne szerkessze a képeket a feltöltés előtt. A Cloud elvégzi ugyanazokat a feldolgozási lépéseket, sőt még többet is.

</details>

<details>

<summary>A MAPIR valaha is támogatni fogja az X funkciót? Nagyon szeretném, ha a MAPIR kínálná az X-et.</summary>

Mindig szívesen fogadjuk a termékeinkkel kapcsolatos visszajelzéseket. Ha problémát tapasztal a termékeinkkel kapcsolatban, vagy javaslata van azok fejlesztésére, kérjük, [LÉPJEN KAPCSOLATBA VELÜNK](https://www.mapir.camera/community/contact), hogy megoszthassa velünk gondolatait. Kutatási és fejlesztési munkánk nagy részét az ügyfeleink legfontosabb igényeinek figyelembevétele irányítja.

</details>

<details>

<summary>Chloros elérhető-e Linux rendszeren?</summary>

Igen! A Chloros 1.3.0 az `.deb` csomagok révén támogatja a Linux amd64 (x86_64) és arm64 (NVIDIA Jetson JetPack 6) rendszereket. A parancssori felület (CLI) és a Python SDK teljes mértékben támogatott Linux rendszeren, beleértve a LATTICE kamerák és DAQ-érzékelők élő vezérlését is. Linux rendszerhez nincs grafikus felhasználói felület (GUI) — minden interakció a [parancssori felületen](CLI.md) vagy a [Python SDK-en](api-python-sdk.md) keresztül történik. A részleteket lásd a [Linux-áttekintés](linux/linux-overview.md) című dokumentumban.

</details>

<details>

<summary>Futtathatom-e aChloros-t az NVIDIA Jetsonon?</summary>

Igen! A Chloros támogatja az NVIDIA Jetson **Orin család** tagjait — az Orin Nano, az Orin NX és az AGX Orin modelleket —, amelyek JetPack 6 rendszert futtatnak. A régebbi modellek (Jetson Nano, TX2, Xavier NX) nem képesek futtatni a JetPack 6-ot, és a jelenlegi csomag nem támogatja őket. A Chloros automatikusan felismeri a Jetson-modelljét, és optimalizálja a feldolgozási stratégiát. A Jetson-csomag a JetPack 6 CUDA-alapú PyTorch-verzióját használja, így a feldolgozás a Jetson GPU-ján történik. A beállítási és telepítési utasításokat az [NVIDIA Jetson útmutató](linux/nvidia-jetson-guide.md) tartalmazza.

</details>

<details>

<summary>Chloros automatikusan optimalizálja a hardveremhez?</summary>

Igen! A Chloros tartalmazza a [dinamikus számítási adaptációt](processing-architecture/dynamic-compute-adaptation.md), amely automatikusan felismeri a CPU-t, a GPU-t, a RAM-ot és (Jetson esetén) a hőérzékelőket. Ezután kiválasztja az optimális feldolgozási stratégiát — az `GPU_PARALLEL`-től a nagy memóriájú GPU-k esetében, az `GPU_SINGLE`-en át a kisebb GPU-kig, egészen az `CPU_PARALLEL`-ig, amely a CUDA GPU nélküli rendszerekre vonatkozik, ahol a PyTorch nem használható. Nincs szükség kézi konfigurálásra.

</details>

<details>

<summary>Mi az a 4-szálas feldolgozási folyamat?</summary>
A

Chloros
4-szálas, csővezetékes architektúrát használ a Chloros
+ felhasználók számára: Az 1. szál (Detection) betölti a képeket és felismeri a kalibrációs célpontokat, a 2. szál (Calibration) kiszámítja a reflexiós kalibrációt, a 3. szál (Processing) elvégzi a GPU-val gyorsított debayeringet és az indexszámítást, míg a 4. szál (Export) írja a kimeneti fájlokat. A maximális átviteli sebesség elérése érdekében több kép is egyidejűleg feldolgozható különböző szálakon. A részleteket lásd a [Feldolgozási folyamat](processing-architecture/processing-pipeline.md) című részben.

</details>

<details>

<summary>Hogyan futtathatok diagnosztikát a Chloros-telepítésemen?</summary>

Az `selftest` parancs segítségével futtathat egy 7 lépésből álló gyorsellenőrzést: verzió, portok elérhetősége, háttérrendszer indítása, API-kapcsolat (`/api/test`), rendszerinformációk (`/api/system-info` — GPU/CUDA/PyTorch), zajszűrő modell jelenléte, valamint a CUDA és a zajszűrő készenléte:

```bash
chloros-cli selftest
```

Ez különösen hasznos Linux-rendszereken a GPU és a CUDA beállításainak ellenőrzéséhez. Jetson-on az `CUDA: False` általában azt jelenti, hogy hiányoznak a JetPack 6 CUDA- és cuDNN-könyvtárai: telepítse őket az `sudo apt install nvidia-jetpack` paranccsal.

</details>
