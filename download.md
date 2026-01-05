---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Letöltés

Töltse le az Chloros legújabb verzióját, hogy elindulhasson a multispektrális képfeldolgozás.

### Rendszerkövetelmények

| Követelmény          | Minimális                         | Ajánlott                     |
| -------------------- | ------------------------------- | ------------------------------- |
| **Operációs rendszer** | Windows 10 (64 bites)             | Windows 11 (64 bites)             |
| **Processzor**        | Intel Core i5 vagy azzal egyenértékű     | Intel Core i7 vagy annál jobb         |
| **Memória (RAM)**     | 8 GB                             | 16 GB vagy több                    |
| **Grafikus kártya**    | DirectX 11 kompatibilis           | NVIDIA GPU 4 GB+ VRAM-mal       |
| **Tárolókapacitás**          | 6 GB szabad hely                  | SSD 10 GB+ szabad hellyel       |
| **Kijelző**          | 1920x1080                       | 2560x1440 vagy nagyobb             |
| **Internet**         | Szükséges a licenc aktiválásához | Szükséges a licenc aktiválásához |

{% hint style=&quot;info&quot; %}
**GPU-gyorsítás**: Az NVIDIA GPU-val (4 GB+ VRAM) rendelkező Chloros+ felhasználók a CUDA-gyorsítást használhatják a jelentősen gyorsabb feldolgozás érdekében. Az Chloros+ felhasználók többszálas feldolgozást is kapnak a maximális sebesség érdekében.
{% endhint %}

***

## Chloros letöltése

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Chloros letöltése itt</a>

### Legújabb stabil kiadás

**Chloros telepítő Windows*** **Verzió**: 1.0.4
* **Kiadás dátuma**: 2026. január 5.
* **Fájlméret (letöltés)**: 1,8 GB
* **Fájlméret (telepítés után)**: 5,7 GB
* **Fájltípus**: .exe (Windows telepítő)

#### **Telepítési lépések:**

