# CLI : Parancssor

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>Az **Chloros CLI** hatékony parancssori hozzáférést biztosít az Chloros képfeldolgozó motorhoz, lehetővé téve az automatizálást, a szkriptelést és a headless működést a képfeldolgozási munkafolyamatokhoz.

### Főbb jellemzők

* 🚀 **Automatizálás** – Több adatkészlet szkriptes kötegelt feldolgozása
* 🔗 **Integráció** – Beágyazás meglévő munkafolyamatokba és folyamatláncokba
* 💻 **Headless működés** – Futtatás GUI nélkül
* 🌍 **Többnyelvűség** – 38 nyelv támogatása
* ⚡ **Párhuzamos feldolgozás** - A [dinamikus számítási adaptáció](processing-architecture/dynamic-compute-adaptation.md) automatikusan optimalizálja a hardverhez

### Követelmények

| Követelmény          | Részletek                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Operációs rendszer** | Windows 10/11 (64 bites), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Licenc**          | Chloros+ ([fizetős csomag szükséges](https://cloud.mapir.camera/pricing)) |
| **Memória**           | Minimum 8 GB RAM (ajánlott: 16 GB)                                  |
| **Internet**         | A licenc aktiválásához szükséges                                     |
| **Lemezterület**       | A projekt méretétől függ                                              |

{% hint style="warning" %}
**Licenc követelmény**: Az CLI fizetős Chloros+ előfizetést igényel. A Standard (ingyenes) csomagok nem biztosítanak hozzáférést az CLI-hez. A frissítéshez látogasson el az [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) oldalra.
{% endhint %}

## Gyors indítás

### Telepítés

#### Windows

Az CLI automatikusan része az Chloros telepítőjének:

1. Töltse le és futtassa az **Chloros Installer.exe** fájlt
2. Végigvezesse a telepítő varázslót
3. Az CLI telepítve: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
A telepítő automatikusan hozzáadja az `chloros-cli` fájlt a rendszer PATH változójához. A telepítés után indítsa újra a terminált.
{% endhint %}

#### Linux

Telepítse az architektúrájához megfelelő `.deb` csomagot:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Az Linux részletes beállításához lásd [Linux Telepítés](linux/linux-installation.md).

### Első beállítás

Az CLI használata előtt aktiválja az Chloros+ licencét:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Alapvető használat

Mappa feldolgozása alapértelmezett beállításokkal:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Parancsok

### Általános szintaxis

```
chloros-cli [global-options] <command> [command-options]
```

***

## Parancsok

### `process` - Képek feldolgozása

Képek feldolgozása egy mappában kalibrálással.

**Szintaxis:**

```bash
chloros-cli process <input-folder> [options]
```

**Példák:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### A feldolgozási parancs opciói

| Opció                | Típus    | Alapértelmezett        | Leírás                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Útvonal    | _Kötelező_     | A RAW/JPG multispektrális képeket tartalmazó mappa                                         |
| `-o, --output`        | Útvonal    | Megegyezik a bemenettel  | A feldolgozott képek kimeneti mappája                                                     |
| `-n, --project-name`  | Karakterlánc  | Automatikusan generált | Egyéni projektnév                                                                    |
| `--vignette`          | Jelző    | Engedélyezve        | Vignettakorrekció engedélyezése                                                             |
| `--no-vignette`       | Jelző    | -              | Vignettakorrekció letiltása                                                            |
| `--reflectance`       | Jelző    | Engedélyezve        | Reflektancia-kalibrálás engedélyezése                                                         |
| `--no-reflectance`    | Jelző    | -              | Reflektancia-kalibrálás letiltása                                                        |
| `--ppk`               | Jelző    | Letiltva       | PPK-korrekciók alkalmazása a .daq fényérzékelő adatokból                                      |
| `--format`            | Választás  | TIFF (16 bites)  | Kimeneti formátum: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Egész szám | Automatikus           | A kalibrációs panel felismeréséhez szükséges minimális célméret pixelben                          |
| `--target-clustering` | Egész szám | Automatikus           | Célcsoportosítási küszöbérték (0–100)                                                    |
| `--debayer`           | Választás  | `standard`     | Debayer-módszer: `standard` vagy `texture-aware` (csak Chloros+)                          |
| `--target`, `--targets` | Jelző  | Letiltva       | Csak a „target” vagy „targets” almappában keressen kalibrációs célpontokat (gyorsítja a feldolgozást) |
| `--indices`           | Lista    | Nincs           | Számítandó vegetációs indexek (pl. `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Karakterlánc  | Nincs           | Az expozíció rögzítése a kameramodellhez (1. pin)                                                 |
| `--exposure-pin-2`    | Karakterlánc  | Nincs           | A kameramodell expozíciójának rögzítése (2. pin)                                                 |
| `--recal-interval`    | Egész szám | Automatikus           | Újrakalibrálási intervallum másodpercben                                                      |
| `--timezone-offset`   | Egész szám | 0              | Időzónaeltérés órában                                                               |

***

### `login` - Fiók hitelesítése

Jelentkezzen be az Chloros+ hitelesítő adataival az CLI feldolgozás engedélyezéséhez.

**Szintaxis:**

```bash
chloros-cli login <email> <password>
```

**Példa:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Különleges karakterek**: Az olyan jelszavaknál, amelyek olyan karaktereket tartalmaznak, mint az `$`, `!` vagy szóközök, használjon egyszerű idézőjeleket.
{% endhint %}

**Kimenet:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Hitelesítő adatok törlése

Törli a tárolt hitelesítő adatokat és kijelentkezik a fiókjából.

**Szintaxis:**

```bash
chloros-cli logout
```

**Példa:**

```bash
chloros-cli logout
```

**Kimenet:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**SDK Felhasználók**: Az Python SDK egy programozási `logout()` metódust is biztosít a hitelesítő adatok törléséhez az Python szkriptekben. A részleteket lásd az [Python SDK dokumentációban](api-python-sdk.md#logout).
{% endhint %}

***

### `status` - Licencállapot ellenőrzése

Megjeleníti az aktuális licenc- és hitelesítési állapotot.

**Szintaxis:**

```bash
chloros-cli status
```

**Példa:**

```bash
chloros-cli status
```

**Kimenet:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` - Exportálás állapotának ellenőrzése

A feldolgozás alatt vagy után figyelemmel kíséri a 4. szál exportálásának állapotát.

**Szintaxis:**

```bash
chloros-cli export-status
```

**Példa:**

```bash
chloros-cli export-status
```

**Használati eset:** Hívja meg ezt a parancsot a feldolgozás futása közben az exportálás előrehaladásának ellenőrzéséhez.***

### `language` - Felület nyelvének kezelése

Az CLI felület nyelvének megtekintése vagy módosítása.

**Szintaxis:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Példák:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Támogatott nyelvek (összesen 38)

| Kód    | Nyelv              | Eredeti név      |
| ------- | --------------------- | ---------------- |
| `en`    | Angol               | English          |
| `es`    | Spanyol               | Español          |
| `pt`    | Portugál            | Português        |
| `fr`    | Francia                | Français         |
| `de`    | Német                | Deutsch          |
| `it`    | Olasz               | Italiano         |
| `ja`    | Japán              | 日本語              |
| `ko`    | Koreai                | 한국어              |
| `zh`    | Kínai (egyszerűsített)  | 简体中文             |
| `zh-TW` | Kínai (hagyományos) | 繁體中文             |
| `ru`    | Orosz               | Русский          |
| `nl`    | Holland                | Nederlands       |
| `ar`    | Arab                | العربية          |
| `pl`    | Lengyel                | Polski           |
| `tr`    | Török               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indonéz            | Bahasa Indonesia |
| `vi`    | Vietnámi            | Tiếng Việt       |
| `th`    | Thai                  | ไทย              |
| `sv`    | Svéd               | Svenska          |
| `da`    | Dán                | Dansk            |
| `no`    | Norvég             | Norsk            |
| `fi`    | Finn               | Suomi            |
| `el`    | Görög                 | Ελληνικά         |
| `cs`    | Cseh                 | Čeština          |
| `hu`    | Magyar             | Magyar           |
| `ro`    | Román              | Română           |
| `uk`    | Ukrán             | Українська       |
| `pt-BR` | brazil portugál  | Português Brasileiro |
| `zh-HK` | kantoni             | 粵語             |
| `ms`    | maláj                 | Bahasa Melayu    |
| `sk`    | Szlovák                | Slovenčina       |
| `bg`    | Bolgár             | Български        |
| `hr`    | Horvát              | Hrvatski         |
| `lt`    | Litván            | Lietuvių         |
| `lv`    | Lett               | Latviešu         |
| `et`    | Észt              | Eesti            |
| `sl`    | Szlovén             | Slovenščina      |

{% hint style="success" %}
**Automatikus megőrzés**: A nyelvi beállítások az `~/.chloros/cli_language.json` fájlba kerülnek mentésre, és minden munkamenet során megmaradnak.
{% endhint %}

***

### `set-project-folder` - Alapértelmezett projektmappa beállítása

Az alapértelmezett projektmappa helyének módosítása (megosztva a GUI-val az Windows fájlban).

**Szintaxis:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Példák:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - Projektmappa megjelenítése

Megjeleníti az aktuális alapértelmezett projektmappa helyét.

**Szintaxis:**

```bash
chloros-cli get-project-folder
```

**Példa:**

```bash
chloros-cli get-project-folder
```

**Kimenet:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` - Alapértelmezett értékre állítás

A projektmappát az alapértelmezett helyre állítja vissza.

**Szintaxis:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` - Rendszerdiagnosztika futtatása

7 diagnosztikai ellenőrzés futtatása a rendszerkonfiguráció ellenőrzéséhez.

**Szintaxis:**

```bash
chloros-cli selftest
```

**Elvégzett diagnosztikai ellenőrzések:**

1. Verzióellenőrzés
2. Port elérhetősége (5000)
3. Backend indítás
4. API kapcsolódási teszt
5. Rendszerinformációk és GPU felismerés
6. Zajszűrő modellek ellenőrzése
7. CUDA elérhetőségi ellenőrzés

{% hint style="info" %}
**Hasznos a hibaelhárításhoz**: A telepítés után futtassa az `selftest` parancsot, hogy ellenőrizze, a rendszer megfelelően van-e konfigurálva, különösen az Linux/Jetson esetében, ahol a GPU és a CUDA beállításait ellenőrizni kell.
{% endhint %}

***

### `update` - Frissítések keresése (csak Linux esetén)

Keresse meg és telepítse az CLI frissítéseket az Linux rendszereken.

**Szintaxis:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Opció    | Leírás                        |
| --------- | ---------------------------------- |
| `--check` | Csak frissítések keresése, telepítés nélkül |

{% hint style="info" %}
Ez a parancs csak az Linux rendszeren érhető el. Az Windows rendszeren a frissítéseket a telepítőprogramon keresztül kapja meg.
{% endhint %}

***

## Általános beállítások

Ezek az opciók minden parancsra vonatkoznak:

| Opció            | Típus    | Alapértelmezett       | Leírás                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Útvonal    | Automatikusan felismerve | A háttérprogram végrehajtható fájljának elérési útja                       |
| `--port`          | Egész szám | 5000          | A háttérprogram API portszáma                          |
| `--restart`       | Jelző    | -             | A háttérprogram újraindításának kényszerítése (a meglévő folyamatok leállítása) |
| `--version`       | Jelző    | -             | Verzióinformációk megjelenítése és kilépés                |
| `--help`          | Jelző    | -             | Súgóinformációk megjelenítése és kilépés                   |

{% hint style="info" %}
**Háttérprogram automatikus felismerése**: Az `--backend-exe` elérési út platformonként automatikusan felismerésre kerül:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (manuális)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Példa globális beállításokkal:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Feldolgozási beállítások útmutató

### Párhuzamos feldolgozás és dinamikus számítási adaptáció

Az Chloros 1.1.0 verzió tartalmazza a [dinamikus számítási adaptációt](processing-architecture/dynamic-compute-adaptation.md) — a feldolgozó motor **automatikusan felismeri a hardvert**, és kiválasztja az optimális stratégiát:

| Platform | Stratégia | Munkások | Folyamat | Megjegyzések |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Memóriahatékony, sorosított |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Párhuzamos GPU-feldolgozás |
| **Asztali számítógép 8 GB-os GPU-val** | `GPU_SINGLE` | 3 | `tiled_gpu` | Jó asztali teljesítmény |
| **Asztali számítógép 12 GB-os vagy nagyobb GPU-val** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Optimális asztali teljesítmény |
| **Csak CPU-val rendelkező rendszer** | `CPU_PARALLEL` | magok - 1 | `cpu_fallback` | Nincs szükség GPU-ra |

{% hint style="success" %}
**Nincs szükség kézi konfigurálásra!** Az Chloros automatikusan felismeri a CPU-t, a GPU-t, a RAM-ot és (Jetson esetén) a hőérzékelőket, majd automatikusan konfigurálja az optimális feldolgozási folyamatot.
{% endhint %}

### Debayer-módszerek

| Módszer | CLI jelző | Minőség | Sebesség | Licenc |
| --- | --- | --- | --- | --- |
| **Standard (Gyors, Közepes minőség)** | `--debayer standard` | Jó | Gyors | Ingyenes / Chloros+ |
| **Textúraérzékeny (lassú, legmagasabb minőség)** | `--debayer texture-aware` | Legmagasabb | Lassú | Csak Chloros+ |

Az alapértelmezett debayer módszer a **Standard**. A**Texture Aware** módszer AI/ML zajszűrő modellt használ a legmagasabb minőségű kimenet érdekében, de ehhez Chloros+ licenc és NVIDIA GPU szükséges.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Vignette-korrekció

**Működése:** Korrigálja a kép széleinél jelentkező fénycsökkenést (a fényképezőgépes képeken gyakran előforduló sötétebb sarkokat).

* **Alapértelmezés szerint engedélyezve** – A legtöbb felhasználónak érdemes ezt engedélyezve hagyni
* Az `--no-vignette` parancs használatával tiltható le

{% hint style="success" %}
**Ajánlás**: Mindig engedélyezze a vignettakorrekciót, hogy a kép egészén egyenletes legyen a fényerő.
{% endhint %}

### Reflektancia-kalibrálás

A kalibrációs panelek segítségével konvertálja a nyers érzékelőértékeket szabványosított reflektancia-százalékokká.

* **Alapértelmezés szerint engedélyezve** – Elengedhetetlen a növényzet elemzéséhez
* Kalibrációs célpanelek szükségesek a képeken
* Az `--no-reflectance` parancs használatával tiltható le

{% hint style="info" %}
**Követelmények**: A pontos fényvisszaverődés-átalakítás érdekében győződjön meg arról, hogy a kalibrációs panelek megfelelően vannak exponálva és láthatók a képein.
{% endhint %}

### PPK-korrekciók

**Működése:** A DAQ-A-SD naplóadatok felhasználásával utólagos kinematikai korrekciókat alkalmaz a GPS-pontosság javítása érdekében.

* **Alapértelmezés szerint letiltva**
* Az engedélyezéshez használja az `--ppk` parancsot
* Ehhez .daq fájlokra van szükség a projektmappában az MAPIR DAQ-A-SD fényérzékelőtől.

### Kimeneti formátumok

<table><thead><tr><th width="197">Formátum</th><th width="130.20001220703125">Bitmélység</th><th width="116.5999755859375">Fájlméret</th><th>Legalkalmasabb</th></tr></thead><tbody><tr><td><strong>TIFF (16 bites)</strong> ⭐</td><td>16 bites egész szám</td><td>Nagy</td><td>GIS-elemzés, fotogrammetria (ajánlott)</td></tr><tr><td><strong>TIFF (32 bites, százalék)</strong></td><td>32 bites lebegőpontos</td><td>Nagyon nagy</td><td>Tudományos elemzés, kutatás</td></tr><tr><td><strong>PNG (8 bites)</strong></td><td>8 bites egész szám</td><td>Közepes</td><td>Szemrevételezés, webes megosztás</td></tr><tr><td><strong>JPG (8 bites)</strong></td><td>8 bites egész szám</td><td>Kicsi</td><td>Gyors előnézet, tömörített kimenet</td></tr></tbody></table>***

## Automatizálás és szkriptelés

### PowerShell kötegelt feldolgozás (Windows)

Több adatkészlet-mappa automatikus feldolgozása az Windows-en:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Windows kötegelt szkript (Windows)

Egyszerű hurok kötegelt feldolgozáshoz az Windows-en:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Bash kötegelt feldolgozás (Linux)

Több adatkészlet-mappa feldolgozása az Linux-en:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Python automatizálási szkript (platformfüggetlen)

Fejlett automatizálás hiba kezeléssel (működik az Windows és az Linux verziókon):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Feldolgozási munkafolyamat

### Standard munkafolyamat

1. **Bemenet**: RAW/JPG kép párokat tartalmazó mappa
2. **Felfedezés**: Az CLI automatikusan átvizsgálja a támogatott képfájlokat
3. **Feldolgozás**: A párhuzamos mód a CPU magok számához igazodik (Chloros+)
4. **Kimenet**: Létrehozza a feldolgozott képeket tartalmazó, kameramodell szerinti almappákat

### Példa a kimeneti struktúrára

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Becsült feldolgozási idő

Tipikus feldolgozási idő 100 kép (egyenként 12 MP) esetén:

| Platform | Mód | Becsült idő | Megjegyzések |
| --- | --- | --- | --- |
| **Asztali számítógép 12 GB+ GPU** | `GPU_PARALLEL` | 5–10 perc | A leggyorsabb lehetőség |
| **Asztali számítógép 8 GB GPU** | `GPU_SINGLE` | 10–15 perc | Jó teljesítmény |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 15–25 perc | Edge számítás |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 30–60 perc | Memória-korlátozott |
| **Csak CPU** | `CPU_PARALLEL` | 20–40 perc | Nincs szükség GPU-ra |

{% hint style="info" %}
**Teljesítményre vonatkozó tipp**: A feldolgozási idő a képek számától, a felbontástól, a debayer módszertől és a hardvertől függően változik. A Texture Aware debayer jelentősen hosszabb időt vesz igénybe, mint a Standard. A részleteket lásd a [Dynamic Compute Adaptation](processing-architecture/dynamic-compute-adaptation.md) oldalon.
{% endhint %}

***

## Hibaelhárítás

### CLI nem található

**Windows hiba:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows megoldások:**

1. Ellenőrizze a telepítési helyet:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Ha nincs a PATH-ben, használja a teljes elérési utat:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Adja hozzá manuálisan a PATH-hez:
   * Nyissa meg a Rendszer tulajdonságait → Környezeti változók
   * Szerkessze a PATH változót
   * Adja hozzá: `C:\Program Files\Chloros\resources\cli`
   * Indítsa újra a terminált

**Linux Hiba:**

```
chloros-cli: command not found
```

**Linux Megoldások:**

1. Ellenőrizze a telepítést:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Töltse be újra a shell-t:

```bash
source ~/.bashrc
```

3. Ellenőrizze a jogosultságokat:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### A háttérprogram indítása sikertelen**Hiba:**

```

Backend failed to start within 30 seconds
```

**Megoldások:**

1. Ellenőrizze, hogy a háttérprogram már fut-e (először zárja be)
2. Ellenőrizze, hogy a tűzfal nem blokkolja-e (Windows), vagy ellenőrizze a port elérhetőségét (Linux: `lsof -i :5000`)
3. Próbáljon ki egy másik portot:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Kényszerítse a háttérprogram újraindítását:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. Az Linux esetében ellenőrizze, hogy létezik-e a háttérprogram futtatható fájlja:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Licenc / Hitelesítési problémák**Hiba:**

```

Chloros+ license required for CLI access
```

**Megoldások:**

1. Ellenőrizze, hogy rendelkezik-e érvényes Chloros+ előfizetéssel
2. Jelentkezzen be a hitelesítő adataival:

```bash
chloros-cli login user@example.com 'password'
```

3. Ellenőrizze a licenc állapotát:

```bash
chloros-cli status
```

4. Vegye fel a kapcsolatot az ügyfélszolgálattal: info@mapir.camera

***

### Nincs kép**Hiba:**

```

No images found in the specified folder
```

**Megoldások:**

1. Ellenőrizze, hogy a mappa támogatott formátumokat tartalmaz-e (.RAW, .TIF, .JPG)
2. Ellenőrizze, hogy a mappa elérési útja helyes-e (szóközökkel rendelkező elérési utak esetén használjon idézőjeleket)
3. Győződjön meg arról, hogy rendelkezik a mappa olvasási jogosultságával
4. Ellenőrizze, hogy a fájlkiterjesztések helyesek-e

***

### A feldolgozás leáll vagy lefagy**Megoldások:**

1. Ellenőrizze a rendelkezésre álló lemezterületet (győződjön meg arról, hogy elegendő-e a kimenethez)
2. Zárja be az egyéb alkalmazásokat a memória felszabadítása érdekében
3. Csökkentse a képek számát (több részletben dolgozza fel)

***

### A port már használatban van**Hiba:**

```

Port 5000 is already in use
```

**Megoldások:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## GYIK

### K: Szükségem van licencre az CLI használatához?

**V:**Igen! Az CLI használatához fizetős**Chloros+ licenc** szükséges.

* ❌ Standard (ingyenes) csomag: az CLI letiltva
* ✅ Chloros+ (fizetős) csomagok: az CLI teljes mértékben engedélyezve

Feliratkozás: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### K: Használhatom az CLI-et GUI nélküli szerveren?**V:** Igen! Az CLI teljesen headless módon fut. Ez az Linux elsődleges felhasználási esete.**Windows szerver:**
* Windows Server 2016 vagy újabb
* Visual C++ Redistributable telepítve

**Linux szerver:**
* Ubuntu 20.04+ / Debian 11+ (amd64) vagy JetPack 6 (arm64)
* Telepítés az `.deb` csomag segítségével

**Mindkét platform:**
* Minimum 8 GB RAM (ajánlott: 16 GB)
* Egyszeri licencaktiválás: `chloros-cli login user@example.com 'password'`

***

### K: Hová kerülnek a feldolgozott képek?**V:**Alapértelmezés szerint a feldolgozott képek a**bemeneti képekkel azonos mappába** kerülnek, a kamera-modell alkönyvtárakba (pl. `Survey3N_RGN/`).

Az `-o` opcióval más kimeneti mappát is megadhat:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### K: Egyszerre több mappát is feldolgozhatok?**V:** Egy parancsban közvetlenül nem, de szkriptek segítségével a mappákat egymás után feldolgozhatja. Lásd az [Automatizálás és szkriptelés](CLI.md#automation--scripting) részt.***

### K: Hogyan menthetem el az CLI kimenetet egy naplófájlba?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### K: Mi történik, ha a feldolgozás közben megnyomom a Ctrl+C billentyűkombinációt?**V:** Az CLI a következőket fogja tenni:

1. A feldolgozást rendben leállítja
2. Leállítja a háttérprogramot
3. 130-as kóddal lép ki

A részben feldolgozott képek a kimeneti mappában maradhatnak.

***

### K: Automatizálhatom az CLI feldolgozását?**V:** Természetesen! Az CLI programot automatizálásra tervezték. Lásd az [Automatizálás és szkriptelés](CLI.md#automation--scripting) részt a PowerShell (Windows), a Batch (Windows), Bash (Linux) és Python (platformok közötti) példákról.***

### K: Hogyan ellenőrizhetem az CLI verziót?**V:**

```bash
chloros-cli --version
```

**Kimenet:**

```

Chloros CLI 1.1.0
```

***

## Súgó

### Parancssori súgó

A súgó információk közvetlenül az CLI-ben tekinthetők meg:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Támogatási csatornák

* **E-mail**: info@mapir.camera
* **Weboldal**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Árak**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Teljes példák

### 1. példa: Alapvető feldolgozás

Feldolgozás alapértelmezett beállításokkal (vignette, reflectance):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### 2. példa: Kiváló minőségű tudományos kimenet

32 bites lebegőpontos TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### 3. példa: Gyors előnézeti feldolgozás

8 bites PNG kalibrálás nélkül a gyors áttekintéshez:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### 4. példa: PPK-korrigált feldolgozás

PPK-korrekciók alkalmazása reflektanciával:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### 5. példa: Egyéni kimeneti hely

Feldolgozás egy másik helyre, meghatározott formátumban:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### 6. példa: Hitelesítési munkafolyamat

Teljes hitelesítési folyamat (minden platformon azonos):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### 7. példa: Többnyelvű használat

Felület nyelvének megváltoztatása (minden platformon azonos):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
