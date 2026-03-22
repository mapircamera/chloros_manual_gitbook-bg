# Ръководство за NVIDIA Jetson

Chloros на NVIDIA Jetson позволява мултиспектрална обработка на изображения в периферната среда — на място, на безпилотни летателни апарати и в отдалечени инсталации. Chloros автоматично разпознава вашия модел Jetson и оптимизира стратегията за обработка според вашия хардуер.

***

## Поддържани модели Jetson

| Модел                | RAM            | Стратегия за обработка                                   | Препоръчителна употреба                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB споделена | `GPU_PARALLEL` (4 работници)                            | Максимална производителност, големи масиви от данни                      |
| **Jetson Orin NX**   | 8–16 GB споделена | `GPU_PARALLEL` (3 работника, 16 GB) / `GPU_SINGLE` (8 GB) | Основна препоръка за внедряване във въздуха и на терен |
| **Jetson Orin Nano** | 8 GB споделена     | `GPU_SINGLE` (1 работник)                               | Изчисления на крайни устройства от начално ниво                                 |
| **Jetson Nano**      | 4–8 GB споделена   | `GPU_SINGLE` (1 работник)                               | Начално ниво, с ограничена памет                          |

{% hint style="info" %}
**По-старите модели Jetson** (TX2, TX1, Xavier NX) може да не се поддържат. Производителността ще варира в зависимост от наличната памет на GPU и възможностите на CUDA.
{% endhint %}

***

## Изисквания

* **JetPack 6.x** (препоръчва се най-новата версия)
* **NVIDIA CUDA** (включено в JetPack)
* **Лиценз за Chloros+** (необходим за достъп до CLI/SDK)

## Инсталиране

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

За общи подробности относно инсталирането на Linux вижте [Инсталиране на Linux](linux-installation.md).

***

## Динамична адаптация на изчисленията на Jetson

Chloros автоматично открива вашия модел Jetson и избира оптималната стратегия за обработка. **Не се изисква ръчна настройка.**

### Как работи

При стартиране Chloros профилира вашата система:

1. **Открива модела Jetson** чрез `/proc/device-tree/model`
2. **Чете наличната GPU/споделена памет**

3.**Избира стратегия за обработка** (`GPU_PARALLEL`, `GPU_SINGLE` или `CPU_PARALLEL`)
4. **Задава броя на работниците, типа на конвейера и разпределението на паметта** автоматично

### Поведение по модели

| Модел Jetson                | Стратегия       | Работни процеси | Конвейер                       | Едновременност |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (ефективно използване на паметта) | Сериализирано  |
| **Jetson Orin Nano 8GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Сериализирано  |
| **Jetson Orin NX 8GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Сериализирано  |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (пълна GPU пътека)    | Едновременни  |
| **Jetson AGX Orin 32-64GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Едновременни  |

{% hint style="success" %}
**Jetson Orin NX 16GB** е идеалният вариант за внедряване в периферията — той използва стратегията `GPU_PARALLEL` с 3 едновременни работника, осигурявайки истинска паралелна GPU обработка в компактен формат.
{% endhint %}

Ключовата разлика между платформите е **паметта**. Jetson Nano с 8 GB споделена памет трябва да обработва изображенията едно по едно, използвайки подход с ефективно използване на паметта, докато Orin NX с 16 GB може да обработва 3 изображения едновременно чрез GPU, използвайки конвергентен конвейер с по-висока производителност.

За пълна справка относно адаптирането на изчислителната мощност вижте [Динамично адаптиране на изчислителната мощност](../processing-architecture/dynamic-compute-adaptation.md).

***

## Термично управление

Устройствата Jetson имат ограничен термичен резерв, особено при затворени или въздушни инсталации. Chloros включва автоматично термично наблюдение и регулиране:

| Температура         | Действие                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70°C**          | Нормална работа — пълна скорост на обработка          |
| **70°C** (Предупреждение)  | Автоматично намаляване на размера на партидата                   |
| **80°C** (Критично) | Агресивно ограничаване — по-ниска паралелност         |
| **90°C** (Изключване) | Пълно спиране на обработката на GPU — необходимо охлаждане |

