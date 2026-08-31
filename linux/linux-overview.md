# Linux Áttekintés

Az Chloros 1.2.0 natív Linux támogatást nyújt az **CLI**és**Python SDK** — headless multispektrális képfeldolgozás, valamint élő LATTICE kamera és DAQ fényérzékelő vezérlés — az Linux munkaállomásokon, szervereken és NVIDIA Jetson edge eszközökön.

{% hint style="info" %}
**Nincs asztali grafikus felhasználói felület (GUI) az Linux-en.**Az Chloros asztali GUI kizárólag az Windows-en érhető el. Az Linux felhasználók az [CLI](../CLI.md) és a [Python SDK](../api-python-sdk.md) menüpontokon keresztül léphetnek kapcsolatba az Chloros-szel. Az `.deb` hozzáad egy**Chloros CLI** elemet az alkalmazás menüjéhez — egyszerűen csak megnyit egy terminálemulátort, amelyen az `chloros-cli` fut.
{% endhint %}

***

## Platformtámogatási táblázat

| Funkció | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Asztali grafikus felület** | Igen | N/A | Nem | Nem |
| **CLI** (`chloros-cli`) | Igen | Igen | Igen | Igen |
| **Python SDK** (`chloros-sdk`) | Igen | Igen | Igen | Igen |
| **Képfeldolgozási folyamat** | Igen | Igen | Igen | Igen |
| **LATTICE kamera vezérlés (élő)** | Igen (Kamerák fül) | Igen (`chloros-cli lattice`, SDK) | Igen | Igen |
| **DAQ fényérzékelők (élő)** | Igen (Fényérzékelők fül) | Igen (`chloros-cli daq pool-*`, SDK) | Igen | Igen |
| **PTP időszinkronizálás (a gazdagép a grandmaster)** | Igen | Igen (`chloros-cli time-sync`) | Igen | Igen |
| **GPU-gyorsítás (CUDA)** | Igen | Igen | Igen | Igen (JetPack 6) |
| **Textúraérzékeny debayer** | Igen (Chloros+) | Igen (Chloros+) | Igen (Chloros+) | Igen (Chloros+) |
| **Dinamikus számítási adaptáció** | Igen | Igen | Igen | Igen |
| **Háttérprogram rendszer szolgáltatásként** (`chloros-backend.service`) | Nem | Nem | Igen (bekapcsolható) | Igen (bekapcsolható) |
| **Helyben történő frissítő** (`chloros-cli update`) | Nem (futtassa a telepítőt) | Nem (futtassa a telepítőt) | Igen | Igen |***

## Támogatott architektúrák

| Architektúra | Leírás | Csomag |
| --- | --- | --- |
| **amd64 (x86_64)** | Szabványos asztali/szerver processzorok (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM processzorok — NVIDIA Jetson Orin család | `chloros_<version>_arm64_jp6.deb` (JetPack 6 build) |

## Támogatott Linux disztribúciók

* **Ubuntu 22.04 LTS vagy újabb** (amd64)
* **Debian 12 vagy újabb** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson Orin platformok)***

## Mit kapnak az Linux felhasználók

