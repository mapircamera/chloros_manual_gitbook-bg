# Използване на Chloros с AI асистенти

Това ръководство е предназначено за две целеви групи: хората и AI асистентите, с които хората все по-често работят. На всяка страница са публикувани точни стойности, стойности по подразбиране и команди, които могат да се копират и поставят, така че асистентът (Claude, ChatGPT, Copilot, агент за програмиране и др.) да може да напише работеща автоматизация с Chloros още от първия опит.

Версия на Chloros: **

1.2.0**. Платформи за CLI/SDK: Windows 10/11 x64 и Linux (x86_64 / Jetson aarch64).

## Какво да предоставите на вашия асистент

| Ресурс | URL | За какво служи |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Машинно четим индекс на всяка страница в това ръководство. |
| **CLI Справочник** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Пълният набор от команди на `chloros-cli`: всяка команда, флаг, стойност по подразбиране, код на изход и правило за папката с изходните данни. Написано за използване от големи езикови модели (LLM). |
| **SDK Справочник** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Пълният `chloros_sdk` Python API: класове, сигнатури, изключения и примери с решения. Написано за LLM. |
| **Всяка страница като суров Markdown** | добавете `.md` към страницата URL | например `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` връща страницата като суров Markdown — идеален за поставяне в контекстен прозорец или извличане от агент. |

Връзки в наръчника: [CLI Справка](reference/cli-reference.md) · [SDK Справка](reference/sdk-reference.md).

{% hint style="info" %}
Двете справочни страници са самостоятелни: асистент, който е прочел една от тях, не се нуждае от останалата част от наръчника, за да напише правилен скрипт.
{% endhint %}

## Готови рецепти

Копирайте, попълнете `<placeholders>` и поставете в вашия асистент.

### 1. Обработка на папка с полети в NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Наблюдавайте пакетно директория със заснети данни

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Свържете матрица LATTICE и заснемете

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Записване на спектри от светлинни сензори на DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Скриптовете за DAQ от командния ред винаги преминават през семейството `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Други подкоманди от типа `daq`, които вашият асистент може да измисли, не са налични в доставяните версии и завършват с грешка.
{% endhint %}

## Защо скриптовете, написани от ИИ, работят добре с Chloros

Всяко от тях е реално, проверено поведение на Chloros 1.2.0 — те премахват класическите начини за провал на автоматизацията, написана от машина:

* **Без сложни настройки.**Помощните функции за интелигентно свързване на SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) и входните точки за обработка (`ChlorosLocal`, `process_folder`)**автоматично стартират локалния бекенд**. Генерираният скрипт не изисква отворен графичен интерфейс или ръчно стартиран сървър — необходимо е само да е инсталиран пакетът desktop/CLI.
* **Целият процес представлява едно извикване.** `chloros_sdk.process_folder("path", indices=["NDVI"])` изпълнява импорт → калибриране → отражателна способност → експорт на индекс от начало до край. По-малко повърхност, по-малко места, където генерираният скрипт може да даде грешка.
* **Изпълненията без резултат се самодиагностицират.** След `process()` обобщението на изпълнението се прикачва към резултата, а всеки съвет за обработката (например *защо* даден цикъл не е дал резултат) също се извежда отново като Python `UserWarning` — така че дори скрипт, който никога не проверява резултатния дикцион, показва диагнозата.
* **CLI се проваля шумно.**Изпълнение с код `chloros-cli process`, което е поискало резултати, но не е записало никакви, отпечатва `Processing finished but wrote no image products.` и**излиза с код, различен от нула**, така че скриптовете на шела и CI го откриват с обикновена проверка на кода на излизане. Успешните изпълнения отчитат `Image products written: N`.

Една асиметрия, която асистентът трябва да знае: `process()` на SDK умишлено **не** генерира изключение при изпълнение без продукти — вместо това докладва чрез обобщението/съветите. Ако пипалинът Python трябва да спре при празно изпълнение, проверете обобщението (рецепта 2 го прави).

## Предупреждения

* **Chloros+ изисква вход.**CLI и SDK изискват**платен** Chloros+ план, наложен от страна на сървъра: заявките се провалят с грешка `401 AUTH_REQUIRED`, когато не сте влезли в системата, и с `403 PLAN_UPGRADE_REQUIRED` при безплатния план. Изпълнете `chloros-cli login` веднъж на всяка машина, преди да стартирате генерираните скриптове. Вижте [Chloros+ Вход](chloros+-login.md).
* **Командите за заснемане управляват реален хардуер.** Командите `lattice` / `daq` / `project` и сесийните обекти SDK осъществяват връзка, предават поточно и задействат физически камери и сензори. Прегледайте генерирания скрипт преди първото му изпълнение и го изпълнете в присъствието на хардуера.
* **Проверете на случаен принцип изходните данни.** Проверете папките с резултати и няколко стойности на пиксели, преди да публикувате резултатите. По-специално, TIFF файловете с отражателна способност се мащабират според източника — прочетете XMP тага на `Chloros:PixelScale` (LATTICE: 32768 = 1,0 отражателна способност; Survey3: 65535), вместо да приемате делител по подразбиране. И двете справочни страници документират това в раздела „Четене на пиксели за отражателна способност“.
* **Малки капани, които пречат на генерирания код:**`pool-record` записва във файловата система на**хоста на бекенда** (по подразбиране `~/Documents/DAQ Live View/`); на машини с няколко мрежови интерфейса предпочитайте `daq pool-connect --eth-host <ip-or-hostname>` пред автоматичното откриване; и използвайте `http://127.0.0.1:5000` (никога `localhost`) навсякъде, където се появява бекенд URL.
