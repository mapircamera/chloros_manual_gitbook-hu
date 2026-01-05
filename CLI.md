# CLI : Parancssor

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>Az **Chloros CLI** hatékony parancssori hozzáférést biztosít az Chloros képfeldolgozó motorhoz, lehetővé téve az automatizálást, a szkriptelést és a headless működést a képalkotási munkafolyamatokhoz.

### Főbb jellemzők

* 🚀 **Automatizálás** – Több adatkészlet szkriptes kötegelt feldolgozása
* 🔗 **Integráció** – Beágyazás meglévő munkafolyamatokba és folyamatokba
* 💻 **Headless működés** – GUI nélkül futtatható
* 🌍 **Többnyelvű** – 38 nyelv támogatása
* ⚡ **Párhuzamos feldolgozás** – Dinamikusan skálázódik a CPU-hoz (akár 16 párhuzamos munkás)

### Követelmények

| Követelmény          | Részletek                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Operációs rendszer** | Windows 10/11 (64 bites)                                              |
| **Licenc**          | Chloros+ ([fizetős csomag szükséges](https://cloud.mapir.camera/pricing)) |
| **Memória**           | Minimum 8 GB RAM (16 GB ajánlott)                                  |
| **Internet**         | A licenc aktiválásához szükséges                                     |
| **Lemezterület**       | A projekt méretétől függ                                              |

{% hint style=&quot;warning&quot; %}
**Licenc követelmény**: Az CLI fizetős Chloros+ előfizetést igényel. A standard (ingyenes) csomagok nem biztosítanak hozzáférést az CLI-hez. A frissítéshez látogasson el az [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) oldalra.
{% endhint %}

## Gyors indítás

### Telepítés

Az CLI automatikusan része az Chloros telepítőnek:

1. Töltse le és futtassa az **Chloros telepítőt.exe**.
2. Végezze el a telepítő varázslót.
3. CLI telepítve: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
A telepítő automatikusan hozzáadja az `chloros-cli` fájlt a rendszer PATH könyvtárához. A telepítés után indítsa újra a terminált.
{% endhint %}

### Első beállítás

Az CLI használata előtt aktiválja az Chloros+ licencét:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Alapvető használat

Feldolgozás egy mappában az alapértelmezett beállításokkal:

```powershell
chloros-cli process "C:\Images\Dataset001"
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

**Példa:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Feldolgozási parancs opciók

| Opció                | Típus    | Alapértelmezett        | Leírás                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Útvonal    | _Szükséges_     | RAW/JPG multispektrális képeket tartalmazó mappa                                         |
| `-o, --output`        | Útvonal    | Megegyezik a bemenettel  | A feldolgozott képek kimeneti mappája                                                     |
| `-n, --project-name`  | Karakterlánc  | Automatikusan generált | Egyéni projektnév                                                                    |
| `--vignette`          | Jelző    | Engedélyezve        | Vignettázás-korrekció engedélyezése                                                             |
| `--no-vignette`       | Jelző    | -              | Vignettázás-korrekció letiltása                                                            |
| `--reflectance`       | Jelző    | Engedélyezve        | Reflektancia kalibrálás engedélyezése                                                         |
| `--no-reflectance`    | Jelző    | -              | Reflektancia kalibrálás letiltása                                                        |
| `--ppk`               | Jelző    | Letiltva       | PPK-korrekciók alkalmazása a .daq fényérzékelő adatai alapján                                      |
| `--format`            | Választás  | TIFF (16 bites)  | Kimeneti formátum: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Egész szám | Automatikus           | A kalibrációs panel felismeréséhez szükséges minimális célméret pixelben                          |
| `--target-clustering` | Egész szám | Automatikus           | Célcsoportosítási küszöbérték (0-100)                                                    |
| `--exposure-pin-1`    | Karakterlánc  | Nincs           | Expozíció rögzítése a kamera modelljéhez (1. pin)                                                 |
| `--exposure-pin-2`    | Karakterlánc  | Nincs           | Expozíció rögzítése a kamera modelljéhez (2. pin)                                                 |
| `--recal-interval`    | Egész szám | Automatikus           | Újrakalibrálási intervallum másodpercben                                                      |
| `--timezone-offset`   | Egész szám | 0              | Időzóna eltérés órában                                                               |

***

### `login` - Fiók hitelesítése

Jelentkezzen be Chloros+ hitelesítő adataival az CLI feldolgozás engedélyezéséhez.

**Szintaxis:**

```bash
chloros-cli login <email> <password>
```

**Példa:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Különleges karakterek**: Használjon egyszerű idézőjeleket az olyan karaktereket tartalmazó jelszavak körül, mint `$`, `!` vagy szóközök.
{% endhint %}

**Kimenet:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Hitelesítő adatok törlése

Törölje a tárolt hitelesítő adatokat, és jelentkezzen ki fiókjából.

**Szintaxis:**

```bash
chloros-cli logout
```

**Példa:**

```powershell
chloros-cli logout
```

**Kimenet:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style=&quot;info&quot; %}
**SDK Felhasználók**: Az Python SDK programozási `logout()` módszert is biztosít a hitelesítő adatok törléséhez az Python szkriptekben. A részleteket lásd az [Python SDK dokumentációban](api-python-sdk.md#logout).
{% endhint %}

***

### `status` - Licencállapot ellenőrzése

A jelenlegi licenc és hitelesítési állapot megjelenítése.

**Szintaxis:**

```bash
chloros-cli status
```

**Példa:**

```powershell
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

### `export-status` - Exportálás előrehaladásának ellenőrzése

A 4. szál exportálásának előrehaladását figyeli a feldolgozás alatt vagy után.

**Szintaxis:**

```bash
chloros-cli export-status
```

**Példa:**

```powershell
chloros-cli export-status
```

**Használati eset:** Hívja meg ezt a parancsot a feldolgozás futása közben az exportálás előrehaladásának ellenőrzéséhez.***

### `language` – Az interfész nyelvének kezelése

Az CLI interfész nyelvének megtekintése vagy módosítása.

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

```powershell
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
| `ko`    | koreai                | 한국어              |
| `zh`    | kínai (egyszerűsített)  | 简体中文             |
| `zh-TW` | kínai (hagyományos) | 繁體中文             |
| `ru`    | Orosz               | Русский          |
| `nl`    | Holland                 | Nederlands       |
| `ar`    | Arab                | العربية          |
| `pl`    | Lengyel                | Polski           |
| `tr`    | török               | Türkçe           |
| `hi`    | hindi                 | हिंदी            |
| `id`    | indonéz            | Bahasa Indonesia |
| `vi`    | vietnámi            | Tiếng Việt       |
| `th`    | thai                  | ไทย              |
| `sv`    | svéd               | Svenska          |
| `da`    | dán                | Dansk            |
| `no`    | Norvég             | Norsk            |
| `fi`    | Finn               | Suomi            |
| `el`    | Görög                 | Ελληνικά         |
| `cs`    | Cseh                 | Čeština          |
| `hu`    | Magyar             | Magyar           |
| `ro`    | Román              | Română           |
| `uk`    | Ukrán             | Українська       |
| `pt-BR` | brazil portugál  | Português Brasileiro |
| `zh-HK` | kantoni             | 粵語             |
| `ms`    | Maláj                 | Bahasa Melayu    |
| `sk`    | Szlovák                | Slovenčina       |
| `bg`    | Bolgár             | Български        |
| `hr`    | Horvát              | Hrvatski         |
| `lt`    | Litván            | Lietuvių         |
| `lv`    | Lett               | Latviešu         |
| `et`    | Észt              | Eesti            |
| `sl`    | Szlovén             | Slovenščina      |

{% hint style=&quot;success&quot; %}
**Automatikus megőrzés**: A nyelvi beállítások az `~/.chloros/cli_language.json` fájlba kerülnek mentésre, és minden munkamenetben megmaradnak.
{% endhint %}

***

### `set-project-folder` - Alapértelmezett projektmappa beállítása

Az alapértelmezett projektmappa helyének módosítása (a GUI-val megosztva).

**Szintaxis:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Példa:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` - Projektmappa megjelenítése

Az aktuális alapértelmezett projektmappa helyének megjelenítése.

**Szintaxis:**

```bash
chloros-cli get-project-folder
```

**Példa:**

```powershell
chloros-cli get-project-folder
```

**Kimenet:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder` - Alapértelmezett értékre visszaállítás

A projektmappa alapértelmezett helyére állítja vissza.

**Szintaxis:**

```bash
chloros-cli reset-project-folder
```

***

## Globális beállítások

Ezek az opciók minden parancsra vonatkoznak:

| Opció          | Típus    | Alapértelmezett       | Leírás                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Útvonal    | Automatikus felismerés | A háttérprogram futtatható fájljának elérési útvonala                       |
| `--port`        | Egész szám | 5000          | Háttérprogram API portszáma                          |
| `--restart`     | Jelző    | -             | A háttérprogram erőltetett újraindítása (a meglévő folyamatok leállítása) |
| `--version`     | Jelző    | -             | Verzióinformációk megjelenítése és kilépés                |
| `--help`        | Jelző    | -             | Súgóinformációk megjelenítése és kilépés                   |

**Példa globális beállításokkal:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Feldolgozási beállítások útmutatója

### Párhuzamos feldolgozás

Chloros+ CLI **automatikusan méretezi**a párhuzamos feldolgozást a számítógép teljesítményéhez igazodva:**Hogyan működik:**

* Felismeri a CPU magokat és a RAM-ot
* Munkások kiosztása: **2× CPU mag** (hiperthreading használata)
* **Maximum: 16 párhuzamos munkás** (a stabilitás érdekében)**Rendszer szintek:**

| Rendszer típus   | CPU        | RAM      | Munkások  | Teljesítmény     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **High-End**  | 16+ mag  | 32+ GB   | Akár 16 | Maximális sebesség   |
| **Középkategória** | 8-15 mag | 16-31 GB | 8-16     | Kiváló sebesség |
| **Alacsony kategória**   | 4-7 mag  | 8-15 GB  | 4-8      | Jó sebesség      |

{% hint style=&quot;success&quot; %}
**Automatikus optimalizálás**: Az CLI automatikusan felismeri a rendszer specifikációit és beállítja az optimális párhuzamos feldolgozást. Nincs szükség manuális konfigurálásra!
{% endhint %}

### Debayer módszerek

Az CLI alapértelmezésként és ajánlott debayer algoritmusként a **High Quality (Faster)** (Kiváló minőség (gyorsabb)) módot használja:

| Módszer                      | Minőség | Sebesség | Leírás                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Magas minőség (gyorsabb)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Élekkel számoló algoritmus (alapértelmezett, ajánlott) |

### Vignette-korrekció

**Működése:** Korrigálja a kép széleinél fellépő fénycsökkenést (a fényképezőgépekkel készített képeken gyakori sötétebb sarkok).

* **Alapértelmezés szerint engedélyezve** – A legtöbb felhasználónak ezt engedélyezve kell hagynia.
* Az `--no-vignette` használatával letiltható.

{% hint style=&quot;success&quot; %}
**Ajánlás**: A képegyenletes fényerejének biztosítása érdekében mindig engedélyezze a vignett korrekciót.
{% endhint %}

### Reflektancia kalibrálás

A kalibrációs panelek segítségével a nyers szenzorértékeket szabványosított reflektancia százalékokra konvertálja.

* **Alapértelmezés szerint engedélyezve** – Elengedhetetlen a növényzet elemzéséhez.
* Kalibrációs célpanelek szükségesek a képeken.
* Az `--no-reflectance` használatával letiltható.

{% hint style=&quot;info&quot; %}
**Követelmények**: A pontos reflektancia-átalakításhoz győződjön meg arról, hogy a kalibrációs panelek megfelelően vannak-e exponálva és láthatók-e a képeken.
{% endhint %}

### PPK-korrekciók

**Működése:** A DAQ-A-SD naplóadatok felhasználásával utólagos kinematikai korrekciókat alkalmaz a GPS-pontosság javítása érdekében.

* **Alapértelmezés szerint letiltva**
* Az engedélyezéshez használja az `--ppk` parancsot.
* Az MAPIR DAQ-A-SD fényérzékelő projektmappájában található .daq fájlok szükségesek.

### Kimeneti formátumok

<table><thead><tr><th width="197">Formátum</th><th width="130.20001220703125">Bitmélység</th><th width="116.5999755859375">Fájlméret</th><th>Legalkalmasabb</th></tr></thead><tbody><tr><td><strong>TIFF (16 bites)</strong> ⭐</td><td>16 bites egész szám</td><td>Nagy</td><td>GIS-elemzés, fotogrammetria (ajánlott)</td></tr><tr><td><strong>TIFF (32 bites, százalék)</strong></td><td>32 bites lebegőpontos</td><td>Nagyon nagy</td><td>Tudományos elemzés, kutatás</td></tr><tr><td><strong>PNG (8 bites)</strong></td><td>8 bites egész</td><td>Közepes</td><td>Vizuális ellenőrzés, webes megosztás</td></tr><tr><td><strong>JPG (8 bites)</strong></td><td>8 bites egész szám</td><td>Kicsi</td><td>Gyors előnézet, tömörített kimenet</td></tr></tbody></table>***

## Automatizálás és szkriptelés

### PowerShell kötegelt feldolgozás

Több adatkészlet-mappa automatikus feldolgozása:

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

### Windows kötegelt szkript

Egyszerű hurok kötegelt feldolgozáshoz:

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

### Python automatizálási szkript

Fejlett automatizálás hiba kezeléssel:

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

1. **Bemenet**: RAW/JPG képeket tartalmazó mappa
2. **Felfedezés**: CLI automatikus keresés a támogatott képfájlok után
3. **Feldolgozás**: Párhuzamos mód, amely a CPU magok számához igazodik (Chloros+)
4. **Kimenet**: A feldolgozott képeket tartalmazó, kameramodell szerinti almappák létrehozása

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

| Mód              | Idő      | Hardver                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Párhuzamos mód** | 5-10 perc  | i7/Ryzen 7, 16 GB RAM, SSD (legfeljebb 16 munkás) |
| **Párhuzamos mód** | 10-15 perc | i5/Ryzen 5, 8 GB RAM, HDD (legfeljebb 8 munkás)   |

{% hint style=&quot;info&quot; %}
**Teljesítménytipp**: A feldolgozási idő a képek számától, felbontásától és a számítógép specifikációitól függ.
{% endhint %}

***

## Hibaelhárítás

### CLI nem található

**Hiba:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Megoldások:**

1. Ellenőrizze a telepítés helyét:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Ha nincs a PATH-ban, használja a teljes elérési utat:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Adja hozzá manuálisan a PATH-hoz:
   * Nyissa meg a Rendszer tulajdonságait → Környezeti változók
   * Szerkessze a PATH változót
   * Adja hozzá: `C:\Program Files\Chloros\resources\cli`
   * Indítsa újra a terminált.

***

### A háttérprogram elindítása sikertelen.**Hiba:**

```

Backend failed to start within 30 seconds
```

**Megoldások:**

1. Ellenőrizze, hogy a háttérprogram már fut-e (először zárja be).
2. Ellenőrizze, hogy az Windows tűzfal nem blokkolja-e.
3. Próbáljon ki egy másik portot:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Kényszerítse a háttérprogram újraindítását:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Licenc/hitelesítési problémák**Hiba:**

```

Chloros+ license required for CLI access
```

**Megoldások:**

1. Ellenőrizze, hogy rendelkezik-e aktív Chloros+ előfizetéssel.
2. Jelentkezzen be a hitelesítő adataival:

```powershell
chloros-cli login user@example.com 'password'
```

3. Ellenőrizze a licenc állapotát:

```powershell
chloros-cli status
```

4. Vegye fel a kapcsolatot az ügyfélszolgálattal: info@mapir.camera

***

### Nincs kép**Hiba:**

```

No images found in the specified folder
```

**Megoldások:**

1. Ellenőrizze, hogy a mappa támogatott formátumokat tartalmaz-e (.RAW, .TIF, .JPG).
2. Ellenőrizze, hogy a mappa elérési útja helyes-e (szóközökkel ellátott elérési utak esetén használjon idézőjeleket).
3. Győződjön meg arról, hogy rendelkezik a mappa olvasási jogosultságával.
4. Ellenőrizze, hogy a fájlkiterjesztések helyesek-e.

***

### A feldolgozás leáll vagy lefagy**Megoldások:**

1. Ellenőrizze a rendelkezésre álló lemezterületet (győződjön meg arról, hogy elegendő a kimenethez).
2. Zárjon be más alkalmazásokat a memória felszabadítása érdekében.
3. Csökkentse a képek számát (folytassa a feldolgozást tételekben).

***

### A port már használatban van**Hiba:**

```

Port 5000 is already in use
```

**Megoldás:**

Adjon meg egy másik portot:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## GYIK

### K: Szükségem van licencre az CLI használatához?

**V:**Igen! Az CLI használatához fizetős**Chloros+ licenc** szükséges.

* ❌ Standard (ingyenes) csomag: CLI letiltva
* ✅ Chloros+ (fizetős) csomagok: CLI teljes mértékben engedélyezve

Iratkozzon fel: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### K: Használhatom az CLI-et GUI nélküli szerveren?**V:** Igen! Az CLI teljesen headless módon fut. Követelmények:

* Windows Server 2016 vagy újabb
* Visual C++ Redistributable telepítve
* Elegendő RAM (minimum 8 GB, ajánlott 16 GB)
* Egyszeri GUI licenc aktiválás bármely gépen

***

### K: Hol kerülnek mentésre a feldolgozott képek?**V:**Alapértelmezés szerint a feldolgozott képek**a bemeneti képekkel azonos mappában** kerülnek mentésre, a kamera modelljének almappáiban (pl. `Survey3N_RGN/`).

Az `-o` opcióval más kimeneti mappát is megadhat:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### K: Több mappát is feldolgozhatok egyszerre?**V:** Egy parancsban nem, de szkript segítségével egymás után feldolgozhatja a mappákat. Lásd az [Automatizálás és szkriptelés](CLI.md#automation--scripting) részt.***

### K: Hogyan menthetem az CLI kimenetet egy naplófájlba?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### K: Mi történik, ha a feldolgozás közben megnyomom a Ctrl+C billentyűkombinációt?**V:** Az CLI:

1. Megállítja a feldolgozást
2. Leállítja a háttérprogramot
3. 130-as kóddal lép ki

A részben feldolgozott képek a kimeneti mappában maradhatnak.

***

### K: Automatizálhatom az CLI feldolgozást?**V:** Természetesen! Az CLI automatizálásra lett tervezve. A PowerShell, Batch és Python példákért lásd az [Automatizálás és szkriptelés](CLI.md#automation--scripting) részt.***

### K: Hogyan ellenőrizhetem az CLI verziót?**V:**

```powershell
chloros-cli --version
```

**Kimenet:**

```

Chloros CLI 1.0.2
```

***

## Súgó

### Parancssori súgó

A súgó információk közvetlenül az CLI-ben tekinthetők meg:

```powershell
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

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### 2. példa: Kiváló minőségű tudományos eredmény

32 bites lebegőpontos TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### 3. példa: Gyors előnézeti feldolgozás

8 bites PNG kalibrálás nélkül, gyors áttekintéshez:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### 4. példa: PPK-korrigált feldolgozás

PPK-korrekciók alkalmazása visszaverődéssel:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### 5. példa: Egyéni kimeneti hely

Feldolgozás különböző meghajtóra, meghatározott formátumban:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### 6. példa: Hitelesítési munkafolyamat

Teljes hitelesítési folyamat:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### 7. példa: Többnyelvű használat

A felület nyelvének megváltoztatása:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