1. Töltse le az `CHLOROS INSTALLER - CURRENT VERSION.exe` fájlt
2. Kattintson duplán a telepítőre a telepítés megkezdéséhez
3. Kövesse a telepítő varázsló utasításait
4. Válassza ki a telepítési könyvtárat (alapértelmezett: `C:\Program Files\[USER]\Chloros\`)
5. Végezze el a telepítést, majd indítsa el az Chloros, Chloros (böngésző) vagy Chloros CLI alkalmazást.
6. Jelentkezzen be [MAPIR Cloud Chloros+ fiókjával](https://cloud.mapir.camera/pricing) (vagy folytassa az ingyenes verzióval)

{% hint style=&quot;success&quot; %}
A telepítő automatikusan hozzáadja az `chloros-cli` fájlt a rendszer PATH-jéhez a parancssori hozzáféréshez.
{% endhint %}

***

## További források

### Python SDK

Fejlesztők és automatizált munkafolyamatok esetén telepítse az Chloros Python SDK programot:

```bash
pip install chloros-sdk
```

**Dokumentáció**: [API: Python SDK](api-python-sdk.md)**Követelmények**: Chloros Desktop telepítése szükséges, Chloros+ licenc bejelentkezés szükséges.***

## Mit tartalmaz

Az Chloros telepítés a következőket tartalmazza:

* ✅ **Chloros** - Teljes funkcionalitású grafikus felület
* ✅ **Chloros (böngésző)** - Webalapú felület alacsonyabb specifikációjú rendszerekhez
* ✅ **Chloros CLI** - Parancssori felület (Chloros+ licenc szükséges)
* ✅ **Chloros SDK** - Python API (Chloros+ licenc szükséges)
* ✅ **Kameraprofilok** - Előre konfigurált MAPIR kamerasablonok***

## Frissítés Chloros+ verzióra

A Chloros+ előfizetéssel fejlett funkciókhoz juthat hozzá:

* 🚀 **Többszálas feldolgozás** – Képek párhuzamos feldolgozása
* ⚡ **GPU (CUDA) gyorsítás** – Az NVIDIA GPU teljesítményének kihasználása
* 💻 **CLI hozzáférés** – Automatizálás parancssori eszközökkel
* 🐍 **Python SDK** – Programozható API hozzáférés
* 📱 **Több eszköz** – 2–10+ eszközön használható (a csomagtól függően)
* 🧮 **Egyéni képletek** – Egyéni multispektrális indexek létrehozása

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Chloros+ csomagok és árak megtekintése</a></p>***

## Telepítési segítség

### Hibaelhárítás

**A telepítés hibaüzenettel meghiúsul:**

* Győződjön meg arról, hogy rendelkezik rendszergazdai jogokkal
* Ideiglenesen tiltsa le a víruskereső szoftvert
* Ellenőrizze, hogy megfelel-e a minimális rendszerkövetelményeknek

**Az alkalmazás nem indul el:**

* Próbálja ki az Chloros (böngésző) verziót
* Ellenőrizze, hogy az Windows 10/11 (64 bites) telepítve van-e
* Frissítse a grafikus illesztőprogramokat
* Ellenőrizze az Windows Eseménynaplóban a hiba részleteit
* Forduljon a támogatáshoz a hibanaplóval

**Licencaktiválási problémák:**

* Ellenőrizze, hogy az internetkapcsolat aktív-e
* Ellenőrizze a hitelesítő adatokat a [https://cloud.mapir.camera](https://cloud.mapir.camera) oldalon
* Ellenőrizze, hogy a tűzfal nem blokkolja-e az Chloros alkalmazást
* Részletes utasításokat a [Chloros+ Bejelentkezés](chloros+-login.md) oldalon talál.

### Támogatás

Segítségre van szüksége a telepítéssel vagy a beállítással kapcsolatban?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Weboldal**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentáció**: [Első lépések](./)
* ❓ **GYIK**: [Gyakran ismételt kérdések](faq.md)***

## Változásnapló

<details>

<summary>1.0.4 verzió</summary>

#### **Kiadás dátuma**: 2026. január 5.**Új funkciók*** **Kép/metaadatok váltás**: A Fájlböngészőbe hozzáadott váltó, amely a kiválasztott kép metaadatait táblázatban jeleníti meg a képek rácsának helyett
* **Képek rácsának nagyítás-csúszka**: Új felhasználói felület csúszka a miniatűrök méretének beállításához (a CTRL + egérkerék kombinációt is támogatja)
* **Képrács export gombok**: A felső sorban található gombok a miniatűrök JPG formátumból feldolgozott export formátumba (célok, visszaverődés, index, LUT) való átváltásához
* **Térkép fül**: Új interaktív 2D térkép, amely a képek GPS-helymegjelölőit mutatja.
  * Támogatja a Google Maps és az ESRI térképcsempéket (a nagyítási szint elérhetősége alapján automatikusan kiválasztja a legjobb csempeszolgáltatást).
  * Egérrel a térképjelölők fölé mozgatva a miniatűrök előnézete megjelenik.

**Hibajavítások*** Javított támogatás az Chloros telepítéséhez nem angol nyelvű számítógépeken.

</details>

<details>

<summary>1.0.3 verzió</summary>

#### **Kiadás dátuma**: 2025. december 20.**Új funkciók*** Első indítás

**Javítások*** Első indítás

**Hibajavítások*** Első indítás

**Ismert problémák*** Első indítás

</details>***

## Licencszerződés**Saját fejlesztésű szoftver** – Copyright (c) 2025 MAPIR Inc.

A jogosulatlan használat, terjesztés vagy módosítás tilos.

**Ingyenes verzió**: Személyes és kereskedelmi használatra elérhető, funkciók korlátozásával.**Chloros+**: Előfizetéses licenc fejlett funkciókhoz és kereskedelmi felhasználáshoz.
