# CLI : Командна линия

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** предоставя мощен достъп чрез командния ред до двигателя за обработка на изображения Chloros, като позволява автоматизация, създаване на скриптове и работа без графичен интерфейс за вашите работни потоци за обработка на изображения.

### Основни характеристики

* 🚀 **Автоматизация** - Скриптова пакетна обработка на множество набори от данни
* 🔗 **Интеграция** - Вграждане в съществуващи работни потоци и пипалини
* 💻 **Работа без графичен интерфейс** - Изпълнение без GUI
* 🌍 **Многоезичност** - Поддръжка на 38 езика
* ⚡ **Паралелна обработка** - [Динамична адаптация на изчислителната мощност](processing-architecture/dynamic-compute-adaptation.md) автоматично се оптимизира за вашия хардуер

### Изисквания

| Изискване          | Подробности                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Операционна система** | Windows 10/11 (64-bit), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Лиценз**          | Chloros+ ([изисква се платен план](https://cloud.mapir.camera/pricing)) |
| **Памет**           | Минимум 8 GB RAM (препоръчват се 16 GB)                                  |
| **Интернет**         | Необходим за активиране на лиценза                                     |
| **Дисково пространство**       | Зависи от размера на проекта                                              |

{% hint style="warning" %}
**Изисквания за лиценз**: CLI изисква платен абонамент за Chloros+. Стандартните (безплатни) планове нямат достъп до CLI. Посетете [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing), за да надстроите.
{% endhint %}

## Бързо стартиране

### Инсталиране

#### Windows

CLI е автоматично включен в инсталатора на Chloros:

1. Изтеглете и стартирайте **Chloros Installer.exe**

2. Изпълнете инструкциите на инсталационния помощник
3. CLI е инсталиран в: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Инсталаторът автоматично добавя `chloros-cli` към системния PATH. Рестартирайте терминала си след инсталацията.
{% endhint %}

#### Linux

Инсталирайте пакета `.deb` за вашата архитектура:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

За подробна настройка на Linux вижте [Инсталиране на Linux](linux/linux-installation.md).

### Първоначална настройка

Преди да използвате CLI, активирайте лиценза си за Chloros+:

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

### Основна употреба

Обработка на папка с настройки по подразбиране:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Справочник за команди

### Общ синтаксис

```
chloros-cli [global-options] <command> [command-options]
```

***

## Команди

### `process` - Обработка на изображения

Обработка на изображения в папка с калибриране.

**Синтаксис:**

```bash
chloros-cli process <input-folder> [options]
```

**Примери:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Опции на командата за обработка

| Опция                | Тип    | По подразбиране        | Описание                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Път    | _Задължително_     | Папка, съдържаща мултиспектрални изображения във формат RAW/JPG                                         |
| `-o, --output`        | Път    | Същото като входните  | Папка за изходните обработени изображения                                                     |
| `-n, --project-name`  | Строка  | Автоматично генерирано | Персонализирано име на проекта                                                                    |
| `--vignette`          | Флаг    | Активирано        | Активиране на корекция на винетиране                                                             |
| `--no-vignette`       | Флаг    | -              | Деактивиране на корекция на винетиране                                                            |
| `--reflectance`       | Флаг    | Активирано        | Активиране на калибриране на отражателната способност                                                         |
| `--no-reflectance`    | Флаг    | -              | Деактивиране на калибриране на отражателната способност                                                        |
| `--ppk`               | Флаг    | Деактивирано       | Прилагане на PPK корекции от данни на светлинния сензор .daq                                      |
| `--format`            | Избор  | TIFF (16-битов)  | Формат на изхода: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Цело число | Автоматично           | Минимален размер на целта в пиксели за откриване на калибрационния панел                          |
| `--target-clustering` | Цело число | Автоматично           | Праг за групиране на целите (0-100)                                                    |
| `--debayer`           | Избор  | `standard`     | Метод за дебайеризация: `standard` или `texture-aware` (само за Chloros+)                          |
| `--target`, `--targets` | Флаг  | Деактивирано       | Търсене само на калибрационни цели в подпапка „target“ или „targets“ (ускорява обработката) |
| `--indices`           | Списък    | Няма           | Индекси на растителността за изчисляване (например, `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Строка  | Няма           | Заключване на експозицията за модел на камера (Pin 1)                                                 |
| `--exposure-pin-2`    | Строка  | Няма           | Заключване на експозицията за модел на камера (Pin 2)                                                 |
| `--recal-interval`    | Цело число | Автоматично           | Интервал на прекалибриране в секунди                                                      |
| `--timezone-offset`   | Цело число | 0              | Разлика в часовата зона в часове                                                               |

***

### `login` - Удостоверяване на акаунт

Влезте с вашите Chloros+ идентификационни данни, за да активирате обработката на CLI.

**Синтаксис:**

```bash
chloros-cli login <email> <password>
```

**Пример:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Специални символи**: Използвайте единични кавички около пароли, съдържащи символи като `$`, `!` или интервали.
{% endhint %}

**Резултат:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Изчистване на удостоверенията

Изчистване на съхранените удостоверения и излизане от профила ви.

**Синтаксис:**

```bash
chloros-cli logout
```

**Пример:**

```bash
chloros-cli logout
```

**Резултат:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**Потребители на SDK**: Python SDK също предоставя програмна метода `logout()` за изчистване на удостоверенията в скриптове Python. Вижте [Python SDK документацията](api-python-sdk.md#logout) за подробности.
{% endhint %}

***

### `status` - Проверка на състоянието на лиценза

Показване на текущото състояние на лиценза и удостоверяването.

**Синтаксис:**

```bash
chloros-cli status
```

**Пример:**

```bash
chloros-cli status
```

**Резултат:**

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

### `export-status` - Проверка на напредъка на експорта

Наблюдавайте напредъка на експорта на Thread 4 по време на или след обработката.

**Синтаксис:**

```bash
chloros-cli export-status
```

**Пример:**

```bash
chloros-cli export-status
```

**Приложение:** Използвайте тази команда по време на обработката, за да проверите напредъка на експорта.***

### `language` - Управление на езика на интерфейса

Преглед или промяна на езика на интерфейса на CLI.

**Синтаксис:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Примери:**

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

#### Поддържани езици (общо 38)

| Код    | Език              | Име на езика      |
| ------- | --------------------- | ---------------- |
| `en`    | Английски               | English          |
| `es`    | Испански               | Español          |
| `pt`    | Португалски            | Português        |
| `fr`    | Френски                | Français         |
| `de`    | Немски                | Deutsch          |
| `it`    | Италиански               | Italiano         |
| `ja`    | Японски              | 日本語              |
| `ko`    | Корейски                | 한국어              |
| `zh`    | Китайски (опростен)  | 简体中文             |
| `zh-TW` | Китайски (традиционен) | 繁體中文             |
| `ru`    | Руски               | Русский          |
| `nl`    | Холандски                | Nederlands       |
| `ar`    | Арабски                | العربية          |
| `pl`    | Полски                | Polski           |
| `tr`    | Турски               | Türkçe           |
| `hi`    | Хинди                 | हिंदी            |
| `id`    | Индонезийски            | Bahasa Indonesia |
| `vi`    | Виетнамски            | Tiếng Việt       |
| `th`    | Тайски                  | ไทย              |
| `sv`    | Шведски               | Svenska          |
| `da`    | Датски                | Dansk            |
| `no`    | Норвежки             | Norsk            |
| `fi`    | Фински               | Suomi            |
| `el`    | Гръцки                 | Ελληνικά         |
| `cs`    | Чешки                 | Čeština          |
| `hu`    | Унгарски             | Magyar           |
| `ro`    | Румънски              | Română           |
| `uk`    | Украински             | Українська       |
| `pt-BR` | Бразилски португалски  | Português Brasileiro |
| `zh-HK` | Кантонски             | 粵語             |
| `ms`    | Малайски                 | Bahasa Melayu    |
| `sk`    | Словашки                | Slovenčina       |
| `bg`    | Български             | Български        |
| `hr`    | Хърватски              | Hrvatski         |
| `lt`    | Литовски            | Lietuvių         |
| `lv`    | Латвийски               | Latviešu         |
| `et`    | Естонски              | Eesti            |
| `sl`    | Словенски             | Slovenščina      |

{% hint style="success" %}
**Автоматично запазване**: Вашите езикови настройки се запазват в `~/.chloros/cli_language.json` и се запазват през всички сесии.
{% endhint %}

***

### `set-project-folder` - Задаване на папка по подразбиране за проекти

Промяна на местоположението на папката по подразбиране за проекти (споделена с GUI в Windows).

**Синтаксис:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Примери:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - Показване на папката с проекти

Показване на текущото местоположение на папката с проекти по подразбиране.

**Синтаксис:**

```bash
chloros-cli get-project-folder
```

**Пример:**

```bash
chloros-cli get-project-folder
```

**Резултат:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` - Възстановяване на настройките по подразбиране

Възстановява местоположението на папката с проекти към местоположението по подразбиране.

**Синтаксис:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` - Изпълнение на системна диагностика

Изпълнява 7 диагностични проверки за потвърждаване на конфигурацията на системата.

**Синтаксис:**

```bash
chloros-cli selftest
```

**Извършени диагностични проверки:**

1. Проверка на версията
2. Наличност на порт (5000)
3. Стартиране на бекенда
4. Тест за свързаност API
5. Информация за системата и откриване на GPU
6. Проверка на моделите за отстраняване на шума
7. Проверка за наличност на CUDA

{% hint style="info" %}
**Полезно за отстраняване на проблеми**: Изпълнете `selftest` след инсталирането, за да проверите дали системата ви е конфигурирана правилно, особено на Linux/Jetson, където може да е необходима проверка на настройките на GPU и CUDA.
{% endhint %}

***

### `update` - Проверка за актуализации (само за Linux)

Проверете за актуализации на CLI и ги инсталирайте на системи Linux.

**Синтаксис:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Опция    | Описание                        |
| --------- | ---------------------------------- |
| `--check` | Само проверка за актуализации, без инсталиране |

{% hint style="info" %}
Тази команда е достъпна само на Linux. На Windows актуализациите се доставят чрез инсталатора.
{% endhint %}

***

## Глобални опции

Тези опции важат за всички команди:

| Опция            | Тип    | По подразбиране       | Описание                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Път    | Автоматично открит | Път към изпълнимия файл на бекенда                       |
| `--port`          | Цело число | 5000          | Номер на порт на бекенда API                          |
| `--restart`       | Флаг    | -             | Принудително рестартиране на бекенда (убива съществуващите процеси) |
| `--version`       | Флаг    | -             | Показване на информация за версията и изход                |
| `--help`          | Флаг    | -             | Показване на информация за помощта и изход                   |

{% hint style="info" %}
**Автоматично откриване на бекенда**: Пътят `--backend-exe` се открива автоматично за всяка платформа:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (ръчно)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Пример с глобални опции:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Ръководство за настройките на обработката

### Паралелна обработка и динамично адаптиране на изчисленията

Chloros 1.1.0 включва [Динамично адаптиране на изчисленията](processing-architecture/dynamic-compute-adaptation.md) — обработващият модул **автоматично открива вашия хардуер** и избира оптималната стратегия:

| Платформа | Стратегия | Работни процеси | Поток | Бележки |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Ефективно използване на паметта, сериализирано |
| **Jetson Orin NX 16GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Едновременна обработка на GPU |
| **Настолен компютър с 8 GB GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | Добра производителност на настолния компютър |
| **Настолен компютър с 12 GB+ GPU** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Оптимална производителност на настолния компютър |
| **Система само с CPU** | `CPU_PARALLEL` | ядра - 1 | `cpu_fallback` | Не се изисква GPU |

{% hint style="success" %}
**Не се изисква ръчна конфигурация!** Chloros автоматично открива вашия CPU, GPU, RAM и (на Jetson) термични сензори, след което автоматично конфигурира оптималния процесор.
{% endhint %}

### Методи за дебайериране

| Метод | CLI Флаг | Качество | Скорост | Лиценз |
| --- | --- | --- | --- | --- |
| **Стандартен (Бърз, Средно качество)** | `--debayer standard` | Добро | Бърз | Безплатен / Chloros+ |
| **С отчитане на текстурата (бавен, най-високо качество)** | `--debayer texture-aware` | Най-високо | Бавен | Само Chloros+ |

По подразбиране методът за дебайериране е **Стандартен**. Методът**С чувствителност към текстурата** използва AI/ML модел за отстраняване на шума за най-високо качество на изхода, но изисква лиценз Chloros+ и NVIDIA GPU.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Корекция на винетиране

**Какво прави:** Коригира загубата на светлина по краищата на изображението (по-тъмните ъгли, често срещани в камерните изображения).

* **Активирано по подразбиране** – Повечето потребители трябва да го оставят активирано
* Използвайте `--no-vignette`, за да го деактивирате

{% hint style="success" %}
**Препоръка**: Винаги активирайте корекцията на винетирането, за да осигурите равномерна яркост в целия кадър.
{% endhint %}

### Калибриране на отражателната способност

Преобразува суровите стойности на сензора в стандартизирани проценти на отражателната способност, като използва калибрационни панели.

* **Активирано по подразбиране** - Необходимо за анализ на растителността
* Изисква калибрационни панели в изображенията
* Използвайте `--no-reflectance`, за да деактивирате

{% hint style="info" %}
**Изисквания**: Уверете се, че калибрационните панели са правилно експонирани и видими във вашите изображения за точно преобразуване на отражателната способност.
{% endhint %}

### PPK корекции

**Какво прави:** Прилага пост-обработени кинематични корекции, използвайки данни от DAQ-A-SD за подобрена GPS точност.

* **Деактивирано по подразбиране**
* Използвайте `--ppk` за активиране
* Изисква .daq файлове в папката на проекта от MAPIR DAQ-A-SD светлинен сензор.

### Формати на изхода

<table><thead><tr><th width="197">Формат</th><th width="130.20001220703125">Битна дълбочина</th><th width="116.5999755859375">Размер на файла</th><th>Най-подходящ за</th></tr></thead><tbody><tr><td><strong>TIFF (16-битов)</strong> ⭐</td><td>16-битово цяло число</td><td>Голям</td><td>ГИС анализ, фотограметрия (препоръчително)</td></tr><tr><td><strong>TIFF (32-битов, процент)</strong></td><td>32-битова плаваща запетая</td><td>Много голям</td><td>Научен анализ, изследвания</td></tr><tr><td><strong>PNG (8-битов)</strong></td><td>8-битово цяло число</td><td>Средно</td><td>Визуална проверка, споделяне в интернет</td></tr><tr><td><strong>JPG (8-битово)</strong></td><td>8-битово цяло число</td><td>Малък</td><td>Бърз преглед, компресиран изход</td></tr></tbody></table>***

## Автоматизация и скриптове

### Пакетна обработка в PowerShell (Windows)

Автоматична обработка на множество папки с набори от данни в Windows:

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

### Скрипт за пакетна обработка Windows (Windows)

Обикновена циклична конструкция за пакетна обработка в Windows:

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

### Батова обработка в Bash (Linux)

Обработка на множество папки с набори от данни в Linux:

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

### Скрипт за автоматизация на Python (мултиплатформен)

Разширена автоматизация с обработка на грешки (работи на Windows и Linux):

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

## Работен поток на обработката

### Стандартен работен поток

1. **Вход**: Папка, съдържаща двойки RAW/JPG изображения
2. **Откриване**: CLI автоматично сканира за поддържани файлове с изображения
3. **Обработка**: Паралелен режим, мащабируем според ядрата на вашия процесор (Chloros+)
4. **Изход**: Създава подпапки за моделите на камерите с обработените изображения

### Примерна структура на изхода

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

### Оценки за времето за обработка

Типично време за обработка на 100 изображения (по 12 MP всяко):

| Платформа | Режим | Очаквано време | Забележки |
| --- | --- | --- | --- |
| **Настолен компютър с 12 GB+ GPU** | `GPU_PARALLEL` | 5–10 мин. | Най-бързият вариант |
| **Настолен компютър с 8 GB GPU** | `GPU_SINGLE` | 10–15 мин. | Добра производителност |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 15–25 мин. | Изчисления на крайни устройства |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 30–60 мин. | Ограничена памет |
| **Само CPU** | `CPU_PARALLEL` | 20-40 мин | Не се изисква GPU |

{% hint style="info" %}
**Съвет за производителност**: Времето за обработка варира в зависимост от броя на изображенията, резолюцията, метода за дебайериране и хардуера. Дебайерирането с отчитане на текстурата отнема значително повече време от стандартното. Вижте [Динамична адаптация на изчисленията](processing-architecture/dynamic-compute-adaptation.md) за подробности.
{% endhint %}

***

## Отстраняване на проблеми

### CLI Не е намерен

**Windows Грешка:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Решения:**

1. Проверете мястото на инсталиране:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Използвайте пълен път, ако не е в PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Добавете ръчно към PATH:
   * Отворете „Свойства на системата“ → „Променливи на средата“
   * Редактирайте променливата PATH
   * Добавете: `C:\Program Files\Chloros\resources\cli`
   * Рестартирайте терминала

**Linux Грешка:**

```
chloros-cli: command not found
```

**Linux Решения:**

1. Проверете инсталацията:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Презаредете вашата shell:

```bash
source ~/.bashrc
```

3. Проверете правата:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Неуспешен старт на бекенда**Грешка:**

```

Backend failed to start within 30 seconds
```

**Решения:**

1. Проверете дали бекендът вече работи (затворете го първо)
2. Проверете дали защитната стена не блокира (Windows) или проверете достъпността на порта (Linux: `lsof -i :5000`)
3. Опитайте с друг порт:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Принудително рестартирайте бекенда:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. На Linux проверете дали изпълнимият файл на бекенда съществува:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Проблеми с лиценза / удостоверяването**Грешка:**

```

Chloros+ license required for CLI access
```

**Решения:**

1. Уверете се, че имате активен абонамент за Chloros+
2. Влезте с вашите данни за достъп:

```bash
chloros-cli login user@example.com 'password'
```

3. Проверете състоянието на лиценза:

```bash
chloros-cli status
```

4. Свържете се с отдела за поддръжка: info@mapir.camera

***

### Не са намерени изображения**Грешка:**

```

No images found in the specified folder
```

**Решения:**

1. Уверете се, че папката съдържа поддържани формати (.RAW, .TIF, .JPG)
2. Проверете дали пътят към папката е правилен (използвайте кавички за пътища с интервали)
3. Уверете се, че имате права за четене на папката
4. Проверете дали разширенията на файловете са правилни

***

### Обработката спира или забива**Решения:**

1. Проверете наличното дисково пространство (уверете се, че е достатъчно за изхода)
2. Затворете други приложения, за да освободите памет
3. Намалете броя на изображенията (обработвайте на партиди)

***

### Портът вече се използва**Грешка:**

```

Port 5000 is already in use
```

**Решения:**

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

## Често задавани въпроси

### В: Нужна ли ми е лицензия за CLI?

**О:**Да! За CLI е необходима платена**лицензия Chloros+**.

* ❌ Стандартен (безплатен) план: CLI е деактивиран
* ✅ Chloros+ (платен) план: CLI е напълно активиран

Абонирайте се на: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### В: Мога ли да използвам CLI на сървър без GUI?**О:** Да! CLI работи изцяло без графичен интерфейс. Това е основният случай на употреба на Linux.**Windows сървър:**
* Windows сървър 2016 или по-нова версия
* Инсталиран Visual C++ Redistributable

**Linux сървър:**
* Ubuntu 20.04+ / Debian 11+ (amd64) или JetPack 6 (arm64)
* Инсталирайте чрез пакета `.deb`

**И двете платформи:**
* Минимум 8 GB RAM (препоръчват се 16 GB)
* Еднократна активация на лиценза: `chloros-cli login user@example.com 'password'`

***

### В: Къде се запазват обработените изображения?**О:**По подразбиране обработените изображения се запазват в**същата папка като входните** в подпапки за моделите на камерите (например `Survey3N_RGN/`).

Използвайте опцията `-o`, за да зададете друга папка за изходни файлове:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### В: Мога ли да обработвам няколко папки едновременно?**О:** Не директно с една команда, но можете да използвате скриптове, за да обработвате папките последователно. Вижте раздела [Автоматизация и скриптове](CLI.md#automation--scripting).***

### В: Как да запазя изхода на CLI в лог файл?**PowerShell:**

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

### В: Какво се случва, ако натисна Ctrl+C по време на обработката?**О:** CLI ще:

1. Спре обработката плавно
2. Затвори бекенда
3. Излезе с код 130

Частично обработени изображения може да останат в папката с резултатите.

***

### В: Мога ли да автоматизирам обработката с CLI?**О:** Разбира се! CLI е проектиран за автоматизация. Вижте [Автоматизация и скриптове](CLI.md#automation--scripting) за PowerShell (Windows), Batch (Windows), Bash (Linux) и Python (мултиплатформен).***

### В: Как да проверя версията на CLI?**О:**

```bash
chloros-cli --version
```

**Резултат:**

```

Chloros CLI 1.1.0
```

***

## Получаване на помощ

### Помощ за командния ред

Прегледайте информацията за помощ директно в CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Канали за поддръжка

* **Имейл**: info@mapir.camera
* **Уебсайт**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Цени**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Пълни примери

### Пример 1: Основна обработка

Обработка с настройки по подразбиране (винет, отражателна способност):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Пример 2: Висококачествен научен резултат

32-битов плаващ TIFF:

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

### Пример 3: Бърза обработка на предварителния преглед

8-битов PNG без калибриране за бърз преглед:

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

### Пример 4: Обработка с PPK корекция

Прилагане на PPK корекции с отражателна способност:

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

### Пример 5: Персонализирано място за изход

Обработка в различно място с конкретен формат:

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

### Пример 6: Работен поток за удостоверяване

Пълен работен поток за удостоверяване (еднакъв на всички платформи):

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

### Пример 7: Използване на няколко езика

Промяна на езика на интерфейса (еднакъв на всички платформи):

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
