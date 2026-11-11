# Инсталиране на Linux

Chloros се разпространява за Linux под формата на пакети `.deb`, които инсталират CLI, бекенд сървъра и Python SDK (за системата `python3`, както и wheel файл, съответстващ на версията, за virtualenvs).

Имената на файловете на пакетите съдържат версията и архитектурата: `chloros_1.3.0_amd64.deb` за x86_64, и `chloros_1.3.0_arm64_jp6.deb` за arm64 (NVIDIA Jetson, JetPack 6). Заместете файла, който сте изтеглили, в командите по-долу.

{% hint style="info" %}
**Ново в 1.3.0: програмите се доставят в разпакетиран вид.** Бекендът и CLI-интерфейсът се инсталират като папки, `/usr/lib/chloros/backend/` и `/usr/lib/chloros/cli/`, вместо като единични саморазпаковащи се файлове. Те вече не се разпаковат сами във временна папка при всяко стартиране, така че стартират много по-бързо (на x86_64 бекендът преди отнемаше около 100 секунди за разпаковане при всяко стартиране) и вече не се нуждаят от няколко GB свободно място в `/tmp`. Размерът на amd64 `.deb` също е около половината от предишния.
{% endhint %}

***

## Linux amd64 (x86_64)

### Системни изисквания

| Изискване | Минимално | Препоръчително |
| --- | --- | --- |
| **Дистрибуция** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Процесор** | x86_64 (Intel/AMD) | Intel Core i7 или по-добър |
| **Памет (RAM)** | 8 GB | 16 GB или повече |
| **Графична карта** | Няма (обработка от процесора) | NVIDIA GPU с 4 GB+ VRAM (12 GB+ отключва `GPU_PARALLEL`, 7 GB+ поддържа Texture Aware изключен при работа с единични изображения) |
| **Памет** | 2 GB свободно място | SSD с 10 GB+ свободно място |
| **Python** | Python 3.7+ (за SDK) | Python 3.10+ |

> **Ubuntu 20.04 и Debian 11 не се поддържат.** Списъкът със зависимости на `.deb` е
> извлечен от това, към което всъщност се свързва бекендът на Chloros, и това включва
> `libc6`, по-нова от glibc 2.31, която се доставя с focal и bullseye, така че `apt` отказва
> инсталацията напълно, вместо да допусне тя да се провали по-късно по време на изпълнение.

### Инсталиране

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Използвайте `apt install ./…`, а не `dpkg -i`.** `apt` разрешава зависимостите на пакета и също така инсталира това, което препоръчва: изискванията на Python SDK, `python3-numpy` и `python3-requests`. `dpkg -i` не инсталира нито едното, нито другото. CLI-то и бекендът са самостоятелни и работят без тях, но `import chloros_sdk` се нуждае от тях; изходът от инсталацията посочва всички липсващи компоненти. Запазете `./`, за да може `apt` да инсталира файла, вместо да търси пакет с това име. Пакетът също така декларира `libgl1`, който се зарежда от включения в пакета OpenCV и може да липсва в минималния образ на сървъра.
{% endhint %}

Проверете инсталацията:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Системни изисквания

| Изискване | Минимално | Препоръчително |
| --- | --- | --- |
| **Платформа** | NVIDIA Jetson с JetPack 6 | Jetson Orin NX 16 GB или AGX Orin |
| **JetPack** | JetPack 6.x | Най-новата версия на JetPack 6 |
| **Памет (RAM)** | 8 GB (споделена между GPU и CPU) | 16 GB+ споделена (12 GB+ е прагът за паралелна обработка) |
| **Памет** | 2 GB свободно място | NVMe SSD с 10 GB+ свободно място |
| **Python** | Python 3.7+ (за SDK) | Python 3.10+ |

