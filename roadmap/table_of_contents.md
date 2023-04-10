# Путеводитель по заметкам

## Начинающим

Советы для тех, кто только начинает изучать `Java`.

1. [Когда надо и когда не надо использовать `static`](../start/static_java.md)
2. [Как надо и как не надо писать код](../start/code_style.md)
3. [Как оформить класс для хранения константных значений](../start/classes_for_static.md)
4. [Ключевое слово final](../start/final.md)
5. [Война с null](../start/null_war.md)
6. [Optional](../start/optional.md)
7. [Проверки и assert](../start/assertions.md)
8. [Подробно о Enum в Java](../start/enum.md)
9. [Comparable и Comparator](../start/comparable_comparator.md)
10. [Общие советы](../start/common_advices.md)

## Объектно-ориентированное программирование

Заметки про `ООП`, зачем нужно, что включает в себя и как это использовать.

1. [Введение в ООП](../oop/intro.md)
2. [Инкапсуляция](../oop/encapsulation.md)
3. [Наследование](../oop/inheritance.md)
4. [Понятие интерфейса](../oop/interface.md)
5. [Понятие абстрактного класса](../oop/abstract_class.md)
6. [Абстрактные классы и интерфейсы](../oop/abstract_vs_interface.md)
7. [Подробно о this и super в Java](../oop/this_super.md)
8. [SOLID](../oop/SOLID.md)

## java.lang.Object

Говорим о `java.lang.Object` - корне иерархии классов в `Java`.

Все о главном классе в `Java` и его методах.

1. [java.lang.Object](../object/intro.md)
2. [toString](../object/toString.md)
3. [equals](../object/equals.md)
4. [hashcode](../object/hashcode.md)
5. [clone](../object/clone.md)
6. [finalize](../object/finalize.md)
7. [getClass](../object/getClass.md)

## Исключения в Java

Важнейшая тема при работе с ЯП `Java`.

Исключения и все о работе с ними.

1. [Исключения в Java](../exceptions/exceptions.md)
2. [Вопросы для проверки по теме исключений](../exceptions/questions.md)

## Коллекции в Java

Все про коллекции в `Java` и `Generics`.

1. [Введение](../collections/intro.md)
2. [java.util.List](../collections/list/intro.md)
    * [java.util.ArrayList](../collections/list/array_list.md)
    * [java.util.LinkedList](../collections/list/linked_list.md)
3. [java.util.Set](../collections/set/intro.md)
    * [java.util.HashSet](../collections/set/hash_set.md)
    * [java.util.LinkedHashSet](../collections/set/linked_hash_set.md)
    * [java.util.TreeSet](../collections/set/tree_set.md)
4. [java.util.Map](../collections/map/intro.md)
    * [java.util.HashMap](../collections/map/hash_map.md)
    * [java.util.LinkedHashMap](../collections/map/linked_hash_map.md)
    * [java.util.TreeMap](../collections/map/tree_map.md)
5. [Generics](../collections/generics/generics.md)
6. Общие советы.
    * [Что делать, когда вам нужна пустая коллекция](../collections/empty_collections.md)

## Concurrency

Многопоточность в `Java`.

1. [Введение в Concurrency Java](../concurrency/intro.md)

## Сериализация

Сериализация в `Java`. Виды, использование, примеры.

## Загрузка классов

Все про загрузчики, порядок инициализации полей при загрузке и т.д.

1. [Загрузчики классов](../classes/class_loading.md)
2. [Порядок инициализации полей класса](../classes/order_of_loading.md)  

## Системы сборки проекта

  Системы сборки проекта в мире `Java`, структура и использование.

  1. [Работа с ресурсами приложения](../build/resources.md)

## Надо знать или иметь представление

  1. [Ссылки в Java](../common/reference.md)
  2. [Overloading и Overriding](../common/over-load-ride.md) - // todo ПЕРЕПИСАТЬ

## Паттерны

Паттерны в `Java` и их использование.

### Порождающие

* [Builder](../patterns/creational/builder.md)
* [Factory Method](../patterns/creational/factory_method.md)
* [Abstract Factory](../patterns/creational/abstract_factory.md)
* [Singleton](../patterns/creational/singleton.md)

### Поведенческие

### Структурные

## Дата и время в Java

1. [Введение](../other/date/intro.md)
2. [java.util.Date и java.util.Calendar](../other/date/date_and_calendar.md)

    Старое `Time Api`. Входит в состав `JDK`. Из-за большого количества недочетов рекомендуется использовать либо новое `Time Api`, либо сторонние реализации, например, `joda-time`.
3. [Java 8 DateTime API](../other/date/java_8_time_api.md)

    Новое `Time Api`. Входит в в состав `JDK`. Рекомендуется для использования.
4. [joda-time](../other/date/joda_time.md)

    Одна из самых распространенных сторонних библиотек в `Java` для работы с временем.

## Алгоритмы и Структуры данных

Все что касается алгоритмов и их реализации на `Java`, с подробным описанием.

### Алгоритмы поиска

* [Бинарный поиск](../algorithms/search/binary.md)

### Алгоритмы сортировки

* [Сортировка пузырьком](../algorithms/sorting/bubble.md)
* [Сортировка простыми вставками](../algorithms/sorting/insertion.md)

## Многопоточное программирование

* [Потоки и пулы потоков](../concurrency/Concurrency.md)

## Разное

* [Garbage Collector](../other/garbage_collector.md)
* [Кодировки](../other/encoding.md)
* [Логирование](../other/logging.md)
* [Аннотации](../common/annotations.md)

## Задачи к собеседованиям

### Алгоритмы и структуры данных

* [Список задач](../interview/algos/intro.md)

### Java Core

* [Список задач](../interview/core/intro.md)

### SQL

* [Список задач](../interview/sql/intro.md)
