# Бързо начало с CLI (pool-*)

Доставяните `chloros-cli` управляват DAQ сензорите чрез семейството команди **`daq pool-*`** — „тънки“ HTTP-клиенти, които управляват сензора чрез постоянния пул от сензори на бекенда на Chloros. Бекендът контролира транспорта, така че GUI, CLI и SDK скриптовете споделят един активен идентификатор, вместо да се борят за порта. Всичко, от което се нуждае клиентът, е достъпно чрез `pool-*`: свързване, стрийминг, запис на калибрирани `.daq` файлове и смяна на профили на капацитета.

`pool-*` е поддържаната DAQ повърхност в пуснатите версии. `chloros-cli daq --help` изброява подкомандите на `pool-*`, а `daq discover` / `daq list` изпълняват същото сканиране като `pool-discover`. Когато подкоманда за директно хардуерно DAQ не е налична, програмата се изключва с явно съобщение за грешка, в което се посочва липсващият пакет и се пренасочва към `pool-*` — нищо не се проваля без предупреждение. (`pip install chloros-sdk` също не предоставя команди за директен достъп до хардуера.)

***

## Предварителни условия

* **Бекендът на Chloros трябва да работи** — командите `pool-*` са HTTP клиенти, а не хардуерни драйвери. В Windows стартирайте настолното приложение на Chloros (то стартира бекенда). На Linux/Jetson без графичен интерфейс активирайте услугата като потребител: `sudo systemctl enable --now chloros-backend@$USER.service` (вижте [Инсталиране на Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Вход в Chloros+ (платен план)**: първо изпълнете `chloros-cli login`. Проверката се извършва от страна на сървъра — без вход командите се провалят с `401 AUTH_REQUIRED`; в безплатния (Iron) план те се провалят с `403 PLAN_UPGRADE_REQUIRED`.
* По подразбиране командите са насочени към `http://127.0.0.1:5000`; семейството `daq pool-*` зачита променливата на средата `CHLOROS_BACKEND_URL`, ако вашият бекенд работи другаде.

***

## Петминутна сесия

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — отваряне на сензор в пула

| Вариант | Значение |
| --- | --- |
| `daq pool-connect` | Интелигентно откриване: намиране на всеки DAQ на тази машина. |
| `daq pool-connect --port PORT` | DAQ-U на конкретен сериен порт (например `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M през BLE, автоматично сканиране на MAC адреса. |
| `daq pool-connect --mac MAC` | DAQ-M на известен BLE MAC (означава `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E при известно име на хост или IP — **надеждният път**. |
| `daq pool-connect --eth` | DAQ-E с автоматично откриване (mDNS, с резервен вариант ARP). Вижте предупреждението по-долу. |

Флагове за настройка, всички са опционални:

| Флаг | Значение |
| --- | --- |
| `--integration-time MS` / `-t MS` | Ръчно зададено време за интегриране в милисекунди. |
| `--frame-avg N` / `-f N` | Средно число кадри за всеки отчетен спектър. |
| `--no-ae` | Деактивиране на автоматичната експозиция (AE е включена по подразбиране). |
| `--no-stream` | Свързване без стартиране на потока (възобновяване по-късно с `pool-stream --start`). |
| `--cap-id CAP` | Профил за корекция на капацитета; по подразбиране за бекенда е `sunshine_cosine`. Вижте [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

Свързването на DAQ-E, който вече се намира в пула под друг от неговите адреси — неговото име на хост `daq-e-<id>.local` или неговия IP адрес — връща съществуващия запис в пула, вместо да отваря устройството за втори път.

{% hint style="warning" %}
**Предупреждение относно автоматичното откриване на `--eth`.** На хост с няколко мрежови интерфейса (повече от един активен мрежов интерфейс) *първият* `pool-connect --eth` след стартиране може да не върне резултат, дори ако сензорът е в изправно състояние — търсенето може да пропусне интерфейса на сензора, докато ARP кешът е празен. Ако `--eth` не открие нищо, опитайте отново или пропуснете изцяло откриването с `--eth-host <ip-or-hostname>`, което е надеждният подход при машини с няколко мрежови интерфейса. Името на хоста на DAQ-E е `daq-e-<id>.local` (например `daq-e-def330.local`); обикновеният му IP адрес също работи.
{% endhint %}

## `pool-discover` — сканиране без свързване

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Сканира за сензори, към които можете да се свържете, и показва адреса за свързване, без да се осъществява връзка. `--only` приема подмножество, разделено със запетая, от `usb,ble,eth`; `--timeout SEC` задава времето за сканиране за всеки транспорт (по подразбиране 5 s, ограничено до 1–20 s). Сензорите, които вече са отворени в пула, не се изброяват (свързан DAQ-M спира да се обявява) — за тях използвайте `pool-list`. В готовия за доставка билд команди `chloros-cli daq discover` и `daq list` изпълняват същото сканиране.

## `pool-list` — показва какво е свързано

Показва всеки сензор в пула на бекенда, включително `sensor_id`, от който се нуждаят всички останали команди:

| Модел | Формат на `sensor_id` | Пример |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5-октет с тире | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

DAQ-E, чийто PTP е бил изключен (настройката се запазва в устройството), се маркира в `pool-list` с `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` и командата, която го включва отново — вижте [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — четене на спектрални рамки

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Връща най-новия кадър или най-новите `--recent N` кадри; `--json` генерира машинно четим изход за скриптове. Кадрите представляват радиометрично калибрирана спектрална интензивност на излъчването (W/m²/nm) върху решетка от 135 точки в диапазона 340–1010 nm, като профилът на капачката на сензора вече е приложен. За количествени стойности на интензитета на излъчването изчислете средната стойност от поне 15 секунди кадри — това е характеристика на уреда, а не дефект.

## `pool-stream` — пауза или възобновяване на стрийминга

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — записване на файл `.daq`

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Флаг | По подразбиране | Значение |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Продължителност на записването в секунди; `0` означава да се изпълнява, докато не издадете `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Директория за изходни файлове. При бекенд на този компютър относителният път се определя спрямо папката, в която сте изпълнили командата; при отдалечен бекенд пътят се определя **на машината, на която се изпълнява бекендът**. |
| `--device-name NAME` | — | Етикет, съхраняван заедно със записа. |
| `--stop` | — | Спиране на текущ запис. |
| `--confirm-after SEC` | `20` | След стартиране изчакайте до този период от време първият ред да бъде записан и изведете ясно предупреждение (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`), ако такъв не пристигне. `0` деактивира проверката. |
| `--heartbeat SEC` | `30` | С `--duration` отпечатвайте броя на редовете на всеки SEC секунди и извеждайте предупреждение, ако той спре да нараства. `0` го деактивира. |

{% hint style="info" %}
Записването се извършва в бекенда, така че файлът `.daq` се съхранява във файловата система на **бекенд машината** — по подразбиране в `~/Documents/DAQ Live View/` там, а не непременно там, където сте стартирали CLI. Имената на файловете включват идентификатора на сензора и времева марка.
{% endhint %}

## `pool-set-cap` — деклариране на монтираната капачка

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

Идентификаторът на капачката избира фабрично измерения коригиращ профил, прилаган към всеки спектър, и **трябва да съвпада с капачката, физически монтирана на сензора** — нито сензорът, нито софтуерът могат сами да разпознаят капачката, а изборът се записва във всеки файл `.daq`. По подразбиране навсякъде се използва `sunshine_cosine` (всеки DAQ се доставя с инсталиран капак „Sunshine“ за косинусна корекция, с ~12× затихване по проект — недекларирана смяна на капака води до неправилна корекция на спектрите приблизително с този коефициент).

| `--cap-id` | Наличен за |
| --- | --- |
| `sunshine_cosine` (по подразбиране) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | само DAQ-U |
| `none` | само DAQ-E — вижте бележката |

Идентификатор на капачката, който не е от набора на сензора, се отхвърля при свързване с ясна грешка. `none` (DAQ-E) означава, че капачката е физически премахната — все още се прилага фабричен геометричен профил за вдлъбнатия стъклен дифузер на DAQ-E, така че това не е „no-op“, а DAQ-E без капачка е конфигурация за лабораторни условия, а не поддържана конфигурация за работа на място. (Голият DAQ-U е напълно гол и изобщо не се нуждае от профил за корекция; DAQ-M се използва със своя капак „Sunshine“.)

## `pool-disconnect` — освобождаване на сензори

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` освобождава сензор, независимо кой го е отворил, така че освобождава и сензор, оставен в пула от сринал се скрипт (записът върху него първо трябва да приключи). Сензор, който не се намира в пула, няма какво да бъде освободен: командата извежда `<sensor-id>: already released (not in the pool)` и излиза от `0`. Освобождаване, което бекендът е отказал или не е потвърдил, излиза от `1`.

## `pool-enable-ptp` — включване на PTP на DAQ-E отново

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

DAQ-E съхранява настройката за включване/изключване на PTP в собствената си памет, така че устройство, чийто PTP е бил изключен, остава изключен — и неговите времеви отметки не са синхронизирани по PTP — докато не бъде включен отново. `pool-list` маркира такова устройство и отпечатва тази команда за него. Chloros никога не включва устройство отново от само себе си, тъй като изключването може да е умишлено. Ново в 1.3.0.

## Настройки на устройството DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` показва фърмуера на сензора (и дали тази версия на Chloros съдържа по-нова), неговия адрес и как го е получил, запазените мрежови настройки, ако се различават от текущите, стрийминг при включване и PTP. `pool-set-network` рестартира сензора, за да приложи промяната, и го свързва отново с предишните му настройки; `--no-reboot` само ги запазва. Статичният адрес трябва да се намира в мрежата на този компютър, в противен случай Chloros го отхвърля; `--force` пренебрегва това ограничение, но само за компютър, който ще бъде преадресиран или пренасочен към него. Рестартирането се отхвърля, докато сензорът записва. `pool-update-firmware` е еквивалентът на **Update to \<version\>** от графичния интерфейс за командния ред и начинът за актуализиране на DAQ-E в Linux, където няма графичен интерфейс. Ново в версия 1.3.0.

***

## Обобщение на командите

| Команда | Предназначение |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Отваряне на сензор в пула на бекенда. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Сканира за сензори без да се свързва (намира BLE MAC адреса на DAQ-M). |
| `daq pool-list` | Показване на всеки сензор в пула заедно с неговия `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Последните N калибрирани спектрални кадри. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Възобновяване / пауза на стрийминга. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Стартиране / спиране на запис на `.daq` (от страна на бекенда). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Смяна на профила за корекция на капацитета по време на изпълнение. |
| `daq pool-disconnect --sensor-id ID [--all]` | Освобождаване на един сензор или на всички. |
| `daq pool-enable-ptp --sensor-id ID` | Включване на PTP за DAQ-E, чийто PTP е изключен. |
| `daq pool-info --sensor-id ID [--json]` | Фърмуер, адрес, запазени мрежови настройки, стрийминг при включване и PTP на DAQ-E. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP или статичен адрес; рестартира и пресъединява сензора. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Стриминг при включване. |
| `daq pool-reboot --sensor-id ID` | Рестартира DAQ-E и го свързва отново. |
| `daq pool-update-firmware --sensor-id ID` | Записва фърмуера на DAQ-E от тази версия на по-стар сензор. |

***

## Отстраняване на проблеми при първото свързване на DAQ-E

1. DAQ-E няма LED индикатор за състоянието — проверете дали има захранване чрез индикатора за PoE/връзка на комутатора или порта на инжектора и изчакайте няколко секунди след включването, за да се стартира и да се присъедини към мрежата.
2. Сървърът трябва да се намира в **същия домейн за разпространение** като сензора — mDNS не преминава през рутери.
3. Защитната стена трябва да пропуска mDNS (UDP 5353), данните от DAQ-E (UDP 5002) и PTP (UDP 319/320). Инсталаторът за Windows добавя правило за това, а за Linux `.deb` ги отваря в `ufw` / `firewalld`, който е бил активен по време на инсталирането; затова е необходимо те да бъдат разрешени от защитна стена на трета страна или от такава, активирана по-късно.
4. Все още няма отговор от `--eth`? Използвайте `--eth-host` с името на хоста на устройството (`daq-e-<id>.local`) или IP адреса, което прескача откриването.

***
{% hint style="info" %}
**Съвет за AI асистентите.** Всяка страница от това ръководство се предоставя като суров Markdown — добавете `.md` към малките букви на URL адреса на страницата (тази страница: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); машинно четимият индекс е `https://mapir.gitbook.io/chloros/llms.txt`. За пълна документация на ниво флагове за `chloros-cli daq` и всяко друго семейство команди, изтеглете [Справочника за CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); Python-пътят е `chloros_sdk.connect_daq_sensor()` в [SDK Reference](../reference/sdk-reference.md).
{% endhint %}
