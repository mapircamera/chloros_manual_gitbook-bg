# Вход в Chloros+

## Вход в Chloros и Chloros (браузър)

Страничното меню <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> страничното меню ви позволява да влезете в акаунта си Chloros+ и да отключите допълнителни функции.

След като влезете, ще се покажат данните за вашия акаунт:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI Вход

Влезте с вашите данни за Chloros+, за да активирате обработката на CLI. На Linux (без графичен интерфейс) това е единственият начин да активирате лиценза си.

**Синтаксис:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Потребители на SDK**: Python SDK също предоставя програмна метода `logout()` за изчистване на кешираните идентификационни данни. Вижте [Python SDK документацията](api-python-sdk.md#logout) за подробности.
{% endhint %}

**Пример:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Специални символи**: Използвайте единични кавички около пароли, съдържащи символи като `$`, `!` или интервали.
{% endhint %}

**Резултат:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Съхранение на удостоверения

Кешираните удостоверения се съхраняват в място, специфично за платформата:

| Платформа | Път към кеша на удостоверенията |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### Изтичане на плана

Изтичането на плана в графичния интерфейс показва кога лицензът ви ще стане невалиден. За периодични месечни абонаменти изтичането е в края на месеца. За годишни абонаменти то е една година след като сте започнали абонамента. Проверката на лиценза изисква месечна интернет връзка за потвърждение, с 30-дневен гратисен период.

### Ограничение за устройства

Всеки план Chloros+ предлага различен брой регистрирани устройства. Всяко устройство, в което влизате с акаунт Chloros+, ще се отчита към броя на регистрираните ви устройства. Можете да преименувате и премахнете устройство на страницата на вашия акаунт в MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">План Chloros+</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">СИЛВЪР</th><th align="center">ЗЛАТО</th></tr></thead><tbody><tr><td align="right">Поддържани устройства</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
