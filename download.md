---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Letöltés

Töltse le a Chloros legújabb verzióját, hogy elindulhasson a multispektrális képfeldolgozás terén.

### Rendszerkövetelmények

#### Windows


| Követelmény          | Minimális                                              | Ajánlott                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Operációs rendszer** | Windows 10 (64 bites)                                  | Windows 11 (64 bites)                                  |
| **Processzor**        | Intel Core i5 vagy azzal egyenértékű                          | Intel Core i7 vagy annál jobb                              |
| **Memória (RAM)**     | 8 GB                                                  | 16 GB vagy több                                         |
| **Grafikus kártya**    | DirectX 11 kompatibilis                                | NVIDIA GPU 4 GB vagy több VRAM-mal                            |
| **Tárhely**          | 6 GB szabad hely                                       | SSD 10 GB vagy több szabad hellyel                            |
| **Kijelző**          | 1920x1080                                            | 2560x1440 vagy nagyobb felbontás                                  |
| **Internet**         | Szükséges a \[opcionális] Chloros+ licenc aktiválásához | Szükséges a \[opcionális] Chloros+ licenc aktiválásához |

#### Linux amd64 (x86\_64)

| Követelmény       | Minimális                    | Ajánlott               |
| ----------------- | -------------------------- | ------------------------- |
| **Disztribúció**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Processzor**     | x86\_64 (Intel/AMD)        | Intel Core i7 vagy jobb   |
| **Memória (RAM)**  | 8 GB                        | 16 GB vagy több              |
| **Grafikus kártya** | Nincs (CPU-feldolgozás)      | NVIDIA GPU 4 GB vagy több VRAM-mal |
| **Tárhely**       | 2 GB szabad hely             | 10 GB vagy több szabad helyű SSD       |
| **Python**        | Python 3.7+ (az SDK-hez)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Követelmény      | Minimális                      | Ajánlott                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Platform**     | NVIDIA Jetson JetPack 6-tal | Jetson Orin NX 16 GB vagy AGX Orin |
| **Memória (RAM)** | 8 GB (megosztott GPU/CPU)         | 16 GB vagy több megosztott                    |
| **Tárhely**      | 2 GB szabad hely               | NVMe SSD 10 GB+ szabad hellyel        |
| **Python**       | Python 3.7+ (az SDK-hez)        | Python 3.10+                    |

{% hint style="info" %}
**GPU-gyorsítás**: Az NVIDIA GPU-val rendelkező Chloros+ felhasználók a CUDA-gyorsítást használhatják a feldolgozás jelentősen gyorsabbá tételéhez. Ez Windows és Linux amd64 rendszereken az NVIDIA asztali GPU-kkal, valamint JetPack 6-tal felszerelt NVIDIA Jetson eszközökön működik. A Chloros+ felhasználók emellett többszálas feldolgozást is igénybe vehetnek a maximális sebesség elérése érdekében.
{% endhint %}

***

## A Chloros
letöltése

### Legújabb stabil kiadás: 1.3.0-s verzió (2026. november 11.)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Chloros</a>
<a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">letöltése Windowsra (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Chloros</a>
<a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">letöltése Linux amd64-re (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Töltse le a Chloros-t Linux arm64 / Jetson rendszerre (.deb)</a>


#### Windows telepítő (GUI + CLI + háttér)

* **Fájltípus**: .exe (Windows telepítő)

**Telepítési lépések:**

