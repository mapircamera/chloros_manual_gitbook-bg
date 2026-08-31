# Chloros Python SDK Референция

**Версия:**

1.2.0**Създадена:**

29.07.2026 г., 19:19 ·**Преработена:**

30.08.2026 г.**Пакет:** `chloros-sdk` (PyPI)**Аудитория:** Оптимизирано за използване от големи езикови модели (LLM); четимо за хора.**Обхват:** Всички публични класове, функции и помощни методи, предоставени от `import chloros_sdk`, с примери, които могат да се копират и поставят, обхващащи обработка на изображения, управление на една камера, синхронизирани масиви, DAQ сензори и автоматизация на проекти.

Ако ви трябват само най-важните моменти, прескочете към:
- [Инсталиране и бързо стартиране](#installation)
- [Smart-Connect за LATTICE масиви](#smart-connect-for-lattice-cameras)
- [Сесии за DAQ сензори](#daq-sensor-sessions)
- [Автоматизация на проекти](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Архитектура за 60 секунди

SDK е тънък слой „Python“ върху бекенда „Chloros“ (същият Flask сървър, който използват десктоп GUI и CLI). За автоматизация импортирате `chloros_sdk` и извиквате методи на високо ниво; „под капака“ всяко извикване се превръща в заявка HTTP към локалния бекенд на порт 5000 — `http://127.0.0.1:5000/api/...` (умишлено не е `localhost`, което първо се преобразува в `::1` на Windows и отнема ~2 s на заявка към бекенд, работещ само с IPv4). Бекендът управлява хардуерния пул — камери, DAQ сензори, профили за изравняване, буфери за кадри — така че скриптовете на SDK могат да съществуват едновременно с графичния интерфейс, без да се конкурират за серийни портове или пропускателна способност на мрежовите карти.

Има три интерфейса, които ще използвате:

1. **`ChlorosLocal` + свободни функции** (`process_folder`, `process_lattice_capture`) — Поток за обработка на изображения. Изпълнете цяла папка през калибриране / дебайериране / експортиране на индекс с едно извикване на Python.
2. **Дръжки за интелигентно свързване** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Отваряне на постоянна сесия в бекенда за хардуер в реално време. Същият поток „smart-prep“, както и в графичния интерфейс: мрежова сонда, автоматичен избор на ниво, PTP, AE сединг, конфигурация на GPIO тригер.
3. **`ChlorosProject` / `open_project`** — Зареждане на запазен проект (папка с `cameras.json` + `sensors.json` + `project.json`), свързва всичко наведнъж и управлява записването чрез именовани дръжки.

Повърхности 1 и 2 **автоматично стартират местен бекенд**, ако такъв все още не е в режим на слушане (същият пакетиран бинарен файл, който GUI/CLI стартират) — така че обикновен скрипт работи от нова терминална среда, без първо да стартирате бекенд. Предайте `auto_start_backend=False`, за да се откажете (например, когато сочите към отдалечен бекенд, който никога не се стартира). Вижте [Автоматично стартиране на бекенда](#backend-auto-start). Повърхност 3 се държи по различен начин: `open_project()` не приема параметъра `auto_start_backend`, а `connect_all()` никога не стартира бекенд — той проверява `http://127.0.0.1:5000` веднъж и, ако нищо не отговори, без предупреждение преминава към директно (без бекенд) управление на устройството `lattice_sdk`. Само `proj.process()` и `stream(..., overlays=True)` създават `ChlorosLocal()` при необходимост (което се стартира автоматично).

И трите изискват-зависими от автентификация: стартирайте `chloros-cli login` веднъж на машината или влезте чрез графичния интерфейс на работния плот. Извиквания на `SDK` без валидна сесия предизвикват грешка `ChlorosAuthenticationError`.

Изисквания:
- Python 3.7+ (съгласно декларацията на пакета; разработен/тестван на 3.10)
- Chloros Desktop, инсталиран локално (бинарният файл на бекенда е включен в инсталатора)
- Активен профил за вход в Chloros+. Минималното ниво за SDK / CLI е **Copper**или по-високо (Copper / Bronze / Silver / Gold); безплатният**Iron**ниво няма достъп до SDK / CLI. Това се прилага**от страна на сървъра**: всяко заявка с флаг SDK / CLI трябва да съдържа както активна сесия, така и платен план, в противен случай бекендът връща `403` с `error_code: PLAN_UPGRADE_REQUIRED` (показан като `ChlorosLicenseError` от `ChlorosLocal`, и като `ChlorosConnectError` от помощните функции `connect_*`). Потребител, който е излязъл от системата, получава `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — двете са различни, защото повторното изпълнение на `chloros-cli login` поправя първата, но не може да поправи втората.
- Използването офлайн се поддържа в рамките на гратисния период на плана: нивото се чете от кеша за валидиране от сървъра (5 минути) или от кеша за подписани, свързани с конкретно устройство лицензи (30 дни за месечни планове, до изтичане на абонамента за годишните). Когато този гратисен период изтече, планът преминава към безплатен и достъпът до SDK / CLI се прекъсва, докато устройството не успее да се свърже със сървъра поне веднъж. `chloros-cli status` (`GET /api/license-status`) остава достъпен в безплатния план, така че причината е видима — това е единственият маршрут SDK / CLI, който е изключен от ограничението на плана.
- Windows 10/11 64-битова версия, **Ubuntu 22.04 LTS или по-нова**, или Jetson (JetPack 6). Ubuntu 20.04**не** се поддържа: зависимостите на `.deb` произтичат от това, към което се свързва бекендът, включително `libc6 (>= 2.34)`, а Focal доставя glibc 2.31.

---

## Инсталиране

Python SDK е тънък слой Python върху бекенда Chloros. За всичко, освен няколко работни потока, предназначени само за DAQ, ви е необходим **Chloros**локално инсталиран десктоп пакет** (инсталатор Windows или Linux `.deb`) — той осигурява бинарния файл на бекенда, средата за изпълнение Arena SDK за камери LATTICE и пакетите за калибриране.

Последни изтегляния: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Стъпка 1 — Инсталиране на пакета за платформата „Chloros“

#### Windows (.exe)

1. Изтеглете `Chloros-Setup-x.y.z.exe` от страницата за изтегляне.
2. Стартирайте инсталатора и следвайте указанията на помощника. Пътят за инсталиране по подразбиране е `C:\Program Files\MAPIR\Chloros\`.
3. Стартирайте Chloros поне веднъж и влезте с вашия акаунт Chloros+.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Стъпка 2 — Инсталирайте Python SDK

**Инсталаторът Chloros включва съответстващ wheel файл SDK.** Всеки инсталатор Windows и .deb файл Linux поставя на диска `chloros_sdk-X.Y.Z-py3-none-any.whl`, който съвпада точно с версията на GUI / CLI / бекенда. Не е необходимо да следите PyPI, за да поддържате синхронизация.

#### Windows

Инсталаторът автоматично стартира `pip install` спрямо приложеното wheel, като използва вашия Python (предпочитан е стартиращият файл `py.exe`, при липса на такъв се използва `python -m pip`). Не се изисква никакво действие — `import chloros_sdk` работи във вашата Python среда след успешна инсталация. Ако на компютъра няма Python, инсталаторът тихо пропуска тази стъпка и GUI + CLI продължават да работят.

#### Linux (.deb)

Файлът .deb поставя пакета wheel в `/usr/lib/chloros/sdk/`. `postinst` отпечатва точната команда — дистрибуциите по PEP 668 по подразбиране отказват глобално записване в pip, затова не извършваме автоматична инсталация:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

За разгръщания на Jetson в изолирана среда това става изцяло офлайн — wheel файлът вече е на диска.

#### Публичен PyPI

За хостове, които използват само pip (без инсталиран десктоп пакет Chloros; работни потоци само с отдалечен бекенд или само с DAQ):

```bash
pip install chloros-sdk
```

PyPI се актуализира при изграждането на инсталаторите за версиите на пускане, така че публикуваният wheel съответства на най-новата стабилна версия. Разработвателските версии (например `1.1.4.dev1`) се доставят само чрез включения в инсталатора „wheel“.

#### Проверка

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Chloros+ се изисква абонамент.** Всички заявки към SDK изискват активен профил в Chloros+. Изпълнете `chloros-cli login user@example.com 'YourPassword'` веднъж на всяка машина; данните за вход се запазват в кеша на `~/.chloros/`.

### Имам ли нужда от пакета за настолни компютри?

Самият pip пакет **не** е достатъчен за повечето работни процеси. Ето какво е необходимо за всяка повърхност на SDK:

| Повърхност наSDK | Необходим ли е Desktop Package? | Защо |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Да** | Автоматично стартира бинарния файл на бекенда в `/usr/lib/chloros/chloros-backend` (Linux) или `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Да**(локално)**/ Не**(отдалечено) | Чисти клиенти „HTTP“ през бекенда. Локален бекенд → необходим е десктоп пакет. Дистанционен бекенд → `backend_url=`**през тунел** (вижте „Режим на дистанционен бекенд“ — доставните бекенди се свързват само с лупбек). |
| `ChlorosProject` / `open_project` | **Да** | Управление на запазени проекти чрез бекенда. |
| Директни LATTICE класове (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Да** | Изисква се родната среда за изпълнение на Arena SDK, която се доставя в пакета за настолни компютри. В противен случай `CAMERA_AVAILABLE` е `False` при импортиране. |
| Директни DAQ класове (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Не** | Чист Python през pyserial/bleak/zeroconf. Среда, използваща само pip, може да управлява DAQ-устройствата от начало до край. |

### Режим „Remote-Backend“ (хост, използващ само pip, през тунел)

> **Доставеният бекенд не е достъпен през LAN.** Производствените
> версии се свързват само с лупбек (и двете групи лупбек) и категорично отказват
> единствения режим, различен от лупбек (`CHLOROS_CLOUD_MODE`), така че
> `backend_url="http://<lan-ip>:5000"` **не може да работи с инсталиран
> Chloros** — този модел е работил единствено с бекенд от типа source/dev
> . За да управлявате бекенд на друга машина, пренасочете сами неговия loopback
> порт и насочете SDK към тунела:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Хостовете без монитор / CI / роботиката могат да запазят една машина с пълна инсталация на десктопа като „Chloros сървър“, а навсякъде другаде да използват `pip install chloros-sdk` — но транспортът между тях е организираният от потребителя тунел по-горе, а не директна LAN връзка URL.

> **Известно ограничение — `ChlorosLocal` не поддържа работа само с pip.** `ChlorosLocal(backend_url=BACKEND)` понастоящем резолвира локален бинарен файл на бекенда в своя конструктор *преди* да провери URL и генерира грешка `ChlorosBackendError` („Не е намерен бекенд Chloros…“) когато няма инсталиран пакет за настолен компютър — дори и при наличие на достъпен отдалечен бекенд. Само интерфейсът за интелигентно свързване, описан по-горе (`connect_camera` / `connect_array` / `connect_daq_sensor`, както и `analyze_array_network` и `list_*` / `discover_*`) работи от хост, на който е инсталиран само pip.

### Работен поток само с DAQ (хост само с pip)

Ако имате нужда само от DAQ сензори и не използвате LATTICE камери или обработка на изображения, пакетът pip е самодостатъчен:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Без бекенд, без .deb, без необходимост от вход в Chloros+ за работа с DAQ директно от хардуера.

---

## Бързо стартиране

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Индекс на най-високо ниво на API

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Обработка на изображения — `ChlorosLocal`

Основният клас за обработка на потоци. Стартира бекенда при първото използване, създава и конфигурира проекти, следи напредъка и връща обобщения след приключване на изпълнението.

### Конструктор

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

### Методи

| Метод | Описание |
| --- | --- |
| `create_project(project_name, camera=None)` | Създава нов проект (по избор с шаблон за камера като `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Импортира изображения във формати RAW/TIF/JPG/DNG **и записи от светлинния сензор `.daq`**. Връща `count` (изображения) и `scan_count` (записи). Извежда предупреждение само ако папката не съдържа нито едното, нито другото. |
| `export_light_sensor(daq=True, csv=True)` | Записва калибрираните `.daq` + `.csv` за всеки запис от светлинния сензор в проекта във файл `<project>/Light Sensor/`X. Вижте [Записи от светлинния сензор](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Настройте регулаторите за обработка. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Стартирайте пипалината. Връща `{"status": "complete", "async": False}`, както и ключ `summary`, когато бекендът предостави такъв — вижте [Обобщение и съвети след изпълнение](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Проверява състоянието на бекенда. |
| `logout()` | Изчиства кешираните удостоверения. |
| `shutdown_backend()` | Прекратете бекенда (ако е стартиран с `SDK`). |
| `discover_cameras()` | Откриване на LATTICE камери **чрез бекенда на този екземпляр** (`/api/camera/discover`). Връща списък от речници (`serial`, `model`, `ip`, …) — със същата структура, каквато виждат GUI/CLI. Празен списък, ако не е намерен такъв или бекендът е недостъпен. |
| `camera_capture(output_dir, format="tiff", **settings)` | Заснемане на един кадър**чрез бекенда**(автоматично стартиран от този хендъл), така че да получи същата подготовка като GUI/ CLI (по подразбиране 12-битов, повторно използване на пула, вградени метаданни за калибриране). Определете целта с `serial=` или `device_index=`; предайте `exposure`/`gain`/`pixel_format`/`preset` като `**settings`. Връща стария речник с метаданни (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Генерира насложени композитни кадри за предварителен преглед от обединена камера — олекотен MJPEG клиент през маршрута `/api/camera/<serial>/stream-annotated` на бекенда (зебра / решетка / мерник / хистограма / пикинг / точка , изчертани от страна на сървъра). `decode=True` предоставя BGR масиви; `False` предоставя необработени JPEG байтове. Достъпно е и по проекти като `ChlorosProject.stream(overlays=True)`. |

Използвайте като мениджър на контекста за гарантирано почистване:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Записи от светлинни сензори — калибрирани `.daq` + `.csv`

DAQ-U / DAQ-M / DAQ-E може да се записва **без** калибрационния си пакет. Това е
това, което публичните [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
записващи устройства (`record_daq.py`) правят по подразбиране: те записват необработените отчитания на сензора и маркират
файла така, че Chloros да извлече фабричната калибрация на този сензор **по сериен номер** — първо от локалния кеш,
след това от облака MAPIR — и да я приложи при импортирането.

Chloros записва резултата обратно като два продукта за всеки запис, под
`<project>/Light Sensor/`:

| Продукт | Какво представлява |
| --- | --- |
| `<name>_calibrated.daq` | Архивът, подлежащ на повторна обработка — със същата схема като запис в реално време, но сега декларира пакета, който го е създал. Повторното му импортиране **не** го калибрира за втори път. |
| `<name>_calibrated.csv` | Спектрална интензивност на излъчването в W/m²/nm по собствената мрежа от дължини на вълните на сензора, по един ред за всяко отчитане, плюс фотометрични колони (обща мощност, фотопични/скотопични луксове, PPFD и неговото разделяне на синьо/зелено/червено, пикова дължина на вълната). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Само за сензори без пакет (DAQ-A).** Сурови спектрални отчитания на сензора — *не* интензитет на излъчването. Вижте по-долу. |

`process()` извършва този експорт като един от етапите си. Той **не** изисква изображения:
самостоятелно летящ сензор за светлина представлява първокласен работен процес, а такъв проект по дефиниция няма
изображения.

**Записите от DAQ-A се експортират като сурови отчитания.** Серията DAQ-A е по-стара от системата с пакети за всеки сериен номер
и няма пакет, който да се извлича — вместо това тя се калибрира на място спрямо
мишена за отражателна способност, поради което никога не е имала нужда от такъв. Тези записи се експортират
под корен `_raw`, а не `_calibrated`: различно име на файла, а не флаг
вътре във файла, тъй като информацията трябва да се запази при изпращане по имейл като голо име. Заглавката
`.csv` посочва `raw spectral sensor counts (NOT irradiance)` и предупреждава, че
стойностите са съпоставими **в рамките** на файла — точно за това ги използва калибрирането, базирано на целеви стойности
— а не между сензори. Фотометричните колони, зависещи от мощността (обща мощност,
фотопичен/скотопичен лукс, PPFD), се връщат като **NULL**, вместо да бъдат интегрирани от отчитанията.

DAQ-U / DAQ-M / DAQ-E, чийто пакет просто не можа да бъде изтеглен, все още се **пропуска**,
а не се записва в суров формат: там пакетът съществува и „пресвързване и преобработка“ е реален съвет.

Старите записи **v1.01 / v1.02** (DAQ-A-SD ги записва) не съдържат епоха за всяко отчитане,
а само времето на запис на файла. Програмата за съпоставяне на изображения↔надолу все още ги отхвърля — съпоставянето на
кадър спрямо времето на запис би било погрешно, без това да се забележи — но експортиращият модул ги чете, а
CSV отпечатва `clock=daq_created_on`, така че продуктът посочва на кой часовник работи.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Запис, чийто калибрационен пакет не може да бъде извлечен (офлайн или сензор без
калибриране в архива), се отчита под `skipped` **с причината**. Той никога не се
записва като „калибриран“ файл, съдържащ необработени отчитания — свържете се с интернет и
изпълнете отново, и експортирането ще приключи.

### Обратни извиквания за напредъка

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Публикуванеизпълнението и съвети

При завършване `process()` извлича `GET /api/processing-summary` и прикачва тялото като `result["summary"]`. Извличането се извършва по принципа „най-добри усилия“ и никога не блокира успешното връщане — ако обобщението не е налично, `process()` преминава към обикновената форма `{"status": "complete", "async": False}`. Всеки запис в `summary["hints"]` — пълни изречения с предложени мерки за отстраняване на проблема, например защо дадено изпълнение е дало нулев резултат — също се-издадена като Python `UserWarning`, така че изпълненията с нулев резултат се самодиагностицират, дори ако никога не проверявате речника:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` е машинно четимата част:

| Ключ | Какво отчита |
| --- | --- |
| `models` | Групи камери в изпълнението. |
| `images_in_groups` | Изходни изображения от тези групи. |
| `targets_found` | Засечени цели за отражателност. |
| `images_calibrated` | Изображения, с които е калибриран цикълът. |
| `exported_files` | **Файлове с изображения, създадени от цикъла.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Записи от светлинния сензор, умишлено преброени отделно — те произхождат от различен етап и съществуват за цикли, при които изобщо няма изображения, така че включването им би направило цикъла само за събиране на данни-само цикъл да изглежда, сякаш е експортирал изображения. |

Наред с тях: `summary["output_dirs"]` (всяка директория, в която е записано),
`summary["light_sensor_export"]`, `summary["stopped"]` (истинско, когато потребителят е прекъснал
изпълнението, така че частичните брояния не се четат като завършено изпълнение с недостатъчна продукция) и
`summary["groups"]` (разбивката по групи).

`exported_files` се записва от конвейера **докато записва**, а не се извлича от
изобразителните обекти на проекта впоследствие. Паралелната и GPU стратегиите създават свои собствени изобразителни
обекти (в работните подпроцеси за GPU пътеките), така че старото сканиране отчиташе
`0 file(s) written` за всяко такова изпълнение и след това излъчваше подсказката за нулеви експорти — при изпълнения,
при които всичко е работило. Ако създадете скрипт въз основа на този номер, едно нормално паралелно изпълнение сега
отчита брой, различен от нула.

Пропуските на сензора за светлина отчитат причината, която четецът действително е установил за всеки файл —
нечетлива схема, липсващ пакет, грешка при запис — **без дублиране**, така че двадесет файла,
, пропуснати поради една и съща причина, се отчитат като една причина, а не като двадесет повторения на нея.

> **`process()` не се генерира, когато изпълнението не създава изображения.** Това е единственото място, където „SDK“ и
> CLI умишлено се различават: `chloros-cli process` третира „имаше заявка за продукти, но нито един не беше
> записан“ като грешка и излиза с ненулев резултат, докато SDK се връща нормално и докладва
> състоянието чрез `summary` / hints. Ако вашият пиплайн трябва да спре при празно изпълнение, проверете го
> сами — проверете `summary` (или пребройте файловете в папката на проекта), вместо да разчитате на
> липсата на изключение. Обичайните причини са папка за входни данни, която не е била разпозната като
> запис, и продукти, пропуснати като неприложими за наличните камери (например радиантност само от камери от типа „RGB“
>).

### Функции за удобство

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Поддържани стойности

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Радиометричен изход (мултиспектрален пиплайн LATTICE)

Нивото на експортиране на мултиспектралните данни (M3C/M3M) от LATTICE в конвейера `process` — `reflectance` (по подразбиране), `radiance`, `sensor-response` или `all` (всеки приложим режим за всяко изображение) — съответства на настройката за обработка **„Радиометричен изход“** на проекта. `configure()` има специално ключово име за това:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Разширената „аварийна врата“ — записването на ключа на проекта `"Radiometric output"` чрез `custom_settings` — все още работи, но имайте предвид, че това замества целия блок с настройки (вижте предупреждението по-долу):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (по подразбиране) разделя радиацията на камерата на **съвпадащия по времеви отпечатък DAQ сигнал**, който се определя автоматично от записан `.daq` (DAQ-U/M/E)**или нативния за DAQ-M `.csv`**, намерен заедно със снимките; всеки пакет за калибриране по камера или DAQ, който липсва локално, се**се изтегля автоматично от AWS** при първата употреба. Файлът „CLI“ представя това като превключватели за всеки тип продукт в `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **заменя** целия блок с изчислени настройки (по проект той заобикаля останалите ключови думи и валидацията на `configure()`). Когато го използвате, включете всеки ключ `Project Settings`, който ви интересува, както е показано в примера по-горе.

---

## Smart-Connect за камери LATTICE

Постоянни сесии на бекенда за хардуер на живо. Използват се същите крайни точки, които използва графичният интерфейс, така че поведението е идентично в SDK / CLI / GUI.

### Една камера — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### Подпис на `connect_camera()`

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### `CameraSession` Методи

| Метод | Описание |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Чете GenICam възли; връща `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Записва възли по дружеско име (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Заснемане на **един** кадър. Връща списък с един елемент, съдържащ речници с метаданни за кадъра. (Заснемането на серии/няколко кадра беше премахнато — извикайте `capture()` в цикъл, ако ви е необходима серия.) |
| `disconnect()` | Освобождава от пула. Няма ефект, ако сме прикачени към вече отворена сесия. |

`capture()` контроли за експортиране (същият модел като масива + GUI):

- `processing` / `levels` — `processing="all"` запазва всеки приложим тип експорт; `levels=["raw","radiance"]` запазва само тези (преопределя `processing`). Пропуснете и двете за използване на подразбиращите се настройки на бекенда.
- `force_daq=True` — запазвате отчетените данни от DAQ/DLS като `.daq` допълнителен файл дори при запис само на необработени данни, така че кадърът да може по-късно да бъде преобразуван в отражателна способност/индекс. Няма ефект, ако няма свързан DAQ.

### Синхронизиран масив — `ArraySession` (Smart-Prep)

`connect_array` е **препоръчителната отправна точка** за конфигурации с няколко камери. Тя изпълнява на заден план пълния GUI поток на Smart-Prep:

1. **Мрежови анализ** (`/api/camera/array/recommend`) — намира най-големия размер на кадър, който се побира в нивото sim-emit, без да се пропускат кадри.
2. **Автоматичен избор на ниво** — `sim-capture-sim-emit`, ако кабелът може да го понесе; в противен случай `sim-capture-ftd-stagger` или `slip-emit-and-capture`.
3. **Автоматично намаляване**— тихо намалява размера на кадрите / увеличава групирането, когато кабелът не може да поддържа изискваната резолюция.**Тази предпазна мярка не обхваща общото пренасищане**: прекалено голям брой камери за кабела не може да се коригира чрез намаляване на кадрите — вижте [Пренасищане](#over-subscription-the-per-cam-floor).
4. **PTP е активиран**по подразбиране — времевите отметки между камерите се синхронизират към един споделен часовник с точност до**~1 ms**. Едновременната експозиция се осигурява от хардуерния тригер на M8 (**&lt; 100 µs** между модулите), а не от PTP: PTP синхронизира *времевите отметки*, а не експозициите.
5. **Автоматичен избор на пикселен формат за всяка камера** — RGB камери → `BayerRG8`, мултиспектрални → `BayerRG12`.
6. **Задаване на AE** — заснема текущото състояние на AE на всяка камера, за да не се нулира експозицията по време на работа.
7. **Конфигурация на GPIO тригера** — `connect_array` активира всяка камера (`TriggerMode=On`, `TriggerSource=Line2`), така че импулсът от главната камера управлява подчинените камери през M8 кабела. Това е стъпка, която се изпълнява само за масива- само стъпка: една камера, отворена с `LatticeCamera`, работи в свободен режим.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### `connect_array()` Сигнатура

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

Стойности на `force_tier`:
- `"sim-capture-sim-emit"` — истинска синхронизация (всички камери задействат сигналите си при един и същ фронт на тактовия импулс).
- `"sim-capture-ftd-stagger"` — гъвкаво разпределение във времевата област (камерите излъчват с леко разминаване във времето, така че пакетите се сериализират по кабела).
- `"slip-emit-and-capture"` — последователно заснемане за всяка камера (без времева синхронизация; единствената опция, когато никой размер на рамката не отговаря на симулацията).

`wire_ceiling_mbps` преопределя **продължителния капацитет на кабела на хоста** в MB/s — единственото
число, от което зависи цялото разпределение на масива. Оставете го `None`, за да се използва автоматично откритата
стойност. Намалете го, когато масивът докладва кадри с повреда по GVSP: автоматичната стойност се изчислява
въз основа на обявената скорост на връзката от мрежовата карта, която надценява USB адаптерите, тънките PCIe линии и
натоварените споделени мрежови структури — а тази надценка се проявява като повредени кадри, а не като
видимо бавна връзка. Стойността се запазва в блока за запис на масива на проекта, така че
повторното отваряне или по-късното задаване на `connect_array` я възстановява като всяка друга настройка на масива.
Вижте [Състояние на масива](#array-health--which-subsystem-is-losing-frames).

#### Превишаване на капацитета (минималният праг на камера)

Регулирането на темпото чрез симулирано излъчване (Sim-emit pacing) разпределя на всяка камера дял от бюджета на кабела, защитен от сблъсъци, с минимална граница от **8 MB/s на камера**(`per_cam_floor_bps`). Веднъж след като `N × floor` надвиши горната граница, защитена от сблъсъци, масивът**превишава капацитета на кабела**— режимът на отказ е загуба на пакети по GVSP, а не по-ниска честота на кадрите — и не съществува решение, свързано с размера на кадрите:**групирането и ROI намаляват броя на байтовете на кадър, а не регулираните байтове в секунда**— това е, което сравнява агрегатната проверка. Практически граници при пълна резолюция на хост с 1 GbE:**6 камери при 1500 MTU, 9 с джъмбо-кадри** (`max_cams_collision_safe` в отговора на анализа посочва границата за вашия кабел). Решения: по-малко камери, джумбо-кадри от край до край или по-бърз мрежов адаптер.

- Отговорите `analyze_array_network()` и `/api/camera/array/connect` съдържат `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` и `per_cam_floor_bps`. Когато `oversubscribed` е true, проекцията **нулира полетата за fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`), вместо да отчита подвеждаща скорост, която е бавна, но все пак работи.
- `POST /api/camera/array/connect` приема параметър в тялото на заявката `pin_resolution` (**само за HTTP — не е ключов аргумент на SDK**; `connect_array` не го излага). Фиксирането премахва предпазната мрежа за постепенно намаляване на биновите размери, така че свръхзаписана връзка с настроено `pin_resolution` се**категорично отхвърля** с грешка, посочваща всички възможни решения. Без фиксиране връзката продължава с постепенното намаляване, но предупреждава, че намаляването не може да изчисти агрегираната стойност.
- Авариен- изход за извънредни ситуации: задайте `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` в средата на бекенда, за да понижите нивото на отказ до силно предупреждение — свързвате се въпреки това и приемате загубата на пакети.

#### Състояние на масива — коя подсистема губи рамки

`GET /api/camera/array/<array_id>/capability` пренася активен блок `health` върху
свързан масив, преоценяван на **10-секундни** прозорец. Той разделя загубата на рамки
на две причини, които изискват противоположни корекции, вместо един „непълен“ процент, който
не посочва нито една от тях:

| Поле | Какво означава | Коя подсистема |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (по сериен номер) | Рамката **е пристигнала и е била структурно повредена**— загуба на пакети в GVSP. |**Мрежа**: капацитет на кабела, темпо, NIC RX пръстен, MTU |
| `never_arrived_rate_pct` (по сериен номер) | Кадърът **изобщо не е пристигнал**— камерата не е задействана или нищо не е излязло от нея. |**Спусък / синхронизация**: M8 кабел, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Най-лошата честота на камерата за всеки от тях. | — |
| `per_cam_rate_pct` | Комбиниран процент на непълни снимки на камера (и двете причини заедно). | — |
| `stable_for_seconds` | Колко дълго всяка камера е оставала под 0,01 %. | — |

Наред с `health`, същият запис отчита числото, от което зависи цялото разпределение:

| Поле | Какво означава |
| --- | --- |
| `wire_ceiling_mbps` | Действащият постоянен кабелен бюджет на хоста, MB/s. |
| `wire_ceiling_source` | Откъде идва тази стойност, с думи — например `USB-capped 200 MB/s (was theoretical 1062; …)` или `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, когато `wire_ceiling_mbps=` го е задал. |
| `nic_is_usb` | `true` за USB Ethernet адаптер. |

Няма SDK обвивка за този ендпойнт — прочетете го директно:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Четене:** `gvsp_corrupt_rate_pct`, различен от нула, при `never_arrived_rate_pct` равен на 0 означава, че
че задействането и синхронизацията на кабела са перфектни и 100 % от загубите са по мрежовия път — намалете
`wire_ceiling_mbps` и се свържете отново. Обратната картина сочи към синхронизационния кабел или
линията за задействане.

> **`target_fps` не е причината за повредените рамки.** Темпото на GevSCPD се задава еднократно при
> свързване, така че намаляването на честотата на задействане променя работния цикъл, а не
> честотата на едновременните импулсни серии. Измереното 5-кратно намаляване на натоварването не доведе до подобрение, докато
> понижаването на максималната скорост на кабела от 240 на 200 MB/s намали процента на повредените данни при същата апаратура от 10,4 % на
> 0,00 %.

> **Автоматичното свиване по време на предаване не е налично във фърмуера на TRI032S.** Работещ масив не може
> да коригира това самостоятелно; прекъснете връзката и я възстановете, за да може модулът за избор на време за свързване да препланира според
> новия таван.

**USB Ethernet адаптерът е ограничен до 200 MB/s** от сондата, независимо от
указаното на табелката: таблицата за ефективност, която превръща скоростта на връзката в устойчива стойност, е
изведена от PCIe, а USB NIC обявява скоростта на Ethernet връзката си, като е ограничен от
USB шината и нейния драйвер. Ограничението е абсолютно, а не относително — USB 1 GbE адаптер
постига ~80 MB/s и не се влияе от това.

#### `ArraySession` Методи

| Метод | Описание |
| --- | --- |
| `status(timeout=10.0)` | Live `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Една синхронизирана група за запис. Връща `CaptureResult` (списък от речници на кадри + `.skipped`). Контроли за експортиране по-долу. |
| `capture(..., smart=True)` | **Интелигентно заснемане** — изчаква AE да се стабилизира на всички камери, след което се задейства. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Най-бързо заснемане: само необработени данни + зададената стойност от DAQ (+ свободният комбиниран индекс). Съответства на бутона „Fastest Capture“ в графичния интерфейс. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Единично / Непрекъснато / Интервално в един ограничен цикъл. Връща `list[CaptureResult]`.**Изисква `count` и/или `duration_s`**, за да може да се прекрати (SDKът няма Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Започва запис на изгледа на комбинирания индекс на живо във видео/GIF → `RecorderHandle`. Един комбиниран рекордер за всеки масив. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Стартира се серия от снимки с висока честота на кадрите-Bayer серия с висока честота на кадрите → `RecorderHandle`. Преобразуване офлайн с `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Офлайн преобразуване на запазена сурова серия в калибрирано видео(и). Блокира до приключване (`wait=True`) и връща `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Проверява офлайн задача за изграждане: `{running, result, error, burst_dir}`. |
| `disconnect()` | Освобождава целия масив. |

`capture()`Контроли за експортиране на  (същият краен пункт, който използват GUI/CLI):

- `processing` / `levels` — `processing="all"` (или `levels=["raw","radiance",…]`) запазва всеки приложим тип експорт за всяка камера; една единствена стойност `processing` запазва само това ниво.
- `aligned=True` — деформира не-суровия експорт на всеки елемент към [профила за подреждане](#array-alignment) на масива (съвместно регистриран); суровите данни остават без деформация, но преобразуването се съдържа в метаданните. Преминава към неподредено състояние (с предупреждение, което се показва в `alignment` на резултата), ако масивът няма профил.
- `render_index=False` — пропуска наслагването на индекса на растителността за всяка камера; по подразбиране го изобразява там, където е конфигурирано.
- `force_daq=True` — запазва присвоеното отчитане на DAQ/DLS като допълнителен файл `.daq`, дори когато никое от избраните нива не го изисква.

**Компресия TIFF (регулатор само за HTTP):**`ArraySession.capture()` не изпраща ключ `compression`, така че се прилага подразбиращото се настройка на бекенда — `POST /api/camera/array/capture` чете параметър от тялото на `compression`, по подразбиране `"deflate"` (zlib L1 без загуба + хоризонтален предиктор, ~4,1 MB на кадър с пълна резолюция). `"none"` записва некомпресирани (~6,3 MB/кадър) с**~5× по-бързо записване** — и двете са без загуба и се четат идентично при импортиране. `SDK` не предоставя ключов аргумент за това; алтернативата е `chloros-cli lattice array-capture --compression none` или суровият HTTP. DEFLATE също задържа GIL на `Python`, така че компресираните записвания не не се паралелизират между нишките за запис на всяка камера — за продължително заснемане с 8 камери при пълна резолюция и скорост на сензора е необходим `compression: "none"`. Подробности: [CLI Справка → array-capture](cli-reference.md).**Преопределяне на експорта за всеки елемент (само HTTP):**същият ендпойнт приема също `exclude_serials` (списък — премахване на елементи от запазения набор; масивът все още се задейства като една синхронизирана група, а изключените елементи се връщат в `excluded`), `serial_levels` (`{serial: [level tokens]}` преопределения на нивопреопределения на ниво камера) и `serial_index` (преопределения на индекс-наслагване за всяка камера по `{serial: bool}`). Това са параметри на тялото, съответстващи на GUI, и**все още не са ключови аргументи от типа „SDK“**; членовете, които липсват в мапите, се заменят с `levels` / `render_index` за целия масив.

##### Проверка на пропуснати камери — `CaptureResult.skipped`

`ArraySession.capture()` връща `CaptureResult`, който е подклас на `list`: итерирайте го, индексирайте го, `len()` го — всеки съществуващ шаблон продължава да работи. Новият код може да провери атрибута `.skipped`, за да види кои камери са били изключени и защо. Най-честият случай е RGB камери в смесенмасив от филтри, когато се изисква `processing="radiance"` или `"reflectance"` — радиантността на базата на Bayer няма смисъл за широколентов сензор, така че бекендът пропуска тези камери, вместо да генерира безсмислици.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Токените за причината следват модела `<level>-not-applicable-to-rgb-cam` (по един запис за всяко пропуснато ниво, като всеки съдържа `level`). Пропуските, свързани с отражателната способност, са `reflectance-skipped-no-fresh-dls` (няма налични нови данни за низходящото излъчване), `reflectance-skipped-bound-daq-unavailable (…)` (не е възможно да се осъществи връзка с свързания DAQ) и `dls-uncalibrated-band-<nm>` — лентата се намира предимно извън радиометрично калибрирания обхват на светлинния сензор на DAQ (~374–974 nm), поради което абсолютната разделителна линия на отражателната способност, базирана на DAQ, се отхвърля и кадърът преминава към режим на работа според отговора на сензора. Сред предлаганите модели само F988 предизвиква това; поддържаният работен процес за тази камера е този с панел за отражателна способност.

Нива на `processing`:

| Ниво | Изход |
| --- | --- |
| `"raw"` | Едноканален Байер (монохромни камери: единична лента) директно от сензора. |
| `"debayered"` *(по подразбиране SDK)* | 3-канален BGR чрез билинеарно демозаициране (монохромни камери: 1-канален в сиви нюанси). |
| `"radiance"` | float32 W/m²/sr/nm чрез пълната радиометрична верига. Само за мултиспектрални камери — камерите от типа „RGB“се пропускат. |
| `"reflectance"` | uint16 0..32768 (съвместимо с Pix4D); изисква свързване с DAQ в реално време за абсолютна референция. Само мултиспектрално. |
| `"display"` | Пълна верига, съответстваща на предварителен преглед в GUI (CCM + WB + гама според профила на камерата). |
| `"all"` | **Един файл за всяко приложимо ниво** за всяка камера (съответстващо на настройката по подразбиране в GUI „Capture All“ / „CLI“). Върнатият `CaptureResult` съдържа по един речник за кадър за всеки `(cam, level)`, като нивото е посочено във всеки речник; неприложимите нива се появяват в `.skipped`. Отчитането от DAQ, използвано за всеки кадър с отражателна способност, се запазва като допълнителен файл `.daq`. |

> **Забележка — стойността по подразбиране се различава от тази в „CLI“.** По подразбиране `ArraySession.capture()` е `processing="debayered"`; по подразбиране командата `chloros-cli lattice array-capture` е `processing="all"`. Предайте `processing="all"` изрично от „SDK“, за да отразите многостепенното запазване на CLI /GUI.

### Режими на заснемане и записващи устройства

Повърхността на масива отразява панела за заснемане на GUI: режими „Единичен“ / „Непрекъснат“ / „Интервал“ / „Най-бърз затвор“, както и две записващи устройства (композитно видео на живо и сурова серия → преобработка офлайн).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**е цикълът „Непрекъснат/Интервал“ на „SDK“. Тъй като няма `Ctrl+C`, с който да го прекъснете от скрипт,**трябва** да предадете `count` и/или `duration_s` (тя спира, когато се достигне някоя от двете). `interval_s` се измерва от началото на всеки цикъл (съответстващо на графичния интерфейс). Останалите kwargs се предават директно към `capture()`.
- **`record`** е от *ниво за мониторинг*: той записва комбинирания индекс в реално време, както се показва на екрана, така че комбинираният поток трябва да е отворен, за да постъпват кадрите. Един записващ модул за композитен сигнал на масив (генерира изключение, ако вече има такъв в действие).
- **`burst` → `build_video`** е *за анализ*: `burst` записва необработени кадри + манифест за всекикадър + по един `.daq` за всяко отделно DLS отчитане под `<output>/bursts/<base>/` при пълната скорост на цикъла на заснемане (без верига, без exiftool, без преглед на живо). `build_video` съпоставя по време всеки кадър към най-близкия `.daq` и изпълнява отново веригата за сияние/отражателна способност/индекс на импортния поток. `products` е списък от `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (по подразбиране: комбинираният индекс). `burst().stop()` също така автоматично стартира изчисление на комбиниран индекс по най-добрия възможен начин, което се връща като `build_job` в крайния резултат.

#### `RecorderHandle`

Връща се от `ArraySession.record()` и `ArraySession.burst()`. Използвайте го като мениджър на контекста за автоматично спиране при излизане от обхвата или го управлявайте ръчно.

| Член | Описание |
| --- | --- |
| `job_id` | Идентификатор на задачата в бекенда (str). |
| `kind` | `"composite"` (от `record`) или `"raw"` (от `burst`). |
| `start_stats` | Речникът, върнат от извикването на `start`. |
| `result` | `None` по време на изпълнение; окончателният речник с резултатите от спирането след спирането. |
| `stats(timeout=10.0)` | Статистика за текущата задача (записани кадри, реализирани кадъра в секунда, изминало време). |
| `stop(timeout=60.0)` | Спира записващото устройство; връща и кешира крайния резултат. Идемпотентно (второ извикване връща кеширания резултат). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Присъединяване към вече свързан масив — `attach_array`

Ако масивът вече е активен (отворен е от графичния интерфейс или предишна сесия на SDK е извикала `connect_array`), използвайте `attach_array`, за да получите идентификатор към него, вместо да се свързвате отново. `connect_array` винаги връща грешка „Камерата <sn>вече е в масива<id>“ в тази ситуация, тъй като POST заявката към `/array/connect` за член на пула не е идемпотентна; `attach_array` чете `/api/camera/array/list` и извършва съвпадение по array_id или по серийни номера.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Шаблон: SDK Скриптовете, които се изпълняват едновременно с графичния интерфейс на работния плот, трябва първо да опитат с `attach_array` и да преминат към `connect_array`, ако все още няма масив в пула.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Важно — излизането от context-manager ДЕЙСТВИТЕЛНО прекъсва връзката.**`ArraySession.disconnect()` винаги изпраща POST към `/array/disconnect`; няма защита „attached-not-owned“, каквато има за `CameraSession` / `DAQSensorSession`. Ако споделяте ресурси с GUI и не искате да разрушите масива при излизане от обхвата,**не използвайте блока `with`** — запазете дръжката в обикновена променлива и пропуснете изричното `disconnect()`:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Помощник за мрежови анализ

Полезен преди отварянето на масива — проверява дали предложените от вас настройки ще са подходящи:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` е един от `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (в противен случай `error`). `auto_capped_fps` означава, че исканата разделителна способност съответства на RX пръстена само при ограничена честота на задействане — запазете резолюцията и преминете от `target_fps=result["recommended"]["recommended_target_fps"]` към `connect_array` (вижте [Пример 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Как да разчитате проекцията** (същият модел като панела „Настройки на масива“ в графичния интерфейс):

- **Серията (`frame_bytes_total`) се сумира за всяка камера поотделно в реалния пикселен формат на всяка камера.**Моно**M3M**камерите предават Mono12 (2 B/px), независимо от стойността на `pixel_format`, която задавате, така че кадър с пълна резолюция от 4 камери е**~25 MB** при три моно камери, а не ~12,6 MB, както би се получило при предположението, че всички са 8-битови. Бекендът определя формата на всяка камера въз основа на нейния модел.
- **Пропускателната способност (`burst_fits_nic_ring`) е съобразена с изчерпването**, а не с цялостния поток спрямо пръстена: симулираното излъчване е подходящо, когато хостът изчерпва RX пръстена по-бързо, отколкото камерите го запълват. Хост 10G + камери 1 GbE**допускат** пълниразделителна способност, дори когато бурстът надхвърля капацитета на пръстена; 1 GbE хост блокира (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` е консервативен таван за последователно извличане** — `max(readout+emit, N×emit)` с излъчване на камера, ограничено до 1 GbE камерен канал, независимо от експозицията. Например ~2,8 кадъра в секунда за масив от 4 камери с пълна резолюция и 12-битова дълбочина (съвпада с измерените от изпълнителната среда ~2,7–3,0). Пълен модел: [CLI Референция → Модел за fps и серийни снимки на масива](cli-reference.md#array-fps--burst-model).
- **Надзаписването (`oversubscribed: true`) означава, че минималната стойност N × на камера надвишава горната граница, осигуряваща защита от сблъсъци** — полетата за кадрите в секунда (`achievable_fps_max` / `fps_bright` / `fps_dark`) показват 0, а автоматичното свиване/групиране не може да реши проблема (те намаляват броя на байтовете на кадър, а не на байтовете в секунда). Решенията са по-малко камери, джъмбо-кадри или по-бърз мрежов адаптер; `max_cams_collision_safe` съобщава горната граница (6 камери с пълна резолюция на 1 GbE при 1500 MTU, 9 с джъмбо). Отговорът съдържа също `aggregate_demand_bps`, `collision_safe_ceiling_bps`, и `per_cam_floor_bps` (8 MB/s). Вижте [Надписание](#over-subscription-the-per-cam-floor).

### Откриване и изброяване

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

Масивите LATTICE изпълняват непрекъсната AE във фонов режим веднага след свързването им, но наскоро насочената сцена отнема малко време, за да се стабилизира. **Smart-capture** е удобна вградена функция: тя проверява експозицията на всяка камера, изчаква, докато масивът се стабилизира в рамките на прозореца, и след това задейства заснемането. Тя е еквивалентна на GUI: бутонът за „умно“ заснемане в настолното приложение извиква същия краен пункт на бекенда.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Когато използвате `ChlorosProject` (следващата секция), получавате повече настройки:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

По подразбиране политиката на „smart-AE“ е консервативна. Затегнете `exposure_tolerance_pct` за прецизна радиометрична работа; разширете я за бързо променящи се сцени, където просто искате „достатъчно близко“.

---

## Сесии на DAQ сензорите

Постоянен пул на бекенда за спектрални сензори (DAQ-U през USB, DAQ-M през BLE, DAQ-E през Ethernet). Отразява повърхността на камерата: smart-detect, повторно използване на пула, идемпотентно свързване.

### Smart-Detect (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Приоритет: Ethernet → BLE → USB. Предайте който и да е един изричен подсказ, за да фиксирате транспорта.

### Фиксиран транспорт

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### Методи на `DAQSensorSession`

| Метод | Описание |
| --- | --- |
| `status(timeout=10.0)` | Обобщение на записите в пула (състояние на стрийминг/запис, диапазон на дължината на вълната, SHA за калибриране, време на интеграция, frame_avg, състояние на AE). |
| `latest(n=1, timeout=10.0)` | Връща до N най-скорошни спектрални кадри. |
| `stream_start()` / `stream_stop()` | Възобновява/спира стрийминга (дескрипторът остава отворен). |
| `record_start(output_dir=None, device_name=None)` | Започва запис на .daq файл. Връща пътя към файла. Отказва се за DAQ-U/M без AWS калибрационен пакет (DAQ-E е изключение). |
| `record_stop()` | Спира записването. Връща `{path, rows}`. |
| `disconnect()` | Освобождава от пула. Няма ефект за прикаченипритежание. |

> **Профилите за корекция на капацитета (`cap_id`) не са регулируеми чрез SDK.** `connect_daq_sensor()` / `DAQSensorSession` не предоставят параметър `cap_id` или метод `set_cap`. Изберете профил за корекция на капацитета на флота чрез CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) или маршрутите „HTTP“ на бекенда (`/api/daq/connect` и `/api/daq/<id>/cap-id` приемат `cap_id`).

### Откриване — намиране на адрес за свързване

`discover_daq_sensors()` сканира USB / BLE / ETH за сензори, които *бихте могли* да отворите. Това е DAQ-аналогът на `discover_lattice_cameras()` и единственият начин да се получи **BLE MAC адресът на DAQ-M** — DAQ-E има име на хост, а DAQ-U — COM порт, но MAC адресът не е отпечатан върху устройството, нито е посочен от операционната система.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Поле | Описание |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM порт / BLE MAC / име на хост — предайте към `connect_daq_sensor` като `port=` / `mac=` / `eth_host=`. |
| `display` | Етикет,разбираем етикет. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` или `None` за порт, който сканирането не може да идентифицира (USB серийните адаптери са неразличими без сонда, затова неизвестните се показват, вместо да се скриват). |
| `extra` | Подробности за всеки транспорт (обявено име на BLE, производител на USB, IP/фърмуер на DAQ-E/…). Празните стойности се пропускат. |

| Параметър | По подразбиране | Описание |
| --- | --- | --- |
| `transports` | и трите | Последователност (или CSV низ), ограничаваща сканирането. Струва си да се зададе, когато знаете какво искате — BLE е бавната част. |
| `scan_timeout` | 5 | Прозорец за сканиране за всеки транспорт в секунди; бекендът ограничава стойността до 1–20. |
| `timeout` | 60.0 | Максимална стойност на „HTTP“ за цялото повикване (както и на други места в „SDK“). |
| `auto_start_backend` | `True` | Стартира локален бекенд, ако такъв не работи. Никога не се стартира за отдалечен `backend_url`. |

> **Сензорите, които вече са отворени в пула, не се показват.** Свързано BLE периферно устройство спира да се рекламира, а отворен COM порт не може да бъде проучен, затова откриването изброява това, което е *налично за свързване*. Очаква се празен резултат веднага след като сте свързали нещо — използвайте `list_daq_sensors()` за това, което вече притежавате. Транспорти, чието сканиране не може да се изпълни (няма инсталиран bleak / zeroconf), се пропускат, вместо да се генерира изключение, така че машина без Bluetooth все пак получава отговорите за USB и ETH.

### Списък

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Съвместно използване с GUI / CLI

Ако GUI вече има отворен сензор, извикването на `connect_daq_sensor(port="COM3")` от Python връща дръжка, маркирана като `already_connected=True`. `disconnect()` на сесията тогава е бездействие, така че скриптът ви SDK не не изважда сензора от GUI при излизане от обхвата.

### Класове за директен достъп до хардуера (без бекенд)

`daq_sdk` се реекспортира от `chloros_sdk`, така че можете да управлявате сензорите от начало до край в рамките на процеса без бекенд:

> **Наличност:**`daq_sdk` се доставя с инсталацията на „Chloros“ за настолни компютри,**но не** с пакета от PyPI — `pip install chloros-sdk` ви предоставя `lattice_sdk`, но изключва `chloros_sdk.DAQ_AVAILABLE == False`. Проверете този флаг, преди да използвате тези класове; на хост, който използва само pip, управлявайте сензора чрез [`connect_daq_sensor()`](#daq-sensor-sessions), който не изисква локални транспортни библиотеки.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Предпочитайте пътя за интелигентно свързване (`connect_daq_sensor`), когато искате споделено притежание с графичния интерфейс; използвайте класовете за директно свързване за скриптове без графичен интерфейс, които притежават сензора изключително.

---

## Автоматизация на проекти — `ChlorosProject`

Запазен проект на „Chloros“ е папка, съдържаща `cameras.json` + `sensors.json` + `project.json`. `open_project` зарежда манифеста, а `connect_all` свързва онлайн всяко запазено устройство със запазените му настройки — същото състояние на хардуера, което би се получило при използване на графичния интерфейс.

### Минимален пример

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Или като мениджър на контекста:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Методи на `ChlorosProject`

| Метод | Описание |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Открива и свързва всяко запаметено устройство. Връща отчет за свързване по класове. Използва работещ бекенд, когато такъв е в режим на слушане на `127.0.0.1:5000`; в противен случай без предупреждение преминава към директно (без-free) управление на устройствата по `lattice_sdk` — никога не създава бекенд. |
| `disconnect_all()` | Прекъсва всички връзки. |
| `capture_all(output_dir=".")` | Един кадър от всяка камера + масив + спектър от всеки сензор. |
| `stream(camera, overlays=False, fps=10.0)` | Генератор, генериращ BGR `numpy` кадри от камера (или масив) с име. `overlays=False` е пряк `lattice_sdk` цикъл за заснемане (масивите генерират `{serial: frame}` речници). `overlays=True` преминава през `ChlorosLocal.camera_stream()` →`/api/camera/<serial>/stream-annotated` MJPEG поток, като запазеният `ui.overlay` блок на камерата се предава като параметри на заявката. Изисква режим на бекенда и **самостоятелна камера**: камера в директен режим генерира `RuntimeError` (бекендът не може да вземе камера, която принадлежи на този процес), а масив генерира `NotImplementedError` (наслагва композитен поток за всяка камера — излъчва елемент по име). Еквивалент за еднократно изпълнение: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Извършва подреждане на всеки масив, който е свързан в момента. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Извършва калибриране / индексиране на изображенията на проекта (wra`ChlorosLocal.process`; тези четири са **единствените** допустими ключови аргументи — `indices=` и т.н. предизвиква `TypeError`; задава индекси чрез `ChlorosLocal.configure()`). Изгражда по-късно `ChlorosLocal()`, което автоматично стартира бекенд. |

Атрибути:
- `proj.cameras` — `Dict[str, CameraHandle]` с ключ по име И сериен номер.
- `proj.arrays` — `Dict[str, ArrayHandle]`, индексиран по име И array_id.
- `proj.sensors` — `Dict[str, SensorHandle]`, индексиран по име И slot_id.
- `proj.config` — речник `project.json["config"]`.

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Нива на обработка.** `capture()`, `grab()` и `frame_stream()` приемат един и същ `processing`
токен, а веригата е кумулативна — всяко ниво изпълнява всичко, което е над него:

| Ниво | Изход | Бележки |
| --- | --- | --- |
| `raw` | 1-канален Bayer, роден за сензора | Без демозаика. Наслагванията не са достъпни на това ниво. |
| `debayered` | 3-канален BGR (**по подразбиране**) | Билинеен демозаик. Единственото ниво, което работи без режим „backend“. |
| `radiance` | float32, W/m²/sr/nm | Пълна радиометрична верига: демозаика + 3×3 разделяне (мултиспектрално) + DSNU + плоско поле + NIST скала, с извадена експозиция × усилване, така че стойностите са абсолютни. |
| `reflectance` | uint16, 32768 = 1,0 | Радианс, разделен на низходящата интензивност на слънчевото лъчение (ρ = π·L/E). Необходимо е отчитане от DLS/DAQ — вижте бележката по-долу. |
| `display` | 8-битов, подобен на sRGB | Рендериране, еквивалентно на GUI: CCM + баланс на бялото + гама чрез активния цветови профил на камерата. |

Всичко, различно от `debayered` изисква режим „backend“; камера в директен режим генерира
`NotImplementedError`. `reflectance` изисква използваема стойност на интензивността на слънчевата радиация — крайната точка на кадъра автоматично извлича
обединените данни от DAQ вDLS слот на камерата автоматично, но без обвързан DAQ веригата отхвърля
изхода на отражателната способност и честно отбелязва понижението в върнатите метаданни, вместо тихо
да върне продукт с по-ниско качество.

> **Скала на отражателната способност DN — не я кодирайте твърдо.** Отражателната способност на LATTICE използва `32768` = ρ 1.0 и отбелязва
> XMP `Chloros:PixelScale=32768`; Survey3 отражателната способност използва `65535` = ρ 1.0 и не съдържа
> `Chloros:*` тагове. Прочетете тага и разделете на него. Той е дефиниран в домейна uint16, така че остава
> `32768` за всеки формат, който премащабира (16-битов TIFF, 8-битов PNG /JPG, 32-битов процент) — първо нормализирайте
> съхранения тип данни обратно към uint16 (×257 от 8-битов, ×65535 от float). Единственото изключение:
> запис от 8-битов източник, записан като 8-битов TIFF, се *орязва*, а не се премащабира, така че няма мащаб, който да го описва
> — Chloros изцяло пропуска `PixelScale` и туплата на MicaSense в този случай. Третирайте липсващ
> таг във файл с отражателна способност на LATTICE като „липса на валиден мащаб“, а не като стойност по подразбиране.

> **EXIF се пренася при експорта.** `process()` копира GPS блока на изходния кадър
> **и неговия ExifIFD** във всеки продукт, така че експортираните файлове съдържат `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` и `CameraSerialNumber`, както и
> геореференцирането. `FocalLength` е това, въз основа на което Pix4D изчислява разстоянието между точките на земната повърхност — без него
> реконструкцията се връща към крайно неправилен мащаб (в един измерен случай обект с дължина 411 м
> в такъв с размер 47,8 км). Копието умишлено не е `-all:all`: структурните тагове на IFD0 нарушават
> изхода на LATTICE, а `ExifImageWidth`/`Height` са изключени, тъй като описват изходния
> запис, а не експортирания растер.

Подфлагове на етапа на заснемане (прилагат се към радиометричните нива — `radiance`, `reflectance`, `display`):

| Флаг | По подразбиране | Значение |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + плоско поле + 3x3 разграждане + радиометрична скала на NIST. |
| `apply_white_balance` | `True` | WB LUT. DLS-съвместимост, когато DAQ е свързан с камерата. |
| `apply_index` | `False` | Оценка на растителния индекс. |
| `index_expression` | `None` | Заместване на формулата. Непразна → автоматично-активира индекса. |
| `annotated` | `False` | Наслагване на GUI декорации (зебра/решетка/пикове). Не е налично за `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Типът на връщаната стойност е `CapturePathMap`, а не `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` е `Dict[str, Union[str, List[str]]]`: еднонивова
> `processing` предоставя на всеки сериен номер по един път, докато многостепенният (`"all"` или
> изричен списък `levels`) му предоставя **подреден списък** с всеки продукт, запазен за тази
> камера. Комбиниран композитен сигнал на живо, ако такъв се излъчва, пристига под допълнителния
> ключ `"combined"`, а не под сериен номер. Кодът, който разчита на `str`, дава грешка при
> списъчната форма, без нито един обект за проверка на типовете да възрази — анотацията посочваше `Dict[str, str]`
> за известно време след пускането на формата на списъка, поради което съществува този псевдоним. Нормализирайте
> когато искате плоската форма:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Подреждане на масиви

`ArrayHandle` предоставя пълния обхват на подреждането. По подразбиране профилите са валидни само за сесията — извикайте изрично `export_alignment()`, за да се запазят.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Изравняване при свързване

`connect_all(align=...)` може автоматично да изравни всеки масив при свързване:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Когато не е посочено друго, се използва `project.json["config"]["auto_align_on_connect"]`.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Директен хардуер (без бекенд)

Когато искате нулева зависимост от бекенда (CI, роботи без графичен интерфейс, вградени системи), импортирайте директно `lattice_sdk` и `daq_sdk` — и двете се реекспортират от `chloros_sdk`. Защита на `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` се намира в пакета PyPI (но изисква наличието на средата за изпълнение Arena SDK), докато `daq_sdk` се доставя само с инсталацията за настолни компютри.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Предварителни настройки и тригер

Три от четирите предварителни настройки са в режим **free-run**: камерата експонира непрекъснато и
`capture()` връща следващия кадър. `triggered` е изключението — той активира
камерата за хардуерен фронт по линия 2, така че не заснема нищо, докато такъв не се появи.

| Предварителна настройка | Тригер | Използвайте я, когато |
| --- | --- | --- |
| `default` | свободен режим | общо предназначение |
| `high_speed` | свободен режим | 8-битов, ограничение 60 кадъра в секунда, кратка експозиция |
| `high_quality` | свободен режим | 12-битов, без ограничение на кадрите в секунда — обичайният избор за снимки |
| `triggered` | **в готовност, линия 2** | камерата е свързана с M8 синхронизиращ кабел и се задейства от нещо друго |

Ако изберете `triggered` (или настроите сами `trigger_mode="On"`), без нищо
да задейства Линия 2, всеки `capture()` ще изтече по време — правилно, тъй като сте поискали
от камерата да изчака. „SDK“ обяснява това, когато се случи; вижте
[SC_ERR_TIMEOUT по време на заснемане](#direct-hardware-backend-free).

> **Забележка — &quot;GVSP probe“ / `SC_ERR_TIMEOUT -1011` съобщенията при свързване не са грешки.**&gt; При свързване SDK се опитва да договори**джумбо-рамки** (GVSP пакети от 9000 байта) за по-висока пропускателна способност. При директна връзка „точка-до-точка“ между мрежови карти (например адрес `169.254.x.x` от типа „link-local“) мрежата обикновено не може да пренася джъмбо-рамки, така че тази проверка изтича и в лога се записват редове като:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Това е **предвиденият резервен вариант**: „SDK“ автоматично преминава към стандартни пакети от 1500 байта и камерата продължава да се свързва нормално (редовете `[chunk-enable …]`, които следват, са част от нормалната последователност на свързване). Записът все още работи.
>
> Можете да пропуснете тази проверка, но **тя не е просто средство за заглушаване на логовете — тя изключва джъмбо-кадрите.** Камерата отговаря на ping-ове с опция „Don&#x27;t-Fragment“ само до 1500 байта, независимо колко добра е мрежата ви, така че самият ping-тест никога не може да открие джъмбо-кадри; тази проверка е единственото нещо, което може да го направи. Деактивирайте го и камерата ще използва стандартни пакети от 1500 байта завинаги, във всяка мрежа:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Струва си само в мрежа, за която *знаете*, че не поддържа джъмбо-пакети, където това спестява приблизително една секунда от времето за свързване на всяка камера. Тъй като това е реална, а не само козметична промяна, приложението „SDK“ вече ви уведомява за това, когато го използвате:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Не го пипайте, освен ако нямате основателна причина.** Ако остане активирана, при всяко свързване се измерва отново реалната ви мрежа: свържете се към комутатор, поддържащ джъмбо пакети, и при следващото свързване джъмбо пакетите се активират автоматично, без да се налага конфигуриране или рестартиране.
>
> Ако *искате* пропускателна способност за джъмбо пакети, активирайте джъмбо от край до край (MTU на мрежовата карта 9000 + суич, който ги пропуска), или го фиксирайте с `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, когато знаете, че връзката го поддържа — макар че е по-добре да използвате `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` за всяка команда, вместо да го настройвате постоянно, тъй като фиксираният размер прескача проучването и спира адаптирането към мрежата пред него. **Всяко** устройство по пътя трябва да пропуска джъмбо пакети — включително всеки PoE сплитер или инжектор, което е обичайната причина, поради която иначе поддържаща джъмбо пакети конфигурация не може да ги пренася.

> **`SC_ERR_TIMEOUT -1011` по време на `capture()` / `grab*()` е различен проблем — това е истинска грешка.**&gt; Горната бележка се отнася само за `-1011`, регистрирано от**сондата за време на свързване**. Същата грешка, възникнала при**запис**, означава, че камерата се е свързала успешно, но не изпраща никакви изображения:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Показател за това е камера, чийто *контролен* канал е в изправност — откриването работи, настройките и записите на `[chunk-enable …]` са успешни — докато *всеки* кадър изтича в тайм-аут.
>
> **Обичайната причина е, че камерата е настроена за хардуерен тригер.** При `trigger_mode="On"` и `trigger_source="Line2"`, камерата не излъчва нищо, докато не се появи електрически импулс по синхронизиращия кабел M8. Ако нямате кабел, който да задвижва тази линия, всяко заснемане чака безкрайно. Камерата не е повредена и мрежата работи нормално — тя прави точно това, което ѝ е казано.
>
> `CameraSettings()` и предварителните настройки `default` / `high_speed` / `high_quality` работят в режим „free-run“, а заснемането, което изтича докато е активирана, дава обяснение, вместо да извежда само `-1011`. `PRESETS["triggered"]` активира Line2, съгласно дизайна.
>
> За да принудите която и да е камера да работи в режим „freeработа в свободен режим:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Ако все още изтича времето с `trigger_mode="Off"`, това означава, че камерата наистина не предава данни — изпратете ни лога и `ip link show`.

#### Цветови профили (преглед на живо в RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` избира цветовия профил на дисплея за **прегледа на живо** на камерите RGB (мултиспектралните камери игнорират тази настройка):

| Профил | Значение |
| --- | --- |
| `raw` | Изцяло прескача радиометричната верига. |
| `linear` | DSNU + flat + WB, без CCM, без гама. |
| `natural` | Линеен + измерен CCM + sRGB гама, само с евтиния финал (изглаждане на хроматичността + десатурация на светлите области) — реалистичната настройка по подразбиране. |
| `enhanced` | `natural` плюс пълната обработка „hub-parity“ (премахване на ореолите, вибранс, локален контраст CLAHE). По-богат вид при приблизително **двойна цена на обработката на кадър**, което води до по-ниска честота на кадрите в реално време. |
| `custom_temp` | `natural`, но балансът на бялото е фиксиран към `custom_cct_k` Келвин (DLS се игнорира; фиксиран в диапазона 2000–10000 K от страна на бекенда). |

Профилът е **само за предварителен преглед на живо** регулатор на скоростта/визуалния вид: запазените кадри винаги получават пълния богат финал, независимо от избрания профил, така че изборът на `natural` с цел спестяване на време за кадър не понижава качеството на това, което се записва на диска. Неизвестен профил повишава `ValueError`; когато бекенд е достъпен, промяната също се изпраща към него чрез POST, така че следващият кадър за преглед да я отрази (потребителите на direct-SDK, които нямат бекенд, също получават промяната в настройките).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Моно (M3M) камери и `Calibration`

Моно **M3M** камерата (`M3M-<lens>-F<wavelength>`) е еднолентова: една равнина в сиви нюанси, без мозайка на Байер, без 3×3 матрица за спектрално пресичане. `Calibration` я разпознава и извежда флаг `is_mono`. Отражателната способност все още се прилага като радиометрична карта за всеки канал (разделянето е идентична матрица), но многолентовите изчисления на една камера дават смислен резултат, вместо да връщат безсмислици:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

За да се изчисли растителен индекс от монохромен хардуер, комбинирайте няколко M3M камери с различни дължини на вълната в подреден многолентов стек (вижте [Изравняване на масиви](#array-alignment)) и изчислете индекса върху този стек, вместо върху една камера.

Директен режим на DAQ:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **`apply_sensor_settings` приети ключове**— точно `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; вече не се използва, заменен е с `cap_id`), `filter_model` (DAQ-M) и `cap_id` (всички видове DAQ; `None`/`""`/`"none"` = чист сензор, без корекция на капацитета). Неизвестните ключове се**игнорират без предупреждение** — например `{"integration_time": 64}` не прави нищо (трябва да бъде `integration_time_ms`). Връща `{"applied": [...], "errors": {...}}` и никога не генерира изключение.

`chloros_sdk` реекспортира само основната повърхност, използвана по-горе. Пълният публичен API на `daq_sdk` (22 имена) добавя следното — импортирайте ги директно от `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Изключения

Уловете базовия клас, за да се справите с „всичко, което е отишло накриво в Chloros“:

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

> `ChlorosAuthenticationError` и `ChlorosConfigurationError` се експортират на най-високо ниво заедно с останалите; те също могат да се импортират от `chloros_sdk.exceptions`, както е показано.

Йерархия:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Примери от начало до край

### 1. Обработка на папка с персонализирана лента за напредък

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Масив LATTICE в реално време → Отражателна способност + DAQ референция

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Кампания за заснемане, ръководена от проект

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Поток от кадри от няколко камери → NumPy тръбопровод

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Скрипт за заснемане без интерфейс, директно към хардуера (без бекенд)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Проверка на възможностите преди свързване на масив от 4 камери

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Еквивалент на рецепта за запис (чист Python)

DSL рецептата на „CLI“ има пряк еквивалент в „Python“:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Автоматично стартиране на бекенда

Входните точки за „smart-connect“ — `connect_camera`, `connect_array`, `connect_daq_sensor` и `discover_lattice_cameras` — са „тънки“ HTTP клиенти, които предполагат, че бекендът слуша на `127.0.0.1:5000` (URL по подразбиране на Smart-Connect повърхността). Когато GUI или CLI вече се изпълняват, такъв вече съществува. При стартиране от обикновен скрипт може да няма такъв — затова тези функции **автоматично стартират прилагаемия бинарен файл на бекенда** (без прозорец, по същия начин, по който го прави `ChlorosLocal`) преди първото си извикване, след което изчакват до `backend_startup_timeout`, докато той се стартира.

Правила:

- **Само URL се стартира.** `backend_url`, сочещ към `localhost` / `127.0.0.1` / `[::1]`, е допустим; всеки друг хост се приема за машина на някой друг машина и никога не се стартира.
- **Бекендът остава да работи за повторна употреба** (също като при „CLI“) — няма имплицитно изключване, когато скриптът ви приключи. При повторно изпълнение на скрипта се използва същият активен бекенд.
- **Изключете тази опция с `auto_start_backend=False`** при което и да е от тези извиквания (например, когато сте посочили отдалечен бекенд или сами управлявате жизнения цикъл на бекенда).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Ако придружаващият бинарен файл не може да бъде намерен или стартиран, последващото извикване на HTTP генерира подлежащо на действие, **съобразено с платформата** изключение `ChlorosConnectError`, вместо просто трасиране на отхвърлена връзка— на Windows ви насочва към настолното приложение или към командата `chloros-cli`; на Linux (без графичен интерфейс) ви насочва към командата `chloros-cli` или към `.deb`.

---

## Среда и заглавни файлове

SDK маркира всяко извикване на бекенда HTTP с `X-Chloros-Client: sdk`. Бекендът прилага лицензионните правила на SDK / CLI (изисква се вход **и** платен план Chloros+), а не безплатния план на графичния интерфейс. Това се настройва автоматично при импортирането — не е необходимо да правите нищо.

`http://localhost` и `http://127.0.0.1` се разпознават като локален бекенд. Извикванията към други хостове (например към вашата собствена аналитична услуга) остават непроменени.

Заменете бекенда URL, като предадете `backend_url=` (или `api_url=` на `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(`backend_url`, който не е loopback, достига само до бекенд от типа source/dev — доставните бекенди се свързват само с loopback; вижте „Режим на отдалечен бекенд“ за модела на тунела.)

---

## Версии и съвместимост

- Версията „SDK“ се представя като `chloros_sdk.__version__`.
- „SDK“ обвързва поведението с версията на включения в пакета бекенд. Комбинирането на по-стар SDK с по-нов бекенд обикновено работи (съвместими с бъдещи версии крайни точки), но комбинирането на по-нов SDK с по-стар бекенд може да доведе до грешки `404` на новите крайни точки — актуализирайте настолното приложение, за да съответства.
- Повърхността на smart-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) и крайната точка за мрежови анализ връщат стабилни схеми JSON; новите полета са допълнителни.

---

## Съвети за отстраняване на проблеми

- **`ChlorosAuthenticationError: Login required`** → Изпълнете `chloros-cli login EMAIL PASSWORD` веднъж на този компютър или влезте чрез настолното приложение „Chloros“.
- **`ChlorosConnectError: No Chloros backend is running …`** → Функцията „Smart-Connect“ стартира автоматично локален бекенд, така че това съобщение се появява само когато включеният бинарен файл не можебъде намерен/стартиран (например хост, който използва само pip и няма пакет за настолен компютър). Съобщението е специфично за платформата: на Windows отворете приложението за настолен компютър или изпълнете някоя от командите `chloros-cli`; на Linux изпълнете командата `chloros-cli` (няма графичен интерфейс) или инсталирайте `.deb`. За отдалечен бекенд предайте `backend_url=` (и `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** при импортиране → `lattice_sdk` не успя да се зареди (обикновено DLL-файловете за изпълнение на Arena SDK не са инсталирани). Повърхността без камера все още работи.
- **Array connect връща резолюция, по-ниска от родната**→ Функциятаавтоматично намалява размера на кадъра, за да се побере в кабела. Използвайте `analyze_array_network()`, за да разберете защо, след което или надградете връзката, или приемете намаляването, или предайте `force_tier="slip-emit-and-capture"` за последователно заснемане. Системата за безопасност при намаляване**не** обхваща общата свръхподписка (`oversubscribed: true`, полета за fps 0): твърде много камери за канала не могат да бъдат коригирани чрез бининг/ROI — намалете броя на камерите, активирайте джъмбо рамки или преминете към по-бърз мрежов адаптер (вижте [Превишаване на капацитета](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` съобщава, че приемният пръстен на мрежовата карта е много малък (~0,26 MB) / свържете гейтовете с „FRAMES WILL DROP“** → Приемният пръстен на мрежовата карта на хоста е на стойността по подразбиране (често се нулира до 32 след актуализация на драйвера на мрежовата карта). При набор от USB 10GbE адаптер задайте `ReceiveBufferLen=256` и `PendingReceives=64` (с повишени права), след което рестартирайте бекенда, за да прочете отново пръстена. Пълна процедура: [CLI Референция → Настройка и оптимизация на хост NIC](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Хостът забива при рестартиране/изключване, по-късно се появяват WMI грешки `Invalid class` / мрежовата карта не се активира** → Остарял драйвер за USB 10GbE, причиняващ грешка `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Актуализирайте драйвера на адаптера до актуална версия (≥ 2026) и приложете отново настройките на приемния пръстен. Вижте [CLI Справочник → Настройка и оптимизация на мрежовата карта на хоста](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Отражателността отказана** → За отражателна способност в абсолютна скала към камерата (или матрицата) трябва да бъде свързан активен DAQ. Свържете го чрез графичния интерфейс или използвайте `processing="radiance"` (W/m²/sr/nm), което не изисква сдвоен сензор.
- **Записът с `smart=True` отнема повече време от очакваното** → Сближаването на AE зависи от динамиката на сцената; увеличете `exposure_tolerance_pct` или съкратете `stability_window_s`, ако искате по-бърз (по-малко стабилен) тригер.

---

## Вижте също

- [Справка за CLI](cli-reference.md) — всяка подкоманда CLI съответства на извикване на SDK.
- [Ръководство за DAQ сензори](../daq/README.md) — правила за свързване, калибриране и запис, специфични за всеки сензор.
- Онлайн документация: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