* **Chloros CLI** — a teljes parancssori felület a kötegelt feldolgozáshoz, automatizáláshoz és szkripteléshez
* **Chloros Python SDK** — programozási Python felület kutatási folyamatokhoz és egyedi eszközökhöz (telepíthető a PyPI-ről, valamint az `.deb` csomagban is megtalálható, a verzióhoz illeszkedő wheel formájában)
* **LATTICE kamera vezérlés** — LATTICE kamerák és szinkronizált többkamerás rendszerek felismerése, csatlakoztatása, konfigurálása és felvétele az `chloros-cli lattice` és az SDK segítségével; az `.deb` csomag tartalmazza az Arena SDK futásidejű környezetet, amelyre a kameráknak szükségük van
* **DAQ fényérzékelő-vezérlés** — csatlakoztassa a DAQ-U/M/E érzékelőket, továbbítson kalibrált spektrumokat, és rögzítsen `.daq` fájlokat az `chloros-cli daq pool-*` és az SDK segítségével
* **PTP időszinkronizálás** — az Chloros háttérprogram futtatja azt a PTP grandmastert, amelyhez a LATTICE kamerák és a DAQ-E érzékelők csatlakoznak; ellenőrizze az `chloros-cli time-sync` segítségével, és az `chloros-backend.service` systemd egységgel biztosítsa a headless üzemmódban való folyamatos futtatását (lásd [Linux telepítés](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Projekt-automatizálás** — a mentett projekteket futtassa headless módban az `chloros-cli project` és az SDK `open_project` segítségével
* **GPU-gyorsítás** — CUDA-gyorsított feldolgozás NVIDIA GPU-kon (asztali és Jetson)
* **Dinamikus számítási adaptáció** — automatikus hardverfelismerés és feldolgozási stratégia kiválasztás, az `CHLOROS_STRATEGY` felülírási lehetőséggel szakértői vészkijáratként
* **Minden feldolgozási funkció** — ugyanaz a feldolgozási folyamat, mint az Windows esetében: kalibrálás, vignettálás-korrekció, vegetációs indexek és minden exportformátum
* **Chloros+ funkciók** — többszálas (csővezetékes) feldolgozás, Texture Aware debayer és egyéni indexek, fizetős Chloros+ csomaggal

## Amit az Linux felhasználók nem kapnak meg

* **Asztali grafikus felület** — nincs grafikus felület; minden interakció az CLI vagy az Python SDK alkalmazáson keresztül történik
* **Képnézegető** — nincs interaktív képnézegető, rácsos nézet vagy térképjelölők
* **Vizuális projektmenedzsment** — a projektek létrehozása és irányítása az CLI parancsok és az SDK hívások segítségével történik (maga a hardver — kamerák, érzékelők, rögzítés — továbbra is teljes mértékben vezérelhető a terminálról)***

## Licenckövetelmény

Az CLI és SDK hozzáféréshez **fizetős Chloros+ szint — Copper vagy magasabb**szükséges (Copper, Bronze, Silver, Gold). Az ingyenes**Iron** csomag nem biztosít hozzáférést az CLI/SDK funkciókhoz. Ezt a korlátozást a háttérrendszer érvényesíti, nem csupán az CLI:

| Helyzet | A háttérrendszer válasza |
| --- | --- |
| Nincs bejelentkezve | `401` az `error_code: AUTH_REQUIRED`-szel |
| Bejelentkezve az ingyenes Iron szintre | `403` és `error_code: PLAN_UPGRADE_REQUIRED` |

Az `chloros-cli status` bármely szinten működik — ez az egyetlen útvonal, amely mentes a kapu alól —, így az elutasítás oka mindig látható.

***

## Az Linux használatának megkezdése

1. **Telepítse az Chloros-et** — az `.deb` telepítéséhez lásd az [Linux telepítését](linux-installation.md)
2. **Ellenőrzés** — az `chloros-cli --version` kinyomtatja az `Chloros CLI 1.2.0`-et; az `chloros-cli selftest` elvégzi a 7 lépéses diagnosztikát
3. **Telepítse az Python és az SDK szoftvereket** (opcionális) — `pip install chloros-sdk`
4. **Jelentkezzen be** — `chloros-cli login your@email.com 'your-password'` (gépenként egyszer, majd minden csomagfrissítés után újra)
5. **Feldolgozza az első adatkészletét** — `chloros-cli process ~/datasets/flight001`

NVIDIA Jetson esetén olvassa el a [NVIDIA Jetson útmutatót](nvidia-jetson-guide.md), amely tartalmazza a platformspecifikus beállításokat, a hőviselkedést és a helyszíni telepítést.

***

## Következő lépések

* [Linux Telepítés](linux-installation.md) — részletes telepítési útmutató, fájlhelyek és hibaelhárítás amd64 és arm64 rendszerekhez
* [NVIDIA Jetson útmutató](nvidia-jetson-guide.md) — Jetson-specifikus beállítások, memória- és hőviselkedés, helyszíni telepítés
* [CLI : Parancssor](../CLI.md) — az CLI útmutató
* [API : Python SDK](../api-python-sdk.md) — az SDK útmutató
* [CLI Referencia](../reference/cli-reference.md) és [SDK Referencia](../reference/sdk-reference.md) — a 1.2.0-s verzióra vonatkozó kimerítő parancs-/API-lista
* [Dinamikus számítási alkalmazkodás](../processing-architecture/dynamic-compute-adaptation.md) — hogyan alkalmazkodik az Chloros a hardveréhez

{% hint style="info" %}
**A kézikönyv programozási úton történő olvasása.** Minden oldal nyers Markdown formátumban is elérhető a saját URL és `.md` (például `https://mapir.gitbook.io/chloros/linux/linux-installation.md`) alatt is elérhető nyers Markdown formátumban, a teljes kézikönyv tartalomjegyzéke pedig a [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt) címen található.
{% endhint %}
