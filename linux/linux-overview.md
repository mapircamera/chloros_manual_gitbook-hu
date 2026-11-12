# Linux-áttekintés

A Chloros 1.3.0 natív Linux-támogatást biztosít a **CLI**és a**Python SDK** számára — fej nélküli multispektrális képfeldolgozás, valamint LATTICE kamerák és DAQ fényérzékelők élő vezérlése — Linux munkaállomásokon, szervereken és NVIDIA Jetson peremkészülékeken.

{% hint style="info" %}
**Linux rendszeren nincs asztali grafikus felület.**A Chloros Desktop GUI kizárólag Windows rendszeren érhető el. A Linux-felhasználók a [CLI](../CLI.md) és a [Python SDK](../api-python-sdk.md) segítségével léphetnek kapcsolatba a Chloros-szal. Az `.deb` valóban hozzáad egy**Chloros CLI** bejegyzést az alkalmazásmenüjéhez — ez egyszerűen megnyit egy terminálemulátort, amelyen az `chloros-cli` fut.
{% endhint %}

***

## Platformtámogatási táblázat

| Funkció | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Asztali grafikus felület** | Igen | N/A | Nem | Nem |
| **Parancssori felület (CLI)** (`chloros-cli`) | Igen | Igen | Igen | Igen |
| **Python SDK** (`chloros-sdk`) | Igen | Igen | Igen | Igen |
| **Képfeldolgozási folyamat** | Igen | Igen | Igen | Igen |
| **LATTICE kamera vezérlés (élő)** | Igen (Kamerák fül) | Igen (`chloros-cli lattice`, SDK) | Igen | Igen |
| **DAQ fényérzékelők (élő)** | Igen (Fényérzékelők fül) | Igen (`chloros-cli daq pool-*`, SDK) | Igen | Igen |
| **PTP időszinkronizálás (a gazdagép a grandmaster)** | Igen | Igen (`chloros-cli time-sync`) | Igen | Igen |
| **GPU-gyorsítás (CUDA)** | Igen | Igen | Igen | Igen (JetPack 6) |
| **Textúraérzékeny debayer** (CUDA GPU szükséges) | Igen (Chloros+) | Igen (Chloros+) | Igen (Chloros+) | Igen (Chloros+) |
| **Dinamikus számítási adaptáció** | Igen | Igen | Igen | Igen |
| **Háttérprogram rendszer szolgáltatásként** (`chloros-backend@<user>.service`) | Nem | Nem | Igen (beállítható) | Igen (beállítható) |
| **Helyben történő frissítő** (`chloros-cli update`) | Nem (futtassa a telepítőt) | Nem (futtassa a telepítőt) | Igen | Igen |

***

## Támogatott architektúrák

