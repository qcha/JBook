# DNS

## Введение

Расшифруем аббревиатуру `DNS` -  это `Domain Name System`,  система доменных имён.

Вы уже наверняка встречались с `DNS`, хотя бы когда вбивали в браузере `www.yandex.ru`. Т.е некоторое доменное имя преобразовывалось в `ip` адрес с которым устанавливалось соединение и происходила дальнейшая работа.

Отсюда можно на вскидку сказать, что задача `DNS` - это преобразование некоторых доменных имен в `ip` адреса.
Хотя это не совсем так, потому что имена `DNS` может преобразовывать не только к `ip` адресам.

Поэтому правильнее было бы сказать, что `DNS` - это некоторая иерархическая система имен, за каждым из которых стоит тот или иной ресурс, например, `ip` адрес.

Типичным приложением `DNS` является `DNS resolver`, т.е преобразование, отображение имен в `ip` адреса (например, в браузере).

Вторым приложением `DNS` является электронная почта.
Вспомним, как выглядит почтовый адрес, на который мы хотим отправить письмо: `aarexer@gmail.com`.
Т.е мы указываем пользователя: `aarexer`, далее идет символ `@`, что значит `at` (на/в) и после идет почтовый домен.
За почтовым доменом стоят сервера, которые ответственны за прием и обработку писем.

И когда отправляется почта на `email` адрес, надо понимать: где взять для пользователя `aarexer` из домена `gmail.ru` сервера, которые обработают почтовый запрос?

И в этом случае, также как и в предыдущем, помогает система `DNS`, а именно ее ресурсы типа `MX`, mail exchanger.
Благодрая чему всегда понятно к какому серверу подключаться.

Возвращаясь к примеру с браузером и преобразованием там имени узла в `ip` адрес (наш пример с `www.yandex.ru`), использовались записи `DNS` типа `A`, address.

Разумеется и обратное действие, преобразование `ip` адреса в имя, также выполняется `DNS`, под это отведены ресурсы типа `PTR`, pointer, обратные указания.

И все эти ресурсы: `A`, `MX`, `PTR` и так далее, хранятся в `DNS`.

### Иерархичность

Как уже было отмечено выше, `DNS` - это иерархическая система имен.
Имена читаются справа налево и таким же образом упорядучиваются в дереве иерархии. Разделителем является точка.

Давайте присмотримся к имени `gmail.com`.

## Команды

### dig

Команда `dig` (domain internet grouper) созвучна с глаголом `копать`.
Названа она так не случайно, так как позволяет копаться в доменной системе имен!

Давайте сделаем запрос:

```bash
dig gmail.com
```

В результате получим ответ:

```bash

; <<>> DiG 9.10.6 <<>> gmail.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 34232
;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;gmail.com.			IN	A

;; ANSWER SECTION:
gmail.com.		282	IN	A	74.125.205.18
gmail.com.		282	IN	A	74.125.205.17
gmail.com.		282	IN	A	74.125.205.83
gmail.com.		282	IN	A	74.125.205.19

;; Query time: 5 msec
;; SERVER: 192.168.1.1#53(192.168.1.1)
;; WHEN: Sun Mar 27 18:35:36 +05 2022
;; MSG SIZE  rcvd: 102
```

Из ответа следует, что вопрос был задан один раз, было получено 4 ответа.

Сам вопрос был вида:

```bash
;; QUESTION SECTION:
;gmail.com.			IN	A
```

Т.е какой адрес имеет имя `gmail.com` в интернет классе тип ресурса `A`?

И этому имени соответствует некоторое количество (в данном случае 4) однотипных ресурсных записей адреса.

Почему их несколько? Ответ прост: из-за большой нагрузки, один адрес был бы не в состоянии обработать все запросы, поэтому это кластер из 4 узлов. И когда мы в браузере набираем `gmail.com` не важно какое имя будет взято, это однотипные узлы. За счет ротации этих узлов и получается распределение нагрузки. В этом можно убедиться, если вызвать команду `dig` еще раз: мы получим уже ротированную секцию `ANSWER SECTION`:

