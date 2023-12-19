## SimpleDateFormat
Разумеется, должен существовать и какой-то специальный класс для парсинга и форматирования даты в `Java`.
Для `Java Time API 1.0` - это `java.text.SimpleDateFormat`.

//todo
И, разумеется, часто дата представляется или передается в приложение в виде строки:
`2016.09.04 AD at 12:08:56 PDT`.
Но работать с таким представлением - крайне неудобно, представим, например,
что вы хотите к текущей дате прибавить нескоолько дней - вообразите насколько сложно это сделать,
насколько это будет подвержено ошибкам, когда дата у вас представлена просто строкой.

При этом, дата в стркое может быть записана по-разному, например, `Fri, June 5 2014` или `21/08/2016`.

Что делать?
Использовать парсер, который разберет строковое представление даты и вернет вам объект с типом даты,
с которым удобно будет работать.


Надо понимать, что он парсит по некоему паттерну, который вы передаете в конструктор класса.
Что можно передавать:

Letter | Date or Time Component | Presentation       |  Examples
------ | ---------------------- | ------------------ | -------------------------------------
G      | Era designator         | Text               | AD
y      | Year                   | Year               | 1996; 96
Y      | Week year              | Year               | 2009; 09
M/L    | Month in year          | Month              | July; Jul; 07
w      | Week in year           | Number             | 27
W      | Week in month          | Number             | 2
D      | Day in year            | Number             | 189
d      | Day in month           | Number             | 10
F      | Day of week in month   | Number             | 2
E      | Day in week            | Text               | Tuesday; Tue
u      | Day number of week     | Number             | 1
a      | Am/pm marker           | Text               | PM
H      | Hour in day (0-23)     | Number             | 0
k      | Hour in day (1-24)     | Number             | 24
K      | Hour in am/pm (0-11)   | Number             | 0
h      | Hour in am/pm (1-12)   | Number             | 12
m      | Minute in hour         | Number             | 30
s      | Second in minute       | Number             | 55
S      | Millisecond            | Number             | 978
z      | Time zone              | General time zone  | Pacific Standard Time; PST; GMT-08:00
Z      | Time zone              | RFC 822 time zone  | -0800
X      | Time zone              | ISO 8601 time zone | -08; -0800; -08:00


Из этого вы можете получить свой формат и паристь дату так, как вам надо.
Например, я как-то столкнулся с подобным представлением даты при парсинге файлов:
`20160915160122`, т.е
2016 год, 09 - месяц, 15 - дата, 16 - часы, 01 - минуты, 22 - секунды.
Соответственно, по таблице получаем, что шаблон для парсинга(паттерн), должен быть:
`yyyyMMddHHmmss`.

Еще примеры:

Input string                        |    Pattern
------------------------------------|----------------------------------
2016.09.04 AD at 12:08:56 PDT       |    yyyy.MM.dd G 'at' HH:mm:ss z
Wed, Jul 4, '01                     |    EEE, MMM d, ''yy
12:08 PM                            |    h:mm a
12 o'clock PM, Pacific Daylight Time|    hh 'o''clock' a, zzzz
0:08 PM, PDT                        |    K:mm a, z
02001.July.04 AD 12:08 PM           |    yyyyy.MMMM.dd GGG hh:mm aaa
Wed, 4 Jul 2001 12:08:56 -0700      |    EEE, d MMM yyyy HH:mm:ss Z
010704120856-0700                   |    yyMMddHHmmssZ                 



В Java:
//todo example

//todo
DateTimeFormatter
