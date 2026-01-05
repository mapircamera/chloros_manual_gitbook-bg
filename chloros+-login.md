# Chloros+ Вход

## Chloros и Chloros (браузър) Вход

Потребителското <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> страничното меню ви позволява да влезете в акаунта си Chloros+ и да отключите допълнителни функции.

Когато влезете в акаунта си, ще се покажат данните ви:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI Вход

Влезте с вашите Chloros+ данни, за да активирате CLI обработка.

**Синтаксис:**

```bash
chloros-cli login <email> <password>
```

{% hint style=&quot;info&quot; %}
**Потребители на SDK**: Python SDK също предоставя програмна `logout()` метода за изчистване на кешираните данни за достъп. За подробности вижте [Python SDK документацията](api-python-sdk.md#logout).
{% endhint %}

**Пример:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Специални символи**: Използвайте единични кавички около пароли, съдържащи символи като `$`, `!` или интервали.
{% endhint %}

**Изход:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Изтичане на плана

Изтичането на плана в графичния интерфейс показва кога лицензът ви ще стане невалиден. За повтарящи се месечни абонаменти изтичането е в края на месеца. За годишни абонаменти то е една година след като сте започнали абонамента. Проверката на лиценза изисква месечна интернет връзка за потвърждение, с 30-дневен гратисен период.

### Ограничение на устройствата

Всеки план Chloros+ предлага различен брой регистрирани устройства. Всяко устройство, в което влизате с акаунт Chloros+, се отчита към броя на регистрираните устройства. Можете да преименувате и премахнете устройство на страницата на акаунта си в MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">План Chloros</th><th align="center">COPPER</th><th align="center">БРОНЗ</th><th align="center">СРЕБЪР</th><th align="center">ЗЛАТО</th></tr></thead><tbody><tr><td align="right">Поддържани устройства</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
