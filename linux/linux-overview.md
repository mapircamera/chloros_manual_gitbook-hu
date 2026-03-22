# Linux Áttekintés

Az Chloros 1.1.0 verzió natív Linux támogatást biztosít az **CLI**és**Python SDK** számára, lehetővé téve a headless multispektrális képfeldolgozást Linux munkaállomásokon, szervereken és NVIDIA Jetson edge eszközökön.

{% hint style="info" %}
**Nincs GUI az Linux-en.** Az Chloros asztali GUI csak az Windows-en érhető el. Az Linux felhasználók az [CLI](../CLI.md) és az [Python SDK](../api-python-sdk.md)
{% endhint %}

***

## Platformtámogatási táblázat

| Funkció | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **Asztali GUI** | Igen | N/A | Nem | Nem |
| **CLI** | Igen | Igen | Igen | Igen |
| **Python SDK** | Igen | Igen | Igen | Igen |
| **GPU-gyorsítás (CUDA)** | Igen | Igen | Igen | Igen (JetPack 6) |
| **Textúraérzékeny debayer** | Igen (Chloros+) | Igen (Chloros+) | Igen (Chloros+) | Igen (Chloros+) |
| **Dinamikus számítási adaptáció** | Igen | Igen | Igen | Igen |***

## Támogatott architektúrák

| Architektúra | Leírás | Telepítési módszer |
| --- | --- | --- |
| **amd64 (x86_64)** | Szabványos asztali/szerver processzorok (Intel, AMD) | `.deb` csomag |
| **arm64 (aarch64)** | ARM-alapú processzorok, elsősorban NVIDIA Jetson | `.deb` csomag (JetPack 6) |

## Támogatott Linux disztribúciók

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — Jetson platformok)***

## Mit kapnak az Linux felhasználók

* **Chloros CLI** — Teljes parancssori felület kötegelt feldolgozáshoz, automatizáláshoz és szkripteléshez
* **Chloros Python SDK** — Programozási Python felület (`pip install chloros-sdk`) kutatási folyamatokba és egyedi eszközökbe történő integráláshoz
* **GPU-gyorsítás** — CUDA-gyorsított feldolgozás NVIDIA GPU-kon (asztali és Jetson)
* **Dinamikus számítási adaptáció** — Automatikus hardverfelismerés és feldolgozási stratégia optimalizálás
* **Minden feldolgozási funkció** — Ugyanaz a multispektrális feldolgozási folyamat, mint az Windows esetében (kalibrálás, vignettakorrekció, vegetációs indexek, minden exportformátum)
* **Chloros+ funkciók** — Többszálas feldolgozás, textúraérzékeny debayer, egyedi indexek (Chloros+ licenccel)

## Amit az Linux felhasználók nem kapnak meg

* **Asztali grafikus felület** — Nincs grafikus felület; minden interakció az CLI vagy az Python SDK segítségével történik
* **Képnézegető** — Nincs interaktív képnézegető, rácsnézet vagy térképjelölők
* **Vizuális projektmenedzsment** — A projekteket az CLI parancsok és az SDK hívások segítségével lehet kezelni***

## Az Linux használatának megkezdése

1. **Telepítse az Chloros-et** — Az `.deb` csomag telepítéséhez lásd az [Linux telepítés](linux-installation.md) című részt
2. **Telepítse az Python SDK** (opcionális) — `pip install chloros-sdk`
3. **Aktiválja a licencét** — `chloros-cli login your@email.com 'password'`
4. **Feldolgozza az első adatkészletét** — `chloros-cli process ~/datasets/flight001`

NVIDIA Jetson felhasználók számára a platformspecifikus beállítások és optimalizálásról a külön [NVIDIA Jetson útmutató](nvidia-jetson-guide.md) nyújt tájékoztatást.

***

## Következő lépések

* [Linux Telepítés](linux-installation.md) — Részletes telepítési útmutató amd64 és arm64 rendszerekhez
* [NVIDIA Jetson útmutató](nvidia-jetson-guide.md) — Jetson-specifikus beállítás, hőkezelés és helyszíni telepítés
* [CLI : Parancssor](../CLI.md) — Teljes CLI referencia
* [API : Python SDK](../api-python-sdk.md) — Teljes SDK referencia
* [Dinamikus számítási adaptáció](../processing-architecture/dynamic-compute-adaptation.md) — Hogyan alkalmazkodik az Chloros a hardveréhez
