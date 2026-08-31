# Инсталиране на Linux

Chloros се разпространява за Linux като пакети `.deb`, които инсталират CLI и сървъра на бекенда. Python SDK е отделен pip пакет (също включен в `.deb` като wheel с съответстваща версия).

Имената на файловете на пакетите съдържат версията и архитектурата: `chloros_1.2.0_amd64.deb` за x86_64 и `chloros_1.2.0_arm64_jp6.deb` за JetPack 6 Jetson версии. Заместете файла, който сте изтеглили, в командите по-долу.

***

## Linux amd64 (x86_64)

### Системни изисквания

| Изискване | Минимално | Препоръчително |
| --- | --- | --- |
| **Дистрибуция** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Процесор** | x86_64 (Intel/AMD) | Intel Core i7 или по-добър |
| **Памет (RAM)** | 8 GB | 16 GB или повече |
| **Графична карта** | Няма (обработка от процесора) | NVIDIA GPU с 4 GB+ VRAM (12 GB+ отключва `GPU_PARALLEL`, 7 GB+ поддържа Texture Aware изключено за пътя с едно изображение) |
| **Памет** | 2 GB свободно място | SSD с 10 GB+ свободно място |
| **Python** | Python 3.7+ (за SDK) | Python 3.10+ |

> **Ubuntu 20.04 и Debian 11 не се поддържат.** Списъкът със зависимости на `.deb` е
> извлечен от това, към което бекендът на Chloros действително се свързва, и това включва
> `libc6 (>= 2.34)`. Focal и bullseye се доставят с glibc 2.31, така че `apt` отказва
> инсталирането напълно, вместо да допусне то да се провали по-късно по време на изпълнение.

### Инсталиране

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` не разрешава зависимостите. Ако докладва липсващи пакети, `sudo apt-get install -f` (или `sudo apt --fix-broken install`) завършва инсталацията — това е нормалният процес, а не грешка.
{% endhint %}

Проверете инсталацията:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Системни изисквания

| Изискване | Минимално | Препоръчително |
| --- | --- | --- |
| **Платформа** | NVIDIA Jetson с JetPack 6 | Jetson Orin NX 16 GB или AGX Orin |
| **JetPack** | JetPack 6.x | Най-новата версия на JetPack 6 |
| **Памет (RAM)** | 8 GB (споделена между GPU и CPU) | 16 GB+ споделена (12 GB+ е прагът за паралелни GPU работници) |
| **Памет** | 2 GB свободно място | NVMe SSD с 10 GB+ свободно място |
| **Python** | Python 3.7+ (за SDK) | Python 3.10+ |

### Инсталиране

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Същата конфигурация като amd64 `.deb`, с CUDA версия, оптимизирана за Jetson Orin / Orin NX / Orin Nano. За поведението на Jetson по отношение на паметта, термичните характеристики и внедряването на място вижте [Ръководството за NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Инсталиране на Python и SDK (всички Linux)

SDK е чист Python HTTP клиент за бекенда, така че един и същ пакет работи както на amd64, така и на arm64. Два източника:**От PyPI** — публикуваната стабилна версия:

```bash
pip install chloros-sdk
```

**От приложеното wheel** — гарантирано, че съответства на CLI/backend, който току-що сте инсталирали (използвайте това, когато вашият `.deb` е по-нов от този в PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**Дистрибуциите по PEP 668** (Ubuntu 23.10+, Debian 12+) не позволяват инсталиране чрез pip в цялата система. Използвайте `pip install --user …`, виртуална среда или `sudo pip install --break-system-packages …`. Инсталаторът на пакета никога не инсталира автоматично SDK във вашата система Python — този избор се оставя на вас.
{% endhint %}

Допълнителни опции:

| Допълнение | Команда | Добавя |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` за стрийминг на напредъка в реално време |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` за BLE (DAQ-M) транспорт |

Проверете SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` инсталира Chloros, CLI и бекенда. Python, SDK комуникира с този бекенд през локална мрежа HTTP API (`http://127.0.0.1:5000`) и го стартира автоматично, когато е необходимо. Винаги използвайте буквалния IPv4 адрес, вместо `localhost` — `localhost` може да се разреши като `::1` и да отнеме около две секунди на заявка.
{% endhint %}

***

## Първоначална настройка

### 1. Влизане