### Инсталиране

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Същата структура като amd64 `.deb` (включва Python SDK, буфери за приемане и бекенд услуги), за Jetson Orin / Orin NX / Orin Nano на **JetPack 6** (L4T R36). PyTorch е CUDA версията на JetPack 6 за Orin, така че обработката на GPU, включително Texture Aware debayer, се изпълнява на GPU-то на Jetson. Пакетът не съдържа самата CUDA: той използва библиотеките CUDA и cuDNN, които JetPack 6 инсталира (`sudo apt install nvidia-jetpack` на образ, в който те липсват). Когато няма наличен CUDA GPU, обработката се изпълнява на CPU и `--debayer texture-aware` използва стандартния дебейър (бекендът записва един ред `[DEBAYER] … needs a CUDA GPU …`, в който се посочва причината). За поведението на Jetson по отношение на паметта, термичните характеристики и внедряването на място вижте [Ръководството за NVIDIA Jetson](nvidia-jetson-guide.md).***

## Инсталиране на Python SDK (за всички версии на Linux)**`.deb` инсталира SDK вместо вас.** И двата пакета (amd64 и Jetson) го инсталират за системата `python3`, така че `import chloros_sdk` работи веднага след инсталацията, без pip и без мрежа. Неговите пакети (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` и заредителите на проекти) се поместват в `/usr/lib/python3/dist-packages`, принадлежащ на пакета: при актуализация те се заменят, а при премахване на пакета се премахват. Версията винаги съответства на инсталирания от вас CLI и бекенд.

