# Използване на Chloros с AI асистенти

Това ръководство е предназначено за две аудитории: хората и AI асистентите, с които хората все по-често работят. Всяка страница съдържа точни стойности, стойности по подразбиране и команди, които могат да се копират и поставят, така че асистентът (Claude, ChatGPT, Copilot, агент за кодиране и др.) да може да напише работеща автоматизация в Chloros още от първия опит.

Версия на Chloros: **

1.3.0**. Платформи за CLI/SDK: Windows 10/11 x64 и Linux (x86\_64 / Jetson aarch64).

## Какво да предоставите на вашия асистент

| Ресурс | URL | За какво служи |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Машинно четим индекс на всяка страница в това ръководство. |
| **Справочник за CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Пълният набор от команди на `chloros-cli`: всяка команда, флаг, стойност по подразбиране, код на изход и правило за папката с резултатите. Написан за използване от LLM. |
| **Справочник за SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Пълният Python API на `chloros_sdk`: класове, сигнатури, изключения и примери с решения. Написан за използване от LLM. |
| **Всяка страница като суров Markdown** | добавете `.md` към URL-а на страницата | например `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` връща страницата като суров Markdown — идеално за поставяне в контекстен прозорец или извличане от агент. |

Връзки в ръководството: [Справочник за CLI](reference/cli-reference.md) · [Справочник за SDK](reference/sdk-reference.md).

{% hint style="info" %}
Двете справочни страници са самостоятелни: асистент, който е прочел една от тях, не се нуждае от останалата част от ръководството, за да напише правилен скрипт.
{% endhint %}

## Позволете на вашия асистент да стартира Chloros директно: `chloros-mcp`

Всичко, описано по-горе, изисква вашият асистент да *напише* скрипт или команда. С `chloros-mcp` асистент, който използва [Model Context Protocol](https://modelcontextprotocol.io) (MCP), като например Claude Desktop, Claude Code или Cursor, управлява самия Chloros. Вие задавате въпроса на обикновен език и той извиква инструментите на сървъра:

* **Обработка на изображения:** създаване на проект, импортиране на папка със заснети изображения, избор на настройки, обработка и отчитане на резултатите от експортирането.
* **Управление на хардуера:** свързване на камери LATTICE, масиви и светлинни сензори DAQ, преглед, настройка на експозицията и цвета, заснемане, запис и актуализиране на фърмуера.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Инструментите на сървъра обхващат функциите на [Python SDK](api-python-sdk.md). Той се инсталира заедно с SDK, така че обикновено не е необходимо да инсталирате нищо друго: в Windows инсталаторът на Chloros поставя `chloros-mcp.exe` в папката `Scripts` на вашия Python, а инсталаторът за Linux `.deb` поставя `chloros-mcp` в `/usr/bin`.

**Код на Claude:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**добавете това към `claude_desktop_config.json` (отваря се от**Настройки &gt; Разработчик &gt; Редактиране на конфигурацията**), след което рестартирайте Claude Desktop.**Курсорът** използва същия запис в `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Ако клиентът съобщи, че сървърът не е успял да стартира, вероятно `chloros-mcp` не фигурира в списъка, който клиентът вижда (`PATH`): посочете пълния път вместо това (`where chloros-mcp` в Windows, `which chloros-mcp` в Linux).

Необходимо е Chloros да е инсталиран на същия компютър и да имате профил там, в приложението или чрез `chloros-cli login`, на някой от платените планове на Chloros+ (Copper или по-висок). Сървърът не може да влезе от ваше име.

{% hint style="info" %}
Настройки на клиента за всяко приложение, опции, всеки инструмент и поведение при изпълнение: [MCP Server (AI Assistants)](reference/sdk-reference.md#mcp-server-ai-assistants) в справочника за SDK.
{% endhint %}

## Рецепти за команди

Копирайте, попълнете `<placeholders>` и поставете в асистента си.

### 1. Преобразуване на папка с полети в NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Групово наблюдение на директория със заснети данни

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Свързване на матрица LATTICE и заснемане

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
Скриптовете за DAQ от командния ред винаги преминават през семейството `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Други подкоманди от типа `daq`, които вашият асистент може да измисли, не са налични в доставните версии и водят до излизане от програмата с грешка.
{% endhint %}

## Защо скриптовете, написани от ИИ, работят добре с Chloros

Всяко от тези е реално, проверено поведение на Chloros 1.3.0 — те премахват класическите начини за провал на автоматизацията, написана от машини:

* **Няма сложни настройки.**Помощните функции за интелигентно свързване на SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) и входните точки за обработка (`ChlorosLocal`, `process_folder`)**автоматично стартират локалния бекенд**. Генерираният скрипт не се нуждае от отворен графичен интерфейс или ръчно стартиран сървър — необходимо е само да е инсталиран пакетът за настолен компютър/CLI.
* **Целият процес се изпълнява с едно извикване.** `chloros_sdk.process_folder("path", indices=["NDVI"])` изпълнява импорт → калибриране → отражателна способност → експорт на индекс от начало до край. По-малко повърхност, по-малко места, където генерираният скрипт може да даде грешка.
* **Изпълненията без резултат се самодиагностицират.** След `process()` обобщението на изпълнението се прикачва към резултата, а всеки съвет за обработката (например *защо* даден цикъл не е дал резултат) също се извежда отново като Python `UserWarning` — така че дори скрипт, който никога не проверява резултатния дикционер, показва диагнозата.
* **CLI-интерфейсът дава ясен сигнал за грешка.**Изпълнение с код `chloros-cli process`, което е поискало резултати, но не е записало никакви, отпечатва `Processing finished but wrote no image products.` и**излиза с ненулев код**, така че скриптовете на шела и CI го откриват с обикновена проверка на кода на излизане. Успешните изпълнения отчитат `Image products written: N`, както и `(plus M clamp masks)`, когато са записани маски. Командите за запис също така посочват всеки файл, който не са успели да запишат, и излизат с `1`.
* **Относителните пътища означават това, което скриптът очаква.** При локален бекенд относителният път за изход (`-o out/` в CLI, `output_dir="out/"` в SDK) се преобразува спрямо папката, от която се изпълнява командата или скриптът.

