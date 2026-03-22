# API : Python SDK

Az **Chloros Python SDK** programozási hozzáférést biztosít az Chloros képfeldolgozó motorhoz, lehetővé téve az automatizálást, az egyedi munkafolyamatokat és a zökkenőmentes integrációt az Python alkalmazásokkal és kutatási folyamatokkal.

### Főbb jellemzők

* 🐍 **Natív Python** - Tiszta, Pythonic API a képfeldolgozáshoz
* 🔧 **Teljes API hozzáférés** - Teljes ellenőrzés az Chloros feldolgozás felett
* 🚀 **Automatizálás** - Egyedi kötegelt feldolgozási munkafolyamatok létrehozása
* 🔗 **Integráció** - Az Chloros beágyazása meglévő Python alkalmazásokba
* 📊 **Kutatásra kész** - Tökéletes tudományos elemzési folyamatokhoz
* ⚡ **Párhuzamos feldolgozás** - A CPU magok számához igazodik (Chloros+)

### Követelmények

| Követelmény          | Részletek                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros telepítve** | Windows: Asztali telepítő; Linux: `.deb` csomag                  |
| **Licenc**          | Chloros+ ([fizetős tervezet szükséges](https://cloud.mapir.camera/pricing)) |
| **Operációs rendszer** | Windows 10/11 (64 bites), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python**           | Python 3.7 vagy újabb                                                |
| **Memória**           | Legalább 8 GB RAM (16 GB ajánlott)                                  |
| **Internet**         | Szükséges a licenc aktiválásához                                     |

{% hint style="warning" %}
**Licenc követelmény**: Az Python SDK használatához fizetős Chloros+ előfizetés szükséges az API hozzáféréshez. A standard (ingyenes) csomagok nem biztosítanak hozzáférést az API/SDK szolgáltatáshoz. A frissítéshez látogasson el a [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) oldalra.
{% endhint %}

## Gyors indítás

### Telepítés

Telepítés pip segítségével:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**Első beállítás**: Az SDK használata előtt aktiválja az Chloros+ licencét az Chloros, Chloros (böngésző) vagy az Chloros CLI alkalmazást, és jelentkezzen be a hitelesítő adataival. Ezt csak egyszer kell elvégeznie. Az Linux (GUI nélkül) alkalmazásban használja a következő parancsot: `chloros-cli login user@example.com 'password'`
{% endhint %}

### Alapvető használat

Feldolgozzon egy mappát néhány sorral:

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**Platformok közötti elérési utak**: Az ezen az oldalon található kódpéldák Windows stílusú elérési utakat használnak (pl. `C:\\DroneImages\\Flight001`). Linux esetén használjon inkább Linux útvonalakat (pl. `/home/user/drone_images/flight001` vagy `~/drone_images/flight001`). Az SDK mindkét platformon azonos módon működik.
{% endhint %}

### Teljes vezérlés

Haladó munkafolyamatokhoz:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")  # Windows
# chloros.import_images("/home/user/drone_images/flight001")  # Linux

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Telepítési útmutató

### Előfeltételek

Az SDK telepítése előtt győződjön meg arról, hogy rendelkezik a következőkkel:

1. **Chloros telepítve** — Windows: Asztali telepítő ([letöltés](download.md)); Linux: `.deb` csomag ([Linux telepítés](linux/linux-installation.md))
2. **Python 3.7+** telepítve ([python.org](https://www.python.org))
3. **Aktív Chloros+ licenc** ([frissítés](https://cloud.mapir.camera/pricing))

### Telepítés pip segítségével

**Normál telepítés:**

```bash
pip install chloros-sdk
```

**Haladásfigyelés támogatással:**

```bash
pip install chloros-sdk[progress]
```

**Fejlesztői telepítés:**

```bash
pip install chloros-sdk[dev]
```

### A telepítés ellenőrzése

Ellenőrizze, hogy az SDK megfelelően van-e telepítve:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Első beállítás

### Licenc aktiválása

Az SDK ugyanazt a licencet használja, mint az Chloros, az Chloros (böngésző) és az Chloros CLI. Aktiválja egyszer a grafikus felhasználói felületen vagy az CLI segítségével:**Windows:**Nyissa meg az**Chloros vagy az Chloros (böngésző)** alkalmazást, és jelentkezzen be a Felhasználó <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> fülön, vagy használja az CLI-et.**Linux:** Használja az CLI-et (GUI nem elérhető):

```bash
chloros-cli login user@example.com 'your_password'
```

A licencet a rendszer helyileg tárolja, és az újraindítás után is megmarad.

{% hint style="success" %}
**Egyszeri beállítás**: A GUI-n vagy az CLI-en keresztül történő bejelentkezés után az SDK automatikusan a gyorsítótárban tárolt licencet használja. Nincs szükség további hitelesítésre!
{% endhint %}

{% hint style="info" %}
**Kijelentkezés**: Az SDK felhasználók programozási úton törölhetik a gyorsítótárban tárolt hitelesítő adatokat az `logout()` metódus használatával. Lásd a [logout() metódust](#logout) az API Referenciában.
{% endhint %}

### Kapcsolat tesztelése

Ellenőrizze, hogy az SDK képes-e csatlakozni az Chloros-hez:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## API referencia

### ChlorosLocal osztály

A helyi Chloros képfeldolgozás fő osztálya.

#### Konstruktor

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Paraméterek:**

| Paraméter                 | Típus | Alapértelmezett                   | Leírás                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL a helyi Chloros háttérprogramhoz          |
| `auto_start_backend`      | bool | `True`                    | A háttérprogram automatikus indítása, ha szükséges |
| `backend_exe`             | str  | `None` (automatikus felismerés)      | A háttérprogram végrehajtható fájljának elérési útja            |
| `timeout`                 | int  | `30`                      | Kérés időtúllépése másodpercben            |
| `backend_startup_timeout` | int  | `60`                      | A háttérprogram indításának időtúllépése (másodperc) |

**Példák:**

```python
# Default (auto-start backend, auto-detect path on Windows and Linux)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path (Windows)
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom backend path (Linux)
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")

# Custom timeout with longer startup (e.g., for Jetson)
chloros = ChlorosLocal(timeout=60, backend_startup_timeout=120)
```

{% hint style="info" %}
**Platformok közötti automatikus felismerés**: Az SDK automatikusan megpróbálja a platformodhoz megfelelő háttérprogram elérési útját:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (manuális)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### Módszerek

#### `create_project(project_name, camera=None)`

Hozzon létre egy új Chloros projektet.

**Paraméterek:**

| Paraméter      | Típus | Szükséges | Leírás                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Igen      | A projekt neve                                     |
| `camera`       | str  | Nem       | Kamera sablon (pl. „Survey3N\_RGN”, „Survey3W\_OCN”) |

**Visszatérési érték:** `dict` - Projekt létrehozási válasz**Példa:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Képek importálása egy mappából.

**Paraméterek:**

| Paraméter     | Típus     | Szükséges | Leírás                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Igen      | A képeket tartalmazó mappa elérési útja         |
| `recursive`   | bool     | Nem       | Almappák keresése (alapértelmezett: False) |

**Visszatérési érték:** `dict` - Importálási eredmények a fájlok számával**Példa:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

A feldolgozási beállítások konfigurálása.

**Paraméterek:**

| Paraméter                 | Típus | Alapértelmezett                 | Leírás                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | &quot;Standard (Gyors, Közepes minőség)&quot; | Debayer módszer            |
| `vignette_correction`     | bool | `True`                  | Vignettakorrekció engedélyezése      |
| `reflectance_calibration` | bool | `True`                  | Reflektancia-kalibrálás engedélyezése  |
| `indices`                 | lista | `None`                  | Számítandó vegetációs indexek |
| `export_format`           | str  | &quot;TIFF (16-bit)&quot;         | Kimeneti formátum                   |
| `ppk`                     | bool | `False`                 | PPK-korrekciók engedélyezése          |
| `custom_settings`         | dict | `None`                  | Speciális egyéni beállítások        |

**Exportformátumok:**

* `"TIFF (16-bit)"` - Ajánlott GIS/fotogrammetria esetén
* `"TIFF (32-bit, Percent)"` - Tudományos elemzés
* `"PNG (8-bit)"` - Vizuális ellenőrzés
* `"JPG (8-bit)"` - Tömörített kimenet

**Elérhető indexek:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 és még sok más.**Példa:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Feldolgozza a projekt képeit.

**Paraméterek:**

| Paraméter           | Típus     | Alapértelmezett      | Leírás                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Feldolgozási mód: „parallel” vagy „serial”   |
| `wait`              | bool     | `True`       | Várjon a befejezésre                       |
| `progress_callback` | callable | `None`       | Haladás visszahívási függvény (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Haladás lekérdezési intervalluma (másodperc)   |

**Visszatérési érték:** `dict` - Feldolgozási eredmények

{% hint style="warning" %}
**Párhuzamos mód**: Chloros+ licenc szükséges. Automatikusan skálázódik a CPU magjaihoz (legfeljebb 16 munkavégző).
{% endhint %}

**Példa:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

A jelenlegi projektkonfiguráció lekérése.

**Visszatérési érték:** `dict` - Jelenlegi projektkonfiguráció**Példa:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

A háttérállapot információinak lekérése, beleértve a szálankénti feldolgozás előrehaladását.

**Visszatérési érték:** `dict` - A háttér állapota a következő szerkezetben:

```python
{
    "running": True,
    "url": "http://localhost:5000",
    "processing": {
        "percent": 75.0,
        "phase": "processing"
    },
    "export": {
        "percent": 50.0,
        "phase": "exporting",
        "active": True
    }
}
```

**Példa:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

A háttérprogram leállítása (ha az SDK parancs indította el).

**Példa:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

A gyorsítótárban tárolt hitelesítő adatok törlése a helyi rendszerből.

**Leírás:**

Programozási úton jelentkezik ki a gyorsítótárban tárolt hitelesítő adatok eltávolításával. Ez a következő esetekben hasznos:
* Különböző Chloros+ fiókok közötti váltás
* Hitelesítő adatok törlése automatizált környezetekben
* Biztonsági célok (pl. a hitelesítő adatok eltávolítása az eltávolítás előtt)

**Visszatérési érték:** `dict` - A kijelentkezési művelet eredménye**Példa:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style="info" %}
**Újra hitelesítés szükséges**: Az `logout()` hívása után újra be kell jelentkeznie az Chloros, Chloros (böngésző) vagy az Chloros CLI parancsot, mielőtt az SDK parancsot használná.
{% endhint %}

***

### Kényelmi funkciók

#### `process_folder(folder_path, **options)`

Egy soros kényelmi funkció egy mappa feldolgozásához.

**Paraméterek:**

| Paraméter                 | Típus     | Alapértelmezett         | Leírás                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Kötelező        | A képeket tartalmazó mappa elérési útja     |
| `project_name`            | str      | Automatikusan generált  | Projekt neve                   |
| `camera`                  | str      | `None`          | Kamera sablon                |
| `indices`                 | list     | `["NDVI"]`      | Számítandó indexek           |
| `vignette_correction`     | bool     | `True`          | Vignettakorrekció engedélyezése     |
| `reflectance_calibration` | bool     | `True`          | Reflektancia-kalibrálás engedélyezése |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | Kimeneti formátum                  |
| `mode`                    | str      | `"parallel"`    | Feldolgozási mód                |
| `progress_callback`       | hívható | `None`          | Haladás visszahívás              |

**Visszatérési érték:** `dict` - Feldolgozási eredmények**Példa:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Kontextuskezelő támogatás

Az SDK támogatja a kontextuskezelőket az automatikus tisztításhoz:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Teljes példák

{% hint style="info" %}
**Linux felhasználók**: Az alábbi példák mindegyike Windows útvonalakat használ. Cserélje ki az `C:\\...` útvonalakat a saját Linux útvonalaira (pl. `/home/user/...` vagy `~/...`). Az SDK összes funkciója minden platformon azonos.
{% endhint %}

### 1. példa: Alapvető feldolgozás

Mappa feldolgozása alapértelmezett beállításokkal:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### 2. példa: Egyéni munkafolyamat

Teljes ellenőrzés a feldolgozási folyamat felett:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### 3. példa: Több mappa kötegelt feldolgozása

Több repülési adatkészlet feldolgozása:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### 4. példa: Kutatási folyamat integrálása

Az Chloros integrálása az adatelemzéssel:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### 5. példa: Egyéni előrehaladás-figyelés

Fejlett előrehaladás-követés naplózással:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### 6. példa: Hiba kezelése

Robusztus hiba kezelés termelési használatra:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros is installed (Windows installer or Linux .deb package)."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### 7. példa: Fiókkezelés és kijelentkezés

A hitelesítő adatok programozási úton történő kezelése:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### 8. példa: Parancssori eszköz

Egyéni CLI eszköz létrehozása az SDK segítségével:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Használat:**

```bash
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## Kivételkezelés

Az SDK különféle hiba típusokhoz specifikus kivételosztályokat biztosít:

### Kivételhierarchia

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Kivételpéldák

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros is installed (Windows installer or Linux .deb package).")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Haladó témák

### Egyéni háttérkonfiguráció

Egyéni háttérhely vagy konfiguráció használata:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Nem blokkoló feldolgozás

Indítsa el a feldolgozást, és folytassa más feladatokkal:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Memóriakezelés

Nagy adathalmazok esetén kötegenként dolgozza fel az adatokat:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Hibaelhárítás

### A háttérprogram nem indul el

**Probléma:** Az SDK nem tudja elindítani a háttérprogramot**Megoldások:**

1. Ellenőrizze, hogy az Chloros telepítve van-e:

```python
import os
import platform

# Auto-detect backend path
if platform.system() == "Windows":
    backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
else:
    backend_path = "/usr/lib/chloros/chloros-backend"

print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Ellenőrizze a tűzfalat (Windows) vagy a port elérhetőségét (Linux: `lsof -i :5000`)
3. Próbálja meg a háttér elérési útját manuálisan:

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### Licenc nem észlelve**Probléma:** Az SDK figyelmeztet a hiányzó licencre**Megoldások:**

1. Nyissa meg az Chloros, Chloros (böngésző) vagy Chloros CLI alkalmazást, és jelentkezzen be.
2. Ellenőrizze, hogy a licenc tárolva van-e:

```python
from pathlib import Path
import os
import platform

# Check cache location
if platform.system() == "Windows":
    cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
else:
    cache_path = Path.home() / '.cache' / 'chloros'

print(f"Cache exists: {cache_path.exists()}")
```

3. Ha hitelesítési problémák merülnek fel, törölje a gyorsítótárban tárolt hitelesítő adatokat, és jelentkezzen be újra:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. Vegye fel a kapcsolatot az ügyfélszolgálattal: info@mapir.camera

***

### Importálási hibák**Probléma:** `ModuleNotFoundError: No module named 'chloros_sdk'`**Megoldások:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Feldolgozási időtúllépés**Probléma:** A feldolgozás időtúllépése**Megoldások:**

1. Növelje az időtúllépést:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Feldolgozzon kisebb adagokat
3. Ellenőrizze a rendelkezésre álló lemezterületet
4. Figyelje a rendszer erőforrásait

***

### A port már használatban van**Probléma:** A 5000-es háttérport foglalt**Megoldások:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Vagy keresse meg és zárja be az ütköző folyamatot:

```powershell
# Windows PowerShell
Get-NetTCPConnection -LocalPort 5000
```

```bash
# Linux
lsof -i :5000
kill $(lsof -t -i :5000)
```

***

## Teljesítményre vonatkozó tippek

### A feldolgozási sebesség optimalizálása

1. **Párhuzamos mód használata** (Chloros+ verzió szükséges)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **A kimeneti felbontás csökkentése** (ha elfogadható)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Kapcsolja ki a felesleges indexeket**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Feldolgozás SSD-n** (nem HDD-n)***

### Memóriaoptimalizálás

Nagy adathalmazok esetén:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Háttérfeldolgozás

Felszabadítsa az Python-et más feladatokhoz:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Integrációs példák

### Django integráció

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## GYIK

### K: Az SDK internetkapcsolatot igényel?

**V:** Csak az első licencaktiváláshoz. Miután bejelentkezett az Chloros, Chloros (böngésző) vagy Chloros CLI segítségével, a licencet a rendszer helyileg tárolja, és 30 napig offline módban is működik.***

### K: Használhatom az SDK-et GUI nélküli szerveren?**V:** Igen! Az SDK headless módban működik mind az Windows, mind az Linux szervereken.**Linux (headless használatra ajánlott):**
* Telepítés az `.deb` csomag segítségével
* Licenc aktiválása: `chloros-cli login user@example.com 'password'`

**Windows szerver:**
* Windows Server 2016 vagy újabb
* Chloros telepítve (egyszeri)
* Licenc aktiválása az CLI segítségével vagy bármely gépen

***

### K: Mi a különbség a Desktop, az CLI és az SDK között?

| Funkció         | Desktop GUI | CLI Parancssor | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Felület**   | Pont-kattintás | Parancssor          | Python API  |
| **Legalkalmasabb**    | Vizuális munka | Szkriptelés        | Integráció |
| **Automatizálás**  | Korlátozott     | Jó             | Kiváló   |
| **Rugalmasság** | Alapvető       | Jó             | Maximális     |
| **Licenc**     | Chloros+    | Chloros+         | Chloros+    |***

### K: Terjeszthetem az SDK-szel készített alkalmazásokat?**V:** Az SDK kód integrálható az Ön alkalmazásaiba, de:

* A végfelhasználóknak telepítve kell lennie az Chloros-nek
* A végfelhasználóknak aktív Chloros+ licenccel kell rendelkezniük
* A kereskedelmi terjesztéshez OEM-licenc szükséges

OEM-megkeresésekkel kapcsolatban vegye fel a kapcsolatot az info@mapir.camera-szel.

***

### K: Hogyan frissíthetem az SDK-et?

```bash
pip install --upgrade chloros-sdk
```

***

### K: Hová kerülnek a feldolgozott képek?

Alapértelmezés szerint a projekt útvonalába:

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### K: Feldolgozhatok képeket az ütemezés szerint futó Python szkriptekből?**V:** Igen! Használja az operációs rendszer ütemezőjét az Python szkriptekkel:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows:** Ütemezze a Feladatütemezőn keresztül napi futtatásra.**Linux:** Ütemezze cron segítségével:

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### K: Támogatja-e az SDK az async/await funkciót?**V:** A jelenlegi verzió szinkron. Aszinkron viselkedéshez használja az `wait=False`-et, vagy futtassa külön szálban:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### K: Hogyan válthatok a különböző Chloros+ fiókok között?**V:** Használja az `logout()` metódust a gyorsítótárban tárolt hitelesítő adatok törléséhez, majd jelentkezzen be újra az új fiókkal:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

A kijelentkezés után hitelesítse magát az új fiókkal a grafikus felhasználói felületen, böngészőben vagy az CLI segítségével, mielőtt újra használná az SDK-et.

***

## Segítség

### Dokumentáció

* **API Referencia**: Ez az oldal

### Támogatási csatornák

* **E-mail**: info@mapir.camera
* **Weboldal**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Árak**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Példakód

Az itt felsorolt összes példa tesztelt és termeléskész. Másolja át és igazítsa saját felhasználási esetéhez.

***

## Licenc**Saját fejlesztésű szoftver** - Copyright (c) 2025 MAPIR Inc.

Az SDK használatához aktív Chloros+ előfizetés szükséges. A jogosulatlan használat, terjesztés vagy módosítás tilos.
