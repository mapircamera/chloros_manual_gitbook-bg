# API: Python SDK

{% hint style="info" %}
**Търсите пълния API?** Тази страница е практическо ръководство. Всички публични класове, методи, точни сигнатури и примери, които можете да копирате и поставите, се намират в [SDK Reference](reference/sdk-reference.md), който е оптимизиран за AI асистенти.**Работите с AI асистент?** Поставете този URL в чата, за да имате достъп до пълното и актуално Chloros API на Chloros 1.3.0:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Всяка страница от това ръководство е достъпна като суров Markdown файл под името си с малки букви + `.md`, а цялото ръководство е индексирано на `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` в PyPI) управлява всичко, което десктоп приложението може да прави от Python: пакетна обработка на изображения, управление на камери LATTICE и масиви в реално време, сесии с DAQ светлинни сензори и автоматизация на запазени проекти. Това е тънък слой върху същия локален бекенд, който използват GUI и CLI (HTTP на `127.0.0.1:5000`), така че поведението е идентично и на трите интерфейса.

## Инсталиране

SDK-ът се доставя заедно с десктоп пакета Chloros, който също предоставя бекенда за обработка и хардуерните среда за изпълнение и винаги съответства на неговата версия.

**Стъпка 1 — Инсталиране на Chloros.** Windows: стартирайте инсталатора за настолни компютри (по подразбиране в `C:\Program Files\Chloros\`) от страницата [Изтегляне](download.md). Linux: инсталирайте пакета `.deb` с `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Инсталиране на Linux](linux/linux-installation.md)).**Стъпка 2 — Проверете SDK** (Python 3.7+). Обикновено не се налага да инсталирате нищо:

* **Windows:** инсталаторът инсталира приложеното SDK wheel в системния ви Python (ако няма инсталиран Python, тази стъпка се пропуска).
* **Linux:** `.deb` инсталира SDK за системния `python3` в `/usr/lib/python3/dist-packages`. Неговите изисквания, `python3-numpy` и `python3-requests`, се препоръчват от пакета: `apt install` ги включва, а `dpkg -i` – не.
* **Virtualenv** не вижда системните пакети; инсталирайте включеното в пакета wheel в него: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` в Linux или `chloros_sdk-*.whl` в папката `sdk` на инсталационната папка за Windows (по подразбиране `C:\Program Files\Chloros\sdk\`).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

На машина без пакет Chloros, чийто SDK управлява бекенд на друга машина (вижте [Какво получава инсталацията само с pip](#what-a-pip-only-install-gets)), инсталирайте го от PyPI: `pip install chloros-sdk`. PyPI се актуализира при излизането на нови версии, така че съответства на най-новата стабилна версия.

**Стъпка 3 — Влезте веднъж на всяка машина:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Удостоверенията се съхраняват в кеш в `~/.chloros/` (и на двете платформи). В Windows можете да влезете по същия начин чрез раздела „Потребител“ <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> на настолното приложение. SDK изисква платен план Chloros+ — вижте [Изисквания за лиценз](#license-requirement) по-долу.

| Изискване | Подробности |
| --- | --- |
| **Инсталиран Chloros** | Windows: инсталатор за настолни компютри; Linux: пакет `.deb` (предоставя бинарния файл на бекенда и SDK) |
| **Python** | 3.7 или по-нова версия (разработен/тестван на 3.10) |
| **Операционна система** | Windows 10/11 64-битова, Ubuntu 22.04 LTS или по-нова версия, или NVIDIA Jetson (JetPack 6) |
| **Лиценз** | Активен акаунт в Chloros+, всеки платен план (Copper или по-висок) |

## Победа за 60 секунди

С едно извикване се създава проект, импортира се папка, конфигурира се обработката и се стартира пипалината — като бекендът се стартира автоматично, ако все още не работи:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(В Linux използвайте Linux пътеки: `/home/user/drone_images/flight001`. SDK работи по същия начин и на двете платформи.)

Обработвате папка със записи от LATTICE? Използвайте LATTICE-съвместимия уиърпър — той прилага подходящите настройки по подразбиране (без откриване на панел-цел, стандартен дебейър):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — пълен контрол над пипалината

За всичко, което надхвърля едноредовата команда, използвайте `ChlorosLocal`. Той използва вече работещ бекенд (на настолното приложение) или стартира инсталирания при първата употреба (`auto_start_backend=True`; в Windows — от папката `backend` в инсталационната папка на Chloros, по подразбиране — `C:\Program Files\Chloros\backend\`), създава и конфигурира проекти, следи напредъка и връща обобщение след приключване на изпълнението.

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

{% hint style="info" %}
Запазете `http://127.0.0.1:5000` по подразбиране, вместо да го замествате с `localhost` — в Windows `localhost` първо се преобразува в `::1` и отнема около 2 секунди на заявка при бекенд, поддържащ само IPv4.
{% endhint %}

