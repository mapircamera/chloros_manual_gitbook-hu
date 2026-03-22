---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Letöltés

Töltse le az Chloros legújabb verzióját, hogy elindulhasson a multispektrális képfeldolgozás terén.

### Rendszerkövetelmények

#### Windows

| Követelmény          | Minimális                                              | Ajánlott                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operációs rendszer** | Windows 10 (64 bites)                                  | Windows 11 (64 bites)                                  |
| **Processzor**        | Intel Core i5 vagy azzal egyenértékű                          | Intel Core i7 vagy annál jobb                              |
| **Memória (RAM)**     | 8 GB                                                  | 16 GB vagy több                                         |
| **Grafikus kártya**    | DirectX 11 kompatibilis                                | NVIDIA GPU 4 GB+ VRAM-mal                            |
| **Tárhely**          | 6 GB szabad hely                                       | SSD 10 GB+ szabad hellyel                            |
| **Kijelző**          | 1920x1080                                            | 2560x1440 vagy nagyobb                                  |
| **Internet**         | Szükséges az \[opcionális] Chloros+ licenc aktiválásához | Szükséges az \[opcionális] Chloros+ licenc aktiválásához |

#### Linux amd64 (x86\_64)

| Követelmény       | Minimális                    | Ajánlott               |
| ----------------- | -------------------------- | ------------------------- |
| **Disztribúció**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Processzor**     | x86\_64 (Intel/AMD)        | Intel Core i7 vagy jobb   |
| **Memória (RAM)**  | 8 GB                        | 16 GB vagy több              |
| **Grafikus kártya** | Nincs (CPU-feldolgozás)      | NVIDIA GPU 4 GB+ VRAM-mal |
| **Tárhely**       | 2 GB szabad hely             | SSD 10 GB+ szabad helygel       |
| **Python**        | Python 3.7+ (az SDK-hez)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Követelmény      | Minimális                      | Ajánlott                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platform**     | NVIDIA Jetson JetPack 6-tal | Jetson Orin NX 16 GB vagy AGX Orin |
| **Memória (RAM)** | 8 GB (megosztott GPU/CPU)         | 16 GB+ megosztott                    |
| **Tároló**      | 2 GB szabad hely               | NVMe SSD 10 GB+ szabad hellyel        |
| **Python**       | Python 3.7+ (az SDK-hez)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-gyorsítás**: Az NVIDIA GPU-val rendelkező Chloros+ felhasználók a CUDA-gyorsítást használhatják a feldolgozás jelentős gyorsításához. Ez mind az Windows (asztali GPU-k), mind az Linux (asztali GPU-k és NVIDIA Jetson) esetében működik. Az Chloros+ felhasználók emellett többszálas feldolgozást is kapnak a maximális sebesség érdekében.
{% endhint %}

***

## Chloros letöltése

### Legújabb stabil kiadás (2026. március 23.): 1.1.0 verzió

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Chloros letöltése Windows (.exe) számára</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Chloros letöltése Linux amd64 (.deb) számára</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Chloros letöltése Linux arm64 / Jetson (.deb)</a>

#### Windows telepítő (GUI + CLI + háttér)

* **Fájltípus**: .exe (Windows telepítő)**Telepítési lépések:**

