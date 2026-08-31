# Chloros CLI Референция

**Версия:**

1.2.0**Създаден:**

29.07.2026 г., 19:19 ·**Преработен:**

30.08.2026 г.**Аудитория:** Оптимизиран за използване от големи езикови модели (LLM); четим от хора.**Обхват:** Всяка подкоманда на `chloros-cli`, предназначена за потребители, с опции и примери, които могат да се копират и поставят.

Настоящият документ представлява пълното справочно ръководство за инструмента за командния ред `chloros-cli`, който се доставя с MAPIR Chloros. Той е умишлено изчерпателен, така че LLM (или човек) да може да състави всеки поддържан работен поток от изброените по-долу команди, без да проверява изходния код.

Ако ви трябват само най-важните моменти, прескочете към:
- [Бързо начало за пет минути](#five-minute-quickstart)
- [Работен поток за първо свързване на камера LATTICE](#lattice-camera-first-connect-workflow)
- [Работен поток за първо свързване на DAQ сензор](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Режими на заснемане, записващи устройства и офлайн преобработка](#capture-modes-recorders--offline-reprocess)

---

## Конвенции

- Всички команди започват с префикса `chloros-cli`. При Windows бинарният файл е `chloros-cli.exe`; при Linux/Jetson той е `chloros-cli`.
- Незадължителните аргументи са показани като `--flag`. Задължителните позиционни аргументи се показват без скоби.
- Когато е зададена стойност по подразбиране, пропускането на флага води до използването на тази стойност.
- CLI е „тънък“ клиент на HTTP върху Chloros бекенд (Flask сървър на `127.0.0.1:5000`). Бекендът се стартира автоматично от повечето команди. `CHLOROS_BACKEND_URL=<url>` насочва семействата команди **`lattice`**,**`project`**и**`daq pool-*`** към отдалечен бекенд — основните команди (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) умишлено фиксират `http://127.0.0.1:<port>` и го игнорират (литералът за IPv4 избягва наказанието от ~2 секунди на заявка за Windows&#x27; `localhost`→`::1`). Вижте [Променливи на средата](#environment-variables).
- За всички SDK/CLI извиквания се изисква вход с акаунт Chloros+ (изпълнете `chloros-cli login` веднъж на всяка машина; кешира се в `~/.chloros/`).
- Примерите използват пътища от типа Linux; в Windows заместете `/home/user/...` с `C:/Users/.../...`.

---

## Общ преглед на най-високо ниво

```
chloros-cli [global options] COMMAND [command options]
```

### Глобални опции

| Флаг | Описание |
| --- | --- |
| `--backend-exe PATH` | Замества автоматично открития изпълним файл на бекенда. |
| `--port N` | Порт на бекенда HTTP (по подразбиране: `5000`). |
| `-v, --verbose` | Активира подробен изход. |
| `--restart` | Принудително рестартиране на бекенда (прекратява всички работещи `backend_server.py`). |
| `--version` | Извежда версията (`Chloros CLI 1.2.0`). |
| `--help` | Показване на помощ на най-високо ниво. |

### Индекс на команди

| Команда | Предназначение |
| --- | --- |
| [`process`](#chloros-cli-process) | Обработва папка със записи от Survey3 или LATTICE от началотоот начало до край. |
| [`login`](#chloros-cli-login) | Удостоверяване на тази машина с акаунт Chloros+. |
| [`logout`](#chloros-cli-logout) | Изчистване на кешираните идентификационни данни. |
| [`status`](#chloros-cli-status) | Показване на текущия статус на лиценза/автентификацията. |
| [`export-status`](#chloros-cli-export-status) | Прогрес на експортирането на Live Thread-4 по време на изпълнение на `process`. |
| [`language`](#chloros-cli-language) | Задаване или изброяване на езика за показване на CLI (поддържат се 38 езика). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Папка по подразбиране за проекти (споделена с графичния интерфейс). |
| [`update`](#chloros-cli-update) | Проверка за и инсталиране на актуализации за CLI (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Системна диагностика + тестове за работоспособност. |
| [`time-sync`](#chloros-cli-time-sync) | Статус/управление на PTP „grandmaster“. |
| [`lattice`](#chloros-cli-lattice) | Управление и заснемане с камера LATTICE (над 45 подкоманди). |
| [`daq`](#chloros-cli-daq) | Управление на спектрални сензори DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Отваряне и управление на запазен проект Chloros (камери + DAQ). |

---

## Инсталиране

`chloros-cli` се доставя като част от инсталатора за настолни компютри Chloros на всяка поддържана платформа — няма отделно изтегляне на CLI. Инсталирането на пакета за платформата добавя `chloros-cli` към вашия `PATH` заедно с десктоп приложението и бинарния файл на бекенда, който то управлява.

Последни изтегляния: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> Инсталаторът включва и удобни скриптове за стартиране (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`), които отварят готова за употреба CLI shell; те са описани в [Ръководството за потребителя на CLI](../CLI.md) и не се повтарят тук.

### Windows (.exe)

1. Изтеглете инсталатора Windows от страницата за изтегляне.
2. Стартирайте `Chloros-Setup-x.y.z.exe` и следвайте указанията на помощника. Пътят за инсталиране по подразбиране е `C:\Program Files\Chloros\` (CLI се намира в `C:\Program Files\Chloros\cli\`, която инсталаторът добавя към PATH).
3. Отворете нов терминал (`cmd.exe`, PowerShell, или Windows Terminal), за да се зареди актуализираният `PATH`.

```powershell
chloros-cli --version
```

Инсталаторът автоматично добавя `chloros-cli.exe` към системния ви `PATH` и включва средата за изпълнение на Arena SDK, необходима за камерите LATTICE.

### Linux amd64 (.deb)

За Ubuntu 22.04 LTS или по-нова версия / работни станции x86_64, базирани на Debian.

> **Ubuntu 20.04 не се поддържа.** Списъкът със зависимости на пакета е извлечен от
> това, към което бекендът действително се свързва, и това включва `libc6 (>= 2.34)`;
> focal доставя glibc 2.31. `apt` отказва инсталацията, вместо да допусне тя да се провали по
> време на изпълнение.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

Файлът .deb инсталира:
- `chloros-cli` до `/usr/bin/chloros-cli`
- Компилираният бекенд към `/usr/lib/chloros/chloros-backend`
- Средата за изпълнение на Arena SDK (за камери LATTICE)
- Модели за отстраняване на шума, пакети за калибриране и конфигурация на канала за актуализации

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Същата структура като .deb файла за amd64, с CUDA версия, оптимизирана за Jetson Orin / Orin NX / Orin Nano.

### Еднократна автентификация за всяка машина

Всяка платформа изисква еднократно влизане в Chloros+, преди да заработят извикванията на SDK/CLI:

```bash
chloros-cli login user@example.com 'YourPassword'
```

Удостоверенията се съхраняват в кеш в `~/.chloros/user_session.json`.

### Проверка на инсталацията

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **Необходим е абонамент за Chloros+.**CLI изисква активен план Chloros+.**Copper**е началният Chloros+ план — всеки платен Chloros+ план има достъп до CLI/SDK; само безплатният**Iron** план няма. (Съответствие на идентификаторите на плановете: `0`=Iron/безплатен, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Направете ъпгрейд на [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Това ограничение се налага от сървъра, а не само от CLI: заявка, маркирана с SDK/CLIбез платен план се отхвърля с `403 PLAN_UPGRADE_REQUIRED`, независимо дали идва от `chloros-cli`, Python SDKили от ръчно създаден клиент HTTP. Излезлият от системата потребител получава кода `401 AUTH_REQUIRED`. Достъпът работи офлайн през гратисния период на плана (30 дни за месечния план, до изтичане на валидността за годишния) и спира, когато този период изтече; `chloros-cli status` продължава да работи, за да е видима причината (това е единственият маршрут SDK/CLI, изключен от ограничението за нива — `GET /api/license-status`).

---

## Бързо начало за пет минути

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Обработка на папка с изображения чрез пълния Chloros поток (откриване на цел → калибриране → винет → отражателна способност → експортиране на индекс).

### Кратко описание

```
chloros-cli process INPUT [OPTIONS]
```

### Аргументи за позиция

| Аргумент | Описание |
| --- | --- |
| `INPUT` | Път към входната папка, съдържаща файлове `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE)или `.dng`. |

### Общи опции

| Флаг | По подразбиране | Описание |
| --- | --- | --- |
| `-o, --output PATH` | нова папка с времева отметка в пътя по подразбиране на проекта (`~/Chloros Projects`, освен ако не е конфигурирано друго) | Папка на проекта, която да се създаде или да се използва отново. Ако папката вече съдържа `project.json`, вместо да се презапише, се създава папка от същия ранг с име `_1`/`_2`. |
| `-n, --project-name NAME` | автоматично (времева марка) | Име на проекта. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` използва невронен дебайер Chloros+; по-бавен, но с по-високо качество. |
| `--vignette / --no-vignette` | `--vignette` | Корекция на винетирането. |
| `--reflectance / --no-reflectance` | `--reflectance` | Калибриране на отражателната способност (използва панелна мишена, ако бъде открита, NIST калибриране за всеки сериен номер за LATTICE). За мултиспектралните данни на LATTICE това служи и като превключвател за **продукта** на отражателната способност — вижте [Превключватели за експортиране по продукти](#превключватели-за-експорт-по-продукт-lattice-мултиспектрален). |
| `--ppk` | изключено | Прилагане на PPK GNSS корекции от sidecar файлове. |
| `--exposure-pin-1 MODEL` | изключено | Закрепване на модела „pin-1“ на двойна камера Survey3. |
| `--exposure-pin-2 MODEL` | изключено | Закрепете модела „pin-2“. |
| `--recal-interval SECONDS` | 0 | Принудително повторно изпълнение на математическите изчисления за калибриране на всеки N секунди от времето на заснемане. |
| `--timezone-offset HOURS` | локално | Преопределяне на отклонението на часовата зона, заложено в метаданните на изхода. |
| `--format FORMAT` | `TIFF (16-bit)` | Един от `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | none | Индекси на растителността (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Налагане на входната точка на конвейера за LATTICE TIFF файлове (Survey3 .raw не се засяга). Също така и „аварийният изход“, който позволява изобщо да се обработва запис **без raw** — вижте [Как изглежда папката с записите](#how-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | включено | Извежда линейния дебайериран резултат (`Debayered_Images`). Вижте [Превключватели за експортиране по продукти](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | включено | Извежда предварителен преглед на екрана (`Preview_Images`): RGB = баланс на бялото (DAQ-източник на осветление, когато е наличен, иначе „gray-world“) + гама; multispec = разтягане с фалшиви цветове. |
| `--radiance / --no-radiance` | включено | Извежда радианс с тип float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Референция за продукта за отражателна способност на LATTICE: `auto` = целта в кадъра, преминала QA, е абсолютната референция, резервен вариант за DAQ-надолу (ρ = π·L/E) като резервен вариант; `target` = строг (без заместване от DAQ); `daq` = авторитетен за DAQ. Вижте [Превключватели за експортиране по продукти](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | няма | Директория с **измерени** сканирани данни за отражателната способност на целите за всяка единица (`<serial>.csv`); при липса се използва номиналният T3/T4P спектър. |
| `--array-alignment / --no-array-alignment` | включено | LATTICE масиви: прилагане на подреждането на модула-към-модул, отбелязано в XMP файла на всеки запис `Chloros:Alignment*`, към всеки обработен продукт (без байеринг / преглед / радиантност / отражателна способност / индекс). Без действие за изображения без тези тагове. |
| `--array-alignment-crop / --no-array-alignment-crop` | изрязване | Изрязва изравнените експортирани данни до общата зона на припокриване на масива, така че всички модули да споделят една площ; `--no-…` запазва пълния сензорен канвас (черно запълване извън източника). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Преизчисляване за изкривяването при подреждането. `nearest` запазва точните динамични номера (DN) на източника (без смесване на радиометричните стойности между пикселите). |

### Опции за откриване на цели

| Флаг | Описание |
| --- | --- |
| `--min-target-size PIXELS` | Минимален размер на панела-цел (px) за детектора. |
| `--target-clustering 0-100` | Чувствителност на групирането. |
| `--target / --targets` | Третира входната папка като съдържаща само панели-цели (пропуска откриването на проучвания). |

### Примери

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Превключватели за експортиране по продукти (LATTICE мултиспектрален)

Обработката с LATTICE се разширява към **всеки приложим продукт в един цикъл**. Четирите превключвателя за всеки тип — `--debayered`, `--preview`, `--radiance`, `--reflectance` — са всички**включени по подразбиране**; използвайте формата `--no-<type>`, за да изключите някоя от тях. Главните камери RGB излъчват само дебайерирани данни + преглед (без радиантност/отражателност по честотни ленти), така че `--radiance`/`--reflectance` са бездействащи за тях. Превключвателите се игнорират за Survey3 и `.raw` (които следват стандартния път за отражателност/цел). *(Старият флаг `--radiometric-output {reflectance,radiance,sensor-response}` беше **премахнат** и заменен с тези превключватели; вече няма ниво `sensor-response`.)*

| Продукт | Изход | Необходимо ли е DAQ за низходящ поток? |
| --- | --- | --- |
| `--debayered` | Линейно демозаициране (`Debayered_Images`). | Не. |
| `--preview` | Предварителен преглед на екрана (`Preview_Images`): RGB = WB + гама; мултиспектрално = разтягане на фалшивите цветове. | № |
| `--radiance` | float32 W/m²/sr/nm от пълната радиометрична верига (`Radiance_Images`). | № |
| `--reflectance` | uint16 отражателна способност ρ (`32768` = 1,0), готова за Pix4D. | **Да**, освен ако не е фиксирана от мишена в кадъра, преминала QA (виж по-долу). |

`--reflectance-source` избира референцията за отражателната способност:**`auto`**(по подразбиране) прави мишената в кадъра, преминала QA,**абсолютна референция**— веригите от емпирични линии, фиксирани към мишената, се сравняват на запазени панели и се прилага измерената най-добра стойност — като се преминава към разделянето на DAQ за низходящо излъчване (ρ = π·L/E) когато няма цел или QA се провали;**`target`**е строг (без заместване от DAQ);**`daq`**избира поведението, при което DAQ има предимство. Геометрията на целта (ArUco / фиксирана ROI / лента) идва от конфигурацията на целта на проекта; `--target-reflectance-dir DIR` запазва**измерените** сканирания (`<serial>.csv`), търсени по серийния номер/QR на целевата единица, като резервен вариант се използват номиналните T3/T4P спектри.

Пътят на отражателната способност на DAQ определя **съвпадащите по време на излъчване**от записан**`.daq`**(DAQ-U/M/E)**или от роден за DAQ-M файл `.csv`**, намерен заедно със снимките. Ако пакетът за калибриране за конкретна камера или DAQ не е кеширан локално, конвейерът**го изтегля автоматично от AWS** при първата употреба (изисква еднократна връзка с интернет; запаметен в кеша под `~/.chloros/`).

#### Четене на пиксели за отражателна способност (Pix4D / Metashape / ваши собствени скриптове)

Отражателната способност се съхранява като цяло число DN, а **DN, което означава ρ = 1,0, зависи от изходната камера**:

| Източник | ρ = 1,0 е | Как да се определи |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (резерв до ρ 2,0) | XMP `Chloros:PixelScale=32768` е отбелязан върху файла. |
| Survey3 | `65535` (ограничено при ρ 1,0) | Липсват XMP етикети с `Chloros:*` — именно тази липса *е* признакът. |

**Прочетете `Chloros:PixelScale` и разделете на него**, вместо да приемате, че е константа. Маркерът е дефиниран в домейна uint16, така че остава `32768` във всички изходни формати, които премащабират — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` и `TIFF (32-bit, Percent)` са самоописателни (първо нормализирайте съхранения тип данни обратно към uint16: ×257 от 8-битов, ×65535 от float).

> **Един случай по дизайн не носи мащаб.** Когато запис от 8-битов източник (BayerRG8) се записва като 8-битов TIFF, потока *ограничава* стойностите до 0..255, вместо да ги премащабира, така че всяка стойност над ρ≈0,008 се изравнява до 255 и файлът не съдържа мащаб. Chloros умишлено пропуска както `Chloros:PixelScale`, така и туплата `MicaSense:RadiometricCalibration` там и записва причината за това. **Ако тагът липсва във файл с отражателна способност на LATTICE, не приемайте, че има мащаб — реекспортирайте в 16-битов или 32-битов формат**, вместо да разделяте пиксели, които никога не са били делими.

#### EXIF данни, пренесени при експорта

`process` копира **GPS блок и неговия ExifIFD** във всеки продукт, така че
експортът пренася `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` и
`CameraSerialNumber` заедно с геореференцирането.

**`FocalLength` не е опционален за фотограметрията.** Pix4D изчислява разстоянието между точките на земната повърхност въз основа на
фокусното разстояние и надморската височина; при липса на този таг се използва крайно неправилен мащаб. При един
полет над портокалова градина с 49 снимки липсващият таг превърна площ от 411 м × 160 м площ в реконструирана
47,8 км × 13 км такава — ортофото с 455 MP, състояща се предимно от „nodata“, което след това беше интерпретирано като проблем с мозайката и
проблем с BigTIFF, преди някой да провери GSD. Ако вашата ортофотография се получи с неправдоподобен
мащаб, първо пуснете `exiftool -FocalLength` върху експортирания продукт.

Копието умишлено **не** е `-all:all`: структурните тагове на IFD0структурните тагове на IFD0 нарушават изхода на LATTICE при
копиране, а `ExifImageWidth` / `ExifImageHeight` са изключени, защото описват
*изходния* запис — експорт, чийто размер някога е бил променен, иначе би съдържал размери,
противоречащи на собствения му растер. XMP се записва директно, а не се копира, защото ExifTool
изтрива XMP-таговете от същото извикване, когато XMP-блокът се копира (което би довело до загуба на MAPIR
таговете за калибриране).

### Къде се съхраняват резултатите

Резултатите се записват **в папката на проекта, групирани по камера и след това по файлов формат**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Папката на камерата е `LATT-<sensor>-<lens>-F<filter>` за LATTICE (съответстваща на EXIF
`Model`) и `<model>_<filter>` за Survey3 — две камери, които споделят един и същ сензор и филтър, но се различават
по обектива, се съхраняват в отделни дървета, тъй като винетирането, зрителното поле и изкривяването са различни. Папката с форматите
следва `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` или `tiff32` за
`TIFF (32-bit, Percent)`.

> **Всеки експортиран продукт запазва името на изходния файл.** Експортираният файл с радиантност на
> `capture_…_raw.tif` все още се нарича `capture_…_raw.tif` — просто се намира в
> `tiff32/Radiance_Images/`. **Папката идентифицира продукта, а не името на файла**, така че търсенето с
> `*radiance*.tif` не дава резултат; вместо това търсете по папката.

### Записи от светлинния сензор — калибрирани `.daq` + `.csv`

`process` обработва и записите `.daq` във вашата входна папка и **не**
се нуждае от никакви изображения, за да го направи: DAQ-U / DAQ-M / DAQ-E, изстрелян самостоятелно, представлява пълно
записване, а папка, съдържаща само файлове `.daq`, е валиден вход.

DAQ може да бъде записан **без** калибриране — точно това правят по подразбиране публичните
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) записващите устройства
(`record_daq.py`) правят по подразбиране: те записват необработените отчитания на сензора и маркират файла, така че
Chloros извлича фабричната калибрация на този сензор **по сериен номер** (първо от локалния кеш,
след това в облака MAPIR) и я прилага. `process` записва резултата обратно:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

`.csv` съдържа по един ред за всяко отчитане: времева марка по UTC, време на интегриране, обща мощност,
фотопичен/скотопичен лукс, PPFD (и неговото разпределение по синьо/зелено/червено), пикова дължина на вълната, след това
пълният спектър по собствената мрежа от дължини на вълните на сензора. `.daq` се импортира отново, без да се
калибрира за втори път.

При успех операцията отчита `Light-sensor products written: N (calibrated .daq + .csv)`.
Текстът в скобите описва какво всъщност е записано, така че се чете
`(RAW COUNTS — this sensor has no calibration bundle)` за сензор без пакет и
`(N calibrated, M raw counts)` за папка, съдържаща и двете. Заглавията на бекенда
`[DAQ-EXPORT]` и `[RUN-SUMMARY]` също се формират по същия начин — нито един от
трите не може да определи суровия експорт като калибриран.

Запис от DAQ-U / DAQ-M / DAQ-E, чийто пакет за калибриране не може да бъде извлечен — вие сте
офлайн или този сензор няма калибриране в архива — се **пропуска с посочване на причина** в
ред `[DAQ-EXPORT]` и никога не се записва като „калибриран“ файл, съдържащ необработени отчитания.
Свържете се с интернет иизпълнете отново. Причината е тази, която четецът действително
е установил за този файл (нечетима схема, липса на пакет, грешка при запис), а обобщението на
изпълнението изброява **отделни** причини — двадесет файла, пропуснати поради една причина, се отчитат като една
причина, а не като двадесет повторения на нея.

#### Записите от DAQ-A се експортират като необработени отчитания

Серията **DAQ-A** е по-стара от системата с пакети за всеки сериен номер и няма калибрационен пакет,
който да се изтегли — вместо това тя се калибрира на място спрямо рефлектометрична мишена, което е
причината, поради която никога не е имала нужда от такъв. Отказът на тези записи не им остави никакъв начин да получат
данните си изобщо, затова ги експортират под **различно име**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Различно име на файла, а не флаг вътре във файла, защото заявката трябва да оцелее
изпращането по имейл като просто име. Заглавката `.csv` посочва
`raw spectral sensor counts (NOT irradiance)` и предупреждава, че стойностите са съпоставими
**в рамките на** файла — което е точно за какво ги използва калибрирането въз основа на еталон — а
не между сензори. Зависимите от мощносттазависими фотометрични колони (обща мощност, фотопичен и
скотопичен лукс, PPFD) се записват като **NULL** вместо да бъдат интегрирани от броя на импулсите, а обобщението на
серията показва `RAW COUNTS`, така че „експортираните“ в лог файла данни не могат да се тълкуват като интензитет на слънчевата радиация.

Старите записи **v1.01 / v1.02** (записвани от DAQ-A-SD ги записва) не съдържат епоха за всяко отчитане,
а само времето на записване на файла. Програмата за съпоставяне на изображения ↔ надолу насочено излъчване все още ги отхвърля — съпоставянето на
кадър спрямо времето на записване би било погрешно, без това да се забележи — но експортерът ги чете, а
CSV отпечатва `clock=daq_created_on`, така че продуктът посочва на кой часовник работи.

### Забележки

- `process` автоматично открива дали папката ви е Survey3, LATTICE или смесена.
- Напредъкът се предава чрез Server-Sent Events; CLI показва текущия напредък за всеки поток (Откриване, Анализиране, Обработка, Експортиране).
- За Linux/Jetson, CLI проверява суап паметта и може да изведе предупреждение преди обработката на големи папки. Дебейърът, отчитащ текстурите, също така автоматично прилага ограничение на честотата на GPU при Jetson устройства с ниска консумация на енергия (Nano, Orin Nano).
- При успех изпълнението отчита колко изображения е записало (`Image products written: N`).

#### Изпълнение, при което не се записват изображения, се счита за неуспешно

Ако сте поискали продукти, а при изпълнението е записано **нито един** — само `project.json` и
`calibration_data.json` — `process` третира това като неуспех: извежда
`Processing finished but wrote no image products.` и **излиза с ненулева стойност**, така че скриптът може
да го засече. Съобщението посочва папката на проекта и обичайните причини:

- входната папка не е била разпозната като заснета (проверете оформлението и `--input-level`), или
- всеки поискан продукт е бил пропуснат като неприложим за тези камери (например, при заявка за
  радиантност/отражателна способност от камери, които поддържат само RGB).

Изпълнете отново с `--verbose` и проверете лога на бекенда за редове `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
които обясняват пропуските по камери, коитоиначе не биха достигнали изхода на CLI.

Умишлено изпълнение само с метаданни — всички продукти са изключени и няма `--indices` — все пак е
**успех**, тъй като празен изход на изображенията е правилният резултат в този случай.

Същото важи и за **лекоизпълнение само със сензора**: папката със записи от `.daq` по дефиниция не съдържа изображения за експортиране
и изпълнението се оценява въз основа на калибрираните `.daq` / `.csv`, които е записало вместо това.

---

## `chloros-cli login`

Удостоверете тази машина с акаунт в облака Chloros+. Удостоверителните данни се съхраняват сигурно в кеш в `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Примери

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (премахване на части от паролата или дублиране на части от нея). При код 401 CLI автоматично прави повторен опит, като добавя отново `$$`, а след това с-дублирана половина от паролата; ако повторението е успешно, системата ви влиза в системата и извежда правилния синтаксис с единични кавички, който да използвате следващия път.

> **Използване без графичен интерфейс/чрез скрипт: липсата на кеширана сесия означава интерактивен прозорец, а не бърза грешка.** Всяка команда, стартираща бекенд (`process`, `status`, `export-status`, `time-sync`, …) без кеширана лицензия/сесия преминава в интерактивен прозорец за въвеждане на команди `Email:` / `Password:` на stdin, преди да продължи. Следователно задача, изпълнявана без надзор и без кеширана сесия, ще зацикли в очакване на входни данни — изпълнете `chloros-cli login EMAIL PASSWORD` веднъж на всяка машина, преди да планирате работа в режим без графичен интерфейс.

---

## `chloros-cli logout`

Изчиства кешираната сесия и налага ново влизане в системата при следващото извикване.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Показва текущото ниво на лиценза (Iron/Copper/Bronze/Silver/Gold), автентифицирания потребител и броя на свързаните устройства.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Проверява текущия прогрес на експортирането на Thread-4. Може безопасно да се извиква **по време на** изпълнение на `process` от друга конзола.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Задаване на езика за показване на CLI (поддържат се 38 езика, включително CJK, RTL и индийски). При конзоли от по-старо поколение, които не могат да визуализират скрипта, езикът се превключва плавно към английски.

```
chloros-cli language [LANG_CODE] [--list]
```

### Примери

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Команди за папката на проекта

Те управляват местоположението по подразбиране на папката на проекта (споделено с графичния интерфейс).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux / Само за Jetson. Проверява `version_url` от `/etc/chloros/update.conf` и предлага да изтегли и инсталира съответния `.deb`.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

На Linux/Jetson CLI също изпълнява **автоматична проверка за актуализации при всяко стартиране** (без блокиране, никога не забавя командата): проверява `/etc/chloros/update.conf`, кешира резултата за 1 час в `~/.chloros/update_cache.json` и извежда `Update available: vX.Y.Z / Run: chloros-cli update`, когато има по-нова версия. При всяка грешка и при Windows.

---

## `chloros-cli selftest`

Извършва 7-етапен тест за работоспособност: версия, наличност на порт, стартиране на бекенда, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), наличие на модел за отстраняване на шума, готовност на CUDA и отстраняването на шума.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Статус и управление на PTP „grandmaster“. Хостът Chloros изпълнява ролята на PTP „grandmaster“; LATTICE cams и DAQ-E устройствата работят като подчинени към него за времеви отметки между устройствата.

| Подкоманда | Описание |
| --- | --- |
| `status` | Показване на състоянието на „grandmaster“, приоритетите на BMCA, идентичност на часовника. |
| `peers` | Изброяване на подчинените устройства, засечени чрез Delay_Req (камери + DAQ-E сензори). |
| `cameras` | Състояние на PTP за всяка камера (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Рестартиране на процеса на главния узел. |
| `set-priority --priority1 N --priority2 N` | Преопределяне на приоритетите на BMCA. |

### Примери

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Управление на камерите LATTICE. Всяка подкоманда преминава през бекенда Chloros; бекендът притежава пула от камери, така че последващите извиквания на CLI използват отново същия отворен идентификатор.

### Общи опции (използвани от повечето подкоманди)

| Флаг | Описание |
| --- | --- |
| `-d, --device N` | Индекс на камерата (по подразбиране: 0). |
| `-s, --serial SN` | Конкретен сериен номер; отменя `--device`. |
| `--serials SN1,SN2,…` | Серийни номера, разделени със запетая, за работа с няколко камери. |
| `--all` | Работа с всяка открита камера. |
| `--exposure US` | Време на експозиция в микросекунди. |
| `--gain DB` | Усилване в dB. |
| `--pixel-format FMT` | например `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Размери на изображението. |
| `--preset {default,high_quality,high_speed,triggered}` | Прилагане на предварително зададени настройки. Всички работят в свободен режим, с изключение на `triggered`, който активира камерата при хардуерен импулс на линия 2 — ако нищо не задейства тази линия, тя ще чака вечно, вместо да заснеме. |
| `-o, --output DIR` | Изходна директория (по подразбиране: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Размер на GVSP пакета. `auto` изпълнява ICMP+GVSP проби; `jumbo` = 9000; `standard` = 1500. |

### Работен процес при първо свързване на камера LATTICE

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Справочник за подкоманди

#### Откриване и информация

| Подкоманда | Цел |
| --- | --- |
| `lattice info` | Изброява свързаните камери (производител, модел, сериен номер, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Анализира хост системата за оптимална конфигурация на камерата. `--no-discover` пропуска откриването на камери (по-бързо, анализ само на мрежовата карта). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Проверка/коригиране на настройките на мрежовата карта; оценка на пропускателната способност/FPS. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Мрежови възможности на бекенда със стабилна схема + препоръка за масив (връща `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` запазва исканата резолюция, но ограничава целевите кадъра в секунда — прочетете `recommended.recommended_target_fps` и го предайте като цел за свързване; третирайте го като успех, а не като грешка. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Анализ „Какво би станало, ако…“ без отваряне на камери. **`--n-active` е общият брой камери в мрежата, а не само тези от този масив**— увеличете го, когато самостоятелни камери предават едновременно или когато бюджетът на мрежата се изчислява въз основа на изискване, което ги подценява (по подразбиране: `len(--models)`). Винаги извежда обобщените редове `Wire budget:` (изисквани MB/s спрямо таван, защитен от сблъсъци) и `Max cameras:`, и маркира `** OVER-SUBSCRIBED**`, когато масивът претоварва кабела — вижте [Модел за fps и бурст на масива](#array-fps--burst-model). |
| `lattice gpu` | Показване на състоянието на GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Проверка или актуализация на фърмуера на камерата. Локалният избор `.fwa` е фиксиран: файлът в `firmware/<MODEL_PREFIX>/`, съответстващ на `MIN_FIRMWARE_VERSION` на версията, се записва, когато е наличен (само най-новата версия като резервен вариант), така че по-нов образ на производителя, подготвен на диска, остава неактивен, докато този избор не бъде променен — по-новите версии достигат устройствата чрез подписания AWS манифест, което се предпочита, когато версията е по-нова. |
| `lattice presets [--apply NAME]` | Изброяване или прилагане на предварителни настройки на камерата. |
| `lattice status` | Статус на камерата на живо. |

#### Заснемане

| Подкоманда | Цел |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Един кадър. **По подразбиране запазва всеки тип експорт** (`--processing all`); вижте [Нива на експорт при заснемане](#capture-export-levels-the-all-default). `--levels` запазва изрично подмножество (отменя `--processing`); `--force-daq` записва зададеното отчитане от DAQ като `.daq` допълнителен файл дори при запис само в суров формат. `--jpeg-quality` = JPEG качество 1–100 (по подразбиране 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Записва на диск до натискане на Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Преглед на живо в MJPEG чрез браузър. `--ae-damping` задава затихване на автоматичната експозиция (0,4–100). |

#### Настройка на сензора

| Подкоманда | Предназначение |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Четене/запис на всеки GenICam възел. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Експозиция и автоматична експозиция (AE). |
| `lattice gain [--auto] [--off] [--set DB]` | Усилване и автоматично усилване. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | Област на интерес (ROI) на сензора и бининг. |
| `lattice format [--set FMT] [--list]` | Формат на пикселите. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Хардуерен/софтуерен тригер. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (без флагове = еднократно балансиране на бялото) | Операции по балансиране на бялото. RGB/само за камери тип Bayer; операция без действие (пропуска се) при монохромния M3M. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | RGB — цветова верига за визуализация. `natural` (по подразбиране) е евтината обработка на живо; `enhanced` добавя премахване на ивици + вибрация + локален контраст CLAHE за пълен вид с паритет на центъра при ~2× по-висока цена на обработката на кадър, което води до по-ниска **в реално време** — запазените записи винаги получават пълната обработка във всеки случай. Само за RGB/Bayer камери; пропуска се при моно M3M. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Настройка на насищането/контраста на дисплея (камери с филтър RGB). Пропуска се при моно M3M. |
| `lattice filter [--set NAME] [--list]` | Задаване на модела на филтъра на камерата (`RGN-IMX265`, `OCN`, `NGB`, …) |
| `lattice power [--sleep]` | Измерване на мощността/термичните възли на сондата; превключване на режим на ниска мощност в режим на готовност. |

#### Калибриране и сензори

| Подкоманда | Предназначение |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Калибриране въз основа на мишена за отражателна способност. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Команди за вградения сензор за падаща светлина. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Прилагане на корекция на винетирането към съществуващи изображения. |

#### Мултикамера (преходни сесии)

| Подкоманда | Предназначение |
| --- | --- |
| `lattice multi-info` | Изброява всички камери със синхронизиращи роли. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Един синхронизиран кадър от всяка камера. Запазва **всички типове експорт по подразбиране**, когато е свързан постоянен масив; преходният резервен вариант без масив е**само с премахване на байеризацията** (за останалите първо изпълнете `array-connect`). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Поточна предаване на синхронизирани кадри (временна). |
| `lattice multi-test [--count N]` | Тест за синхронизация на GPIO. |
| `lattice multi-detect [--line LINE] [--json]` | Автоматично откриване на GPIO свързване „господар/роб“. |

#### Изравняване

| Подкоманда | Цел |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — плюс регулатори за детектор/съвпадение `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, регулатори за RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, комбинация от няколко кадра `[--averaging mean\|median\|inlier_weighted]`, геометрични ограничения `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, пространствено ограничение `[--roi X0,Y0,X1,Y1] [--mask PATH]` и преопределения за всеки „slave“ `[--per-cam-override SN:KEY=VALUE]` (повторими) | Изчисляване на профила на изравняване от камери в реално време. `--prefilter` по подразбиране използва `gradient` (карта на ръбовете; съвпада с GUI/матричния алигнатор — ръбовете се запазват в различните спектрални ленти). `--matcher flann` дава добри резултати при над ~5000 характеристики; `--averaging median` е устойчив на един лош кадър, `inlier_weighted` претегля според броя на съвпаденията; `--lock-scale` проектира към най-близкото завъртане (без мащабиране), `--lock-axis` нулира една компонента на преместване; `--mask` се прилага за всяка камера (използвайте `--per-cam-override` за настройки за всяка камера, например `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` отказва да запише калибриране, чийто RMS на репроекцията надвишава прага. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Заснемане на един подравнен многолентов кадър. `--bit-depth` по подразбиране се настройва според камерата; `--no-crop` запазва целия кадър (запълва с черно); `--interpolation` (по подразбиране `linear`) и `--border-mode`/`--border-value` (по подразбиране `constant`/0) контролират изкривяването на CPU — пътят на GPU е билинеарен във всеки случай. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Мултилентови кадри, подравнени по поток (същите регулатори за изкривяване като `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Показване на подробности за профила. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Промяна на подреждането на слоевете. |

#### Индекс / Математика на растителността

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Пълен набор от флагове: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (повторяемо), `--capture-level raw|debayered|radiance|reflectance|unknown` (преопределя нивото на запис, записано в източника TIFF; по подразбиране: прочетено от метаданните на TIFF), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. При `--live` се прилагат и регулаторите за изкривяване на подреждането: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **Символите на `--channel` са чувствителни към регистър-чувствителни.** Страната със символите трябва да съвпада точно с имената на каналите в предварителните настройки (предварителните настройки използват малки букви, например NDVI = `red`, `nir` — проверете `--list-presets`), а частта с лентата трябва да съвпада с името на лента в подредения стек (или да бъде индекс на лента, започващ от 0, в офлайн режим). `--channel red=Red_660 --channel nir=NIR_850` работи; `--channel RED=660` дава грешка с грешка `channel_map missing entries`.

#### Постоянни връзки (Smart-Prep, поток, еквивалентен на GUI)

Тези команди поддържат камерите отворени в пула на бекенда при всички извиквания на CLI.

| Подкоманда | Цел |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Добавяне на една камера към пула (една камера, без масив). |
| `lattice cam-disconnect [--serial SN] [--all]` | Освобождаване. |
| `lattice cam-list` | Изброява камерите в пула. |
| **`lattice array-connect`**|**Свързва постоянен синхронизиран масив (ПРЕПОРЪЧИТЕЛНАТА отправна точка).** Изпълнява пълния GUI поток за интелигентна подготовка. |
| `lattice array-disconnect [--array-id ID] [--all]` | Освобождаване на масив. |
| `lattice array-list` | Изброяване на свързаните масиви. |
| `lattice array-status [--array-id ID]` | Кадри в секунда на живо, PTP, последна грешка. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Едно синхронизирано заснемане от масива на живо — Единичен / Непрекъснат / Интервал / Най-бърз. **По подразбиране е `all`** (един файл за всеки приложим тип експорт за всяка камера). Пропуснатите камери (например RGB, изключена от излъчване/отражателност) се отчитат с `Skipped: SN:<serial> (<reason>)`; отчитането от DAQ, използвано за отражателната способност, се запазва заедно с тях и се отчита с `DAQ: <path>`. Вижте [Режими на заснемане, записващи устройства и офлайн преобработка](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Записва изгледа на комбинирания индекс на живо във видео/GIF (за мониторинг; изисква отворен комбиниран поток). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Високаfps серия от необработени Bayer изображения (за анализ; преобработване офлайн). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Преобработване на запазена серия от необработени изображения в калибрирани видеоклипове. |

##### `array-connect` Опции

| Флаг | По подразбиране | Описание |
| --- | --- | --- |
| `--serials SN1,SN2,…` | автоматично откриване на всички камери LATTICE (необходими са ≥2) | Първият по сериен номер е МАСТЪР. Когато се пропусне, откриването филтрира моделите LATTICE (`TRI032*`) и свързва всички тях. |
| `--line {Line0,Line2,Line3}` | `Line2` | Линия за синхронизация на GPIO. |
| `--target-fps F` | автоматично | Честота на задействане на Master. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | автоматично | Преопределяне на избора на ниво. |
| `--wire-ceiling-mbps MB_PER_S` | автоматично открит | **Продължителният капацитет на връзката на хоста, в MB/s — стойността, от която зависи цялото разпределение на масива.** Намалете го, когато масивът докладва кадри с повреден GVSP: автоматичната стойност се изчислява въз основа на обявената скорост на връзката от мрежовата карта, която надценява USB адаптерите, тънките PCIe линии и натоварените споделени мрежови структури. Запазва се в блока за запис на масива на проекта, така че повторно отваряне / CLI / SDK го възстановява. Вижте [Състояние на масива](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Хардуерно групиране. |
| `--no-recommend` | off | Пропускане на стъпката за мрежов анализ. |
| `--no-ptp` | изключено | Деактивира PTP (в този случай времевите отметки между камерите **не** са съпоставими). |

### Smart-AE / Smart-Capture

Масивите LATTICE изпълняват непрекъснато AE във фонов режим веднага след свързването им, но наскоро насочената сцена отнема малко време, за да достигне конвергенция. `array-capture --smart` е **удобно решение**: изчаква AE да се стабилизира във всяка камера от масива, след което задейства заснемането. Използвайте го, когато сменяте сцени по време на сесията.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

Политиката за стабилизиране е консервативна по по подразбиране: таймаут от 5 секунди, прозорец за стабилност от 1,5 секунди, толеранс на разпределение на експозицията ±5 %. Настройте чрез SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`), ако се нуждаете от различно поведение на автоматизацията.

### Нива на експортиране на заснетите данни (по подразбиране `all`)

От тази версия нататък `lattice capture`, `lattice multi-capture` и `lattice array-capture` **по подразбиране са `--processing all`** — по един запазен файл за всеки тип експорт, който се прилага за всяка камера, съответстващ на поведението на „Заснемане на всичко“ в графичния интерфейс. Нивата са:

| Ниво | Изход | Прилага се за |
| --- | --- | --- |
| `raw` | Едноканален Bayer (монохромни камери: един канал) директно от сензора. | Всички камери. |
| `debayered` | 3-канален BGR демозаик (монохромни камери: 1-канален в сиви нюанси). | Всички камери. |
| `radiance` | float32 W/m²/sr/nm чрез пълната радиометрична верига. | Само мултиспектрални (M3C/M3M) — **пропуска се за камери с RGB-филтър**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), готова за Pix4D. | Само мултиспектрална и **само когато е свързан DAQ + камерата е калибрирана**; в противен случай се пропуска. |
| `preview` / `display` | Пълна верига за предварителен преглед в GUI (CCM + WB + гама според профила на камерата). `lattice capture` нарича това `preview`; `array-capture`/`multi-capture` използват `display`. | Всички камери. |

Предайте едно ниво, за да запазите само това (`--processing debayered`). Когато поискате `all`, нивата, които не се отнасят за дадена камера, се пропускат (и се отчитат), но не се отчитат като грешка — камера, която не е свързана или не е калибрирана, все пак получава `raw` / `debayered` / `preview`.

За всеки кадър с отражателна способност действително използваното от DAQ отчитане на низходящото излъчване се записва в **`.daq`** файл до изображението (така че заснемането да може да бъде преобработено по-късно) и се отчита в ред `DAQ:`.

### Как изглежда папката със заснеманията

Всеки тип експорт се съхранява в **собствена подпапка** под `-o`, така че при многостепенно заснемане типовете никога не се смесват:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` е времевата марка на заснемането, а `<serial>` – серийният номер на камерата, така че една синхронизирана група споделя една и съща
времева марка за всички камери. **Обърнете внимание на асиметрия:** нивото `display` се съхранява в папка
с име `preview/`, докато самите файлове запазват `_display` в името — папката и суфиксът се различават
само за това ниво. Неизвестните нива се съхраняват в папка със същото име като тях, а ако подпапката
не може да бъде създадена, файлът се записва в кореновата папка за изход, вместо да бъде изгубен.

**Повторна обработка на папка със записи:**насочете `chloros-cli process` към**коренната папка с заснетите изображения**
(`output/`). `process` обикновено импортира само папката, която сте посочили, но когато тази папка не съдържа
изображения и има подпапки, програмата автоматично преминава към тях — така че подпапките на нивото на корена и
коренните `.daq` се извличат наведнъж. Всяко ниво на заснемане се импортира като едно изображение, като
останалите нива са достъпни като режими, а не като по едно изображение на ниво.

Именуването на **подпапка на ниво** директно (например `output/raw/`) също работи. По този начин кореновата
папка `.daq` остава настрана, затова копирайте или посочете отчета от DAQ заедно с нея, когато преизчислявате радиометричен
продукт от `raw/` — в противен случай съвпадението на времевата марка няма с какво да се съпостави.

**Обработката винаги започва от `raw`.** В рамките на всяко заснемане суровият кадър е източникът на конвейера;
`debayered`, `radiance`, `reflectance` и `preview` се появяват като режими за преглед, но никога не се връщат
обратно през тръбопровода. Повторната обработка на производен продукт би довела до повторно прилагане на винетиране, CCM и
математически изчисления за радиантност, които вече са вградени в неговите пиксели, затова Chloros отказва, вместо да
извършва двойна обработка. Две последствия, които си заслужава да се знаят:

- Рендерите `index/` и `composite/` **никога** не се обработват. Те са изходни файлове, а не заснети изображения —
  рендерингът на LUT от типа NDVI няма смислена интерпретация на радиантността.
- Папка с заснети изображения, експортирана **без** `raw` (например `array-capture --processing reflectance`) няма
  легитимен източник в производствения поток. Тези заснети файлове се импортират и показват нормално, но `process` ги пропуска
  и показва следното съобщение:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Ако наистина се налага да прокарате производен продукт — сесия от хъб, заснета с
  активиран `demosaic`, или папка от по-стара версия — `--input-level {raw,debayered,processed}` насилва въвеждането
  и отменя прескачането. Този флаг е преднамерена аварийна опция; `auto` (по подразбиране)
  никога не обработва заснемане, което няма суров файл.

### Прескочени заснемания в масиви със смесени филтри

Когато смесвате RGB и мултиспектрални камери в един масив, `array-capture --processing radiance` (или `reflectance`) запазва мултиспектралните кадри и **пропуска** RGB — радиантността на пиксел по Байер няма смисъл за широколентов сензор. CLI отпечатва всеки запазен файл (с нивото му на експортиране), всяко записано `.daq` и всяко изпускане изрично, така че броят на файловете не е изненадващ:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Токените за причината за изпускане следват модела `<level>-not-applicable-to-rgb-cam`. Отразяванетоance също може да пропусне с `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, както и с `dls-uncalibrated-band-<nm>`, когато лентата се намира предимно извън радиометрично калибрирания диапазон на светлинния сензор на DAQ (~374–974 nm) — сред предлаганите артикули само F988, чийто поддържан път е работният процес с панел за отражателна способност.

Използвайте `--processing debayered` (или `display`), за да включите всяка камера, независимо от типа на филтъра, или стандартния `all`, за да получите всички приложими нива за всяка камера на един път.

---

## Режими на заснемане, записващи устройства и офлайн преобработка

Всички те работят върху **постоянен масив** (първо изпълнете `array-connect`). Те отразяват панела за заснемане в графичния интерфейс.

### Режими на `array-capture`

`array-capture` е единична команда с четири режима на затвора, както и набор от превключватели за експортиране:

| Режим | Флаг | Поведение |
| --- | --- | --- |
| **Единичен** *(по подразбиране)* | (няма) | Една синхронизирана група заснемания, след което излиза. |
| **Непрекъснат** | `--continuous` | Последователни цикли до `Ctrl+C`, `--count N` или `--duration S`. |
| **Интервал** | `--interval S` | Един цикъл на всеки `S` секунди (измерени от началото на всеки цикъл), същите граници. |
| **Най-бърз** | `--fastest` | Само необработени данни + зададеното DAQ отчитане + композитният индекс; пропуска изчисленията за радиантност/отражателност/показване, за да се зареди кадърът бързо. Включва `--processing raw --force-daq`. По-късно преобразува запазеното `.daq` в калибрирани продукти по-късно. |

Превключватели за експортиране (комбинират се с всеки режим; всички споделят GUI/SDK крайна точка):

| Флаг | Ефект |
| --- | --- |
| `--processing LEVEL` | Едно ниво на експортиране или `all` (по подразбиране). |
| `--levels L1,L2,…` | Изрично подмножество от типове за експортиране (например `raw,radiance,reflectance`); **отменя `--processing`**. |
| `--aligned` / `--no-aligned` | Извършва трансформация на всеки не-суров износ на елемент към [профила на подравняване](#alignment) (съвместно регистриран). Необработените данни остават без преобразуване, но преобразуването се съхранява в метаданните. Преминава към несъвместяване (с предупреждение), ако масивът няма профил. |
| `--index` / `--no-index` | Запазване / пропускане на наслагването на индекса на растителността за всяка камера, когато такова е конфигурирано. По подразбиране: рендерира се. |
| `--force-daq` | Запазвайте присвоеното отчитане от DAQ/DLS като `.daq` допълнителен файл, дори когато никое избрано ниво не го изисква (например при заснемане само на необработени данни), за да могат кадрите да бъдат преобработени офлайн в отражателна способност/индекс. |
| `--smart` | Изчаква AE да се стабилизира за всички камери преди задействане (виж [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | TIFF компресия на пикселите. `deflate` (по подразбиране) = zlib L1 без загуба + хоризонтален предиктор, ~4,1 MB на кадър с пълна резолюция; `none` = некомпресирано, ~5× по-бързо записване при ~6,3 MB на кадър — използвайте за максимална устойчива скорост, когато дискът позволява. И двете са без загуба на качество и се четат идентично при импортиране. |

> **TIFF с еднократно записване + моделът с устойчива скорост.**Записите се записват в**един**проход на TIFF файл, съдържащ пиксели + XMP + IFD0 Марка/Модел (измерено при Mono12 с пълна резолюция: 36 ms компресирано / 6,5 ms некомпресирано, спрямо ~148 ms за стария метод „записване, след което презаписване с ExifTool“); единствената оставаща работа с ExifTool (доизпипване на EXIF sub-IFD) се изпълнява от асинхронен фонов процес, а кадърът е завършен и готов за импортиране, дори ако този процес никога не се изпълни. Имайте предвид, че DEFLATE компресията задържа GIL на Python, така че записването на компресирани данни**не**се паралелизира между нишките на записващия модул за всяка камера — непрекъснат запис на 8камери със запис при пълна резолюция със скоростта на сензора (~10,4 кадъра в секунда) изисква `--compression none`**и** диск от NVMe-клас (~500 MB/s устойчиво записване). Същият параметър е достъпен като `compression` в `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — видео/GIF с комбиниран индекс (клас за мониторинг)

Записва всичко, което **изгледът на комбинирания индекс на живо** показва към `.avi` (и по избор към `.gif`). Тъй като извлича данни от композитния сигнал на живо, комбинираният поток трябва да е отворен (например масивът се преглежда в графичния интерфейс), за да могат кадрите да се запишат. Проверява напредъка на всеки 2 секунди и спира при `--duration`, `Ctrl+C` или когато записващото устройство се самоизключи.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Флаг | По подразбиране | Описание |
| --- | --- | --- |
| `--array-id ID` | само масив | Целеви масив (пропуснете, ако е свързан само един). |
| `-o, --output DIR` | `output` | Изходна директория (локална за бекенда). |
| `--fps F` | `10` | Честота на записване на кадрите. |
| `--duration S` | до Ctrl+C | Автоматично спиране след `S` секунди. |
| `--gif` | изключено | Записване и на анимиран GIF. |
| `--gif-only` | изключено | Записва само GIF (без `.avi`). |

### `array-burst` — серия от снимки в суров формат Bayer с висока честота на кадрите (за аналитични цели)

Чете директно буфера на синхронизираната група от цикъла за заснемане — **не се изисква верига за калибриране, exiftool или преглед на живо** — така че работи с пълната честота на заснемане на камерата. Записва сурови кадри + манифест за всеки кадър + един `.daq` за всяко отделно DLS отчитане под `<output>/bursts/<base>/`. Преобразувайте офлайн (следваща команда) или предайте `--build`, за да се извърши незабавно при спиране.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Флаг | По подразбиране | Описание |
| --- | --- | --- |
| `--array-id ID` | само масив | Целеви масив. |
| `-o, --output DIR` | `output` | Изходна директория (потоците се записват в `<DIR>/bursts/<base>/`). |
| `--duration S` | до Ctrl+C | Автоматично спиране след `S` секунди. |
| `--max-frames N` | без ограничение | Автоматично спиране след `N` необработени кадъра. |
| `--build` | изключено | След спиране, незабавно преобработване на серията (същото като `array-build-video`). |
| `--products …` | `combined:index` | С `--build`: колко видеода се създаде (виж по-долу). |
| `--fps F` | `10` | С `--build`: fps на изходното видео. |
| `--save-tiffs` | изключено | С `--build`: запазвайте също и калибрирани TIFF файлове за всеки кадър. |
| `--gif` | изключено | С `--build`: записва и анимирани GIF файлове. |

### `array-build-video` — офлайн преобработка на запазена серия

Съвпада времето на всеки суров кадър с най-близкото запазено `.daq` отчитане и го прекарва през **същата верига от радиантност / отражателност / индекс като конвейера за импортиране**, като генерира едно или повече видеоклипове.

`--products` е списък, разделен със запетая, от `kind:level` елементи, където `kind` ∈ `per_cam` | `combined` и `level` ∈ `radiance` | `reflectance` | `index`. Един самостоятелният `level` (без `kind:`) по подразбиране приема стойността на `per_cam`. Стойността по подразбиране е `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Флаг | По подразбиране | Описание |
| --- | --- | --- |
| `--burst-dir DIR` | (задължително) | Път към папката за серийни снимки (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | Списък `kind:level`, както по-горе. |
| `--fps F` | `10` | Кадри в секунда на изходното видео. |
| `--save-tiffs` | изключено | Също така запазвайте калибрирани TIFF файлове за всеки кадър заедно с видеото(ата). |
| `--gif` | изключено | Също така записва анимирани GIF файлове. |

> **Изберете подходящия рекордер.** `array-record` е *за мониторинг* — записва композитния сигнал на живо, както се показва на екрана, и изисква потокът да е отворен. `array-burst` → `array-build-video` е *за анализ* — запазва необработени данни от сензора с пълна честота и възстановява калибрирани видеоклипове за сияние/отражение/индекс, без да се изисква изглед на живо.

### Моно (M3M) еднолентови камери

Серията **M3M**е моно версията на Bayer**M3C**: един теснолентов интерферентен филтър на камера (`M3M-<lens>-F<wavelength>`, например `M3M-L87-F685`), така че сензорът предоставя**един-единствен канал в сиво** без мозайка на Байер. Няма какво да се демозаицира, нямаканално преливане, което да се раздели, и няма баланс на бялото, който да се настройва — целият цветови поток за дисплея RGB просто не се прилага.

Какво означава това за CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**разпознават монохромна камера и**пропускат с едноредово съобщение**, вместо да налагат безсмислени настройки. Те все пак работят нормално с камера RGB/Bayer M3C в същата сесия.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** все още работят — радиантността и отражателната способност са *радиометрични карти за всяка лента* и са напълно добре дефинирани за една лента. Монохромните кадри носят **идентична** матрица на отговора на сензора (без 3×3 разделяне), така че равнината преминава през математиката на калибрирането без промяна.
- **Една монохромна камера не може да генерира индекс на растителността.**NDVI/NDRE/и т.н. се нуждаят от поне две ленти (напр. Red + NIR). За да получите индекс от монохромен хардуер, насочете**няколко** M3M камери към различни дължини на вълната, подредете ги в един многолентов стек и изчислете индекса *на него*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` символите трябва да съвпадат **точно** (различава се главна и малка буква; NDVI са с малки букви `red`, `nir` — вижте `--list-presets`), а имената на страните на честотната лента трябва да съответстват на лента в подредения стек (офлайн режимът приема и индекси на лентите, започващи от 0, например `--channel red=0 --channel nir=1`).

Разграничителният елемент в целия стек е токенът `M3M` в модела на низа (той никога не се появява в низа `M3C`), който се показва в GUI/SDK като `is_mono`.

---

## Настройка и оптимизация на мрежовата карта на хоста (масиви LATTICE)

Камерите LATTICE предават GVSP през Ethernet адаптера на хоста, така че при масиви скамерни масиви **драйверът**на адаптера и**размерът на приемащия пръстен** са също толкова важни, колкото и скоростта на връзката. Неправилните настройки се появяват като врата `FRAMES WILL DROP` / `Reduce ROI to enable` в панела „Настройки на масива“ (и в `lattice network-analysis` / `analyze_array_network()` на SDK), дори когато самите камери работят нормално.

### USB 10GbE адаптери — Realtek RTL8157 („Realtek USB 10GbE Family Controller“)

| Елемент | Необходима стойност | Защо е важно |
| --- | --- | --- |
| **Версия на драйвера**|**≥ v10.67 (януари 2026 г.)**, INF `rtump64x64sta.inf` | Старият драйвер от**2016 г.**(v10.65, `rtump64x64.inf`) не обработва правилно изключването и генерира грешки с**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**при изключване/рестартиране/преминаване в режим на заспиване. Преходът зацикля (таймаут от ~5 минути), потребителят изключва насила устройството, а повтарящите се неправилни изключвания**повреждат хранилището на WMI**(PowerShell/инструментите започват да дават грешки с `Invalid class`) и**блокират USB стека** при следващото стартиране (сетевата карта не се активира; USB устройствата спират да се разпознават). Изтеглете актуализация от realtek.com (или от производителя на донгъла), преди да разчитате на „чисти“ рестарти. |
| **Буфери за приемане**— ключова дума `ReceiveBufferLen` |**256**(максимум за драйвера) | RX пръстенът на мрежовата карта. Настройката по подразбиране на драйвера**32**оставя само ~0,26 MB използваем пръстен — твърде малък за серия от данни от няколко камери — затова панелът на масива отчита `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` и блокира свързването. При**256**пръстенът е голям (**~13,5 MB, измерени на лабораторния 10GbE хост**), което дава на RX конвейера реален резерв за многокамерни GVSP серии. (Дали дадена конфигурация действително *се свързва* се определя от две проверки — проверката за **drain-aware**достъп и проверката за**агрегирана свръхрезервация** — а не чрез просто сравнение между серията и пръстена; виж [Модел за fps и серии на масива](#array-fps--burst-model).) |
| **URB за приемане**— ключова дума `PendingReceives` |**64** (макс.) | Блокове USB заявки в процес на предаване; увеличават се успоредно с буферите за приемане за поглъщане на изблиците. |
| **Джъмбо рамка** — ключова дума `*JumboPacket` | **9014** | Необходима за GVSP пакети от 9000 байта (6 пъти по-малко пакети на рамка в сравнение с 1500). |

> ⚠️ **Актуализацията на драйвера на мрежовата карта ВЪЗСТАНОВЯВА тези разширени свойства към техните стойности по подразбиране.**След актуализиране или подмяна на драйвера на адаптера**приложете отново** `ReceiveBufferLen=256` и `PendingReceives=64`, в противен случай панелът на масива ще се блокира отново, въпреки че „нищо не се е променило в хардуера“. Това е основната причина за внезапния отказ на преди това работеща система да се свърже.

Приложете от **привилегирован** PowerShell (заместете името на вашия адаптер, например `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` обхваща USB 10GbE адаптери.** Сега той разпознава типа на адаптера и настройва правилната ключова дума за приемния пръстен: `*ReceiveBuffers`→2048 за PCIe мрежови карти (Intel I219и др.) или `ReceiveBufferLen`→256 + `PendingReceives`→64 за **USB** 10GbE контролера на Realtek (който не предоставя `*ReceiveBuffers`). Стойноститестойностите са ограничени до максималната стойност, отчитана от всеки драйвер (`NumericParameterMaxValue`), така че никога не се записва стойност извън диапазона. Изпълнете го от терминал с **повишени** права; както при всяко настройване, свързано с регистъра, промяната влиза в сила след рестартиране на адаптера или рестартиране на системата. Ръчните команди `Set-NetAdapterAdvancedProperty` по-горе остават добра алтернатива — те се прилагат на момента (превързват адаптера) без рестартиране.

### Основи на мрежата (всички връзки на LATTICE)

- **Адресиране:** link-local `169.254.0.0/16` (GigE Vision LLA). Хостът приема статичен `169.254.x.x/16`; камерите и DAQ-E си присвояват адреси самостоятелно в същия диапазон. Не се изисква DHCP/шлюз.
- **Размер на пакета:**предпочитайте джъмбо (9000), но оставете автоматичното проучване да го определи — то преизмерва при всяко свързване и вече заобикаля ограничението от 1500 байта за ICMP на камерата чрез GVSP проба, така че се установява на джъмбо, където кабелът действително го пренася. Задавайте фиксирана стойност само с `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` когато знаете повече от пробата и предпочитате настройка за всяка команда пред постоянна: фиксирането прескача пробата, така че ако пътят всъщност не може да пренесе 9000,**всяко** заснемане ще изтече с `SC_ERR_TIMEOUT -1011` (вижте [Променливи на средата](#environment-variables)).
- **RX пръстенът се мащабира с `ReceiveBufferLen`:**при стойността по подразбиране `32` използваемият пръстен е ~0,26 MB (твърде малък за всякакъв мултикамерен бурст); при максималната стойност `256` той е голям (~13,5 MB, измерен на лабораторния 10GbE хост), което осигурява реален резерв. Дали дадена конфигурация ще се свърже, се решава след това от проверката за достъп, отчитаща изчерпването на капацитета,**и** от сумирането напроверката за подписка по-долу — а не чрез пряко сравнение между серийния поток и капацитета на пръстена.

### Модел на кадрите в секунда (fps) и серийните снимки на масива

Как да четем панела „Настройки на масива“ (и `lattice analyze-array` / `analyze_array_network` на SDK):

- **Бърстът се сумира за всяка камера поотделно в реалния пикселен формат на съответната камера.**Моно**M3M**камерите предават**Mono12 (2 B/px)**;**M3C**камерите с Bayer матрица предават 8- или 12-битови данни (TRI032S незабележимо излъчва BayerRG12, дори когато е поискано BayerRG8). Така че кадър с пълна резолюция от 4 камери е**~12,6 MB, ако всички са 8-битови, но ~25 MB с три 12-битови моно камери**. Проекцията определя формата на всяка камера въз основа на нейния модел (кеш за идентичност), така че серията от данни съответства на това, което кабелът действително пренася — а не на еднозначното предположение за BayerRG8.
- **USB Ethernet адаптерът е ограничен до 200 MB/s, независимо от посоченото на табелката му.** Таблицата за ефективност, която превръща скоростта на връзката в устойчива стойност, е изведена от PCIe; USB мрежовата карта обявява своята *Ethernet* скорост на връзката, но е ограничена от USB шината и нейния драйвер. USB 10GbE донгъл постигаше ~1063 MB/s „устойчива“ скорост — стойност, която никога не беше тествана — а полученото регулиране на скоростта повреждаше 6–18 % от кадрите, въпреки че все още отчиташе нормален целеви брой кадъра в секунда (fps). USB мрежовите карти вече са ограничени до **200 MB/s** като абсолютна стойност (ограничението е в шината, така че не се мащабира според номиналните характеристики; USB 1 GbE адаптерът постига ~80 MB/s и не е засегнат). `wire_ceiling_source` в записа за възможностите го посочва с думи, а `nic_is_usb` го маркира. И в двата случая това може да се пренебрегне с `--wire-ceiling-mbps`.
- **Достъпът се определя според натоварването на дрена, а не според цялата серия срещу пръстена.** Една едновременна серия трябва да се побере само в *преходния натрупан обем* = `max(0, Σ per-cam arrival − host drain) × emit_window`, а не на целия пакет. При структура с бърз хост / бавни камери (**PCIe**10G хост + 4× 1 GbE камери: пристигане ≈ 320 MB/s, извеждане ≈ 1063 MB/s) хостът извежда по-бързо, отколкото камерите се пълнят, закъснението е ≈ 0, така че симулацията на излъчване с пълна резолюция**се допуска**, въпреки че пакетът от 25 MB надвишава пръстена от 13,5 MB. Поставете същите четири камери зад**USB**10GbE адаптер и изходът е 200 MB/s, а не 1063 — пристигането го изпреварва, а загубата се проявява като повредени кадри, а не като по-ниска честота на кадрите. На хост с 1 GbE долната граница на DLThr на камерите от 31,25 MB/s кара пристигащите данни да изпреварват извеждането → тя правилно**блокира** (за *този* клас блокиране, намалете ROI или използвайте бининг ≥ 2). Достъпът се осъществява през една от **двете** свързващи врати — другата е проверката за агрегирана свръхрезервация по-долу.
- **Прогнозираните fps са консервативен таван за последователно извличане.**Цикълът за извличане на хоста понастоящем извлича буфера на всяка камера**последователно**(~по един прозорец за излъчване на камера), така че цикълът е ограничен от `max(readout+emit, N × emit)`, като излъчването на камера е фиксирано към**връзката за достъп**(1 GbE ≈ 80 MB/s), а не от връзката за качване към хоста. За масив от 4 камери с пълна резолюция и 12-битова дълбочина това е**~2,8 кадъра в секунда**, което съответства на измерените ~2,7–3,0 кадъра в секунда. Този показател е умишлено**независим от експозицията**, така че при слабо осветени сцени реалната стойност може леко да падне под горната граница, когато експозицията се удължи. Последователното извличане е истинският ограничител на кадрите в секунда; паралелизирането му би повишило горната граница до скоростта на единично излъчване.
- **Агрегираното превишаване на капацитета е сериозен препятствие за свързване.**Минималното разпределение на пропускателна способност за всяка камера е**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`), така че след като минималната стойност се достигне, агрегираното търсене (`per_cam × N`) може да надхвърли**максималната граница на кабела, защитена от сблъсъци**(`sustained × sim_emit_factor`). Практическите максимални граници при пълна резолюция по 1 GbE:**6 камери при 1500 MTU, 9 с джъмбо**. Този таван зависи единствено от кабела и долната граница — той е**независим от размера на кадъра**, така че**групирането и по-малките ROI НЕ помагат** (те намаляват байтовете на *кадър*, а не байтовете на *секунда*, определяни от GevSCPD); единствените решения са по-малко камери, джъмбо-рамки от край до край или по-бърз мрежов адаптер. Симптомът би бил загуба на пакети в GVSP, а не плавно намаляване на fps, така че `analyze-array` нулира достижимитеps и извежда `**OVER-SUBSCRIBED**`, а `array-connect` с фиксирана резолюция **отказва да се свърже** (в противен случай „walk-down“ обединява кадрите, което също не отстранява този тип блокиране). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` понижава нивото на отказ до силно предупреждение за работа в тестова среда — вижте [Променливи на средата](#environment-variables).

### Състояние на масива — коя подсистема губи кадри

`GET /api/camera/array/<array_id>/capability` на свързан масив съдържа активен
`health` блок, преоценяван в непрекъснато обновяващ се **10-секунден** прозорец. Той разделя загубата на кадри
на две причини, които изискват противоположни корекции, вместо да докладва една „непълна“
честота, която не посочва нито една от двете:

| Поле | Какво означава | Коя подсистема |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (по сериен номер) | Кадърът **е пристигнал, но е бил структурно повреден**— загуба на GVSP пакет. |**Мрежа**: капацитет на кабела, темпо, RX пръстен на мрежовата карта, MTU |
| `never_arrived_rate_pct` (по сериен номер) | Рамката **изобщо не е пристигнала**— камерата не се е задействала или нищо не е излязло от нея. |**Задействане / синхронизация**: M8 кабел, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Най-лошата честота на камерата за всяка от тях. | — |
| `per_cam_rate_pct` | Комбиниран процент на непълни данни за всяка камера (двете причини заедно). | — |
| `stable_for_seconds` | Колко дълго всяка камера е оставала под 0,01 %. | — |

При над 5 % бекендът записва ред `[array-health <id>] WARN`, посочващ разделянето — при
първото нарушение, при промяна на нивото на сериозност, веднъж в минута, докато то продължава, и веднъж, когато
се отстрани. Повредената половина отпечатва `[gvsp-corrupt <SN>]` при първото засегнато събитие за всяка камера и
причина, след което се обновявана всеки 60 секунди. Всяка оценка все пак се записва в лог файла на бекенда;
броячите се актуализират при всеки буфер, независимо от това какво се отпечатва.

Същият запис съобщава номера, към който е прикрепено цялото разпределение:

| Поле | Какво означава |
| --- | --- |
| `wire_ceiling_mbps` | Действащият постоянен капацитет на кабела на хоста, MB/s. |
| `wire_ceiling_source` | Откъде идва тази цифра, с думи — например `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` или `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, когато `--wire-ceiling-mbps` (или полето **Бюджет за кабелна връзка** в графичния интерфейс) го задава. |
| `nic_is_usb` | `true` за USB Ethernet адаптер — вижте ограничението от 200 MB/s по-горе. |

**Четене:** `gvsp_corrupt_rate_pct`, различен от нула, при `never_arrived_rate_pct` равен на 0
означава, че задействането и синхронизацията на кабела са перфектни и 100 % от загубата е по мрежовия
път — намалете `--wire-ceiling-mbps` и се свържете отново. Обратният модел сочи към
синхронизационния кабел или линията за задействане.

> **`--target-fps` не е показател за повредени рамки.** Темпото на GevSCPD се записва
> веднъж при свързване, така че намаляването на честотата на задействане променя работния цикъл, а не
> скоростта на едновременните излъчвани пакети. Измерено намаляване на търсенето с 5× не доведе до подобрение;
> понижаването на максималната скорост на кабела от 240 на 200 MB/s намали процента на повредените пакети при същата система от 10,4 %
> на 0,00 %.

> **Автоматичнотопоток не е налично във фърмуера на TRI032S.** Работещ масив
> не може да коригира това сам; прекъснете и възстановете връзката, за да може инструментът за избор на време за свързване да
> препланира с новия таван.

### Симптом → решение

| Симптом (Настройки на масива / свързване / `analyze_array_network`) | Причина | Решение |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` се нулира на 32 (обикновено след актуализация на драйвера) | Задайте `ReceiveBufferLen`→256, `PendingReceives`→64; отворете отново панела (рестартирайте бекенда, ако е запазил в кеша стария размер на пръстена) |
| Зависване при рестартиране/изключване; по-късно `Invalid class` грешки в WMI, NIC не се активира, липсват USB устройства | Стар драйвер Realtek USB 10GbE от 2016 г. → BSOD `0x9F` → принудителни изключвания | Актуализирайте драйвера на адаптера до версия ≥ v10.67 (2026), след което приложете отново настройките за приемния пръстен по-горе |
| Свързването е успешно, но се връща резолюция, по-ниска от естествената | Smart-prep автоматично смалява рамката, за да се побере в кабела | Актуализирайте връзката / приемете смаляването / `--force-tier slip-emit-and-capture` |
| Масивът отчита нормална целева честота на кадрите, но предоставя само част от нея; `health.gvsp_corrupt_rate_pct` различна от нула, `never_arrived_rate_pct` 0 | Изчисленият превишава това, което той действително поддържа (типично за USB Ethernet адаптер, тънка PCIe линия или споделена мрежова структура) | Свържете се отново с по-ниска стойност на `--wire-ceiling-mbps` и проверете отново блока за състоянието. **Не** `--target-fps` — темпото на GevSCPD е фиксирано при свързване |
| Липсват камери от публикуваните групи; `health.never_arrived_rate_pct` е различно от нула, `gvsp_corrupt_rate_pct` 0 | Път за задействане/синхронизация — камерите не се задействат, не е проблем с мрежата | Проверете M8 синхронизиращия кабел и `--line`; потвърдете, че всеки член е активиран (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` са надвишени в `analyze-array` или отказ на връзка с фиксирано разрешение (`array over-subscribes the wire`) | Общото на камера (минимално 8 MB/s × N камери) надвишава горната граница на кабела, при която няма сблъсъци — 6 камери с пълна резолюция по 1 GbE при 1500 MTU, 9 с джъмбо | По-малко камери, джумбо-кадри от край до край или по-бърз мрежов адаптер. **ROI/бининг НЯМА да помогнат** (максималният праг не зависи от размера на кадрите). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` преодолява ограничението при тестове (приема загубата на пакети) |

---

## `chloros-cli daq`

Команди за спектрален сензор. Два класа:
- **`pool-*`**— олекотени HTTP клиенти, които управляват сензора чрез постоянния пул на бекенда.**Това е поддържаният път, и единственият, наличен в доставяния CLI.** Бекендът притежава транспорта, така че GUI, скриптовете CLI и SDK споделят един активен идентификатор, вместо да се борят за серийния порт.
- **Всичко останало**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — директен достъп до хардуера, описан по-долу за пълнота. За тях е необходим пакетът `daq` Python, който**не е включен в нито един от доставяните артефакти**: компилираният CLI го изключва (`scripts/Build-CLI.ps1` задава `--nofollow-import-to=daq`, а транспортите `pyserial` / `bleak` / `zeroconf` заедно с него), а пакетът PyPI SDK също не го съдържа. Те се изпълняват само от изтеглен изходен код, така че ги третирайте като MAPIR-вътрешен път за разработка, а не като нещо, към което да се стремите.
- **`discover` / `list`** се намират между двете: те са директни команди към хардуера от изтеглен изходен код, но в готовата версия се пренасочват към `pool-discover` и сканирането се извършва от бекенда. Така сканирането работи навсякъде — което е важно, защото това е единственият начин да се разбере BLE MAC адреса на DAQ-M.

> **`chloros-cli daq --help`** (и `-h` / `help`) изброява подкомандите на `pool-*` — помощната информация умишлено се пренасочва към клиента на пула, за да отразява командите, които действително се изпълняват. Ако извикате директнаподкоманда, свързана с хардуера, върху доставна версия, тя ще се излезе с явна грешка, посочваща липсващия пакет и насочваща ви обратно към `pool-*`; нищо не се проваля без предупреждение. (`discover` / `list` са изключение — те пренасочват към `pool-discover` и просто работят.)
>
> **Всичко, от което клиентът се нуждае, е достъпно чрез `pool-*`** — свързване, стрийминг, записвайте калибрирани `.daq` файлове и сменяйте профилите на капацитета. DAQ може също да се управлява от Python с `chloros_sdk.connect_daq_sensor()`, което използва същия обединен път.

### DAQ Работен процес при първо свързване на сензор

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Справка за `pool-*`

| Подкоманда | Цел |
| --- | --- |
| `daq pool-connect` (smart-detect) | Отваряне на сензор от пула на бекенда. |
| `daq pool-connect --port PORT` | DAQ-U на конкретен сериен порт. |
| `daq pool-connect --ble` | DAQ-M през BLE, автоматично сканиране на MAC. |
| `daq pool-connect --mac MAC` | DAQ-M през BLE на известен MAC (което предполага `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E през Ethernet на известен хост. |
| `daq pool-connect --eth` | DAQ-E през Ethernet, хостът еоткриване (mDNS + ARP като резервен вариант; работи при празен ARP кеш при Windows и Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Настройка на прозореца за интеграция / състоянието на AE. |
| `daq pool-connect --no-stream` | Свързване, но все още не започва стрийминг (възобновява се с `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Профил за корекция на капацитета. По подразбиране в бекенда е `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Сканира всеки транспорт за сензори, към които можете да се свържете, без да се свързвате. **По този начин се намира BLE MAC адреса на DAQ-M.** `daq discover` / `daq list` се насочва автоматично тук в готовите версии. Сензорите, които вече са отворени в пула, не се изброяват — свързан DAQ-M спира да се рекламира — затова използвайте `pool-list` за тях. |
| `daq pool-list` | Показване на всеки сензор в пула на бекенда. |
| `daq pool-disconnect --sensor-id ID [--all]` | Освобождаване. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Най-новите N спектрални рамки. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Възобновяване / пауза на стрийминга. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Стартиране / спиране на .daq запис. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Смяна на профила за корекция на капацитета по време на изпълнение. |

### Подкоманди за директен достъп до хардуера (само при изтегляне на изходния код — не са включени в готовите версии)

> Изброени са за пълнота. Те изискват пакета `daq` Python, както и `pyserial` / `bleak` / `zeroconf`, нито една от които не се доставя в компилираната версия CLI или в PyPI SDK — те се изпълняват само от изтеглен изходен код на MAPIR. **Ако използвате пусната версия на Chloros, използвайте вместо това командите `pool-*` по-горе**; те обхващат свързване, стрийминг, запис и избор на капацитет.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Отваряне, свързване и управление на запазен проект Chloros (папка с `cameras.json` + `sensors.json` + `project.json`). Всичко преминава през бекенда, така че графичният интерфейс и CLI генерират идентично състояние на хардуера.

### Справочник за подкоманди

| Подкоманда | Предназначение |
| --- | --- |
| `project open PATH` | Отпечатва списъка с устройствата в проекта (камери, масиви, сензори). |
| `project devices PATH [--reconnect]` | Извежда списък или повторно стартира откриването. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Свързване на всяка запазена камера / масив / сензор. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Единичен запис от посочена камера или масив. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Серия от N кадъра от посочена камера или масив (`-n/--count` по подразбиране 5; `-i/--interval` секунди между кадрите, по подразбиране 0). Сериите от масиви премахват дублираните синхронизирани групи (контрол за остаряване), така че масив с частичен цикъл не може да върне N копия на един кадър; отпечатва резултатите за всяка итерация. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Поток към диск чрез задача на бекенда. `--poll-interval` = секунди между `/stats` запитвания (по подразбиране 2,0). |
| `project sensor read PATH NAME [--json]` | Последната рамка на спектъра. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Записва .daq. |
| `project run PATH RECIPE.yaml` | Изпълнява рецепта за запис YAML/JSON. `--dry-run` проверява валидността без да изпълнява. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Изчисляване на подреждането за масив — вижте [таблицата с флагове по-долу](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Отпечатване на текущия профил за подреждане. |
| `project align clear PATH NAME` | Изтриване на кеширания профил. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Леко изместване на трансформацията на един подчинен. |
| `project align export PATH NAME --to FILE` | Запазване на профила в JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Зареждане на запазен профил. |

#### Опции на `project align calibrate`

| Флаг | По подразбиране | Описание |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Метод за подреждане. **Тези наименования се различават от `lattice align-calibrate`**, който използва съкратените форми `orb` / `akaze` / `phase`; двете команди не са взаимозаменяеми при този флаг. |
| `--model {translation, rigid, affine, homography}` | `affine` | Трансформиране на модела за прилягане. |
| `--frames N` | `1` | Синхронизирани снимки на кадри към средната стойност. |
| `--reference SN` | главната | Сериен номер на референтната камера; всички останали камери се деформират спрямо нея. |
| `--max-features N` | `5000` | Ограничение на броя на характеристиките на ORB. |
| `--ratio-threshold F` | `0.75` | Тест на съотношението на Лоу. |
| `--ransac-threshold-px F` | `3.0` | RANSAC припраг на вътрешните точки. |
| `--min-matches N` | `15` | **Праг за качество** — отхвърляне на решението, ако броят на съвпаденията с вътрешни точки е по-малък от този праг. |
| `--max-reproj-err-px F` | `4.0` | **Праг за качество** — отхвърля решаването над тази RMS грешка при репроекция. |
| `--checkerboard RxC` | — | Геометрия на платка за `--method checkerboard`, например `9x6`. |
| `--name PROFILE` | празно | Име на профила, вградено в запазения JSON. **Не е името на масива** — това е позиционният `NAME`. |

Двата контролни механизма за качество са причината, поради която калибрирането може да успее при решаването, но все пак
да откаже запазването: профил, който не отговаря на някой от тях, би довел до неправилна регистрация на всяко
последващо заснемане, затова се отхвърля, вместо да се запази.

### Примери

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### DSL за рецепти

`project run RECIPE.yaml` приема YAML или JSON файл, описващ поредица от действия:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Поддържани действия: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. Действието `burst` приема `name` (задължително), `count` (по подразбиране 5), `interval` (секунди, по подразбиране 0), `output`, `format` и `settings` (същата форма на настройките за всяка камера като `apply`); сериите от снимки използват същия „watchdog“ за наскоро синхронизирана група като `project burst`.

Изпълнете го:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Променливи на средата

| Променлива | Ефект |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Замества задната част URL (по подразбиране `http://127.0.0.1:5000`) — **се спазва само от семействата команди `lattice`, `project` и `daq pool-*`.** Основните команди (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) свързват `http://127.0.0.1:<port>` и игнорират тази променлива (литералът IPv4 заобикаля наказанието от ~2 секунди на заявка при Windows `localhost`→`::1`), така че те винаги се насочват към локалната машина. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` понижава нивото на отказ на свързване при свръхзаписване на масива (общо търсене на камера &gt; праг на кабела, защитен от сблъсъци, с `pin_resolution`) до силно предупреждение-и-продължаване, като се приема загуба на пакети по GVSP. Само за тестови цели — вижте [Модел за fps и пикова производителност на масива](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Задаван от самия CLI; указва на бекенда да активира паралелната обработка. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` пропуска GVSP пробата за резервен вариант (само ICMP резултати). **Това изключва джъмбо пакетите, а не просто заглушава лога** — камерата отговаря на DF пингове само до 1500 по всеки път, така че тази проверка е единственото нещо, което може да открие джъмбо пакети. Спестява ~1 с на камера на свързване; струва ~1,45× максималния капацитет на линията, ако мрежата *би могла* да пренася джъмбо пакети. SDK предупреждава, когато го настроите. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Фиксира размера на пакета GVSP на N байта; изцяло пропуска проучването. Предпочитайте настройката за всяка команда (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) пред постоянната настройка: фиксираният размер спира адаптирането към мрежата пред него, а фиксирането на 9000 по път, който не може да пренася джъмбо пакети, води до изтичане на времето за **всяко** заснемане с `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | Преопределяне на директорията за извличане на файлове от Nuitka. CLI автоматично използва `/mnt/ssd/tmp`, ако такъв съществува. |

---

## Кодове за изход

| Код | Означение |
| --- | --- |
| `0` | Успех. |
| `1` | Обща грешка (повечето грешки при подкоманди). |
| `2` | Грешка в аргумента. |
| `130` | Прекъснато с Ctrl+C. |

---

## Съвети за отстраняване на проблеми

- **„Изисква се вход“** → Изпълнете `chloros-cli login EMAIL PASSWORD` веднъж на този компютър.
- **„backend unreachable“** → Стартирайте десктоп приложението Chloros или изпълнете директно бинарния файл на бекенда (`chloros-backend`), или проверете `CHLOROS_BACKEND_URL`, ако е отдалечено.
- **Командите на `lattice` се провалят с съобщение „Не са намерени драйвери за камерата LATTICE“** → Средата за изпълнение на Arena SDK не е инсталирана; CLI се доставя с `win32api`, включен в Windows, но C runtime е част от инсталатора на графичния интерфейс.
- **В „Array connect“ / „Array Settings“ се показва „FRAMES WILL DROP“ или „Reduce ROI to enable“** → Приемният пръстен на мрежовата карта на хоста е прекалено малък (обикновено се възстановява на 32 след актуализация на драйвера на мрежовата карта). Вижте [Настройка и оптимизация на хост NIC](#host-nic-setup--tuning-lattice-arrays) — задайте `ReceiveBufferLen=256`, `PendingReceives=64`.
- **Машината зависва при рестартиране/изключване, след което WMI `Invalid class` / мрежовата карта не се активира / липсват USB устройства** → Остарял драйвер на USB 10GbE адаптера, причиняващ `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Актуализирайте драйвера на адаптера — вижте [Настройка и оптимизация на мрежовата карта на хоста](#host-nic-setup--tuning-lattice-arrays).
- **Предупреждение за swap на Jetson** → Добавете swap, поддържан от файл; CLI отпечатва точните команди `fallocate` / `swapon`.
- **Липсват директни команди за DAQ** → Очаквано: доставеният `chloros-cli` умишлено изключва пакета `daq`, така че е наличен само `pool-*` (PyPI SDK също не съдържа го). Използвайте `pool-*`, който управлява същия сензор чрез бекенда, или `chloros_sdk.connect_daq_sensor()` от Python.

---

## Вижте също

- [Справка за Python и SDK](sdk-reference.md) — програмно еквивалент на всяка команда на CLI.
- [Ръководство за DAQ сензори](../daq/README.md) — специфично за сензорите окабеляване + калибриране.
- Онлайн документация: `https://mapir.gitbook.io/chloros/cli`