Използвайте го като мениджър на контекста. Напускането на блока прекратява използването на бекенда от този екземпляр, а не самия бекенд: той продължава да работи за следващия скрипт и се спира сам след 30 минути бездействие. `shutdown_backend()` спира този екземпляр, след като е стартиран.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` приема следните ключови думи: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` и `custom_settings`. Основните стойности:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Специфичните за LATTICE регулатори (`input_level`, `radiometric_output`, семейството `array_alignment*`) са документирани с пълните си таблици със стойности в [Справочник за SDK](reference/sdk-reference.md#supported-values).

### Наблюдение на напредъка

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Четене на обобщението след изпълнение — и засичане на празни изпълнения

При завършване `process()` прикачва обобщението за обработката на бекенда като `result["summary"]`. Всеки запис в `summary["hints"]` е пълно изречение, което обяснява всичко важно — например, защо даден цикъл не е дал никакъв резултат — и всеки съвет се извежда отново като Python `UserWarning`, така че празните цикли се самодиагностицират, дори ако никога не проверявате речника:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` не генерира изключение, когато изпълнението не създава изображения.** Това е единственото място, където SDK и CLI умишлено се различават: `chloros-cli process` третира „имаше заявка за продукти, но нито един не беше записан“ като грешка и излиза с не нулев резултат, докато SDK-то се връща нормално и докладва състоянието чрез `summary` / подсказки. Ако вашият пиплайн трябва да спре при празно изпълнение, проверете това сами — проверете `summary` (или пребройте файловете в папката на проекта), вместо да разчитате на изключение.
{% endhint %}

## Smart Connect — хардуер в реално време

Три помощни програми отварят постоянни сесии в хардуерния пул на бекенда — същият пул, който използват CLI, други скриптове и разделите за камери в GUI, така че SDK скриптовете съжителстват с тях, без да се конкурират за серийни портове или мрежова пропускателна способност. И трите автоматично стартират локален бекенд, ако такъв не работи. DAQ панелът на GUI не е част от пула: затворете сензора за осветеност в GUI, преди да го отворите от скрипт.

### Една камера LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Относителните пътища сочат към мястото, където се изпълнява вашият скрипт (променено във версия 1.3.0).** При локален бекенд относителният `output_dir` (включително `"output"` по подразбиране) — както и останалите пътища, които бекендът отваря (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — се преобразуват спрямо работната директория на вашия скрипт, така че `cam.capture("output/")` записва в `<current folder>/output/`. В версия 1.2.0 инсталираният бекенд ги преобразуваше в собствената си инсталационна папка и даваше грешка „Достъпът е отказан“. При отдалечен бекенд пътеките се изпращат така, както са въведени.
{% endhint %}

### Синхронизиран масив — `connect_array`

`connect_array` е препоръчителната отправна точка за конфигурации с няколко камери. Той изпълнява същия поток за интелигентна подготовка като графичния интерфейс: мрежов анализ, автоматичен избор на ниво синхронизация, PTP синхронизация на времето, избор на формат на пикселите за всяка камера, задаване на AE и активиране на GPIO тригера. **Първият сериен номер е главният** (той задейства хардуерния тригер); останалите са подчинени.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Добавете `smart=True` към всяко заснемане на масив, за да изчакате автоматичната експозиция да се стабилизира във всички камери, преди да се задейства тригерът.

Новост във версия 1.3.0: `ArraySession` също ви предоставя синхронизирани кадри директно, без да записва файлове:

* `arr.grab()` връща по един кадър на камера, всички от едно задействане, като масиви numpy (`{serial: array}`); `.group_id` е споделеният тригер, а `.errors` посочва всяка камера, която е пропусната.
* `arr.frame_stream(fps=5.0)` предоставя по един такъв набор на всеки тик, за обработка в реално време във вашия собствен поток.
* `arr.continuous("output/")` стартира собственото непрекъснато заснемане на бекенда (режим „Continuous“ на графичния интерфейс) и връща `ContinuousHandle` с `status()`, `wait()` и `stop()` — използвайте го вместо цикъл на запис, когато се налага всеки тригер да бъде запазен с висока честота.

За режимите на запис (Единичен / Непрекъснат / Интервал / Най-бърз), рекордерите, burst-to-video и подреждането на масиви вижте [Справочник за SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### DAQ сензор за осветеност — `connect_daq_sensor`

Без аргументи, `connect_daq_sensor()` автоматично открива транспортния протокол (поредност: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Всеки кадър съдържа 135-точковия `spectrum` (W/m²/nm при калибриране), флаг `is_saturated` и CIE `x`, `y`, `z`. За да зададете конкретен сензор или транспорт, предайте един изричен указател:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Имайте предвид, че профилите за корекция на капацитета (`cap_id`) **не** са параметър на SDK — изберете ги чрез `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Запазени проекти — `open_project`

Запазен проект в Chloros запазва свързания с него хардуер (`cameras.json` + `sensors.json` заедно с `project.json`), а `chloros_sdk.open_project(path)` може да свърже отново камерите, масивите и светлинните сензори наведнъж и да управлява заснемането по име на устройството. Светлинните сензори се свързват отново чрез работещ бекенд на Chloros, като запазват настройките си и капацитета си. Вижте [Автоматизация на проекти](reference/sdk-reference.md#project-automation--chlorosproject) в справочника.

## Какво получава инсталацията само с pip

Проверете флаговете за наличност на ниво модул, преди да използвате хардуерни повърхности:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

На хост с **само** `pip install chloros-sdk` и без десктоп пакет на Chloros:

* `ChlorosLocal`, `process_folder` и `process_lattice_capture` **не** работят — те се нуждаят от бинарния файл на бекенда, който се доставя в инсталатора на десктопа.
* Помощните програми за „smart-connect“ (`connect_camera`, `connect_array`, `connect_daq_sensor`) са чисто HTTP-клиенти, така че работят с бекенд на друга машина — но доставните бекенди се свързват само с лупбек, така че трябва сами да пренасочите порта (например `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) и да предадете `backend_url="http://127.0.0.1:5000"` заедно с `auto_start_backend=False`. Вижте [Режим „Remote-Backend“](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Класовете LATTICE с директен хардуерен достъп (`LatticeCamera`, `CameraPool`, …) се импортират, но се нуждаят от средата за изпълнение на Arena SDK от пакета за настолни компютри — без нея `CAMERA_AVAILABLE` е `False`.
* DAQ сензорите се управляват само чрез (тунелиран) бекенд — `connect_daq_sensor()` или `proj.sensors` на даден проект; бекендът отваря сензора на собствената си машина.

## AI асистенти — `chloros-mcp`

SDK-ът включва `chloros-mcp` — MCP сървър, който позволява на Claude Desktop, Claude Code, Cursor и други MCP клиенти да управляват Chloros: да обработват изображения и да управляват LATTICE камери, масиви и DAQ светлинни сензори. Неговите инструменти обхващат функционалността на този SDK. Той се инсталира заедно с SDK, така че за Claude Code настройката се състои от един ред:

```bash
claude mcp add chloros -- chloros-mcp
```

Други клиенти: [Използване на Chloros с AI асистенти](ai-assistants.md). Опции и всички инструменти: [MCP сървър (AI Assistants)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Изисквания за лиценз

Достъпът до SDK изисква активен профил в Chloros+ на някой от платените планове — **Copper или по-висок**(Copper / Bronze / Silver / Gold); безплатният план „Iron“ няма достъп до SDK/CLI. Проверката се извършва**от страна на сървъра**: всяко SDK заявка трябва да съдържа както активна сесия, така и платен план, в противен случай бекендът връща `403` / `PLAN_UPGRADE_REQUIRED` (генерирани като `ChlorosLicenseError` от `ChlorosLocal` и като `ChlorosConnectError` от помощните функции `connect_*`). Излезлият от системата потребител получава `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — повторното изпълнение на `chloros-cli login` решава първия случай, но не и втория.

Използването офлайн работи в рамките на гратисния период на плана: нивото се чете от кеша за валидиране от сървъра (5 минути) или от кеша за подписани, обвързани с устройството лицензи (30 дни за месечни планове; до изтичане на абонамента за годишните). Когато гратисният период изтече, планът преминава към безплатен и достъпът до SDK се прекъсва, докато устройството не се свърже със сървъра поне веднъж. `chloros-cli status` остава достъпен в безплатния план, така че причината винаги е видима. Вижте [Вход в Chloros+](chloros+-login.md).

## Изключения

Уловете базовия клас, за да обработите „всичко, което е отишло накриво в Chloros“:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

Всички изключения в пипалината (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) произтичат от `ChlorosError`. Едно уточнение: `ChlorosConnectError` — генерира се само от `connect_camera` / `connect_array` / `connect_daq_sensor` — произтича от обикновения `Exception`, **а не** от `ChlorosError`, така че `except ChlorosError` няма да го улови. Пълната йерархия е в [Справочник за SDK](reference/sdk-reference.md#exceptions).

## Вижте също

* [Справочник за SDK](reference/sdk-reference.md) — пълният набор от API функции, оптимизиран за AI асистенти.
* [Справочник за CLI](reference/cli-reference.md) — всяка подкоманда на CLI отразява извикване на SDK.
* [Използване на Chloros с AI асистенти](ai-assistants.md) — рецепти за настройка и подсказки за `chloros-mcp`.
* [Изтегляне](download.md) — инсталатори за Windows и Linux.
