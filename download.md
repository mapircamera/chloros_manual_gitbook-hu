---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Letöltés

Töltse le az Chloros legújabb verzióját, hogy belekezdjen a multispektrális képfeldolgozásba.

### Rendszerkövetelmények

#### Windows

| Követelmény          | Minimális                                              | Ajánlott                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operációs rendszer** | Windows 10 (64 bites)                                  | Windows 11 (64 bites)                                  |
| **Processzor**        | Intel Core i5 vagy azzal egyenértékű                          | Intel Core i7 vagy annál jobb                              |
| **Memória (RAM)**     | 8 GB                                                  | 16 GB vagy több                                         |
| **Grafikus kártya**    | DirectX 11-kompatibilis                                | NVIDIA GPU 4 GB vagy több VRAM-mal                            |
| **Tárhely**          | 6 GB szabad hely                                       | SSD 10 GB vagy több szabad hellyel                            |
| **Kijelző**          | 1920x1080                                            | 2560x1440 vagy nagyobb felbontás                                  |
| **Internet**         | Szükséges az \[opcionális] Chloros+ licenc aktiválásához | Szükséges az \[opcionális] Chloros+ licenc aktiválásához |

#### Linux amd64 (x86_64)

| Követelmény       | Minimális                    | Ajánlott               |
| ----------------- | -------------------------- | ------------------------- |
| **Disztribúció**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Processzor**     | x86\_64 (Intel/AMD)        | Intel Core i7 vagy jobb   |
| **Memória (RAM)**  | 8 GB                        | 16 GB vagy több              |
| **Grafikus kártya** | Nincs (CPU-feldolgozás)      | NVIDIA GPU 4 GB vagy több VRAM-mal |
| **Tárhely**       | 2 GB szabad hely             | SSD 10 GB vagy több szabad hellyel       |
| **Python**        | Python 3.7+ (az SDK-hez)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Követelmény      | Minimális                      | Ajánlott                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platform**     | NVIDIA Jetson JetPack 6-tal | Jetson Orin NX 16 GB vagy AGX Orin |
| **Memória (RAM)** | 8 GB (megosztott GPU/CPU)         | 16 GB+ megosztott                    |
| **Tárolókapacitás**      | 2 GB szabad hely               | NVMe SSD 10 GB+ szabad hellyel        |
| **Python**       | Python 3.7+ (az SDK esetében)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-gyorsítás**: Az NVIDIA GPU-val rendelkező Chloros+ felhasználók a CUDA-gyorsítást használhatják a feldolgozás jelentős gyorsításához. Ez mind az Windows (asztali GPU-k), mind az Linux (asztali GPU-k és NVIDIA Jetson) esetében működik. Az Chloros+ felhasználók emellett többszálas feldolgozást is igénybe vehetnek a maximális sebesség elérése érdekében.
{% endhint %}

***

## Chloros letöltése

### Legújabb stabil kiadás: 1.2.0-s

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->

verzió

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Az Chloros letöltése az Windows (.exe) számára</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Chloros letöltése az Linux amd64 (.deb) verzióhoz</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Chloros letöltése az Linux arm64 / Jetson rendszerhez (.deb)</a>

#### Windows telepítő (GUI + CLI + háttérprogram)

* **Fájltípus**: .exe (Windows telepítő)**Telepítési lépések:**