1. Töltse le a fenti .exe fájlt
2. Kattintson duplán a telepítőre a telepítés elindításához
3. Kövesse a telepítő varázsló utasításait
4. Válassza ki a telepítési könyvtárat (alapértelmezett: `C:\Program Files\Chloros\`)
5. A **További feladatok kiválasztása**oldalon hagyja bejelölve mindkét**Rendszerbeállítások** opciót (ajánlott — lásd alább)
6. Fejezze be a telepítést, majd indítsa el a Chloros vagy a Chloros CLI programot 7. Jelentkezzen be a [MAPIR Cloud Chloros+ fiókjával](https://cloud.mapir.camera/pricing) (vagy folytassa az ingyenes verzióval)

**Rendszerbeállítások opciók** (alapértelmezés szerint mindkettő be van jelölve):

| Opció, a telepítőben szereplő megfogalmazás szerint | Mit csinál | Csendes telepítés kikapcsolása |
| --- | --- | --- |
| **Windows hosszú elérési útvonalak engedélyezése** (lehetővé teszi, hogy a projektmappák elérési útvonala meghaladja a 260 karaktert; ajánlott) | A Windows rendszer egészére vonatkozó `LongPathsEnabled` beállítást 1-re állítja. A Windows alapértelmezés szerint kikapcsolt állapotban szállítja, és ilyenkor egyetlen fájlútvonal sem lehet hosszabb 260 karakternél; a Chloros-termékek a projektmappa alatt körülbelül 130 karakterrel helyezkednek el, így a hosszú útvonalak kikapcsolása esetén a mélyebben elhelyezett projektmappát a rendszer elutasítja. A beállítás a telepítés után elindított programokra vonatkozik. Az eltávolítás után is aktív marad, mivel más programok is támaszkodhatnak rá. | `/MERGETASKS="!longpaths"` |
| **A Chloros-háttérprogram kizárása a Microsoft Defender vizsgálatából** (sokkal gyorsabb kamerafelvétel; ajánlott) | Hozzáad egy Microsoft Defender-folyamatkizárást az `chloros-backend.exe` fájlhoz a telepítési mappában, így a valós idejű védelem nem vizsgálja át minden fájlt, amit a háttérprogram ír, függetlenül attól, hová írja. Magas rögzítési sebesség mellett a vizsgálat a rögzítési sebesség körülbelül 30%-át tette ki (9 kamerás folyamatos rögzítés esetén 77,7 szemben 52,7 rögzítéssel másodpercenként). Egyéb programok nem kerülnek kizárásra, és a kizárás az eltávolításkor megszűnik. Ha a Defender nincs telepítve, vagy a házirend letiltja, ez a lépés észrevétlenül meghiúsul, és a telepítés folytatódik. | `/MERGETASKS="!defenderexclusion"` |

A csendes telepítés (`/SILENT` vagy `/VERYSILENT`) mindkét beállítást alkalmazza, kivéve, ha a megjelenített `/MERGETASKS` értékkel leállítja.

{% hint style="success" %}
A telepítő egyszer hozzáadja a CLI mappát (`C:\Program Files\Chloros\cli\`) a rendszeréhez `PATH` — az újratelepítés vagy frissítés már nem hoz létre duplikált bejegyzést, és eltávolítja a korábbi telepítők által hozzáadott duplikátumokat. A telepítés után nyisson meg egy új terminált az `chloros-cli` használatához. A telepítő emellett minden hálózati profilhoz két Windows-tűzfal-szabályt is hozzáad a háttérprogramhoz: **Chloros Backend (UDP-In)**, kameraképek, PTP, mDNS és a DAQ-E adatfolyam számára, valamint**Chloros Backend (TCP-In)**, amely a helyi alhálózatra korlátozódik, a DAQ-E firmware-frissítéshez, amikor az érzékelő visszacsatlakozik erre a számítógépre a képek begyűjtése érdekében. Mindkettő eltávolításra kerül az eltávolításkor.
{% endhint %}

#### Linux amd64 (.deb csomag — CLI + háttér)

* **Fájltípus**: .deb (Debian/Ubuntu csomag)
* **Architektúra**: x86\_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux arm64 — NVIDIA Jetson (.deb csomag — CLI + háttérprogram)

* **Fájltípus**: .deb (JetPack 6)
* **Architektúra**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Használja a ténylegesen letöltött fájlnevet, és tartsa meg az `./` nevet, hogy az `apt` azt a fájlt telepítse, ahelyett, hogy név alapján keresne csomagot. Telepítse az `apt install` paranccsal, ne az `dpkg -i` parancsot: az apt ugyanis telepíti a Python SDK követelményeit is, amelyeket a csomag inkább ajánl, mint előír, az `dpkg -i` parancs pedig kihagyja őket. Az 1.3.0-s verziótól kezdve a háttérprogram és a CLI-parancssor kicsomagolatlanul kerül telepítésre az `/usr/lib/chloros/` alatt, így elindulnak anélkül, hogy előbb kicsomagolnák magukat.

A Jetson-csomag a JetPack 6 CUDA-alapú PyTorch-verzióját használja, így a feldolgozás – beleértve a Texture Aware debayert is – a Jetson GPU-ján fut. A JetPack 6 által telepített CUDA- és cuDNN-könyvtárakat használja (`sudo apt install nvidia-jetpack` egy olyan képen, amelyen ezek hiányoznak).

A részletes beállítási utasításokat lásd a [Linux-telepítés](linux/linux-installation.md) című részben, a Jetson-specifikus útmutatást pedig az [NVIDIA Jetson útmutató](linux/nvidia-jetson-guide.md) című részben.

#### Python SDK (minden platform)

Minden telepítő tartalmaz egy megfelelő `chloros_sdk` wheel-t, így az SDK verziója mindig egyezik a telepített GUI/CLI/háttérprogrammal. Windows rendszeren a telepítő automatikusan telepíti a rendszer Python-jébe. Linux rendszeren az `.deb` a rendszer `python3` számára telepíti az SDK-t (az `/usr/lib/python3/dist-packages`-be), így az `import chloros_sdk` már a telepítés után azonnal működik, pip és hálózat nélkül. Csak egy olyan virtualenv esetében, amely nem látja a rendszer csomagjait, szükséges a csomagban lévő wheel telepítése:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

A kizárólag pip-et használó gazdagépek esetében (ahol nincs telepítve Chloros-csomag; az SDK ilyenkor egy másik gépen futó háttérprogramot vezérel) az SDK a PyPI-n is elérhető:

```bash
pip install chloros-sdk
```

A dokumentációt lásd az [API: Python SDK](api-python-sdk.md) és az [SDK-referencia](reference/sdk-reference.md) oldalakon.

{% hint style="info" %}
**Linux-felhasználók**: Az `.deb` csomag telepíti a CLI-t, a háttérprogramot és a Python SDK-t. Linux-hez nincs grafikus felhasználói felület — minden interakció a CLI-en vagy az SDK-en keresztül történik.
{% endhint %}

***

## További források

### Python SDK


A fejlesztők és az automatizált munkafolyamatok számára a Chloros Python SDK minden telepítővel együtt jár: a Windows-telepítő a Python-ba telepíti, míg a Linux `.deb` a rendszer `python3` számára telepíti. Chloros-csomag nélküli gépen (amely egy másik gépen futó háttérrendszert vezérel) telepítse a PyPI-ről:

```bash
pip install chloros-sdk
```

**Dokumentáció**: [API: Python SDK](api-python-sdk.md)

**Követelmények**: egy vezérelendő Chloros háttérprogram (a Windows-telepítő vagy a Linux `.deb` csomag telepít egyet), valamint egy fizetős szintű Chloros+ bejelentkezés

***

## A csomag tartalma

### Windows-telepítő

* ✅ **Chloros GUI** – Teljes funkcionalitású grafikus felület
* ✅ **Chloros CLI** – Parancssori felület (Chloros+ licenc szükséges)
* ✅ **Chloros háttérprogram** – Feldolgozó motor
* ✅ **Python SDK** – telepítésre kerül a Python környezetbe, ha az rendelkezésre áll (Chloros+ licenc szükséges)
* ✅ **Kamera profilok** – Előre konfigurált MAPIR kamera sablonok

### Linux.deb csomag

* ✅ **Chloros CLI** – Parancssori felület (Chloros+ licenc szükséges)
* ✅ **Chloros Backend** – Feldolgozó motor
* ✅ **Python SDK** – telepítve az `python3` rendszerre (Chloros+ licenc szükséges)
* ✅ **Kamera profilok** – Előre konfigurált MAPIR kamera sablonok
* ❌ Nincs grafikus felület — a Linux kizárólag parancssori (CLI)/SDK felületet támogat

### Python SDK (PyPI, Chloros csomaggal nem rendelkező gépekhez)

* ✅ **Chloros SDK** – Python API (Chloros+ licenc szükséges)

***

## Frissítés Chloros+-ra

Chloros+ előfizetéssel hozzáférhet a fejlett funkciókhoz:

* 🚀 **Többszálas feldolgozás** – Képek párhuzamos feldolgozása
* ⚡ **GPU (CUDA) gyorsítás** - Használja ki az NVIDIA GPU teljesítményét
* 💻 **CLI-hozzáférés** – Automatizálás parancssori eszközökkel
* 🐍 **Python SDK** – Programozási API-hozzáférés
* 📱 **Több eszköz** – Használható 2–10+ eszközön (a csomagtól függően)
* **🐻 Fejlett, textúraérzékeny debayer-módszer** – kiváló minőségű, élekkel számoló debayer, amely egy AI/ML zajszűrő modellel kombinálva szinte teljesen eltávolítja a debayeringből származó zajt.
* 🧮 **Egyéni képletek** – Egyéni multispektrális indexek létrehozása

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Chloros+ csomagok és árak megtekintése</a></p>

***

## Telepítési segítség

### Hibaelhárítás

**A telepítés hibaüzenettel meghiúsul:**

* Győződjön meg róla, hogy rendszergazdai jogosultságokkal rendelkezik
* Ideiglenesen tiltsa le a víruskereső szoftvert
* Ellenőrizze, hogy megfelel-e a minimális rendszerkövetelményeknek

**Az alkalmazás nem indul el (Windows):**

* Ellenőrizze, hogy a Windows 10/11 (64 bites) rendszer telepítve van-e
* Frissítse a grafikus illesztőprogramokat
* Ellenőrizze a Windows Eseménynaplóban a hiba részleteit
* Vegye fel a kapcsolatot az ügyfélszolgálattal a hibajelentésekkel

**A CLI nem indul el (Linux):**

* Ellenőrizze, hogy az `.deb` csomag megfelelően van-e telepítve: `dpkg -l | grep chloros`
* Ellenőrizze a jogosultságokat: `sudo chmod +x /usr/bin/chloros-cli`
* Futtassa a diagnosztikát: `chloros-cli selftest`
* Ellenőrizze, hogy nincsenek-e hiányzó könyvtárak: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (az 1.3.0-s verziótól kezdve az `/usr/lib/chloros/chloros-backend` egy kis indító szkript; maga a program az `/usr/lib/chloros/backend/` fájlban található)

**Licenc-aktiválási problémák:**

* Győződjön meg arról, hogy aktív az internetkapcsolat
* Ellenőrizze a hitelesítő adatait a [https://cloud.mapir.camera](https://cloud.mapir.camera) oldalon
* Ellenőrizze, hogy a tűzfal nem blokkolja-e a Chloros szolgáltatást* Részletes utasításokat a [Chloros+ Bejelentkezés](chloros+-login.md) oldalon talál

### Támogatás igénylése

Segítségre van szüksége a telepítéssel vagy a beállítással kapcsolatban?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Weboldal**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Dokumentáció**: [Első lépések](./)
* ❓ **GYIK**: [Gyakran feltett kérdések](faq.md)

***

## Szoftverfrissítések A

Chloros ellenőrzi a frissítéseket, értesít, ha új verzió érhető el, és linket biztosít erre a letöltési oldalra — a frissítéshez futtassa az új, aláírt telepítőt. A beállításai, projektjei és bejelentkezési adatai a frissítés után is megmaradnak. Linux és Jetson rendszereken az `chloros-cli update` ellenőrzi, hogy van-e újabb verzió, és felajánlja a megfelelő `.deb` letöltését és telepítését (ez a parancs csak Linux rendszereken használható).

***

## Változásnapló**

1.3.0-s verzió (legújabb, 2026. november 11.)**— a teljes listát lásd az [Első lépések](./) oldalon a**Chloros 1.3.0 újdonságai** című részben.**

1.2.0-s verzió**— a teljes listát lásd a [Bevezetés](./) oldalon a**Chloros 1.2.0 újdonságai** című részben.

<details>

<summary>1.0.5-ös verzió</summary>

**Kiadás dátuma: 2026. február 10.**

**Új funkciók**

* **Textúraérzékeny debayer-módszer \[Csak Chloros+] -** A Textúraérzékeny funkció egy kiváló minőségű, élekkel számoló debayer-módszert használ, amelyet egy AI/ML zajszűrő modellel kombinálnak, így szinte az összes debayer-zajt eltávolítja.
* **T4P kalibrációs célok támogatása**
* **Gyorsabb Chloros+ GPU-feldolgozás, jobb memóriakezelés**

**Hibajavítások**

* Teljesen új felhasználói felület (GUI), amely mostantól minden Windows-számítógépen működnie kell.

</details>

<details>

<summary>1.0.4-es verzió</summary>

**Kiadás dátuma: 2026. január 5.**

**Új funkciók**

* **Kép/metaadatok váltás**: Hozzáadott váltógomb a Fájlkezelőben, amellyel a kiválasztott kép metaadatai táblázatban jeleníthetők meg a képrajz helyett
* **Képrajz nagyítási csúszka**: Új felhasználói felületi csúszka a miniatűrök méretének beállításához (a CTRL + egérgörgő kombinációt is támogatja)
* **Képrács export gombok**: A felső sorban található gombok segítségével a miniatűrök JPG formátumból feldolgozott exportformátumokra (Célok, Reflektancia, Index, LUT) válthatók át
* **Térkép fül**: Új interaktív 2D-térkép, amelyen a képek GPS-helyjelölői láthatók
  * Támogatja a Google Maps és az ESRI térképcsempéket (a nagyítási szint elérhetősége alapján automatikusan kiválasztja a legjobb csempeszolgáltatást)
  * Egérrel a térképjelölők fölé viszve megjelenik a miniatűr előnézete

**Hibajavítások**

* Javított támogatás a Chloros telepítéséhez nem angol nyelvű számítógépeken

</details>

<details>

<summary>1.0.3-as verzió</summary>

**Kiadás dátuma: 2025. december 20.**

**Új funkciók**

* Első kiadás

**Továbbfejlesztések**

* Első kiadás

**Hibajavítások**

* Első kiadás

**Ismert problémák**

* Első kiadás

</details>

***

## Licencszerződés**Saját fejlesztésű szoftver** – Szerzői jog (c) 2026 MAPIR Inc.

A jogosulatlan használat, terjesztés vagy módosítás tilos.

**Ingyenes verzió**: Személyes és kereskedelmi célokra egyaránt elérhető, funkciók korlátozásával**Chloros+**: Előfizetéses licenc a fejlett funkciókhoz és kereskedelmi felhasználáshoz