{% hint style="warning" %}
**Осигурете адекватна вентилация и отвод на топлината** за продължителна обработка, особено в затворени полеви корпуси или въздушни системи. Термичното ограничаване ще намали производителността на обработката, за да защити хардуера.
{% endhint %}

***

## Управление на паметта

Устройствата Jetson използват **унифицирана памет** — GPU и CPU споделят една и съща физическа RAM. Това означава, че отчетената VRAM (например 15,3 GB на Orin NX 16 GB) не е специална памет за GPU; тя се споделя с операционната система и други процеси.

### Препоръки за суап

За големи масиви от данни или обработка с Texture Aware debayer, Chloros може да препоръча създаването на суап пространство:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Оценки за паметта на изображение:**

* Стандартен дебайер: ~10 MB на изображение
* Дебайер с разпознаване на текстури: ~15 MB на изображение

Chloros автоматично изчислява необходимата памет въз основа на размера на вашия набор от данни и ви предупреждава, ако се препоръчва суап.

### Резервен вариант при OOM (Out of Memory)

Ако по време на обработката бъде открито състояние на изчерпване на паметта:

1. Chloros автоматично намалява броя на работните процеси на GPU
2. Преминава от `fused_gpu` към `tiled_gpu` пипалин (по-ефективен по отношение на паметта)
3. Продължава обработката с намалена производителност, вместо да се срине

***

## Разгръщане на място

### Съображения относно захранването

| Модел Jetson     | Типична консумация на енергия | Бележки                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5-10 W              | USB-C или цилиндричен конектор    |
| Jetson Orin Nano | 7-15 W              | DC цилиндричен конектор          |
| Jetson Orin NX   | 10-25 W             | DC цилиндричен конектор          |
| Jetson AGX Orin  | 15-60 W             | USB-C PD или цилиндричен конектор |

Планирайте енергийния си бюджет за продължителна обработка — пиковото потребление на енергия се наблюдава по време на Thread 3 (Processing), който натоварва интензивно графичния процесор (GPU).

### Препоръки за съхранение

* **NVMe SSD** се препоръчва силно за arm64 разгръщания
* SD картите са твърде бавни за обработка — използвайте ги само като носители за стартиране
* Планирайте 2-3 пъти по-голям размер на обработените данни в сравнение с размера на необработените изображения

### Работа без монитор чрез SSH

Chloros CLI е идеален за инсталации на Jetson без монитор:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Автоматизирана обработка със systemd

Създайте услуга systemd за автоматизирана обработка:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

Съчетайте я с таймер systemd за планирана обработка:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Примери за работни потоци

### Основна обработка на Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK на Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Пакетна обработка на множество полети

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Препоръчителни системи Jetson за използване на терен

За разгръщане на терен и във въздуха, обмислете следните варианти на носещи платки Jetson Orin NX 16GB:

* **Въздушно/дрон**: Системи с вибрационна устойчивост (MIL-STD), леки (под 300g), пасивно охлаждане
* **Устойчиви на тежки условия**: Водоустойчиви кутии IP67/IP69K с възможност за свързване на PoE GigE камера
* **Минимални/икономични**: Комплекти за разработчици с допълнителни кутии

Свържете се с [MAPIR Поддръжка](https://www.mapir.camera/community/contact) за конкретни препоръки за хардуер за вашия сценарий на внедряване.

***

## Следващи стъпки

* [Linux Инсталиране](linux-installation.md) — Общи подробности за инсталирането на Linux
* [Динамична адаптация на изчислителната мощност](../processing-architecture/dynamic-compute-adaptation.md) — Пълно справочно ръководство за стратегиите за изчислителна мощност
* [Процесорна тръба](../processing-architecture/processing-pipeline.md) — Разбиране на 4-нишковата тръба
* [CLI : Командна линия](../CLI.md) — Пълно справочник за CLI
* [API : Python SDK](../api-python-sdk.md) — Пълен справочник за SDK