```bash
;; ANSWER SECTION:
gmail.com.		105	IN	A	64.233.161.17
gmail.com.		105	IN	A	64.233.161.83
gmail.com.		105	IN	A	64.233.161.18
gmail.com.		105	IN	A	64.233.161.19
```

Вопрос: а эти ли сервера ответственны за почту?
Нет! Потому что здесь указаны записи типа `A`, в то время как за почту отвечают `MX` записи.

Для выборки `MX` записей необходимо указать команде `dig` параметр `-t MX`, т.е. типы интересующих нас записей:

```bash
dig -t MX gmail.com
```

Ответом будет:

```bash
; <<>> DiG 9.10.6 <<>> -t MX gmail.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 48178
;; flags: qr rd ra; QUERY: 1, ANSWER: 5, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;gmail.com.			IN	MX

;; ANSWER SECTION:
gmail.com.		860	IN	MX	10 alt1.gmail-smtp-in.l.google.com.
gmail.com.		860	IN	MX	40 alt4.gmail-smtp-in.l.google.com.
gmail.com.		860	IN	MX	30 alt3.gmail-smtp-in.l.google.com.
gmail.com.		860	IN	MX	20 alt2.gmail-smtp-in.l.google.com.
gmail.com.		860	IN	MX	5 gmail-smtp-in.l.google.com.

;; Query time: 15 msec
;; SERVER: 192.168.1.1#53(192.168.1.1)
;; WHEN: Sun Mar 27 19:00:23 +05 2022
;; MSG SIZE  rcvd: 161
```

Заметим, что перед именем сервера теперь указывается некоторое число:

```bash
10 alt1.gmail-smtp-in.l.google.com.
40 alt4.gmail-smtp-in.l.google.com.
30 alt3.gmail-smtp-in.l.google.com.
20 alt2.gmail-smtp-in.l.google.com.
5 gmail-smtp-in.l.google.com.
```

Это вес, предпочтение.

Теперь посмотрим на те сервера, что ответственны за обработку почты у `yandex`:

```bash
dig -t MX yandex.ru
```

Результат:

```bash
; <<>> DiG 9.10.6 <<>> -t MX yandex.ru
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 33090
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 3, ADDITIONAL: 7

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;yandex.ru.			IN	MX

;; ANSWER SECTION:
yandex.ru.		2920	IN	MX	10 mx.yandex.ru.

;; AUTHORITY SECTION:
yandex.ru.		604800	IN	NS	ns9.z5h64q92x9.net.
yandex.ru.		604800	IN	NS	ns1.yandex.ru.
yandex.ru.		604800	IN	NS	ns2.yandex.ru.

;; ADDITIONAL SECTION:
mx.yandex.ru.		600	IN	A	77.88.21.249
ns1.yandex.ru.		334550	IN	A	213.180.193.1
ns2.yandex.ru.		264274	IN	A	93.158.134.1
mx.yandex.ru.		600	IN	AAAA	2a02:6b8::311
ns1.yandex.ru.		2040	IN	AAAA	2a02:6b8::1
ns2.yandex.ru.		2893	IN	AAAA	2a02:6b8:0:1::1

;; Query time: 59 msec
;; SERVER: 192.168.1.1#53(192.168.1.1)
;; WHEN: Sun Mar 27 19:19:23 +05 2022
;; MSG SIZE  rcvd: 257
```

И тут, обратите внимание, что в отличие от похожего запроса в `gmail.com`, нам вернулась еще и дополнительная секция:

```bash
;; ADDITIONAL SECTION:
mx.yandex.ru.		600	IN	A	77.88.21.249
ns1.yandex.ru.		334550	IN	A	213.180.193.1
ns2.yandex.ru.		264274	IN	A	93.158.134.1
mx.yandex.ru.		600	IN	AAAA	2a02:6b8::311
ns1.yandex.ru.		2040	IN	AAAA	2a02:6b8::1
ns2.yandex.ru.		2893	IN	AAAA	2a02:6b8:0:1::1
```

Возникает вопрос, а что, у серверов `gmail.com` нет ip?
Проверим:

```bash
dig  alt1.gmail-smtp-in.l.google.com.
```

И увидим, что все есть:

