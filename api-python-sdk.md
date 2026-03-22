# API : Python SDK

**Chloros Python SDK** осигурява програмно достъп до Chloros двигателя за обработка на изображения, позволявайки автоматизация, персонализирани работни потоци и безпроблемна интеграция с вашите Python приложения и изследователски процеси.

### Ключови характеристики

* 🐍 **Нативен Python** - Чист, Pythonic API за обработка на изображения
* 🔧 **Пълен достъп до API** - Пълен контрол върху обработката на Chloros
* 🚀 **Автоматизация** - Създавайте персонализирани работни потоци за пакетна обработка
* 🔗 **Интеграция** - Вградете Chloros в съществуващи Python приложения
* 📊 **Готов за научни изследвания** - Идеален за научни аналитични процеси
* ⚡ **Паралелна обработка** - Мащабира се според ядрата на вашия процесор (Chloros+)

### Изисквания

| Изискване          | Подробности                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros инсталиран** | Windows: Инсталатор за настолни компютри; Linux: `.deb` пакет                  |
| **Лиценз**          | Chloros+ ([изисква се платен план](https://cloud.mapir.camera/pricing)) |
| **Операционна система** | Windows 10/11 (64-битова), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python**           | Python 3.7 или по-нова версия                                                |
| **Памет**           | Минимум 8 GB RAM (препоръчителни 16 GB)                                  |
| **Интернет**         | Необходим за активиране на лиценза                                     |

{% hint style="warning" %}
**Изисквания за лиценз**: Python SDK изисква платен абонамент за Chloros+ за достъп до API. Стандартните (безплатни) планове не включват достъп до API/SDK. Посетете [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing), за да надстроите.
{% endhint %}

## Бързо стартиране

### Инсталиране

Инсталирайте чрез pip:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**Първоначална настройка**: Преди да използвате SDK, активирайте лиценза си за Chloros+ като отворите Chloros, Chloros (браузър) или Chloros CLI и влизайки с вашите данни за достъп. Това трябва да се направи само веднъж. В Linux (без GUI) използвайте: `chloros-cli login user@example.com 'password'`
{% endhint %}

### Основна употреба

Обработете папка само с няколко реда:

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**Междуплатформени пътища**: Примерите за код на тази страница използват пътища в стил Windows (например, `C:\\DroneImages\\Flight001`). В Linux използвайте вместо това пътеки от типа Linux (например `/home/user/drone_images/flight001` или `~/drone_images/flight001`). SDK работи по същия начин и на двете платформи.
{% endhint %}

### Пълен контрол

За разширени работни процеси:

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

## Ръководство за инсталиране

### Предварителни условия

Преди да инсталирате SDK, уверете се, че разполагате с:

1. **Chloros инсталиран** — Windows: Инсталатор за настолни компютри ([изтегляне](download.md)); Linux: Пакет `.deb` ([Инсталиране на Linux](linux/linux-installation.md))
2. **Python 3.7+** инсталиран ([python.org](https://www.python.org))
3. **Активен лиценз за Chloros+** ([ъпгрейд](https://cloud.mapir.camera/pricing))

### Инсталиране чрез pip

**Стандартна инсталация:**

```bash
pip install chloros-sdk
```

**С поддръжка за наблюдение на напредъка:**

```bash
pip install chloros-sdk[progress]
```

**Инсталиране за разработка:**

```bash
pip install chloros-sdk[dev]
```

### Проверка на инсталирането

Проверете дали SDK е инсталиран правилно:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Първоначална настройка

### Активиране на лиценза

SDK използва същия лиценз като Chloros, Chloros (браузър) и Chloros CLI. Активирайте веднъж чрез графичния интерфейс или CLI:**Windows:**Отворете**Chloros или Chloros (браузър)** и влезте в раздела „Потребител“ <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> или използвайте CLI.**Linux:** Използвайте CLI (няма GUI):

```bash
chloros-cli login user@example.com 'your_password'
```

Лицензът се кешира локално и се запазва при рестартиране.

{% hint style="success" %}
**Еднократна настройка**: След влизане чрез GUI или CLI, SDK автоматично използва кеширания лиценз. Не се изисква допълнителна автентификация!
{% endhint %}

{% hint style="info" %}
**Излизане**: Потребителите на SDK могат програмно да изчистят кешираните идентификационни данни, като използват метода `logout()`. Вижте [метода logout()](#logout) в справочника за API.
{% endhint %}

### Тест на връзката

Проверете дали SDK може да се свърже с Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## Справочник за API

### Клас ChlorosLocal

Основен клас за локална обработка на изображения Chloros.

#### Конструктор

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Параметри:**

| Параметър                 | Тип | По подразбиране                   | Описание                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL на локалния Chloros бекенд          |
| `auto_start_backend`      | bool | `True`                    | Автоматично стартиране на бекенда, ако е необходимо |
| `backend_exe`             | str  | `None` (автоматично откриване)      | Път към изпълнимия файл на бекенда            |
| `timeout`                 | int  | `30`                      | Тайм-аут на заявката в секунди            |
| `backend_startup_timeout` | int  | `60`                      | Тайм-аут за стартиране на бекенда (секунди) |

**Примери:**

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
**Автоматично откриване на платформата**: SDK автоматично пробва правилния път към бекенда за вашата платформа:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (ръчно)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### Методи

#### `create_project(project_name, camera=None)`

Създаване на нов проект Chloros.

**Параметри:**

| Параметър      | Тип | Задължително | Описание                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Да      | Име на проекта                                     |
| `camera`       | str  | Не       | Шаблон на камерата (например, &quot;Survey3N\_RGN&quot;, „Survey3W\_OCN“) |

**Връща:** `dict` – Отговор за създаване на проект**Пример:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Импортиране на изображения от папка.

**Параметри:**

| Параметър     | Тип     | Задължително | Описание                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Да      | Път към папка с изображения         |
| `recursive`   | bool     | Не       | Търсене в подпапки (по подразбиране: False) |

**Връща:** `dict` - Резултати от импорта с брой файлове**Пример:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Конфигуриране на настройките за обработка.

**Параметри:**

| Параметър                 | Тип | По подразбиране                 | Описание                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | &quot;Стандартен (Бърз, Средно качество)&quot; | Метод за дебайериране            |
| `vignette_correction`     | bool | `True`                  | Активиране на корекция на винетиране      |
| `reflectance_calibration` | bool | `True`                  | Активиране на калибриране на отражателната способност  |
| `indices`                 | списък | `None`                  | Индекси на растителността за изчисляване |
| `export_format`           | стр.  | &quot;TIFF (16-bit)&quot;         | Формат на изхода                   |
| `ppk`                     | bool | `False`                 | Активиране на PPK корекции          |
| `custom_settings`         | dict | `None`                  | Разширени потребителски настройки        |

**Формати за експортиране:**

* `"TIFF (16-bit)"` - Препоръчва се за ГИС/фотограметрия
* `"TIFF (32-bit, Percent)"` - Научен анализ
* `"PNG (8-bit)"` - Визуална инспекция
* `"JPG (8-bit)"` - Компресиран изход

**Налични индекси:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 и др.**Пример:**

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

Обработка на изображенията от проекта.

**Параметри:**

| Параметър           | Тип     | По подразбиране      | Описание                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Режим на обработка: &quot;parallel&quot; или &quot;serial&quot;   |
| `wait`              | bool     | `True`       | Изчакване на завършване                       |
| `progress_callback` | callable | `None`       | Функция за обратно извикване за напредък (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Интервал на проверка за напредък (секунди)   |

**Връща:** `dict` - Резултати от обработката

{% hint style="warning" %}
**Паралелен режим**: Изисква лиценз Chloros+. Автоматично се мащабира според ядрата на вашия процесор (до 16 работници).
{% endhint %}

**Пример:**

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

Получаване на текущата конфигурация на проекта.

**Връща:** `dict` - Текуща конфигурация на проекта**Пример:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Получаване на информация за състоянието на бекенда, включително напредъка на обработката за всеки поток.

**Връща:** `dict` - Статус на бекенда със следната структура:

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

**Пример:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

Изключване на бекенда (ако е стартиран от SDK).

**Пример:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

Изчистване на кешираните удостоверения от локалната система.

**Описание:**

Излизане от системата чрез програмиране чрез премахване на кешираните удостоверения за автентификация. Това е полезно за:
* Превключване между различни Chloros+ акаунти
* Изчистване на удостоверенията в автоматизирани среди
* Цели, свързани със сигурността (например, премахване на удостоверенията преди деинсталиране)

**Връща:** `dict` - Резултат от операцията по излизане**Пример:**

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
**Необходима повторна автентификация**: След извикване на `logout()`, трябва да влезете отново чрез Chloros, Chloros (браузър) или Chloros CLI, преди да използвате SDK.
{% endhint %}

***

### Удобни функции

#### `process_folder(folder_path, **options)`

Удобна функция от един ред за обработка на папка.

**Параметри:**

| Параметър                 | Тип     | По подразбиране         | Описание                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Задължително        | Път към папката с изображения     |
| `project_name`            | str      | Генерира се автоматично  | Име на проекта                   |
| `camera`                  | str      | `None`          | Шаблон на камерата                |
| `indices`                 | списък     | `["NDVI"]`      | Индекси за изчисление           |
| `vignette_correction`     | bool     | `True`          | Активиране на корекция на винетиране     |
| `reflectance_calibration` | bool     | `True`          | Активиране на калибриране на отражателната способност |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | Формат на изхода                  |
| `mode`                    | str      | `"parallel"`    | Режим на обработка                |
| `progress_callback`       | callable | `None`          | Обратно извикване за напредък              |

**Връща:** `dict` - Резултати от обработката**Пример:**

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

## Поддръжка на мениджъри на контекст

SDK поддържа мениджъри на контекст за автоматично почистване:

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

## Пълни примери

{% hint style="info" %}
**Потребители на Linux**: Всички примери по-долу използват Windows пътеки. Заменете `C:\\...` пътищата с вашите Linux пътища (например, `/home/user/...` или `~/...`). Всички SDK функционалности са идентични на всички платформи.
{% endhint %}

### Пример 1: Основна обработка

Обработка на папка с настройки по подразбиране:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Пример 2: Персонализиран работен поток

Пълен контрол върху процеса на обработка:

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

### Пример 3: Пакетна обработка на няколко папки

Обработка на множество набори от данни за полети:

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

### Пример 4: Интеграция в изследователски процес

Интегриране на Chloros с анализ на данни:

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

### Пример 5: Персонализирано проследяване на напредъка

Разширено проследяване на напредъка с регистриране:

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

### Пример 6: Обработка на грешки

Надеждна обработка на грешки за производствена употреба:

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

### Пример 7: Управление на акаунта и излизане

Управление на удостоверенията чрез програмиране:

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

### Пример 8: Инструмент за командния ред

Създаване на персонализиран инструмент CLI с SDK:

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

**Употреба:**

```bash
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## Обработка на изключения

SDK предоставя специфични класове изключения за различни типове грешки:

### Йерархия на изключенията

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Примери за изключения

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

## Разширени теми

### Персонализирана конфигурация на бекенда

Използвайте персонализирано местоположение или конфигурация на бекенда:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Неблокираща обработка

Започнете обработката и продължете с други задачи:

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

### Управление на паметта

За големи масиви от данни обработвайте на партиди:

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

## Отстраняване на проблеми

### Бекендът не стартира

**Проблем:** SDK не успява да стартира бекенда**Решения:**

1. Проверете дали Chloros е инсталиран:

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

2. Проверете защитната стена (Windows) или достъпността на порта (Linux: `lsof -i :5000`)
3. Опитайте ръчен път към бекенда:

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### Лицензът не е открит**Проблем:** SDK предупреждава за липсващ лиценз**Решения:**

1. Отворете Chloros, Chloros (браузър) или Chloros CLI и влезте в системата.
2. Проверете дали лицензът е запазен в кеша:

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

3. Ако имате проблеми с данните за достъп, изчистете кешираните данни за достъп и се влезте отново:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. Свържете се с поддръжката: info@mapir.camera

***

### Грешки при импортиране**Проблем:** `ModuleNotFoundError: No module named 'chloros_sdk'`**Решения:**

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

### Изтичане на времето за обработка**Проблем:** Изтичане на времето за обработка**Решения:**

1. Увеличете времето за изчакване:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Обработвайте по-малки партиди
3. Проверете наличното дисково пространство
4. Наблюдавайте системните ресурси

***

### Портът вече се използва**Проблем:** Порт 5000 на бекенда е зает**Решения:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Или намерете и затворете конфликтния процес:

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

## Съвети за производителност

### Оптимизирайте скоростта на обработка

1. **Използвайте паралелен режим** (изисква Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Намалете резолюцията на изхода** (ако е приемливо)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Деактивирайте ненужните индекси**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Извършвайте обработката на SSD** (не на HDD)***

### Оптимизация на паметта

За големи масиви от данни:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Обработка на заден план

Освободете Python за други задачи:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Примери за интеграция

### Интеграция с Django

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

## Често задавани въпроси

### В: Изисква ли SDK интернет връзка?

**О:** Само за първоначалната активация на лиценза. След влизане чрез Chloros, Chloros (браузър) или Chloros CLI лицензът се запазва локално и работи офлайн в продължение на 30 дни.***

### В: Мога ли да използвам SDK на сървър без графичен интерфейс?**О:** Да! SDK работи без графичен интерфейс както на Windows, така и на Linux сървъри.**Linux (препоръчва се за работа без графичен интерфейс):**
* Инсталирайте чрез пакета `.deb`
* Активирайте лиценза: `chloros-cli login user@example.com 'password'`

**Windows сървър:**
* Windows сървър 2016 или по-нова версия
* Chloros инсталиран (еднократно)
* Лиценз активиран чрез CLI или на която и да е машина

***

### В: Каква е разликата между Desktop, CLI и SDK?

| Функция         | Desktop GUI | CLI Command Line | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Интерфейс**   | Посочи и кликни | Команда          | Python API  |
| **Най-подходящ за**    | Визуална работа | Скриптове        | Интеграция |
| **Автоматизация**  | Ограничена     | Добра             | Отлична   |
| **Гъвкавост** | Основна       | Добра             | Максимална     |
| **Лиценз**     | Chloros+    | Chloros+         | Chloros+    |***

### В: Мога ли да разпространявам приложения, създадени с SDK?**О:** Кодът на SDK може да бъде интегриран във вашите приложения, но:

* Крайните потребители трябва да имат инсталиран Chloros
* Крайните потребители трябва да имат активни лицензи за Chloros+
* Търговското разпространение изисква OEM лицензиране

Свържете се с info@mapir.camera за запитвания относно OEM.

***

### В: Как да актуализирам SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### В: Къде се запазват обработените изображения?

По подразбиране, в пътя на проекта:

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### В: Мога ли да обработвам изображения от скриптове на Python, изпълнявани по график?**О:** Да! Използвайте планиращия модул на операционната си система със скриптове на Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows:** Настройте чрез Планиращ модул да се изпълнява ежедневно.**Linux:** Настройте чрез cron:

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### В: Поддържа ли SDK async/await?**О:** Текущата версия е синхронна. За асинхронно поведение използвайте `wait=False` или изпълнете в отделен поток:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### В: Как да превключвам между различни акаунти Chloros+?**О:** Използвайте метода `logout()`, за да изчистите кешираните идентификационни данни, след което се логнете отново с новия акаунт:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

След излизане от системата се удостоверете с новия акаунт чрез GUI, браузър или CLI, преди да използвате отново SDK.

***

## Получаване на помощ

### Документация

* **API Справочник**: Тази страница

### Канали за поддръжка

* **Имейл**: info@mapir.camera
* **Уебсайт**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Цени**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Примерни кодове

Всички примери, изброени тук, са тествани и готови за употреба. Копирайте ги и ги адаптирайте за вашия случай на употреба.

***

## Лиценз**Собствен софтуер** - Авторски права (c) 2025 MAPIR Inc.

SDK изисква активен абонамент за Chloros+. Неоторизираното използване, разпространение или модифициране е забранено.