1. Töltse le a fenti .exe fájlt
2. Kattintson duplán a telepítőre a telepítés elindításához
3. Kövesse a telepítő varázsló utasításait
4. Válassza ki a telepítési könyvtárat (alapértelmezett: `C:\Program Files\MAPIR\Chloros\`)
5. Fejezze be a telepítést, majd indítsa el az Chloros vagy az Chloros CLI programot
6. Jelentkezzen be a [MAPIR Cloud Chloros+ fiókjával](https://cloud.mapir.camera/pricing) (vagy folytassa az ingyenes verzióval)

{% hint style="success" %}
A telepítő automatikusan hozzáadja az `chloros-cli` parancsot a rendszer PATH változójához a parancssori hozzáférés érdekében.
{% endhint %}

#### Linux amd64 (.deb csomag — CLI + háttérprogram)

* **Fájltípus**: .deb (Debian/Ubuntu csomag)
* **Architektúra**: x86_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (.deb csomag — CLI + háttérprogram)

* **Fájltípus**: .deb (JetPack 6)
* **Architektúra**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

A részletes telepítési utasításokat lásd az [Linux telepítés](linux/linux-installation.md) című részben, a Jetson-specifikus útmutatást pedig az [NVIDIA Jetson útmutató](linux/nvidia-jetson-guide.md) című részben.

#### Python SDK (minden platform)

Minden telepítő tartalmaz egy megfelelő `chloros_sdk` wheel fájlt, így az SDK verzió mindig illeszkedik a telepített GUI/CLI/backend verzióhoz. Az Windows rendszeren a telepítő automatikusan az Python rendszerbe telepíti; az Linux verzión az `.deb` a wheel fájlt az `/usr/lib/chloros/sdk/` helyre helyezi, és kinyomtatja a telepítési parancsot:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

A kizárólag pip-et használó gazdagépek esetében (ahol nincs telepítve az Chloros csomag), az SDK a PyPI-n is elérhető:

```bash
pip install chloros-sdk
```

Lásd [API : Python SDK](api-python-sdk.md) és a [SDK Referencia](reference/sdk-reference.md) a dokumentációhoz.

{% hint style="info" %}
**Linux felhasználók**: Az `.deb` csomag telepíti az CLI-et és a háttérprogramot. Az Linux-hez nincs grafikus felhasználói felület — minden interakció az CLI vagy az SDK segítségével történik.
{% endhint %}

***

## További források

### Python SDK

Fejlesztők és automatizált munkafolyamatok esetén telepítse az Chloros, Python és SDK fájlokat:

```bash
pip install chloros-sdk
```

**Dokumentáció**: [API: Python SDK](api-python-sdk.md)**Követelmények**: Az Chloros-et telepíteni kell (Windows telepítő vagy Linux `.deb` csomag), Chloros+ licencbejelentkezés szükséges***

## A csomag tartalma

### Windows telepítő

* ✅ **Chloros GUI** – Teljes funkcionalitású grafikus felület
* ✅ **Chloros CLI** - Parancssori felület (Chloros+ licenc szükséges)
* ✅ **Chloros háttérprogram** – Feldolgozó motor
* ✅ **Kameraprofilok** – Előre konfigurált MAPIR kamerasablonok

### Linux .deb csomag

* ✅ **Chloros CLI** – Parancssori felület (Chloros+ licenc szükséges)
* ✅ **Chloros háttérprogram** – Feldolgozó motor
* ✅ **Kameraprofilok** – Előre konfigurált MAPIR kamerasablonok
* ❌ Nincs grafikus felhasználói felület — az Linux kizárólag headless CLI/SDK

### Python SDK (pip, minden platform)

* ✅ **Chloros SDK** – Python API (Chloros+ licenc szükséges)***

## Frissítés Chloros+ verzióra

Fejlett funkciók elérése Chloros+ előfizetéssel:

* 🚀 **Többszálas feldolgozás** – Képek párhuzamos feldolgozása
* ⚡ **GPU (CUDA) gyorsítás** – Az NVIDIA GPU teljesítményének kihasználása
* 💻 **CLI hozzáférés** - Automatizálás parancssori eszközökkel
* 🐍 **Python SDK** - Programozott API-hozzáférés
* 📱 **Több eszköz** – Használható 2–10+ eszközön (a csomagtól függően)
* **🐻 Fejlett, textúrát figyelembe vevő debayer-módszer** – kiváló minőségű, élek figyelembevételével működő debayer, amely egy AI/ML zajszűrő modellel kombinálva szinte teljesen eltávolítja a debayeringből származó zajt.
* 🧮 **Egyéni képletek** – Egyéni multispektrális indexek létrehozása

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Az Chloros+ csomagok és árak megtekintése</a></p>***

## Telepítési segítség

### Hibaelhárítás

**A telepítés sikertelen, a következő hibaüzenet jelenik meg:**

* Győződjön meg arról, hogy rendelkezik rendszergazdai jogosultságokkal
* Ideiglenesen tiltsa le a víruskereső szoftvert
* Ellenőrizze, hogy a rendszer megfelel-e a minimális rendszerkövetelményeknek

**Az alkalmazás nem indul el (Windows):**

* Ellenőrizze, hogy az Windows 10/11 (64 bites) telepítve van-e
* Frissítse a grafikus illesztőprogramokat
* Ellenőrizze az Windows Eseménynaplóban a hiba részleteit
* Forduljon a támogatáshoz a hibajelentésekkel

**Az CLI nem indul el (Linux):**

* Ellenőrizze, hogy az `.deb` csomag megfelelően van-e telepítve: `dpkg -l | grep chloros`
* Ellenőrizze a jogosultságokat: `sudo chmod +x /usr/bin/chloros-cli`
* Futtassa a diagnosztikát: `chloros-cli selftest`
* Ellenőrizze, hogy nincsenek-e hiányzó könyvtárak: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Licenc-aktiválási problémák:**

* Győződjön meg arról, hogy aktív az internetkapcsolat
* Ellenőrizze a hitelesítő adatait a [https://cloud.mapir.camera](https://cloud.mapir.camera) oldalon
* Ellenőrizze, hogy a tűzfal nem blokkolja-e az Chloros alkalmazást
* Részletes utasításokat a [Chloros+ Bejelentkezés](chloros+-login.md) oldalon talál

### Támogatás igénybevétele

Segítségre van szüksége a telepítéssel vagy a beállítással kapcsolatban?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Weboldal**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentáció**: [Első lépések](./)
* ❓ **GYIK**: [Gyakran ismételt kérdések](faq.md)***

## Szoftverfrissítések

Az Chloros ellenőrzi a frissítéseket, értesít, ha új verzió érhető el, és linket biztosít erre a letöltési oldalra — a frissítéshez futtassa az új, aláírt telepítőt. A beállításai és projektjei a frissítés után is megmaradnak. Az Linux és a Jetson rendszereken az `chloros-cli update` ellenőrzi, hogy van-e újabb verzió, és felajánlja a megfelelő `.deb` letöltését és telepítését letöltését és telepítését (ez a parancs kizárólag az Linux verzióban érhető el).

***

## Változásnapló**

1.2.0-s verzió (legújabb)**— a teljes funkciólistát lásd az**Újdonságok az Chloros 1.2.0-ban** című részben a [Bevezetés](./) oldalon.

<details>

<summary>1.0.5-ös verzió</summary>

**Kiadás dátuma: 2026. február 10.**

**Új funkciók*** **Texture Aware debayer módszer \[Csak Chloros+] -** A Texture Aware egy kiváló minőségű, élekhez igazodó debayer módszert alkalmaz, amely egy AI/ML zajszűrő modellel kombinálva szinte teljesen eltávolítja a debayeringből származó zajt.
* **T4P kalibrációs célok támogatása*** **Gyorsabb Chloros+ GPU-feldolgozás, jobb memóriakezelés**

**Hibajavítások*** Teljesen új felhasználói felület (GUI), amely most már minden Windows számítógépen működik.

</details>

<details>

<summary>1.0.4-es verzió</summary>

**Kiadás dátuma: 2026. január 5.**

**Új funkciók*** **Kép/metaadatok váltás**: Hozzáadott váltógomb a Fájlkezelőben, amellyel a kiválasztott kép metaadatai táblázatban jeleníthetők meg a képrajz helyett
* **Képrajz nagyítási csúszka**: Új felhasználói felületi csúszka a miniatűrök méretének beállításához (a CTRL + egérgörgő kombinációt is támogatja)
* **Képrács exportgombok**: A felső sorban található gombok a miniatűrök JPG-formátumról feldolgozott exportformátumokra (Célok, Reflektancia, Index, LUT) való átváltásához
* **Térkép fül**: Új interaktív 2D-térkép, amelyen a képek GPS-helyjelölői láthatók
  * Támogatja a Google Maps és az ESRI térképcsempéket (a nagyítási szint elérhetősége alapján automatikusan kiválasztja a legjobb csempeszolgáltatást)
  * Egérrel a térképjelölők fölé viszve megjelenik a miniatűr előnézete

**Hibajavítások*** Javított támogatás az Chloros telepítéséhez nem angol nyelvű számítógépeken

</details>

<details>

<summary>1.0.3-as verzió</summary>

**Kiadás dátuma: 2025. december 20.**

**Új funkciók*** Első kiadás

**Továbbfejlesztések*** Első kiadás

**Hibajavítások*** Első kiadás

**Ismert problémák*** Első kiadás

</details>***

## Licencszerződés**Saját fejlesztésű szoftver** – Szerzői jog (c) 2026 MAPIR Inc.

A jogosulatlan használat, terjesztés vagy módosítás tilos.

**Ingyenes verzió**: Személyes és kereskedelmi célokra is használható, de a funkciók korlátozottak**Chloros+**: Előfizetéses licenc a fejlett funkciókhoz és kereskedelmi célú telepítésekhez