Достъпът до CLI и SDK изисква платен план Chloros+ (**Copper** или по-висок), което се прилага от страна на сървъра: потребител, който не е влязъл в системата, получава `401 AUTH_REQUIRED`, а потребител с безплатен план (Iron) получава `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Удостоверенията се кешират в `~/.chloros/user_session.json`.

{% hint style="warning" %}
**Трябва да влезете отново след всяка инсталация или актуализация.** Скриптът `prerm` на пакета умишлено изчиства `~/.chloros/user_session.json` и кеширания лиценз за всеки потребител на компютъра, така че всяка нова версия винаги проверява валидността на лиценза, вместо да се доверява на остарял кеш.
{% endhint %}

### 2. Проверете състоянието на лиценза си

```bash
chloros-cli status
```

`chloros-cli status` работи на всеки план (включително безплатния), така че винаги можете да видите защо достъпът е или не е наличен.

### 3. Изпълнете системна диагностика

```bash
chloros-cli selftest
```

Извършват се седем проверки поред, а командата излиза с резултат, различен от нула, ако някоя от тях се провали:

| # | Проверка | Какво доказва |
| --- | --- | --- |
| 1 | **Версия** | CLI съобщава своята версия (`v1.2.0`). |
| 2 | **Портът е свободен** | Порт 5000 е свободен, *или* вече е отговорен от изправен Chloros бекенд (което се счита за успех). |
| 3 | **Стартиране на бекенда** | Бейнендът се стартира. |
| 4 | **Тест на API (`/api/test`)** | Бекендът отговаря с `status: ok`. |
| 5 | **Информация за системата** | Извежда `GPU: <name>, CUDA: <bool>, PyTorch: <version>` от `/api/system-info`. |
| 6 | **Модели за отстраняване на шума** | Намира модели `*.pth.enc` (в Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + Филтър за отстраняване на шума**| Функцията „Texture Aware“ действително може да се използва — изисква CUDA**и** поне един файл с модел. |

Изпълнението приключва с `N/7 checks passed`, като изброява всички грешки по име.

### 4. Обработете първия си набор от данни

```bash
chloros-cli process ~/datasets/flight001
```

***

## Файлове и директории

### За всеки потребител

Chloros съхранява своите идентификационни данни, а CLI — конфигурацията си в една единствена междуплатформена директория, **`~/.chloros/`** (в Windows, `%USERPROFILE%\.chloros\`). Два кеша, специфични за Linux, следват конвенциите на XDG — те спазват настройките на `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`, когато са зададени.

| Път | Предназначение |
| --- | --- |
| `~/.chloros/user_session.json` | Кеш на сесията за вход, записан от `chloros-cli login` (изчиства се при всяка инсталация/актуализация на пакет) |
| `~/.chloros/working_directory.txt` | Заместване на папката по подразбиране за проекта (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Настройка за език на CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Настройката на езика се споделя с графичния интерфейс на Windows — тук `language` има приоритет пред `cli_language.json` |
| `~/.chloros/update_cache.json` | Едночасов кеш за проверката за актуализации при стартиране на Linux/Jetson |
| `~/.chloros/backend.log` | Лог на бекенда при стартирането му от CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Кеширани пакети за калибриране на LATTICE за всяка камера, сортирани по сериен номер и хеш на пакета |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Опционални потребителски настройки за профилите за корекция на капацитета на DAQ |
| `~/.config/chloros/system_config.json` | Кеширан хардуерен профил от Dynamic Compute Adaptation — изтрийте го, за да принудите ново разпознаване на хардуера |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Логи на сървъра на бекенда, по един файл за всяко стартиране |
| `~/Chloros Projects/` | Папка по подразбиране за проекта, когато не е зададено преопределяне |

### За цялата система

| Път | Предназначение |
| --- | --- |
| `/usr/bin/chloros-cli` | Скрипт-обвивка — задава `LD_LIBRARY_PATH` за включените в пакета native библиотеки, след което изпълнява действителния бинарен файл |
| `/usr/bin/chloros-backend` | Скрипт-обвивка — същото, плюс `CHLOROS_PRODUCTION=1`, за да не може авторизационният шлюз на бекенда никога да се деактивира тихо |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Компилираните бинарни файлове |
| `/usr/lib/chloros/arena_runtime/` | Среда за изпълнение Arena SDK, изисквана от камерите LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Криптирани модели за премахване на шума, използвани от Texture Aware debayer |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK колело, съответстващо точно на тази версия |
| `/usr/lib/chloros/exiftool` | Включен exiftool (създава се символна връзка към `/usr/local/bin/exiftool` само ако в системата няма exiftool) |
| `/etc/chloros/update.conf` | Конфигурация на канала за актуализации, четена от `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Настройва `net.ipv4.ip_unprivileged_port_start = 319`, така че бекендът да може да свърже PTP портовете без права на root |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Насочва динамичния заредител към `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Предоставя на вписания потребител достъп до USB-серийния мост DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Включване на постоянно активна бекенд услуга (инсталирана, **неактивирана**) |
| `/usr/share/applications/chloros-cli.desktop` | Елемент в менюто на приложението „Chloros CLI“, който отваря терминал |

## Местоположение на изпълнимия файл на бекенда

CLI и SDK автоматично откриват бекенда:

| Компонент | Път |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Бекенд | `/usr/lib/chloros/chloros-backend` |

Преопределете пътя към бекенда с флага `--backend-exe` CLI или с параметъра на конструктора `backend_exe` SDK, а портът – с `--port` (по подразбиране `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` сочи към **`lattice`**,**`project`**и**`daq pool-*`** към отдалечен бекенд. Основните команди (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) умишлено го игнорират и винаги се насочват към `http://127.0.0.1:<port>`.
{% endhint %}