| Architektúra | Leírás | Csomag |
| --- | --- | --- |
| **amd64 (x86_64)** | Szabványos asztali/szerver processzorok (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | ARM processzorok — NVIDIA Jetson Orin család (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Támogatott Linux-disztribúciók

* **Ubuntu 22.04 LTS vagy újabb** (amd64)
* **Debian 12 vagy újabb** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson Orin platformok)

***

## Mit kapnak a Linux-felhasználók

* **Chloros CLI** — a teljes parancssori felület kötegelt feldolgozáshoz, automatizáláshoz és szkripteléshez
* **Chloros Python SDK** — programozási Python-felület kutatási folyamatokhoz és egyedi eszközökhöz, amelyet az `.deb` telepít a rendszerhez (`python3`) (a virtualenvs-hez illeszkedő verziójú wheel az `/usr/lib/chloros/sdk/`-ben található; a PyPI azoknak a gépeknek szól, amelyek nem rendelkeznek Chloros-csomaggal)
* **LATTICE kamera vezérlés** — LATTICE kamerák és szinkronizált többkamerás rendszerek felismerése, csatlakoztatása, konfigurálása és felvételek készítése az `chloros-cli lattice` és az SDK segítségével; az `.deb` tartalmazza az Arena SDK futtatókörnyezetet, amelyre a kameráknak szükségük van
* **DAQ fényérzékelő-vezérlés** — DAQ-U/M/E érzékelők csatlakoztatása, kalibrált spektrumok streamelése és `.daq` fájlok rögzítése az `chloros-cli daq pool-*` és az SDK segítségével
* **PTP időszinkronizálás** — az Chloros-háttérprogram futtatja a PTP grandmastert, amelyhez a LATTICE kamerák és a DAQ-E érzékelők csatlakoznak; ellenőrizze az `chloros-cli time-sync` segítségével, és tartsa futtatásban headless módban az `chloros-backend@<user>.service` systemd egységgel (lásd [Linux-telepítés](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Projekt automatizálás** — a mentett projekteket futtassa headless módban az `chloros-cli project` és az SDK `open_project` segítségével
* **GPU-gyorsítás** — CUDA-gyorsított feldolgozás NVIDIA GPU-kon (amd64-es asztali GPU-kon, valamint a Jetson GPU-ján a JetPack 6 segítségével)
* **Dinamikus számítási adaptáció** — automatikus hardverfelismerés és feldolgozási stratégia kiválasztás, az `CHLOROS_STRATEGY` felülírási lehetőséggel szakértői menekülési útvonalaként
* **Minden feldolgozási funkció** — ugyanaz a feldolgozási folyamat, mint a Windows-ban: kalibrálás, vignettakorrekció, vegetációs indexek és minden exportformátum
* **Chloros+ funkciók** — többszálas (pipeline-alapú) feldolgozás, Texture Aware debayer (CUDA GPU-n) és egyéni indexek, fizetős Chloros+ csomaggal

## Amit a Linux-felhasználók nem kapnak meg

* **Asztali grafikus felület** — nincs grafikus felület; minden interakció CLI-en vagy a Python SDK-en keresztül történik
* **Képnézegető** — nincs interaktív képnézegető, rácsnézet vagy térképjelölők
* **Vizuális projektkezelés** — a projektek létrehozása és irányítása CLI-parancsokkal és SDK-hívásokkal történik (maga a hardver — kamerák, érzékelők, rögzítés — továbbra is teljes mértékben vezérelhető a terminálról)

***

## Licenckövetelmény

A CLI-hez és az SDK-hez való hozzáféréshez **fizetős Chloros+ csomag — Copper vagy magasabb**(Copper, Bronze, Silver, Gold) szükséges. Az ingyenes**Iron** csomag nem biztosít CLI/SDK hozzáférést. Ezt a korlátozást a háttérrendszer érvényesíti, nem csupán a CLI:

| Helyzet | Háttérrendszer válasza |
| --- | --- |
| Nincs bejelentkezve | `401` és `error_code: AUTH_REQUIRED` |
| Bejelentkezve az ingyenes Iron szintre | `403` és `error_code: PLAN_UPGRADE_REQUIRED` |

Az `chloros-cli status` bármely szinten működik — ez az egyetlen útvonal, amely mentesül a korlátozás alól — így az elutasítás oka mindig látható.

***

## Első lépések Linuxon

1. **Telepítse a Chloros-t** — az `.deb` telepítéséhez lásd a [Linux-telepítés](linux-installation.md) című részt
2. **Ellenőrzés** — az `chloros-cli --version` kinyomtatja az `Chloros CLI 1.3.0` parancsot; az `chloros-cli selftest` futtatja a 7 lépéses diagnosztikát
3. **Ellenőrizze a Python SDK-t** — az `.deb` telepíti: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (virtuális környezetben: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Jelentkezzen be** — `chloros-cli login your@email.com 'your-password'` (gépenként egyszer; a csomagfrissítések megőrzik a bejelentkezési adatokat, a csomag eltávolítása törli azokat)
5. **Feldolgozza az első adatkészletét** — `chloros-cli process ~/datasets/flight001`

NVIDIA Jetson esetén olvassa el a [NVIDIA Jetson útmutatót](nvidia-jetson-guide.md), amely a platformspecifikus beállításokat, a hőviselkedést és a terepi telepítést ismerteti.

***

## Következő lépések

* [Linux-telepítés](linux-installation.md) — részletes telepítési útmutató, fájlhelyek és hibaelhárítás amd64 és arm64 architektúrákhoz
* [NVIDIA Jetson útmutató](nvidia-jetson-guide.md) — Jetson-specifikus beállítás, memória- és hőviselkedés, helyszíni telepítés
* [CLI: Parancssor](../CLI.md) — a CLI útmutató
* [API: Python SDK](../api-python-sdk.md) — az SDK útmutató
* [CLI-referencia](../reference/cli-reference.md) és [SDK-referencia](../reference/sdk-reference.md) — a 1.3.0-s verzió teljes parancs- és API-listája
* [Dinamikus számítási alkalmazkodás](../processing-architecture/dynamic-compute-adaptation.md) — hogyan alkalmazkodik aChlorosa a hardveréhez

{% hint style="info" %}
**A kézikönyv programozási úton történő olvasása.** Minden oldal nyers Markdown formátumban is elérhető a saját URL-jén, kiegészítve az `.md` kóddal (például `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), és a teljes kézikönyv tartalomjegyzéke a [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt) címen.
{% endhint %}
