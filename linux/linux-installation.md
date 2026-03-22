# Инсталиране на Linux

Chloros се разпространява за Linux под формата на пакети `.deb`, които инсталират CLI и бекенда. Python SDK се инсталира отделно чрез pip.

***

## Linux amd64 (x86_64)

### Системни изисквания

| Изискване | Минимално | Препоръчително |
| --- | --- | --- |
| **Дистрибуция** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Процесор** | x86_64 (Intel/AMD) | Intel Core i7 или по-добър |
| **Памет (RAM)** | 8 GB | 16 GB или повече |
| **Графична карта** | Няма (обработка от процесора) | NVIDIA GPU с 4 GB+ VRAM |
| **Памет** | 2 GB свободно място | SSD с 10 GB+ свободно място |
| **Python** | Python 3.7+ (за SDK) | Python 3.10+ |

### Инсталиране

Изтеглете пакета `.deb` и инсталирайте:

```bash
sudo dpkg -i chloros-amd64.deb
```

Проверете инсталацията:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Системни изисквания

| Изискване | Минимално | Препоръчително |
| --- | --- | --- |
| **Платформа** | NVIDIA Jetson с JetPack 6 | Jetson Orin NX 16GB или AGX Orin |
| **JetPack** | JetPack 6.x | Най-новата версия на JetPack 6 |
| **Памет (RAM)** | 8 GB (споделена GPU/CPU) | 16 GB+ споделена |
| **Памет** | 2 GB свободно място | NVMe SSD с 10 GB+ свободно място |
| **Python** | Python 3.7+ (за SDK) | Python 3.10+ |

### Инсталиране

Изтеглете пакета JetPack 6 `.deb` и инсталирайте:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Проверете инсталацията:

```bash
chloros-cli --version
```

За подробна настройка на Jetson, включително термично управление и внедряване на място, вижте [Ръководството за NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Python SDK Инсталиране (Всички Linux)

Python SDK се инсталира отделно чрез pip и работи както на amd64, така и на arm64:

```bash
pip install chloros-sdk
```

За да включите опционална поддръжка за стрийминг на напредъка:

```bash
pip install chloros-sdk[progress]
```

Проверете SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Пакетът `.deb` инсталира Chloros CLI и бекенда. Python SDK е отделен pip пакет, който комуникира с бекенда чрез локален HTTP API.
{% endhint %}

***

## Конфигурационни директории

Chloros на Linux следва [Спецификацията за основна директория на XDG](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Цел | Linux Път | Windows Еквивалент |
| --- | --- | --- |
| **Конфигурация** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Данни / Проекти** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Кеш / Удостоверения** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Местоположения на изпълнимите файлове на бекенда

Пакетът `.deb` инсталира бекенда в стандартно местоположение. CLI и SDK автоматично откриват пътя към бекенда:

| Метод на инсталиране | Път към бекенда |
| --- | --- |
| Пакет `.deb` | `/usr/lib/chloros/chloros-backend` |
| Ръчно / по избор | `/opt/mapir/chloros/backend/chloros-backend` |

Можете да преопределите пътя към бекенда с флага `--backend-exe` CLI или с параметъра на конструктора `backend_exe` SDK.

***

## Първоначална настройка

### 1. Активирайте лиценза си

За достъп до CLI и SDK е необходим лиценз Chloros+:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Проверете състоянието на лиценза си

```bash
chloros-cli status
```

### 3. Обработете първия си набор от данни

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Изпълнете системна диагностика

Уверете се, че системата ви е конфигурирана правилно:

```bash
chloros-cli selftest
```

Това изпълнява 7 диагностични проверки, включително версия, стартиране на бекенда, API свързаност и наличност на CUDA/GPU.

***

## Примери за Bash скриптове

### Обработка на множество набори от данни

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Обработка с персонализирани настройки

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Автоматизирана обработка с Cron

Добавете към crontab (`crontab -e`), за да обработвате автоматично нови набори от данни:

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Python SDK Пример

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Отстраняване на проблеми

### CLI Не е намерен след инсталиране

Ако `chloros-cli` не е намерен след инсталиране на пакета `.deb`:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Отказ на достъп

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### Неуспешен старт на бекенда

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

### CUDA не е открит

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Липсват споделени библиотеки

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Актуализиране на Chloros на Linux

Използвайте вградената команда за актуализиране, за да проверите за налични актуализации и да ги инсталирате:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Следващи стъпки

* [Ръководство за NVIDIA Jetson](nvidia-jetson-guide.md) — Оптимизация и внедряване, специфични за Jetson
* [CLI : Командна линия](../CLI.md) — Пълен справочник за команди CLI
* [API : Python SDK](../api-python-sdk.md) — Пълен справочник за SDK
* [Динамична адаптация на изчисленията](../processing-architecture/dynamic-compute-adaptation.md) — Как Chloros се адаптира към вашия хардуер