***

## Камери LATTICE и светлинни сензори DAQ на Linux

Всички групи команди за работа с хардуер в реално време работят на Linux (amd64 и Jetson):

* **`chloros-cli lattice`** — откриване, свързване, конфигуриране и запис от камери LATTICE и синхронизирани масиви. `.deb` обединява необходимата им среда за изпълнение Arena SDK и я регистрира в динамичния заряждащ модул.
* **`chloros-cli daq pool-*`** — свързване на светлинни сензори DAQ-U/M/E чрез бекенд пула, предавайте калибрирани спектри и записвайте файлове `.daq`. Компилираният CLI включва само семейството `pool-*`: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — стартирайте запазен проект (камерите, сензорите и настройките за обработка) в режим без интерфейс.
* **`chloros-cli time-sync`** — проверява PTP „grandmaster“, на който работи бекендът Chloros за камери LATTICE и сензори DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` е задължително условие за `pool-latest`, `pool-stream`, `pool-record` и `pool-set-cap`; `pool-list` показва идентификаторите, които в момента се намират в пула.

{% hint style="info" %}
**Предпочитайте `--eth-host` за първото свързване на DAQ-E към машина с няколко мрежови интерфейса.** Автоматичното откриване претърсва mDNS и може да пропусне интерфейса на сензора поради празен ARP кеш, така че първият `pool-connect --eth` след стартиране може да се провали, дори когато сензорът е напълно изправен. Предаването на IP адреса или името на хоста на сензора изключва напълно процеса на откриване.
{% endhint %}

**Разрешенията за сериен порт на DAQ-U** се управляват от инсталираното правило на udev (`uaccess` + група `dialout`). Ако сензор, който вече е бил включен, остава недостъпен, презаредете правилата или го включете отново:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Вижте [справочника за CLI](../CLI.md) за пълния набор от команди.

### Постоянно активен PTP за хостове без монитор

При първата инсталация се генерира systemd модулът `chloros-backend.service`, но той **не е активиран**. На Jetson без монитор или сървър, който трябва да поддържа непрекъсната синхронизация на времето по PTP за DAQ-E сензори и LATTICE камери, активирайте го:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Без нея PTP работи само докато бекендът Chloros е активен — т.е. по време на активна сесия на CLI/SDK.

Устройството свързва бекенда с `127.0.0.1:5000` (настройки на средата `CHLOROS_HOST` / `CHLOROS_PORT` вътре в устройството; преопределяне с `sudo systemctl edit chloros-backend.service`) и го рестартира при отказ след 5 секунди.

**Как PTP получава своите портове.** PTP използва UDP 319/320, и двата под нормалния праг от 1024 за привилегировани портове. `postinst` от пакета записва `/etc/sysctl.d/60-chloros-ptp.conf` с `net.ipv4.ip_unprivileged_port_start = 319`, което позволява на бекенда да ги свърже, докато работи като вашия потребител. То също така прилага `setcap cap_net_bind_service,cap_net_raw=+ep` към бинарния файл на бекенда като допълнителна предпазна мярка — ето защо `libcap2-bin` е декларирана зависимост на пакета.***

## Примери за скриптове в Bash

{% hint style="info" %}
**Кодове за изход, подходящи за скриптове.**`chloros-cli process` извежда `0` при успех и**код, различен от нула, при неуспех — включително при изпълнение, което е поискало изображения, но не е записало нито едно** (извежда `Processing finished but wrote no image products.` и посочва името на папката на проекта и обичайните причини). Успешните изпълнения съобщават колко изображения са били записани (`Image products written: N`). Кодове за изход: `0` — успех, `1` — неуспех, `2` — грешка в аргументите, `130` — прекъсване.
{% endhint %}

### Обработка на няколко набора от данни

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Обработка с персонализирани настройки

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Валидните стойности за `--format` са точно четири и съдържат интервали — винаги ги поставяйте в кавички:

| Стойност на `--format` | Папка за изходни данни |
| --- | --- |
| `TIFF (16-bit)` *(по подразбиране)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` приема `standard` (по подразбиране) или `texture-aware` (Chloros+).

