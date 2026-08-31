# CLI : Командна линия

> **Пълно ръководство:**[CLI Reference](reference/cli-reference.md) документира**всеки флаг на всяка подкоманда** и е оптимизирано за AI асистенти — поставете неговия URL във вашия асистент и попитайте за работеща команда: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Съвет за AI инструменти:** всяка страница от това ръководство е достъпна като суров Markdown код, като добавите `.md` към нейния URL (например `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), а `https://mapir.gitbook.io/chloros/llms.txt` индексира цялото ръководство за използване от големи езикови модели (LLM).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## Какво представлява „CLI
“

`chloros-cli` е интерфейсът за командния ред към същия процесор, който използва и настолното приложениеChloros
. Това е „тънък“ клиент (HTTP
) върху бекендаChloros
(локален сървър на `127.0.0.1:5000`) — повечето команди стартират бекенда автоматично, така че един-единствен извикване на `chloros-cli process …` е всичко, от което се нуждае един скрипт.

Работи на **Windows
10/11 (x64)**и**Linux
(x86_64, както и NVIDIA Jetson arm64 на JetPack 6)**, във всеки терминал, без да се изисква графичен интерфейс. Проверете инсталацията си с:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Кратък преглед на групите команди:

* **Обработка и акаунт** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 езика — вижте [Поддържани езици](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (само заLinux
/Jetson)
* **Активен хардуер** — `lattice` (управление на камера LATTICE, над 45 подкоманди), `daq pool-*` (светлинни сензори DAQ), `time-sync` (PTP)
* **Автоматизация** — `project` (задвижване на запазен проект отChloros
без графичен интерфейс, включително YAML рецепти за запис)

Глобални опции, които си струва да знаете: `--port N` (порт на бекенда, по подразбиране `5000`), `-v/--verbose`, `--restart` (принудително рестартиране на бекенда), `--backend-exe PATH`. Вижте [CLI
Справочник](reference/cli-reference.md) за пълния списък.

***

## Инсталиране

CLI
**се доставя в инсталатора наChloros** за всяка платформа — няма отделно изтегляне наCLI
. Изтеглете инсталатора от страницата [Изтегляне](download.md).

###Windows


Инсталаторът поставяCLI
на адрес:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

и добавя тази папка към вашата система `PATH` — **отворете нов терминал**след инсталирането, за да се зареди актуализираният `PATH`. Инсталаторът също така поставя скриптове за стартиране (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) в кореновата директория на инсталацията, както и**Chloros
CLI
** преки път в менюто „Старт“, като всеки от тях отваря терминал с `chloros-cli`, готов за употреба.

###Linux


Инсталирайте `.deb` за вашата архитектура:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Това инсталира `chloros-cli` до `/usr/bin/chloros-cli` (вече на `PATH`) и бекенда до `/usr/lib/chloros/chloros-backend`, заедно с необходимата за камерите LATTICE среда за изпълнение на ArenaSDK
. Вижте [Инсталиране наLinux
](linux/linux-installation.md) за подробности.

### Проверка

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Вход и лицензиране

CLI
(иPython
SDK
) изисква **платен планChloros
+**— всеки платен план го включва; безплатният план не го включва. Ограничението се налага**от страна на сървъра** от бекенда, а не от бинарния файлCLI
: повикване без вход се отхвърля с `401 AUTH_REQUIRED`, а заявката от потребител, влязъл в системата, на безплатния план — с кода `403 PLAN_UPGRADE_REQUIRED`, независимо дали идва от `chloros-cli`,SDK
или ръчно създаден клиентHTTP
. Направете ъпгрейд на [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Влезте **веднъж на всяка машина**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Пароли със специални символи**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` се изкривява от командния интерпретатор (CLI
го открива при код 401 и автоматично повтаря опита, но единичните кавички избягват проблема напълно).
{% endhint %}

Сесията се запазва в кеша в `~/.chloros/user_session.json` и продължава да работи офлайн през гратисния период на плана (30 дни за месечните планове, до изтичане на валидността за годишните). `chloros-cli status` работи дори без платен план, така че причината за отказ винаги е видима.

{% hint style="danger" %}
**Планирате работа в режим „headless“? Първо влезте в системата.**Командите за стартиране на бекенд процеси (`process`, `status`, `export-status`, …) изпълнена**без кеширана сесия**не дава бърза грешка — преминава в интерактивен прозорец `Email:` / `Password:` на stdin. Поради това автоматизирана cron задача или CI стъпка ще**зависне в очакване на входни данни**. Изпълнете `chloros-cli login EMAIL 'PASSWORD'` веднъж на машината, преди да планирате каквото и да било.
{% endhint %}

***

## Първото ви изпълнение на обработката

Насочете `process` към папка със заснети данни — той автоматично откриваSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` или комбинация от тях:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Потокът на напредъка се извежда на живо за всеки поток на тръбопровода (откриване, анализ, обработка, експортиране), а успешното изпълнение приключва с отчитане на броя на записаните изображения (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Къде се съхраняват резултатите

`process` записва в **папка на проекта**, а не във вашата папка с входни данни:

* Без `-o`: проектът се създава в папката по подразбиране за проекти (споделена с графичния интерфейс; управлявайте я с `get-project-folder` / `set-project-folder`, резервен вариант `~/Chloros Projects`), като името му е `-n/--project-name` или времева марка (`YYYYMMDD_HHMMSS`), когато не е посочено.
* С `-o PATH`: тази папка **е** папката на проекта. Ако тя вече съдържа `project.json`, вместо да се презапише, се създава папка със същото име, но с суфикс `_1`/`_2`…

В рамките на проекта продуктите са групирани **по фотоапарат, а след това по формат на файла**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Папката на камерата е `LATT-<sensor>-<lens>-F<filter>` за LATTICE (съответстваща на EXIF данните на заснетия кадър `Model`) и `<model>_<filter>` (например `Survey3N_RGN`) за „Survey3
“. Папката за формат следва `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` или `tiff32` за `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Всеки експортиран продукт запазва името на ИЗТОЧНИКОВИЯ файл.**Експортът на радиантност от `capture_..._raw.tif` все още се нарича `capture_..._raw.tif` — просто се намира в `tiff32/Radiance_Images/`.**Папката идентифицира продукта, а не името на файла**, затова използвайте глобален шаблон за директорията, а не за суфикса `*radiance*`.
{% endhint %}

### Опциите, които действително ще използвате

| Флаг | По подразбиране | Какво прави |
| --- | --- | --- |
| `-o, --output PATH` | папка по подразбиране за проекта | Местоположение на папката на проекта (вижте по-горе). |
| `-n, --project-name NAME` | времева марка | Име на проекта. |
| `--format FMT` | `TIFF (16-bit)` | Едно от `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | няма | Индекси на растителността за експортиране (вижте [Индекси на растителността](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = невронен дебайер, по-бавен, най-високо качество (Chloros
+, NVIDIA GPU). |
| `--vignette / --no-vignette` | включено | Корекция на винет. |
| `--reflectance / --no-reflectance` | включено | Калибриране на отражателната способност; за LATTICE това е и превключвател за продукта на отражателната способност. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Налагане на началната точка на процеса за TIFF файлове на LATTICE. |

За всичко останало — настройка на откриването на цели, PPK, пинове за експозиция, флагове за подреждане на масиви — вижте [раздела `process` от справочника за „CLI
“](reference/cli-reference.md).

***

## Избор на това, което да се експортира (продукти на LATTICE)

Обработката на LATTICE се разпределя към **всеки приложим продукт в един цикъл**. Четирите превключвателя за всеки продукт са**включени по подразбиране**; използвайте формуляра `--no-`, за да изключите един от тях:

| Превключвател | Продукт |
| --- | --- |
| `--debayered` | Линеен демозаик → `Debayered_Images/` |
| `--preview` | Предварителен преглед на дисплея (баланс на бялото + гама; разтягане с фалшиви цветове за мултиспектрални изображения) → `Preview_Images/` |
| `--radiance` | float32 радиантност, W/m²/sr/nm → `Radiance_Images/` (винаги `tiff32/`) |
| `--reflectance` | uint16 отражателна способност, готова за Pix4D → `Reflectance_Calibrated_Images/` |

RGB
главните камери винаги излъчват само дебайерирани данни + предварителен преглед — радиантността/отражателната способност по отделни ленти няма смисъл за широколентов сензор, така че тези превключватели за тях са бездействащи.Survey3
`.raw` игнорира превключвателите и следва стандартния път за отражателна способност/целева стойност.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (по подразбиране `auto`) избира референцията за отражателност: `auto` създава [калибрационна мишена](calibration-targets.md) в кадъра, която отговаря на изискванията за качество като абсолютна референция и преминава към разделянето на низходящото излъчване на светлинния сензор на DAQ (ρ = π·L/E), когато няма цел; `target` е строг (без заместване от DAQ); `daq` се основава на данните от DAQ. Сканиранията на измерваните цели на единица могат да се предоставят с `--target-reflectance-dir`.

{% hint style="info" %}
**Четене на пиксели на отражателната способност:**DN със значение ρ = 1,0 е**за всеки източник** — Файловете LATTICE вграждат `Chloros:PixelScale=32768` в XMP; файловетеSurvey3
използват 65535 (и не съдържат `Chloros:*` тагове). Прочетете тага и разделете на него, вместо да приемате, че е константа. Подробности и един умишлено изключен краен случай без мащаб са в [CLI
Справочник](reference/cli-reference.md).
{% endhint %}

**Обработката винаги започва от `raw`.** Производните продукти (експортирани данни за дебайериране, радиантност и отражателна способност) никога не се връщат обратно в процеса — повторното им импортиране и обработка би довело до двойно прилагане на математиката за калибриране, затоваChloros
ги пропуска и ви уведомява за това. `--input-level` е преднамерено „спасително средство“, когато наистина се налага да се наложи начална точка.

***

## Когато изпълнението се провали

От версия 1.2.0 нататък `process` дава явен сигнал за неуспех, вместо да „успее“ без да покаже нищо:

* Изпълнение, което **е поискало продукти, но не е записало нито един**— само `project.json` и `calibration_data.json` — извежда `Processing finished but wrote no image products.` и**излиза с резултат, различен от нула**, така че скриптовете могат да го засекат. Обичайните причини: входната папка не е била разпозната като заснета (проверете оформлението и `--input-level`) или всеки поискан продукт е бил неприложим за тези камери (например, искане на радиантност/отражателност от камери, които заснемат самоRGB
).
* **Умишлено изпълнение само с метаданни** (всички продукти са изключени, без `--indices`) все пак се счита за успешно — празен изход на изображението е правилният резултат в този случай.
* Изпълнете отново с `--verbose` и проверете лога на бекенда за редове `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, които обясняват пропуските за всяка камера.

Кодове за изход: `0` — успех · `1` — обща грешка · `2` — грешка в аргумента · `130` — прекъсване с Ctrl+C.

***

## Индекси на растителността

Изпълнете `--indices` с едно или повече имена на предварителни настройки; всеки индекс се запазва в собствена папка `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

22-те предварително зададени имена, които `process --indices` приема:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Има три списъка с индекси — не ги бъркайте.**Падащото меню „Настройки на проекта“ в графичния интерфейс съдържа 27 формули (добавя `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — тези пет са само за графичния интерфейс и**не** са валидни за `--indices`). Командата за работа на живо/офлайн `lattice index --preset` използва свой собствен списък с 22 предварителни настройки. Формулите и математическите изчисления за честотните ленти са документирани в [Формули за мултиспектрални индекси](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Светлинни сензори DAQ: Кратък преглед

Серията `daq pool-*` управлява спектралните сензори за събиране на данни (MAPIR
) (DAQ-U през USB, DAQ-M през BLE, DAQ-E през Ethernet) чрез постоянния пул на бекенда — графичният интерфейс (GUI),CLI
иSDK
споделят един и същ дескриптор за работа в реално време. **`pool-*` е поддържаната DAQ пътека в доставяната версия наCLI
**; другите подкоманди `daq`, към които може да видите препратки, са вътрешен източник само заMAPIR
и се излизат с явна грешка, насочваща ви към `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` без `--duration` се изпълнява до `pool-record --stop`; директорията по подразбиране за изходни данни е `~/Documents/DAQ Live View/` **на машината на бекенда**. Профилът за корекция на капацитета се избира при свързване (`--cap-id`, подразбиращ се за бекенда `sunshine_cosine`) и може да бъде сменен на живо с `pool-set-cap` — профилите за капацитет и калибрираният обхват на сензора са разгледани в главите за DAQ на това ръководство.

{% hint style="warning" %}
**DAQ-E на хост с няколко мрежови карти:** първото автоматично откриване на `pool-connect --eth` след стартиране може да се провали дори при изправен сензор. `--eth-host <ip-or-hostname>` е надеждната опция — използвайте я винаги, когато откриването не даде резултат.
{% endhint %}

***

## Камери LATTICE, PTP и автоматизация на проекти

Серията `lattice` (над 45 подкоманди) обхваща цялостната работа с камери LATTICE: откриване, единични заснемания, постоянни синхронизирани масиви с потока за интелигентна подготовка на GUI, преглед на живо в браузъра, подреждане, индексни изчисления и диагностика на хост-NIC. Ето един пример:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Наред с това: `chloros-cli time-sync` предоставя информация за PTP „grandmaster“, който се изпълнява на хоста „Chloros
“ (камерите LATTICE и сензорите DAQ-E работят като „slave“ към него за синхронизирани времеви отметки между устройствата), а `chloros-cli project` отваря запазен проект „Chloros
“ и управлява камерите, масивите и сензорите му без графичен интерфейс — включително скриптови YAML рецепти за заснемане.

Тези три групи (`lattice`, `project`, `daq pool-*`) са и единствените, които поддържат `CHLOROS_BACKEND_URL` за управление на **отдалечен** бекенд; основните команди винаги са насочени към локалната машина.

Пълни постъпкови инструкции можете да намерите в главите за LATTICE в това ръководство; всеки флаг е описан в [CLI
Справочник](reference/cli-reference.md).

***

## Отстраняване на проблеми: Топ 5

| Симптом | Решение |
| --- | --- |
| `Login required` или планирана задача зацикля на подкана `Email:` | Изпълнете `chloros-cli login EMAIL 'PASSWORD'` веднъж на тази машина — команди без кеширана сесия се изпълняват интерактивно, вместо да дават грешка веднага. |
| `backend unreachable` | Стартирайте настолното приложение „Chloros
“ или изпълнете директно бинарния файл на бекенда (`chloros-backend`). Ако насочите `lattice`/`project`/`daq pool-*` към отдалечен бекенд, проверете `CHLOROS_BACKEND_URL`. |
| Свързването с масива е блокирано: `FRAMES WILL DROP` / `Reduce ROI to enable` | Пръстенът за приемане на мрежовата карта на хоста е възстановен към заводските настройки — това е основната причина за отказ на свързване на система, която преди това е работила, обикновено след актуализация на драйвера на мрежовата карта. Изпълнете `chloros-cli lattice network --fix` от терминал с **повишени** права (или задайте `ReceiveBufferLen=256`, `PendingReceives=64`); вижте раздела *Настройка и оптимизация на мрежовата карта на хоста* в справочника. |
| Подкомандата `daq` излиза с съобщение: „изисква пълния DAQ пакет…“ | Очаквано при доставките — компилираниятCLI
съдържа само семейството `daq pool-*`, което обхваща свързване, поток, запис и избор на капацитет. Използвайте `pool-*` (или `chloros_sdk.connect_daq_sensor()` отPython
). |
| Jetson извежда предупреждение за суап преди големи папки | Добавете суап, поддържан от файл —CLI
извежда точните команди `fallocate`/`swapon`, които трябва да се изпълнят. |

***

## Получаване на помощ

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Всяка опция, всяка подкоманда:** [CLI
Справочник](reference/cli-reference.md)
* **Еквивалент вPython
:** [Python
SDK
](api-python-sdk.md) и [SDK
Справочник](reference/sdk-reference.md)
* **Поддръжка:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
