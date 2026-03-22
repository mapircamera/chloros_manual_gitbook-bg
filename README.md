---
metaLinks: {}
---

# Първи стъпки

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>Chloros е софтуерно приложение от [MAPIR](https://www.mapir.camera) за обработка на изображения и други данни от сензори.

***{% hint style="success" %}**Какво ново в Chloros 1.1.0**: Вградена поддръжка на Linux (amd64 и arm64), NVIDIA Jetson edge computing, Dynamic Compute Adaptation, 4-нишкова обработваща верига, нови команди и опции на CLI. Вижте [Изтегляне](download.md) за пълния списък с промените.
{% endhint %}

Chloros е наличен в 3 режима на работа:

## Chloros: Десктоп GUI приложение

Самостоятелен отделен прозорец с всички функции. _Само за Windows._

## [Chloros CLI: Интерфейс на командния ред](CLI.md)

Пакетна обработка от командния ред. Идеално за автоматизация, създаване на скриптове и работа без графичен интерфейс. Налично на **Windows, Linux amd64 и Linux arm64 (NVIDIA Jetson)**. _CLI изисква лиценз Chloros+ за достъп._

## [Chloros API: Python SDK](api-python-sdk.md)

Програмен интерфейс Python за автоматизация и персонализирани работни потоци. Идеален за изследователски процеси, интеграция със съществуващи приложения Python и създаване на персонализирани инструменти. Наличен на **всички платформи** чрез `pip install chloros-sdk`. _API изисква лиценз Chloros+ за достъп._***

## Поддържани платформи

| Платформа | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Да | Да | Да |
| **Linux amd64 (x86_64)** | Не | Да | Да |
| **Linux arm64 (NVIDIA Jetson)** | Не | Да | Да |

За инструкции за инсталиране на Linux вижте раздела [Linux &amp; Edge Computing](linux/linux-overview.md).

***

## Chloros+

Макар Chloros да е безплатен за повечето задачи, може да установите, че искате повече. Ето тук платената лицензия за Chloros+ може да ви бъде от полза. С лиценз за Chloros+ можете да отключите нови функции, като например:

* **Многонишкова обработка**: значително ускорява обработката на изображения за по-големи проекти чрез едновременна обработка на изображенията по веригата.
* **Ускорение чрез GPU (CUDA)**: възползвайте се от по-големите възможности на днешните GPU памети, за да ускорите още повече процеса на обработка на изображенията. Препоръчваме 4 GB или повече VRAM за най-добри резултати.
* **Chloros+**[**CLI**](CLI.md)**Достъп**: стартирайте Chloros+ от командния ред, за да автоматизирате и интегрирате в собствения си софтуер.
* **Chloros+**[**API**](api-python-sdk.md)**Достъп:** стартирайте Chloros+ от Python за програмно управление, което позволява безпроблемна интеграция с вашите изследователски процеси, работни потоци за анализ на данни и персонализирани приложения.
* **Използване на няколко устройства**: всяка лицензия за Chloros+ позволява регистрирането на 2 или повече устройства. Използвайте своя акаунт в MAPIR Cloud, за да управлявате регистрираните устройства. Добавете поддръжка за повече устройства, като надстроите лицензията си за Chloros+.
* **Усъвършенстван метод за дебайеринг с отчитане на текстурата:** висококачествен дебайеринг с отчитане на ръбовете, комбиниран с AI/ML модел за отстраняване на шума, който премахва почти целия шум при дебайеринга. 
* **Персонализирани формули за мултиспектрални индекси:** въведете персонализирани мултиспектрални индекси в растерните калкулатори на Chloros, както за обработка, така и за тестовата среда за преглед на изображения.
* **Linux &amp; Edge Computing:** стартирайте Chloros на Linux x86\_64 и ARM64 платформи, включително NVIDIA Jetson, за обработка на място и в периферията. Вижте [Linux Общ преглед](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Chloros+ Цени и регистрация</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