```bash
; <<>> DiG 9.10.6 <<>> alt1.gmail-smtp-in.l.google.com.
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 59708
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;alt1.gmail-smtp-in.l.google.com. IN	A

;; ANSWER SECTION:
alt1.gmail-smtp-in.l.google.com. 286 IN	A	142.250.157.27

;; Query time: 7 msec
;; SERVER: 192.168.1.1#53(192.168.1.1)
;; WHEN: Sun Mar 27 19:21:27 +05 2022
;; MSG SIZE  rcvd: 76
```

Почему же в одном случае нам дополнительная секция вернула значения, а в другом нет?

Потому что так решил сервер, **отвечающий** на запросы нашего клиента `dig`!

Сделаем запрос на `www.gmail.com`:

```bash
; <<>> DiG 9.10.6 <<>> www.gmail.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 5543
;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.gmail.com.			IN	A

;; ANSWER SECTION:
www.gmail.com.		4638	IN	CNAME	mail.google.com.
mail.google.com.	599005	IN	CNAME	googlemail.l.google.com.
googlemail.l.google.com. 200	IN	A	64.233.161.19
googlemail.l.google.com. 200	IN	A	64.233.161.17
googlemail.l.google.com. 200	IN	A	64.233.161.83
googlemail.l.google.com. 200	IN	A	64.233.161.18
```






Все взаимодействие в интернете - это клиент-серверное взаимодействие, `dig` - это клиент, который по определенному протоколу задает вопрос с
Команда `dig` - это клиент, задающий вопрос 

## nslookup

Name server lookup

Вызва команду:

```bash
nslookup gmail.com
```

Мы узнаем какому конкретно серверу и на какой порт был послан запрос:

```bash
Server:		192.168.1.1
Address:	192.168.1.1#53

Non-authoritative answer:
Name:	gmail.com
Address: 64.233.165.17
Name:	gmail.com
Address: 64.233.165.18
Name:	gmail.com
Address: 64.233.165.83
Name:	gmail.com
Address: 64.233.165.19
```

Из диапазона `192.168.xxx.xxx` мы знаем, что это адреса приватной сети, т.е обращение происходит к локальному серверу локальной сети.

RFC 1034 и RFC 1035

Показать картинку корневой домен и ниже .

Эта иерархия очень похоже на понятие абсолютного путевого имени файла на файловой системе, но в системе DNS - это называется FQDN, что означает полностью определенное доменное имя, Fuly Qualified Domain Name.

forum.dev.cisco.com.

Точка - это показатель абсолютного, полного имени.

И обратите внимание на то, что в командах выше мы указывали `gmail.com`, а вопрос был задан командой `dig`: `gmail.com.`, с точкой в конце!

В таком случае возникает вопрос - а почему и как работает браузер, когда мы не ставим точку в конце?

Браузер допускает, что мы указали полное имя, но не указали точку (забыли или просто поленились) и делает запрос в `DNS` с точкой, проверяя гипотезу о том, что это полное имя. Если это так - возвращается ресурс и ip адрес, как обычно.

Если же это не так, то проверяется гипотеза, что это не полное, а относительное имя.
Точно также как в файловой системе, когда мы можем указать полное имя до файла или имя, относительно текущей директории.

Похожим образом работает и `DNS`, добавляя имя поискового домена, если оно указано в настройках (resolv.conf).

Разница лишь в том, что поисковых доменов может быть несколько, а текущая директория может быть только одна.

В моем случае resolv.conf:

```bash
#
# macOS Notice
#
# This file is not consulted for DNS hostname resolution, address
# resolution, or the DNS query routing mechanism used by most
# processes on this system.
#
# To view the DNS configuration used by this system, use:
#   scutil --dns
#
# SEE ALSO
#   dns-sd(1), scutil(8)
#
# This file is automatically generated.
#
search localnet
nameserver 192.168.1.1
```

Поисковый домен у меня указан один и это localnet.

Ответим на вопрос: что это за 192.168.1.1?
Это кеширующий `DNS`

Информация о служебных серверах DNS также хранится в DNS, за это отвечают ресурсы `NS`: name server.

Сервера корневого домена - кто они?

```bash
dig -t NS .
```

Кто ответственен за зону?

```bash
dig -t SOA .
```
