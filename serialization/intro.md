# Serialization

## Введение

Сериализация - это процесс перевода структуры данных (объекта) в формат (например, двоичный или текстовый), который может быть сохранен или передан, чтобы впоследствии однозначно воссоздать передаваемую структуру. Обратный процесс называется, как не трудно догадаться, десериализация.

Зачастую также встречается термин маршаллинг (`marshalling`) - это процесс преобразования структуры данных в последовательность битов. Обратный процесс, соответственно, называется`unmarshalling`.

В основном оперируют терминами сериализация/десериализация, так как зачастую речь идет именно о задачах преобразования в формат для передачи/воссоздания структуры.

Для корректной работы современного приложения ему необходимо уметь обмениваться данными с другими сервисами и компонентами системы, сохранять свое состояние.
И большинство этих сервисов и компонентов выходит далеко за рамки не только нашего приложения, но и вовсе устройства, на котором оно запущено.

Даже для простейшего приложения, например, 'Телефонная книга', рано или поздно понадобится сохранять состояние (контакты, пользователей и т.д.) в том или ином виде, а также уметь это состояние восстановить.

Поэтому без обмена `состоянием` сейчас невозможно представить практически ни одно приложение, а сериализация и дессериализация играет важную роль в этом обмене.

## Сериализация в Java

Здесь мы рассмотрим наиболее популярные виды сериализации, которые встречаются в работе.

### Бинарная

Язык программирования `Java` содержит в стандартной поставке все необходимое для преобразования объекта в последовательность бит.
Для этого существует интерфейс-маркер `java.io.Serializable` и интерфейс `java.io.Externalizable`.

Об этом подробно описано [здесь](binary/binary.md).

### CSV

Расшифровывается как comma-separated values - это текстовый формат, предназначенный для представления табличных данных. Чтобы выделить 'колонки' в таблице используется разделитель `,`:

```csv
Aleksanr,Kuchuk,MIPT
```

По сути `csv` является более подтипом `DSV` (delimiter-separated values). Разница состоит в том, что `CSV` использует запятую  как разделитель, а `DSV` любой разделитель по выбору.

Часто тогда, когда говорят `csv` имеют в виду `dsv` - `delimiter-separated values`, т.е значения разделенные некоторым разделителем, не обязательно точкой с запятой. Большинство библиотек для работы с `csv` поддерживают возможность указать разделитель.

### Xml

Расшифровывается как eXtensible markup language`: расширяемый язык разметки.

```xml
<?xml version="1.0"?>
<greeting>Hello, world!</greeting>
```

Достаточно популярный формат, особенно в крупных банках и старых проектах. Большое количество конфигурационных файлов также до сих пор хранится в этом формате, например, конфигурации контекста в старых проектах на `Spring`-е.

### Json

Расшифровывается как javascript object notation: текстовый формат обмена данными, основанный на `JavaScript`.

```json
{
   "firstName": "Иван",
   "lastName": "Иванов",
   "address": {
       "streetAddress": "Московское ш., 101, кв.101",
       "city": "Ленинград",
       "postalCode": 101101
   },
   "phoneNumbers": [
       "812 123-1234",
       "916 123-4567"
   ]
}
```

В данный момент это один из самых популярных форматов для веб-серверного обмена данными.