1. Töltse le a fenti .exe fájlt
2. Kattintson duplán a telepítőre a telepítés megkezdéséhez
3. Kövesse a telepítő varázsló utasításait
4. Válassza ki a telepítési könyvtárat (alapértelmezett: `C:\Program Files\[USER]\Chloros\`)
5. Fejezze be a telepítést, és indítsa el az Chloros vagy az Chloros CLI programot
6. Jelentkezzen be [MAPIR Cloud Chloros+ fiókjával](https://cloud.mapir.camera/pricing) (vagy folytassa az ingyenes verzióval)

{% hint style="success" %}
A telepítő automatikusan hozzáadja az `chloros-cli` fájlt a rendszer PATH-jéhez a parancssori hozzáférés érdekében.
{% endhint %}

#### Linux amd64 (.deb csomag — CLI + Backend)

* **Fájltípus**: .deb (Debian/Ubuntu csomag)
* **Architektúra**: x86\_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (.deb csomag — CLI + Backend)

* **Fájltípus**: .deb (JetPack 6)
* **Architektúra**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

A részletes telepítési utasításokat lásd az [Linux telepítés](linux/linux-installation.md) című részben, a Jetson-specifikus útmutatást pedig az [NVIDIA Jetson útmutató](linux/nvidia-jetson-guide.md) című részben.

#### Python SDK (Minden platform)

```bash
pip install chloros-sdk
```

A dokumentációt lásd az [API : Python SDK](api-python-sdk.md) oldalon.

{% hint style="info" %}
**Linux felhasználók**: Az `.deb` csomag telepíti az CLI-et és a háttérprogramot. Az Python SDK külön telepíthető a pip segítségével. Az Linux-hez nincs grafikus felhasználói felület — minden interakció az CLI vagy az SDK segítségével történik.
{% endhint %}

***

## További források

### Python SDK

Fejlesztők és automatizált munkafolyamatok esetén telepítse az Chloros Python SDK-et:

```bash
pip install chloros-sdk
```

**Dokumentáció**: [API: Python SDK](api-python-sdk.md)**Követelmények**: Az Chloros telepítése szükséges (Windows telepítő vagy Linux `.deb` csomag), Chloros+ licenc bejelentkezés szükséges***

## A csomag tartalma

### Windows telepítő

* ✅ **Chloros GUI** - Teljes funkcionalitású grafikus felület
* ✅ **Chloros CLI** - Parancssori felület (Chloros+ licenc szükséges)
* ✅ **Chloros háttérprogram** - Feldolgozó motor
* ✅ **Kamera profilok** - Előre konfigurált MAPIR kamera sablonok

### Linux .deb csomag

* ✅ **Chloros CLI** - Parancssori felület (Chloros+ licenc szükséges)
* ✅ **Chloros Backend** - Feldolgozó motor
* ✅ **Kamera profilok** - Előre konfigurált MAPIR kamera sablonok
* ❌ Nincs grafikus felhasználói felület — az Linux kizárólag headless CLI/SDK

### Python SDK (pip, minden platform)

* ✅ **Chloros SDK** - Python API (Chloros+ licenc szükséges)***

## Frissítés Chloros+ verzióra

Fejlett funkciók elérése Chloros+ előfizetéssel:

* 🚀 **Többszálas feldolgozás** - Képek párhuzamos feldolgozása
* ⚡ **GPU (CUDA) gyorsítás** - Az NVIDIA GPU teljesítményének kihasználása
* 💻 **CLI hozzáférés** – Automatizálás parancssori eszközökkel
* 🐍 **Python SDK** – Programozott API hozzáférés
* 📱 **Több eszköz** - Használható 2–10+ eszközön (a csomagtól függően)
* **🐻 Fejlett textúraérzékeny debayer módszer** - kiváló minőségű, élekkel számoló debayer, AI/ML zajszűrő modellel kombinálva, amely szinte az összes debayering zajt eltávolítja.
* 🧮 **Egyéni képletek** - Egyéni multispektrális indexek létrehozása

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Az Chloros+ csomagok és árak megtekintése</a></p>***

## Telepítési segítség

### Hibaelhárítás

**A telepítés hibaüzenettel meghiúsul:**

* Győződjön meg arról, hogy rendelkezik rendszergazdai jogokkal
* Ideiglenesen tiltsa le a víruskereső szoftvert
* Ellenőrizze, hogy megfelel-e a minimális rendszerkövetelményeknek

**Az alkalmazás nem indul el (Windows):**

* Ellenőrizze, hogy az Windows 10/11 (64 bites) telepítve van-e
* Frissítse a grafikus illesztőprogramokat
* Ellenőrizze az Windows Eseménynaplóban a hiba részleteit
* Vegye fel a kapcsolatot az ügyfélszolgálattal a hibajelentésekkel

**Az CLI nem indul el (Linux):**

* Ellenőrizze, hogy az `.deb` csomag megfelelően van-e telepítve: `dpkg -l | grep chloros`
* Ellenőrizze a jogosultságokat: `sudo chmod +x /usr/bin/chloros-cli`
* Futtassa a diagnosztikát: `chloros-cli selftest`
* Ellenőrizze, hogy hiányoznak-e könyvtárak: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Licenc-aktiválási problémák:**

* Győződjön meg arról, hogy az internetkapcsolat aktív
* Ellenőrizze a hitelesítő adatokat a [https://cloud.mapir.camera](https://cloud.mapir.camera) oldalon
* Ellenőrizze, hogy a tűzfal nem blokkolja-e az Chloros-et
* Részletes utasításokat a [Chloros+ Bejelentkezés](chloros+-login.md) oldalon talál

### Támogatás

Segítségre van szüksége a telepítéssel vagy a beállítással kapcsolatban?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Weboldal**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentáció**: [Első lépések](./)
* ❓ **GYIK**: [Gyakran ismételt kérdések](faq.md)***

## Változásnapló

<details>

<summary>1.1.0 verzió (legújabb)</summary>

**Kiadás dátuma: 2026. március**

**Új funkciók*** **Linux támogatás** — Natív CLI és SDK az Linux amd64 (x86\_64) és arm64 (NVIDIA Jetson JetPack 6) platformokhoz. Telepítés `.deb` csomagok segítségével.
* **NVIDIA Jetson támogatás** — Optimalizált feldolgozás Jetson Nano, Orin Nano, Orin NX és AGX Orin edge eszközökhöz.
* **Dinamikus számítási adaptáció** — Automatikus hardverfelismerés és feldolgozási stratégia optimalizálás. Az Chloros a Jetson Nano-tól a több GPU-s munkaállomásig alkalmazkodik a hardveréhez.
* **4-szálas feldolgozási folyamat** — Párhuzamos felismerési, kalibrálási, feldolgozási és exportálási szálak dinamikus GPU-memória-allokációval.
* **Új CLI parancsok** — `selftest` (rendszerdiagnosztika) és `update` (Linux frissítéskezelés).
* **Új CLI folyamatjelzők** — `--debayer` (standard/textúra-érzékeny), `--indices` (indexek megadása), `--target` (először a cél almappában keres a gyorsabb felismerés érdekében).
* **Új GUI menüpontok** — A Fájlok hozzáadása, Mappa hozzáadása és Feldolgozás indítása/leállítása menüpontok mostantól elérhetők a főmenü legördülő menüjéből.**Javítások**

* Platformok közötti háttérprogram automatikus felismerése (Windows és Linux útvonalak)
* Továbbfejlesztett SDK `get_status()` szálonkénti előrehaladás-követéssel
* Új SDK kivételek: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Hőkezelés és adaptív fojtás az NVIDIA Jetson számára
* Automatikus memóriakezelés OOM-visszaeséssel a csempés GPU-feldolgozásra

</details>

<details>

<summary>1.0.5 verzió</summary>

**Kiadás dátuma: 2026. február 10.**

**Új funkciók*** **Texture Aware debayer módszer \[Csak Chloros+] -** A Texture Aware egy kiváló minőségű, élekkel számoló debayert használ, amelyhez egy AI/ML zajszűrő modell társul, amely szinte az összes debayering zajt eltávolítja.
* **T4P kalibrációs célok támogatása*** **Gyorsabb Chloros+ GPU-feldolgozás, jobb memóriakezelés**

**Hibajavítások*** Teljesen új felhasználói felület (GUI), most már minden Windows számítógépen működnie kell.

</details>

<details>

<summary>1.0.4-es verzió</summary>

**Kiadás dátuma: 2026. január 5.**

**Új funkciók*** **Kép/metaadatok váltás**: Váltó gombot adtunk a Fájlkezelőhöz, amellyel a kiválasztott kép metaadatai táblázatban jeleníthetők meg a kép rács helyett
* **Képrács nagyítás csúszka**: Új felhasználói felületi csúszka a miniatűrök méretének beállításához (támogatja a CTRL + egérgörgő kombinációt is)
* **Képrács export gombok**: Gombok a felső sorban a miniatűrök JPG-ről feldolgozott exportokra (Célok, Reflektancia, Index, LUT) váltásához
* **Térkép fül**: Új interaktív 2D térkép, amely megjeleníti a képek GPS-helyjelölőit
  * Támogatja a Google Maps és az ESRI térképlapokat (a nagyítási szint elérhetősége alapján automatikusan kiválasztja a legjobb térképlap-szolgáltatást)
  * Egérrel az egérmutatóval a térképjelölőkön a miniatűrök előnézete

**Hibajavítások*** Javított támogatás az Chloros telepítéséhez nem angol nyelvű számítógépeken

</details>

<details>

<summary>1.0.3-as verzió</summary>

**Kiadás dátuma: 2025. december 20.**

**Új funkciók*** Első kiadás

**Javítások*** Első kiadás

**Hibajavítások*** Első kiadás

**Ismert problémák*** Első kiadás

</details>***

## Licencszerződés**Saját fejlesztésű szoftver** - Copyright (c) 2026 MAPIR Inc.

A jogosulatlan használat, terjesztés vagy módosítás tilos.

**Ingyenes verzió**: Személyes és kereskedelmi használatra elérhető, funkciók korlátozásával**Chloros+**: Előfizetéses licenc a fejlett funkciókhoz és kereskedelmi telepítésekhez
