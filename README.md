---
metaLinks: {}
---

# Első lépések

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Az Chloros egy [MAPIR](https://www.mapir.camera) által készített szoftveralkalmazás képek és egyéb érzékelőadatok feldolgozására.

***{% hint style="success" %}**Újdonságok az Chloros 1.1.0 verzióban**: Natív Linux támogatás (amd64 és arm64), NVIDIA Jetson edge computing, Dynamic Compute Adaptation, 4-szálas feldolgozási folyamat, új CLI parancsok és opciók. A teljes változásnaplóért lásd a [Letöltés](download.md) oldalt.
{% endhint %}

Az Chloros 3 alkalmazási módban érhető el:

## Chloros: Asztali GUI alkalmazás

Önálló, külön ablak, minden funkcióval. _Csak Windows._

## [Chloros CLI: Parancssori felület](CLI.md)

Parancssori kötegelt feldolgozás. Tökéletes automatizáláshoz, szkripteléshez és headless működéshez. Elérhető **Windows, Linux amd64 és Linux arm64 (NVIDIA Jetson)** rendszereken. _A CLI eléréséhez Chloros+ licenc szükséges._

## [Chloros API: Python SDK](api-python-sdk.md)

Programozási Python felület automatizáláshoz és egyedi munkafolyamatokhoz. Tökéletes kutatási folyamatokhoz, meglévő Python alkalmazásokkal való integrációhoz és egyedi eszközök készítéséhez. **Minden platformon** elérhető az `pip install chloros-sdk`-en keresztül. _Az API eléréséhez Chloros+ licenc szükséges._***

## Támogatott platformok

| Platform | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Igen | Igen | Igen |
| **Linux amd64 (x86_64)** | Nem | Igen | Igen |
| **Linux arm64 (NVIDIA Jetson)** | Nem | Igen | Igen |

Az Linux telepítési utasításait lásd az [Linux és Edge Computing](linux/linux-overview.md) szakaszban.

***

## Chloros+

Bár az Chloros a legtöbb feladatra ingyenesen használható, előfordulhat, hogy többet szeretne. Ilyenkor az Chloros+ fizetős licenc előnyös lehet az Ön számára. Az Chloros+ licenccel új funkciókat nyithat meg, például:

* **Többszálas feldolgozás**: jelentősen felgyorsíthatja a képfeldolgozást nagyobb projektek esetén azáltal, hogy a képeket egyszerre dolgozza fel a folyamat során.
* **GPU (CUDA) gyorsítás**: használja ki a mai nagyobb GPU-memória opciókat a képfeldolgozási folyamat további felgyorsításához. A legjobb eredmények elérése érdekében 4 GB vagy annál nagyobb VRAM-ot javaslunk.
* **Chloros+**[**CLI**](CLI.md)**Hozzáférés**: futtassa az Chloros+ programot a parancssorból a saját szoftverébe történő automatizálás és integrálás érdekében.
* **Chloros+**[**API**](api-python-sdk.md)**Hozzáférés:** futtassa az Chloros+ parancsot az Python-ből a programozási vezérléshez, amely zökkenőmentes integrációt tesz lehetővé a kutatási folyamatokkal, az adatelemzési munkafolyamatokkal és az egyedi alkalmazásokkal.
* **Több eszköz használata**: minden Chloros+ licenc 2 vagy több eszköz regisztrációját teszi lehetővé. A regisztrált eszközök kezeléséhez használja MAPIR Cloud fiókját. További eszközök támogatását az Chloros+ licenc frissítésével biztosíthatja.
* **Fejlett textúraérzékeny debayer módszer:** kiváló minőségű, élekkel számoló debayer, amely egy AI/ML zajszűrő modellel kombinálva szinte teljesen eltávolítja a debayeringből származó zajt. 
* **Egyéni multispektrális indexképletek:** adjon meg egyéni multispektrális indexeket az Chloros raszterkalkulátorokban, mind a feldolgozáshoz, mind a képnézegető sandboxhoz.
* **Linux és Edge Computing:** futtassa az Chloros-et az Linux x86\_64 és ARM64 platformokon, beleértve az NVIDIA Jetsont is, terepi és edge feldolgozáshoz. Lásd [Linux áttekintés](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Árak és regisztráció</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