Собствените изисквания на SDK, `python3-numpy` и `python3-requests`, са **препоръчителни, а не задължителни** от пакета: `sudo apt install ./chloros_<version>_<arch>.deb` ги инсталира, а `dpkg -i` – не. Изходът от инсталацията отчита версията на SDK, която `python3` импортира, всички липсващи изисквания, както и всеки `chloros_sdk`, намиращ се другаде в пътя на `python3`, който засенчва копието на пакета (като например такъв, инсталиран по-рано с `pip install --user`).**В virtualenv**, който не вижда системните пакети, инсталирайте приложеното wheel в него (изцяло офлайн):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**От PyPI** — само за машина, на която няма инсталиран пакет Chloros, чийто SDK управлява бекенд на друга машина (вижте [Режим „Remote-Backend“](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Допълнителни опции за инсталиране чрез pip (в virtualenv или PyPI):

| Допълнение | Команда | Добавя |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` за стрийминг на напредъка в реално време |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` за BLE (DAQ-M) транспорт |

Проверете SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Python SDK-то комуникира с бекенда, който `.deb` инсталира чрез локален HTTP API (`http://127.0.0.1:5000`). `ChlorosLocal` и помощните програми за свързване го стартират автоматично, когато е необходимо; `connect_all()` на запазен проект никога не го прави, затова на машина без монитор стартирайте бекенда като [услуга](#always-on-ptp-for-headless-hosts). Винаги използвайте буквалния IPv4 адрес, а не `localhost` — `localhost` може да се преобразува в `::1` и да отнеме около две секунди на заявка.
{% endhint %}

***

## Първоначална настройка

### 1. Влизане

Достъпът до CLI и SDK изисква платен план Chloros+ (**Copper** или по-висок), което се налага от страна на сървъра: излезлият от системата потребител получава `401 AUTH_REQUIRED`, а потребител с безплатен план (Iron) получава `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Удостоверенията се съхраняват в кеш в `~/.chloros/user_session.json`.

{% hint style="info" %}
**При ъпгрейд данните за вход се запазват; при премахване те се изтриват.** Инсталирането на по-нова версия `.deb` върху съществуваща инсталация (включително чрез `chloros-cli update`) запазва `~/.chloros/user_session.json` и кеширания лиценз, така че задачите в CLI/SDK без графичен интерфейс продължават да работят. Премахването на пакета (`sudo apt remove chloros`) изчиства запазените данни за вход и кеша на лиценза за всеки потребител на компютъра, така че при нова инсталация отново е необходим `chloros-cli login`.
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
| 1 | **Версия** | CLI-то отчита своята версия (`v1.3.0`). |
| 2 | **Порт наличен** | Порт 5000 е свободен, *или* вече е отговорен от изправен бекенд на Chloros (което се счита за успех). |
| 3 | **Стартиране на бекенда** | Бинарният файл на бекенда се стартира. |
| 4 | **Тест на API (`/api/test`)** | Бекендът отговаря с `status: ok`. |
| 5 | **Системна информация** | Извежда `GPU: <name>, CUDA: <bool>, PyTorch: <version>` от `/api/system-info`. |
| 6 | **Модели за отстраняване на шума** | Намира модели `*.pth.enc` (в Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + Филтър за отстраняване на шума**| Texture Aware всъщност може да се използва — изисква CUDA**и** поне един файл с модел. |

Изпълнението завършва с `N/7 checks passed`, като изброява всички грешки по име.

### 4. Обработете първия си набор от данни

```bash
chloros-cli process ~/datasets/flight001
```

***

## Файлове и директории

### За всеки потребител

Chloros съхранява своите идентификационни данни и конфигурацията на CLI в една единствена междуплатформена директория, **`~/.chloros/`** (в Windows — `%USERPROFILE%\.chloros\`). Два кеша, специфични за Linux, следват конвенциите на XDG — те използват `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`, когато са зададени.

| Път | Предназначение |
| --- | --- |
| `~/.chloros/user_session.json` | Кеш на сесията за вход, записан от `chloros-cli login` (запазва се при актуализации на пакета; изчиства се при премахване на пакета) |
| `~/.chloros/working_directory.txt` | Заместване на папката по подразбиране за проекта (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Настройка за езика на CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Настройка на езика, споделена с графичния интерфейс на Windows — тук `language` има предимство пред `cli_language.json` |
| `~/.chloros/update_cache.json` | Едночасов кеш за проверката за актуализации при стартиране на Linux/Jetson |
| `~/.chloros/backend.log` | Лог на бекенда при стартирането му чрез CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Кеширани пакети за калибриране на LATTICE за всяка камера, индексирани по сериен номер и хеш на пакета |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Опционални потребителски настройки за профилите за корекция на капацитета на DAQ |
| `~/.config/chloros/system_config.json` | Кеширан хардуерен профил от Dynamic Compute Adaptation — изтрийте го, за да принудите ново откриване на хардуера |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Лог файлове на бекенд сървъра, по един файл за всяко стартиране |
| `~/Documents/MAPIR/Chloros Projects/` | Папка по подразбиране за проекта, когато не е зададено преопределяне |

### На ниво система

| Път | Предназначение |
| --- | --- |
| `/usr/bin/chloros-cli` | Скрипт-обвивка — задава `LD_LIBRARY_PATH` за включените в пакета native библиотеки (и `TMPDIR` към `/mnt/ssd/tmp`, когато тази папка съществува), след което изпълнява действителния бинарен файл |
| `/usr/bin/chloros-backend` | Скрипт-обвивка — същото за бекенда, който маркира като инсталирана производствена версия |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | Компилираните CLI и бекенд, инсталирани в разпакетиран вид (новост в 1.3.0); програмите са `cli/chloros-cli` и `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Двуредни скриптове за стартиране, които пускат горепосочените програми — запазени на тези пътища, защото SDK, CLI и по-старите скриптове ги търсят тук |
| `/usr/lib/chloros/arena_runtime/` | Среда за изпълнение на Arena SDK, необходима за камерите LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Криптирани модели за премахване на шума, използвани от Texture Aware debayer |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (и `lattice_sdk`, `lattice_cam`, `arena_api` — заредителите на проекта) | Python SDK, инсталиран за системата `python3` и принадлежащ на пакета |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK wheel, съответстващ точно на тази версия, за virtualenv |
| `/usr/lib/chloros/exiftool` | Включен стартиращ файл за ExifTool (с символна връзка към `/usr/local/bin/exiftool` само ако не съществува системен exiftool). От версия 1.3.0 нататък скриптът ExifTool се доставя със собствена Perl библиотека в `/usr/lib/chloros/exiftool-dist/`, така че работи без системния пакет `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | Конфигурация на канала за актуализации, четена от `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Задава `net.ipv4.ip_unprivileged_port_start = 319`, така че бекендът да може да свърже PTP портовете без права на root |
| `/etc/sysctl.d/60-chloros-gige.conf` | Увеличава буферите за приемане на сокети на ядрото (`net.core.rmem_max`, `net.core.rmem_default`) до 32 MB за потоците от камери LATTICE и ги прилага при инсталирането (вече зададена по-висока стойност се запазва). При стандартните настройки на ядрото пакетите от камерата се изхвърлят в ядрото и масивът губи кадри и честота на кадрите; `chloros-cli lattice network` сигнализира за нивото на буферите, когато то е ниско. Вижте [Хостове с Linux — буфери за приемане на сокети](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Насочва динамичния заредител към `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Предоставя на вписания потребител достъп до USB серийния мост DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Услуга за постоянно активен бекенд, която се стартира по избор и изпълнява бекенда като потребителя, който посочите (инсталирана, **не е активирана**) — тази, която трябва да активирате, вижте [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Същата услуга, която изпълнява бекенда като **root**(инсталирана,**не е активирана**), запазена, за да може машината, на която е активирана, да продължи да работи |
| `/var/log/chloros/` | Подробният лог на услугата с права на root, достъпен за четене от групата `adm` (изтрива се при премахване на пакета) |
| `/usr/share/applications/chloros-cli.desktop` | Елемент в менюто на приложението „Chloros CLI“, който отваря терминал |

**Защитна стена (ново в 1.3.0).** Ако `ufw` или `firewalld` са активни по време на инсталирането, пакетът отваря UDP портове `5002` (данни от DAQ-E), `5353` (откриване чрез mDNS), `319` и `320` (PTP) и премахва тези правила при деинсталиране на пакета. Ако е инсталиран, но не е активен, той не добавя нищо и извежда портовете, които трябва да бъдат разрешени, в случай че го активирате по-късно. Той не отваря TCP портове: актуализацията на фърмуера на DAQ-E, при която сензорът се свързва обратно с този компютър през TCP, за да събере изображението, се проваля с грешка `… device never connected back on TCP <port> …`, докато активна защитна стена блокира тази връзка.

## Местоположение на изпълнимия файл на бекенда

CLI и SDK автоматично откриват бекенда:

| Компонент | Път |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Бекенд | `/usr/lib/chloros/chloros-backend` (стартиращ файл за `/usr/lib/chloros/backend/chloros-backend`) |

Преопределете пътя към бекенда с флага на CLI-а `--backend-exe` или с параметъра на конструктора на SDK-а `backend_exe`, а портът – с `--port` (по подразбиране `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` насочва семействата команди **`lattice`**,**`project`**и**`daq pool-*`** към отдалечен бекенд. Основните команди (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) умишлено го игнорират и винаги се насочват към `http://127.0.0.1:<port>`.
{% endhint %}

***

## Камери LATTICE и светлинни сензори DAQ под Linux

Всички групи команди от типа „live-hardware“ работят под Linux (amd64 и Jetson):

* **`chloros-cli lattice`** — открива, свързва, конфигурира и записва от камери LATTICE и синхронизирани масиви. `.deb` обединява необходимата им среда за изпълнение на Arena SDK и я регистрира в динамичния заредител.
* **`chloros-cli daq pool-*`** — свързване на светлинни сензори DAQ-U/M/E чрез бекенд пула, стрийминг на калибрирани спектри, запис на файлове `.daq` и размяна на профили на капачки, а за DAQ-E — промяна на мрежовите му настройки, рестартиране и актуализиране на фърмуера му. `chloros-cli daq discover` сканира за сензори, към които можете да се свържете, а `chloros-cli daq --help` изброява подкомандите на `pool-*`.
* **`chloros-cli project`** — управлява запазен проект (камерите, сензорите и настройките за обработка) без монитор.
* **`chloros-cli time-sync`** — проверява PTP „grandmaster“, който бекендът на Chloros използва за камери LATTICE и сензори DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` е задължителна за `pool-latest`, `pool-stream`, `pool-record` и `pool-set-cap`; `pool-list` показва идентификаторите, които в момента се намират в пула.

{% hint style="info" %}
**`--eth-host` пропуска откриването.** `pool-connect --eth` открива DAQ-E чрез mDNS, с ARP резервен вариант, който работи от празен ARP кеш; като се предаде IP адресът или името на хоста на сензора с `--eth-host`, се осъществява директна връзка с него.
{% endhint %}

**Разрешенията за сериен порт на DAQ-U** се управляват от инсталираното правило на udev (`uaccess` + група `dialout`). Ако сензор, който вече е бил включен, остава недостъпен, презаредете правилата или го включете отново:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Вижте [справочника за CLI](../CLI.md) за пълния набор от команди.

### Постоянно активен PTP за хостове без монитор

Пакетът инсталира две systemd услуги за бекенда, като нито една от тях не е активирана. На Jetson без монитор или сървър, който трябва да поддържа непрекъсната PTP синхронизация на времето за DAQ-E сензори и LATTICE камери, или където скриптове на SDK свързват запазени проекти (`connect_all()` никога не стартира бекенда), активирайте услугата **като ваш потребител**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` изпълнява бекенда като този потребител: записите, които създава, са негови, подробният му лог се намира в неговия `~/.cache/chloros/logs`, а той използва неговия `chloros-cli login` (`~/.chloros`), затова първо се влезте веднъж като този потребител. При инсталирането на пакета се извежда тази команда с попълнено вашето потребителско име.

Без услуга бекендът (а с него и PTP грандмастърът) работи само докато нещо го е стартирало — команда от типа `chloros-cli` или скрипт от типа SDK.

Устройството свързва бекенда с настройките на средата от типа `127.0.0.1:5000` (настройки на средата `CHLOROS_HOST` / `CHLOROS_PORT` вътре в модула; преопределете с `sudo systemctl edit chloros-backend@.service`) и го рестартира при отказ след 5 секунди.

**Услугата като root.**`chloros-backend.service` изпълнява бекенда като**root**. Тя все още е инсталирана, така че машината, на която е активирана, да продължава да работи, но всичко, което бекендът с права на root записва, принадлежи на root, така че потребителят не може да изтрие собствените си записи; логът му се намира в `/var/log/chloros` и е достъпен за четене от групата `adm`. За да превключите към своя потребител:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**Бекендът на дадена услуга е споделен и никога не се спира от клиент.** Скриптовете `chloros-cli` и SDK го използват така, както го намерят: `process` се изпълнява върху него, като зарежда собствения си проект на мястото на последния. Когато CLI, в противен случай, би рестартирал бекенда (`--restart` или `process`, чийто импорт не е намерил нищо), той посочва името на модула, извежда `sudo systemctl restart <unit>` и излиза от `1`, тъй като бекенд, спрян по този начин, би останал изключен. Рестартирайте го сами с `sudo systemctl restart chloros-backend@$USER.service`. Бекендът на дадена услуга никога не се спира сам в режим на бездействие, а `chloros-cli backend stop` го отхвърля, като посочва командата `systemctl stop`.**Как PTP получава своите портове.** PTP използва UDP 319/320, и двата под нормалния праг от 1024 за привилегированите портове. `postinst` на пакета записва `/etc/sysctl.d/60-chloros-ptp.conf` с `net.ipv4.ip_unprivileged_port_start = 319`, което позволява на бекенда да ги свърже, докато работи като вашия потребител. От версия 1.3.0 нататък пакетът не задава файлови права (`setcap`) на бекенда, а `libcap2-bin` вече не е зависимост: при разпакения бекенд файловите права биха попречили на стартирането му, затова не добавяйте такава сами.***

## Примери за скриптове в Bash

{% hint style="info" %}
**Кодове за изход, подходящи за скриптове.**`chloros-cli process` излиза с `0` при успех и**с не нулева стойност при неуспех — включително при изпълнение, което е поискало изображения, но не е записало нито едно** (извежда `Processing finished but wrote no image products.` и посочва името на папката на проекта и обичайните причини). Успешните изпълнения отчитат колко изображения са записани (`Image products written: N`, плюс `(plus M clamp masks)`, когато са записани маски). Кодове за изход: `0` — успех, `1` — неуспех, `2` — грешка в аргумента, `130` — прекъсване.
{% endhint %}

### Обработка на множество набори от данни

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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Валидните стойности на `--format` са точно четири и съдържат интервали — винаги ги поставяйте в кавички:

| Стойност на `--format` | Папка за изходни данни |
| --- | --- |
| `TIFF (16-bit)` *(по подразбиране)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` приема `standard` (по подразбиране) или `texture-aware` (Chloros+).

### Автоматизирана обработка с Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Пример за SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Бекендът не стартира след добавяне на файлови права

Не изпълнявайте `setcap` на бекенда. От версия 1.3.0 нататък пакетът не задава файлови възможности, а портовете 319/320 на PTP произтичат от настройката `sysctl`, описана в [Как PTP получава своите портове](#always-on-ptp-for-headless-hosts). Файловата способност поставя разпакетирания бекенд в режим на сигурно изпълнение, в който той не може да намери собствените си библиотеки, поради което не стартира. Ако сте добавили такава, премахнете я:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP не стартира / не може да се свърже с порт 319

Уверете се, че прагът за портове без привилегии е бил понижен, и го приложете отново за текущото зареждане, ако това не е било направено:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

След това проверете главния сървър:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### Командите `lattice` се провалят с съобщение „Не са намерени драйвери за камери LATTICE“

Бекендът не можа да зареди средата за изпълнение на камерата LATTICE, с която е инсталиран. Всяка команда `lattice` се изпълнява на бекенда; самият CLI не съдържа драйвери за камери. Преинсталирайте Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): `.deb` инсталира средата за изпълнение до бекенда, в `/usr/lib/chloros/arena_runtime`, и я регистрира в динамичния заредител. За да проверите тази регистрация:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
ls /usr/lib/chloros/arena_runtime | head
```

### Бекендът не успя да стартира

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Лог файловете на бекенда за неуспешния старт се намират в `~/.cache/chloros/logs/`. Ако порт 5000 принадлежи на услуга на бекенда (`systemctl status chloros-backend@$USER.service`), рестартирайте услугата, вместо да прекратявате процеса ѝ.

### CUDA не е открит

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` отчита същото в един ред: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Липсващи споделени библиотеки

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Бавно стартиране на системи с SD-карти

От версия 1.3.0 нататък бекендът и CLI-то се инсталират в разпакетиран вид, така че вече не се извличат в временна директория при всяко стартиране. Временните файлове се записват автоматично в `/mnt/ssd/tmp`, ако тази папка съществува; в противен случай можете да ги насочите към бърза файлова система:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Актуализиране на Chloros на Linux

Командата `update` е достъпна само за Linux/Jetson. Тя проверява версията, публикувана в канала за актуализации, конфигуриран в `/etc/chloros/update.conf`, и предлага да изтегли и инсталира съответната `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

В Linux/Jetson CLI също извършва неблокираща проверка за актуализации при всяко стартиране (резултатът се съхранява в кеш за един час в `~/.chloros/update_cache.json`) и извежда `Update available: vX.Y.Z`, когато съществува по-нова версия. Вашите настройки, проекти и данни за вход се запазват след актуализацията.

Актуализирането на пакета спира всички бекенд услуги по време на инсталирането, така че изпълняващ се скрипт губи достъпа си до камерите и сензорите, а след това стартира отново всяка от тях, която е активирана (тази, която е стартирана ръчно, но никога не е била активирана, остава спряна, както след рестартиране).

## Деинсталиране

```bash
sudo apt remove chloros
```

Деинсталирането спира и деактивира всички бекенд услуги, възстановява стандартната граница за портове без привилегии (1024), премахва файла с настройките на буфера за приемане (текущите стойности остават до рестартиране), символната връзка към включения в пакета exiftool, конфигурацията на Arena loader, правилата на защитната стена, добавени от пакета, и Python SDK, който е инсталирал, и изчиства кешираните идентификационни данни. Вашите проекти и файловете с данни на `~/.chloros/` остават непокътнати. `sudo apt purge chloros` също така изтрива `/var/log/chloros`.

***

## Следващи стъпки

* [Ръководство за NVIDIA Jetson](nvidia-jetson-guide.md) — Оптимизация и внедряване, специфични за Jetson
* [CLI: Командна линия](../CLI.md) — ръководството за CLI
* [API: Python SDK](../api-python-sdk.md) — ръководството за SDK
* [Справочник за CLI](../reference/cli-reference.md) и [Справочник за SDK](../reference/sdk-reference.md) — изчерпателни списъци с команди/API за версия 1.3.0
* [Динамична адаптация на изчислителната мощност](../processing-architecture/dynamic-compute-adaptation.md) — как Chloros се адаптира към вашия хардуер
