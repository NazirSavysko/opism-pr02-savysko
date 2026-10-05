# Практична робота № 2
 
**Дисципліна:** Основи побудови інформаційних систем та мереж (ОК-13)
 
**Тема:** Структура повідомлень прикладного протоколу HTTP. Формування запиту вручну
 
| Поле | Значення |
|---|---|
| Студент (прізвище, ім'я, по батькові) | Сависько Назір Михайлович |
| Група | ІПЗ-2.01 |
| Номер варіанта | 27 |
| Індивідуальний домен | `freebsd.org` |
| «Чужий» домен для завдання A.3.1 | `cern.ch` (варіант 12) |
| Середовище виконання | Linux (Ubuntu), OpenBSD netcat 1.234, curl 8.18.0 |
| Дата виконання | 05.10.2026 |
 
> Бланк заповнюють, не змінюючи структури розділів. Порожні заготовки блоків коду замінюють власними виводами. Позначки-підказки в кутових дужках вилучають.
 
---
 
## Частина A. Збір експериментальних даних
 
### Завдання A.1. Формування запиту вручну
 
**Команда:**
 
```
nc -C freebsd.org 80
```
 
**Набраний запит:**
 
```
GET / HTTP/1.1
Host: freebsd.org
Connection: close
 
```
 
**Відповідь:**
 
```
HTTP/1.1 301 Moved Permanently
Server: nginx
Content-Type: text/html
Content-Length: 162
Connection: close
Location: https://freebsd.org/
 
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
```
 
---
 
### Завдання A.2. Запит без поля `Host` у версії 1.1
 
**Команда:**
 
```
printf 'GET / HTTP/1.1\r\nConnection: close\r\n\r\n' | nc freebsd.org 80
```
 
**Вивід:**
 
```
HTTP/1.1 400 Bad Request
Server: nginx
Content-Type: text/html
Content-Length: 150
Connection: close
 
<html>
<head><title>400 Bad Request</title></head>
<body>
<center><h1>400 Bad Request</h1></center>
<hr><center>nginx</center>
</body>
</html>
```
 
---
 
### Завдання A.3. Вплив поля `Host` на відповідь сервера
 
#### A.3.1. Чуже доменне ім'я в полі `Host`
 
**Команда:**
 
```
printf 'GET / HTTP/1.1\r\nHost: cern.ch\r\nConnection: close\r\n\r\n' | nc freebsd.org 80
```
 
**Вивід:**
 
```
HTTP/1.1 301 Moved Permanently
Server: nginx
Content-Type: text/html
Content-Length: 162
Connection: close
Location: https://cern.ch/
 
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
```
 
#### A.3.2. Неіснуюче ім'я в полі `Host`
 
**Команда:**
 
```
printf 'GET / HTTP/1.1\r\nHost: opism-pr02.invalid\r\nConnection: close\r\n\r\n' | nc freebsd.org 80
```
 
**Вивід:**
 
```
HTTP/1.1 301 Moved Permanently
Server: nginx
Content-Type: text/html
Content-Length: 162
Connection: close
Location: https://opism-pr02.invalid/
 
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
```
 
#### A.3.3. Запит без поля `Host` у версії 1.0
 
**Команда:**
 
```
printf 'GET / HTTP/1.0\r\n\r\n' | nc freebsd.org 80
```
 
**Вивід:**
 
```
HTTP/1.1 301 Moved Permanently
Server: nginx
Content-Type: text/html
Content-Length: 162
Connection: close
Location: https:///
 
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
```
 
Зведення результатів наведено в **Додатку Д**.
 
---
 
### Завдання A.4. Два запити в одному з'єднанні
 
**Команда:**
 
```
printf 'GET /opism-pr02-12345 HTTP/1.1\r\nHost: freebsd.org\r\n\r\nGET / HTTP/1.1\r\nHost: freebsd.org\r\nConnection: close\r\n\r\n' | nc -C freebsd.org 80
```
 
**Вивід:**
 
```
HTTP/1.1 301 Moved Permanently
Server: nginx
Content-Type: text/html
Content-Length: 162
Connection: keep-alive
Location: https://freebsd.org/opism-pr02-12345
 
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
HTTP/1.1 301 Moved Permanently
Server: nginx
Content-Type: text/html
Content-Length: 162
Connection: close
Location: https://freebsd.org/
 
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
```
 
**Кількість отриманих відповідей:** 2
 
**Коди стану отриманих відповідей:** 301, 301
 
---
 
### Завдання A.5. Запит за допомогою клієнтської програми
 
**Команда:**
 
```
curl -v --http1.1 http://freebsd.org/ -o /dev/null
```
 
**Вивід:**
 
```
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
  0      0   0      0   0      0      0      0                              0* Host freebsd.org:80 was resolved.
* IPv6: 2610:1c1:1:606c::50:15
* IPv4: 96.47.72.84
*   Trying [2610:1c1:1:606c::50:15]:80...
* Immediate connect fail for 2610:1c1:1:606c::50:15: Не вдалося отримати доступ до мережі
*   Trying 96.47.72.84:80...
* Established connection to freebsd.org (96.47.72.84 port 80) from 192.168.10.153 port 59188 
* using HTTP/1.x
> GET / HTTP/1.1
> Host: freebsd.org
> User-Agent: curl/8.18.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 301 Moved Permanently
< Server: nginx
< Content-Type: text/html
< Content-Length: 162
< Connection: keep-alive
< Location: https://freebsd.org/
< 
{ [162 bytes data]
100    162 100    162   0      0    449      0                              0
* Connection #0 to host freebsd.org:80 left intact
```
 
---
 
### Завдання A.6. Запит через захищене з'єднання
 
**Ресурс, на якому виконано завдання:** власний домен `freebsd.org`
 
**Підстава для використання резервного ресурсу (заповнюють за потреби):** не знадобилася — у завданні A.1 отримано код 301, з'єднання з портом 443 встановлено.
 
**Команда:**
 
```
openssl s_client -connect freebsd.org:443 -servername freebsd.org -crlf -quiet
```
 
**Набраний запит:**
 
```
GET / HTTP/1.1
Host: freebsd.org
Connection: close
 
```
 
**Вивід:**
 
```
HTTP/1.1 301 Moved Permanently
Server: nginx
Content-Type: text/html
Content-Length: 162
Connection: close
Location: https://www.freebsd.org/
Strict-Transport-Security: max-age=31536000
X-Content-Type-Options: nosniff
X-XSS-Protection: 1; mode=block
X-Frame-Options: SAMEORIGIN
 
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
```
 
---
 
## Частина B. Розбір полів заголовка
 
Розбирається відповідь, отримана в завданні A.1.
 
**Загальна кількість полів заголовка у відповіді:** 5
 
| № | Поле заголовка | Значення | Призначення (власне формулювання) | Походження: сервер / проміжний вузол / не визначено | Обґрунтування |
|---|---|---|---|---|---|
| 1 | `Server` | `nginx` | Назва програми сервера | не визначено | Порт 80 тільки перенаправляє, тому це може бути вузол перед сайтом. Полів `Via`, `Age`, `X-Cache` немає, тож точно не визначити |
| 2 | `Content-Type` | `text/html` | Тип вмісту: HTML-сторінка | сервер | Описує тіло відповіді. Сервер сам видав коротку HTML-сторінку з написом `301 Moved Permanently`. |
| 3 | `Content-Length` | `162` | Розмір тіла в байтах | сервер | Число порахував сам сервер. В а.1 тіло `301` має 162 байти, а в а.2 тіло `400` інше, тому там `150`. Клієнт за цим числом знає, де закінчується відповідь |
| 4 | `Connection` | `close` | Закрити з'єднання після відповіді | сервер | Повторює моє прохання з запиту. Коли в а.4 я його не вказав, сервер відповів `keep-alive` |
| 5 | `Location` | `https://freebsd.org/` | Куди перейти | сервер | Береться з мого `Host`: з `cern.ch` вийшло `https://cern.ch/` |
 
> Рядок наводять на кожне поле, яке реально надійшло. Зайві рядки вилучають, за потреби додають нові. Поле, походження якого встановити не вдалося, зазначають із позначкою «не визначено» та поясненням утруднення.
 
---
 
## Частина D. Висновки
Обсяг — 150–300 слів. Висновки спираються на власні спостереження.
 
**D.1.** Що з поведінки сервера виявилося неочевидним або несподіваним. Конкретно, з посиланням на рядок виводу.
 
Мене здивувало, що сервер не перевіряє ім'я в Host. З Host: opism-pr02.invalid він просто перенаправив на https://opism-pr02.invalid/ (A.3.2)
 
**D.2.** Яке з полів заголовка викликало найбільше утруднення при визначенні походження (частина B) та з якої причини.
 
Найважче було визначити, чи відповідає сам сервер, чи проміжний вузол (проксі). Через Server: nginx це не видно, а полів Via, Age, X-Cache немає. Тому я поставив «не визначено»
 
**D.3.** Яке питання залишилося без відповіді після виконання роботи.
 
Не зрозумів, чому http і https перенаправляють по-різному. Через http мене відправило на https://freebsd.org/, а через https уже на https://www.freebsd.org/
 
---
 
## Контрольні питання
 
**1.** У завданні A.1 сервер не надсилав відповіді, доки не було введено порожній рядок. Чим це зумовлено?
 
Порожній рядок означає кінець запиту, тому сервер чекав, поки я його введу
 
**2.** Порівняйте результати завдань A.1, A.2 та A.3.1–A.3.3 (таблиця Додатка Д). За яких значень поля `Host` і за якої версії протоколу сервер обслуговує запит, а за яких — ні? Яку задачу розв'язує поле `Host`? Відповідь має посилатися на конкретні рядки ваших виводів.
 
Тільки 1.1 без Host дав 400. Усі інші запити дали 301, і навіть чуже та неіснуюче ім'я сервер просто підставив у Location, наприклад https://cern.ch/. Без Host у версії 1.0 теж 301, але Location: https:///. Host потрібен, бо на одній IP може бути кілька сайтів і сервер має знати, який віддати
 
**3.** Скільки відповідей надійшло у завданні A.4 і з якими кодами стану? Чи залежить відповідь сервера на порту 80 від запитаного шляху — і що це говорить про роль цього сервера? Якщо надійшла одна відповідь, знайдіть у ній поле заголовка, яке це пояснює, або зазначте, що такого поля немає. Якщо надійшло дві — що це означає для клієнтської програми, яка завантажує сторінку з великою кількістю вкладених ресурсів?
 
Прийшло дві відповіді, обидві 301. Від шляху відповідь не залежить: сервер на порту 80 тільки перенаправляє на https, шлях переносить у Location. Дві відповіді означають, що з'єднання не закрилось після першої, тож браузер може качати багато файлів через одне з'єднання (в першій відповіді Connection: keep-alive)
 
**4.** Які поля заголовка програма `curl` додала самостійно (завдання A.5)? Ці поля не є обов'язковими — сервер відповів і без них у завданні A.1. З якою метою їх додано?
 
`curl` сам додав `User-Agent: curl/8.18.0` і `Accept: */*`. `User-Agent` каже серверу, яка програма запитує, а `Accept` — що клієнт приймає будь-який тип даних
 
**5.** За якими ознаками у вашому виводі виявляється присутність проміжного вузла? Якщо таких ознак не виявлено, поясніть, що з цього випливає.
 
Полів типу `Via`, `X-Cache` чи `Age` немає, тому явних ознак проміжного вузла не видно. Але це не означає, що його точно немає: він може просто не додавати своїх полів
 
**6.** Три рядки, про які не йшлося на лекції, наведено в **Додатку В**.
 
---
 
## Додаток В. Відповіді на питання 6
 
| № | Рядок виводу | Джерело (номер завдання) |
|---|---|---|
| 1 | `Strict-Transport-Security: max-age=31536000` | A.6 |
| 2 | `Connection: keep-alive` | A.4 |
| 3 | `* Immediate connect fail for 2610:1c1:1:606c::50:15: Не вдалося отримати доступ до мережі` | A.5 |
 
---
 
## Додаток Д. Зведення результатів завдання A.3
 
**Вузол, з яким установлювалося з'єднання (у всіх пробах однаковий):** `freebsd.org` (96.47.72.84, порт 80)
 
| Проба | Значення поля `Host` | Версія | Код стану | Обсяг тіла відповіді | Збігається з A.1 (так / ні) |
|---|---|---|---|---|---|
| A.1 (вихідна) | `freebsd.org` | 1.1 | 301 | 162 | — |
| A.2 | поле відсутнє | 1.1 | 400 | 150 | ні |
| A.3.1 | `cern.ch` | 1.1 | 301 | 162 | ні — `Location: https://cern.ch/` |
| A.3.2 | `opism-pr02.invalid` | 1.1 | 301 | 162 | ні — `Location: https://opism-pr02.invalid/` |
| A.3.3 | поле відсутнє | 1.0 | 301 | 162 | ні — `Location: https:///` |
 
**Висновок за таблицею (2–4 речення):** що саме змінювалося у запиті від проби до проби і як на це реагував сервер.
 
Змінювались тільки `Host` і версія. Сервер відмовив лише тоді, коли в 1.1 не було `Host` (`400`). З будь-яким іменем він давав `301` і підставляв це ім'я в `Location`, а в 1.0 без `Host` вийшло `https:///`.
 
---
## Декларування використання технологій штучного інтелекту

Для цієї роботи встановлено **рівень Р3 — ШІ як співвиконавець**.

Виводи команд частини A не можуть бути згенеровані та мають бути отримані внаслідок фактичного виконання команд.

**Чи використовувалися технології ШІ під час виконання роботи:** ні

Якщо так, заповнюють таблицю. Якщо ні, таблицю вилучають.

| № | Інструмент (назва, версія) | Етап роботи | Дослівний текст запиту (промпту) | Як використано результат |
|---|---|---|---|---|
| 1 | | | | |
| 2 | | | | |

**Підтвердження:** усі виводи команд, наведені в частині A, отримано внаслідок фактичного виконання команд на зазначеному індивідуальному домені.

---

*ОПІСМ (ОК-13) · Практична робота № 2 · бланк звіту*
 