Една асиметрия, която асистентът трябва да знае: `process()` на SDK умишлено **не** генерира изключение при изпълнение с нулев резултат — вместо това докладва чрез обобщението/подсказките. Ако Python-пипалината трябва да спре при празно изпълнение, проверете обобщението (рецепта 2 го прави).

## Предупреждения

* **Необходим е Chloros+ login.**CLI и SDK изискват**платен** план на Chloros+, което се налага от страна на сървъра: заявките се провалят с грешка `401 AUTH_REQUIRED`, когато не сте влезли в системата, и с грешка `403 PLAN_UPGRADE_REQUIRED` при безплатния план. Изпълнете `chloros-cli login` веднъж на всяка машина, преди да стартирате генерираните скриптове. Вижте [Вход в Chloros+](chloros+-login.md).
* **Командите за заснемане управляват реален хардуер.** Командите `lattice` / `daq` / `project`, обектите на сесията от SDK и инструментите на `chloros-mcp` се свързват, предават поток и задействат физически камери и сензори. Прегледайте генерирания скрипт преди първото му изпълнение и го изпълнете, докато имате достъп до хардуера.
* **Проверете на случаен принцип изходните данни.** Проверете папките с продуктите и няколко стойности на пикселите, преди да публикувате резултатите. По-специално, TIFF файловете с отражателна способност се мащабират според източника — прочетете XMP тага на `Chloros:PixelScale` (файловете LATTICE го съдържат: 32768 = 1,0 отражателна способност; файловете Survey3 не го съдържат и използват 65535 = 1,0), вместо да приемате делител. И двете справочни страници документират това в раздела „Четене на пиксели за отражателна способност“.
* **Малки капани, които затрудняват генерирания код:**`pool-record` записва във файловата система на**хоста на бекенда** (по подразбиране `~/Documents/DAQ Live View/`); на машини с няколко мрежови интерфейса предпочитайте `daq pool-connect --eth-host <ip-or-hostname>` пред автоматичното откриване; използвайте `http://127.0.0.1:5000` (никога `localhost`) навсякъде, където се появява URL на бекенда; скрипт, който обхожда дървото с изходни файлове на проекта, трябва да пропуска папките `clamp_masks/` (файловете в тях с имена `*_clampmask.tif` са маски, а не изображения — вижте [Формати на изходните изображения](output-image-formats.md#clamp-masks)); и в Windows, където дългите пътища са деактивирани, поддържайте папките на проекта къси — папка на проект, чието име е по-дълго от около 130 символа, се отхвърля преди започването на обработката.