### Автоматизирана обработка с Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Пример за Python и SDK

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

### CLI не се намира след инсталирането

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Отказ на достъп

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### „setcap failed“ по време на инсталирането

`.deb` прилага `cap_net_bind_service` към `/usr/lib/chloros/chloros-backend`, за да може да свърже PTP портове 319/320 без root права. Ако `libcap2-bin` е липсвало по време на инсталирането, извикването се пропуска. Инсталирайте го и преинсталирайте пакета:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP не стартира / не може да свърже порт 319

Уверете се, че прагът за портовете без привилегии е бил намален, и го приложете отново за текущото зареждане, ако това не е било направено:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

След това проверете главния сървър:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### „Не са намерени драйвери за камери LATTICE“

Средата за изпълнение на Arena SDK не се разрешава. Уверете се, че конфигурацията на зарядното устройство, която пакетът записва, е налична и актуализирана:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
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

Лог файловете на бекенда за неуспешния старт се намират в `~/.cache/chloros/logs/`.

### CUDA не е открита

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` съобщава същото в един ред: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Липсващи споделени библиотеки

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Бавно стартиране на системи с SD-карти

Компилираните бинарни файлове се разпаковат във временна директория при всяко стартиране. Ако съществува `/mnt/ssd/tmp`, Chloros го използва автоматично; в противен случай задайте `TMPDIR` към бърза файлова система:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Актуализиране на Chloros на Linux

Командата `update` е достъпна само за Linux/Jetson. Тя проверява версията, публикувана в канала за актуализации, конфигуриран в `/etc/chloros/update.conf`, и предлага да изтегли и инсталира съответстващия `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

На Linux/Jetson CLI също извършва неблокираща проверка за актуализации при всяко стартиране (резултатът се съхранява в кеш за един час в `~/.chloros/update_cache.json`) и извежда `Update available: vX.Y.Z`, когато съществува по-нова версия. Настройките и проектите ви се запазват при актуализация; след това ще трябва да влезете отново в системата.

## Деинсталиране

```bash
sudo apt remove chloros
```

Деинсталирането спира `chloros-backend.service`, възстановява стандартния праг за портове без привилегии (1024), премахва символната връзка към exiftool и конфигурацията на Arena loader, както и изчиства кешираните идентификационни данни. Вашите проекти и файлове с данни `~/.chloros/` остават непроменени.

***

## Следващи стъпки

* [Ръководство за NVIDIA Jetson](nvidia-jetson-guide.md) — оптимизация и внедряване, специфични за Jetson
* [CLI : Командна линия](../CLI.md) — ръководството за CLI
* [API : Python SDK](../api-python-sdk.md) — ръководството SDK
* [CLI Справочник](../reference/cli-reference.md) и [SDK Справочник](../reference/sdk-reference.md) — изчерпателен списък с команди/API за версия 1.2.0
* [Динамична адаптация на изчислителната мощност](../processing-architecture/dynamic-compute-adaptation.md) — как Chloros се адаптира към вашия хардуер
