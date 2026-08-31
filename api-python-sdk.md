# API : Python SDK

{% hint style="info" %}
**Търсите пълния API?** Тази страница е практическо ръководство. Всеки публичен клас, метод, точен сигнатура и пример, който може да се копира и постави, се намира в [Справочник за SDK](reference/sdk-reference.md), който е оптимизиран за AI асистенти.**Работите с AI асистент?** Поставете този URL в чата, за да разполага с пълния, актуален Chloros 1.2.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Всяка страница от това ръководство е достъпна като суров Markdown файл под името си с малки букви + `.md`, а цялото ръководство е индексирано на `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` в PyPI) управлява всичко, което десктопното приложение може да прави от Python нататък: пакетна обработка на изображения, управление на камерата LATTICE и масива в реално време, сесии с DAQ светлинни сензори и автоматизация на запазени проекти. Това е тънък слой върху същия локален бекенд, който използват GUI и CLI (HTTP на `127.0.0.1:5000`), така че поведението е идентично и на трите платформи.

## Инсталиране

Инсталирането се извършва в две стъпки: първо се инсталира пакетът за настолни компютри Chloros (той осигурява бекенда за обработка и хардуерните изпълнителни среди), а след това пакетът Python.

**Стъпка 1 — Инсталирайте Chloros.** Windows: стартирайте инсталатора за настолни компютри (по подразбиране `C:\Program Files\MAPIR\Chloros\`) от страницата [Изтегляне](download.md). Linux: инсталирайте пакета `.deb` ([Инсталиране на Linux](linux/linux-installation.md)).**Стъпка 2 — Инсталирайте SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Може дори да не ви е нужен pip: всеки инсталатор съдържа съответстващ SDK wheel. Инсталаторът Windows го инсталира автоматично в системата ви Python; Linux `.deb` го поставя в `/usr/lib/chloros/sdk/` и извежда точната команда `pip install --user`. PyPI се актуализира при излизането на новите версии, така че `pip install chloros-sdk` съответства на най-новата стабилна версия.

**Стъпка 3 — Влезте веднъж на всеки компютър:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Удостоверенията се съхраняват в кеш в `~/.chloros/` (и на двете платформи). В Windows можете да влезете по същия начин чрез раздела „Потребител“ (User) <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> в приложението за настолни компютри. SDK изисква платен план Chloros+ — вижте [Изисквания за лиценз](#license-requirement) по-долу.

| Изискване | Подробности |
| --- | --- |
| **Chloros инсталиран** | Windows: инсталатор за настолен компютър; Linux: `.deb` пакет (предоставя бинарния файл на бекенда) |
| **Python** | 3.7 или по-нова версия (разработено/тествано на 3.10) |
| **Операционна система** | Windows 10/11 64-битова, Ubuntu 22.04 LTS или по-нова версия, или NVIDIA Jetson (JetPack 6) |
| **Лиценз** | Активен профил в Chloros+, всеки платен план (Copper или по-висок) |

## Победа за 60 секунди

С едно извикване се създава проект, импортира се папка, конфигурира се обработката и се стартира пипалината — като се стартира автоматично бекендът, ако такъв все още не работи:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(В Linux използвайте пътеките от Linux: `/home/user/drone_images/flight001`. SDK работи по същия начин и на двете платформи.)

Обработвате папка със записи от LATTICE? Използвайте LATTICE-съвместимия уиндър — той прилага подходящите настройки по подразбиране (без откриване на панел-цел, стандартен дебайер):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — пълен контрол върху процеса

За всичко, което надхвърля едноредов скрипт, използвайте `ChlorosLocal`. Той стартира бекенда при първата употреба (`auto_start_backend=True`), създава и конфигурира проекти, следи напредъка и връща обобщение след приключване на процеса.

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
Запазете стандартния `http://127.0.0.1:5000`, вместо да го замествате с `localhost` — при Windows, `localhost` първо се преобразува в `::1` и отнема ~2 секунди на заявка при бекенд, поддържащ само IPv4.
{% endhint %}

Използвайте го като мениджър на контекста за гарантирано почистване:

```python
import chloros_sdk

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

`configure()` приема следните ключови думи: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` и `custom_settings`. Основните стойности:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
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

### Прочитане на обобщението след изпълнение — и откриване на празни изпълнения

При завършване `process()` прикачва обобщението за обработката на бекенда като `result["summary"]`. Всеки запис в `summary["hints"]` е пълно изречение, което обяснява всичко забележително — например, защо даден цикъл не е дал никакъв резултат — и всеки съвет също се извежда повторно като Python `UserWarning`, така че циклите без резултат се самодиагностицират, дори и да не проверявате речника:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` не се генерира, когато дадено изпълнение не създава изображения.** Това е единственото място, където SDK и CLI умишлено се различават: `chloros-cli process` третира „имаше заявка за продукти, но нито един не беше записан“ като грешка и излиза с резултат, различен от нула, докато SDK се връща нормално и докладва състоянието чрез `summary` / подсказки. Ако вашият пиплайн трябва да спре при празно изпълнение, проверете това сами — прегледайте `summary` (или пребройте файловете в папката на проекта), вместо да разчитате на изключение.
{% endhint %}

## Smart Connect — хардуер в реално време

Три помощни програми отварят постоянни сесии в хардуерния пул на бекенда — същия пул, който използва графичният интерфейс, така че скриптовете SDK съжителстват с настолното приложение, без да се конкурират за серийни портове или мрежова пропускателна способност. И трите автоматично стартират локален бекенд, ако такъв не работи.

### Единична камера LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Синхронизирана матрица — `connect_array`

`connect_array` е препоръчителната отправна точка за системи с няколко камери. Той изпълнява същия поток за интелигентна подготовка като графичния интерфейс: мрежов анализ, автоматичен избор на ниво синхронизация, PTP синхронизация на времето, избор на формат на пикселите за всяка камера, инициализиране на AE и активиране на GPIO тригера. **Първата серийна камера е главната** (тя задейства хардуерния тригер); останалите са подчинени.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Добавете `smart=True` към всяко заснемане на масив, за да изчакате автоматичната експозиция да се стабилизира във всички камери преди задействане. За режимите на заснемане (Единичен / Непрекъснат / Интервал / Най-бърз), рекордери, серийно заснемане във видео и подреждане на масива, вижте [Справочник за SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Светлинен сензор DAQ — `connect_daq_sensor`

Без аргументи, `connect_daq_sensor()` автоматично разпознава транспортния протокол (приоритет: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Всеки кадър съдържа 135-точковия `spectrum` (W/m²/nm при калибриране), флаг `is_saturated` и CIE `x`, `y`, `z`. За да се определи конкретен сензор или транспорт — надеждният избор на хостове с множество мрежови интерфейси, където автоматичното откриване по Ethernet може да пропусне работещ DAQ-E при първия опит — предайте един изричен указател:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Имайте предвид, че профилите за корекция на капацитета (`cap_id`) **не** са параметър от типа SDK — изберете ги чрез `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Запазени проекти — `open_project`

Запазен проект Chloros запазва свързания с него хардуер (`cameras.json` + `sensors.json` заедно с `project.json`), а `chloros_sdk.open_project(path)` може да свърже отново всичко наведнъж и да управлява записването по име на устройство. Вижте [Автоматизация на проекти](reference/sdk-reference.md#project-automation--chlorosproject) в справочника.

## Какво получавате при инсталация само чрез pip

Проверете флаговете за наличност на ниво модул, преди да използвате хардуерни повърхности:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

На хост с **само** `pip install chloros-sdk` и без десктоп пакет Chloros:

* `ChlorosLocal`, `process_folder` и `process_lattice_capture` **не** работят — те се нуждаят от бинарния файл на бекенда, който се доставя в инсталатора за десктоп.
* Помощните програми за „smart-connect“ (`connect_camera`, `connect_array`, `connect_daq_sensor`) са чисто HTTP клиенти, така че работят с бекенд на друга машина — но доставяните бекенди се свързват само с лупбек, така че трябва сами да пренасочите порта (например `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) и да предадете `backend_url="http://127.0.0.1:5000"` заедно с `auto_start_backend=False`. Вижте [Режим „Remote-Backend“](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Класовете LATTICE за директен достъп до хардуера (`LatticeCamera`, `CameraPool`, …) се импортират, но се нуждаят от средата за изпълнение Arena SDK от пакета за настолни компютри — без нея `CAMERA_AVAILABLE` е `False`.
* `daq_sdk` (класовете за директен DAQ) се доставя с инсталацията за настолни компютри, а не с пакета от PyPI, така че `DAQ_AVAILABLE` е `False` на хост, който използва само pip — вместо това управлявайте DAQ сензорите чрез `connect_daq_sensor()` спрямо (тунелиран) бекенд.

## Изисквания за лиценз

Достъпът до SDK изисква активен акаунт за Chloros+ на някой от платените планове — **Copper или по-висок**(Copper / Bronze / Silver / Gold); безплатният план „Iron“ няма достъп до SDK/CLI. Контролът се извършва**от страна на сървъра**: всяко SDK заявка трябва да съдържа както активна сесия, така и платен план, в противен случай бекендът връща `403` / `PLAN_UPGRADE_REQUIRED` (генерирани като `ChlorosLicenseError` от `ChlorosLocal` и като `ChlorosConnectError` от `connect_*` помощните функции). Излезлият от системата потребител получава `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — повторното изпълнение на `chloros-cli login` решава първия случай, но не и втория.

Използването офлайн работи в рамките на гратисния период на плана: нивото се чете от кеша за валидиране от сървъра (5 минути) или от кеша с подписани, обвързани с устройството лицензи (30 дни за месечни планове; до изтичане на абонамента за годишни). Когато гратисният период изтече, планът преминава към безплатен и достъпът с SDK спира, докато устройството не се свърже със сървъра поне веднъж. `chloros-cli status` остава достъпен в безплатния план, така че причината винаги е видима. Вижте [Chloros+ Вход](chloros+-login.md).

## Изключения

Използвайте базовия клас, за да обработвате „всичко, което Chloros е отишло накриво“:

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

Всички изключения от пипалината (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) произтичат от `ChlorosError`. Едно уточнение: `ChlorosConnectError` — генерира се само от `connect_camera` / `connect_array` / `connect_daq_sensor` — произтича от обикновения `Exception`, **а не** от `ChlorosError`, така че `except ChlorosError` няма да го улови. Пълната йерархия е в [Справочник за SDK](reference/sdk-reference.md#exceptions).

## Вижте също

* [SDK Reference](reference/sdk-reference.md) — пълната повърхност API, оптимизирана за AI асистенти.
* [CLI Reference](reference/cli-reference.md) — всяка подкоманда на CLI отразява извикване на SDK.
* [Изтегляне](download.md) — инсталационни програми за Windows и Linux.
