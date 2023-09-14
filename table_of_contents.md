# Путеводитель по заметкам

## Начинающим

Советы для тех, кто только начинает изучать `Java`.

1. [Когда надо и когда не надо использовать `static`](jcore/beginner/static_java.md)
2. [Как надо и как не надо писать код](jcore/beginner/code_style.md)
3. [Как оформить класс для хранения константных значений](jcore/beginner/classes_for_static.md)
4. [Ключевое слово final](jcore/beginner/final.md)
5. [Война с null](jcore/beginner/null_war.md)
6. [Optional](jcore/beginner/optional.md)
7. [Проверки и assert](jcore/beginner/assertions.md)
8. [Подробно о Enum в Java](jcore/beginner/enum.md)
9. [Comparable и Comparator](jcore/beginner/comparable_comparator.md)
10. [Общие советы](jcore/beginner/common_advices.md)

## Объектно-ориентированное программирование

Заметки про `ООП`, зачем нужно, что включает в себя и как это использовать.

1. [Введение в ООП](jcore/oop/intro.md)
2. [Инкапсуляция](jcore/oop/encapsulation.md)
3. [Наследование](jcore/oop/inheritance.md)
4. [Понятие интерфейса](jcore/oop/interface.md)
5. [Понятие абстрактного класса](jcore/oop/abstract_class.md)
6. [Абстрактные классы и интерфейсы](jcore/oop/abstract_vs_interface.md)
7. [Подробно о this и super в Java](jcore/oop/this_super.md)
8. [SOLID](jcore/oop/SOLID.md)

## java.lang.Object

Говорим о `java.lang.Object` - корне иерархии классов в `Java`.

Все о главном классе в `Java` и его методах.

1. [java.lang.Object](jcore/object/intro.md)
2. [toString](jcore/object/toString.md)
3. [equals](jcore/object/equals.md)
4. [hashcode](jcore/object/hashcode.md)
5. [clone](jcore/object/clone.md)
6. [finalize](jcore/object/finalize.md)
7. [getClass](jcore/object/getClass.md)

## Исключения в Java

Важнейшая тема при работе с ЯП `Java`.

Исключения и все о работе с ними.

1. [Исключения в Java](jcore/exceptions/exceptions.md)
2. [Вопросы для проверки по теме исключений](jcore/exceptions/questions.md)

## Коллекции в Java

Все про коллекции в `Java` и `Generics`.

1. [Введение](jcore/collections/intro.md)
2. [java.util.List](jcore/collections/list/intro.md)
    * [java.util.ArrayList](jcore/collections/list/array_list.md)
    * [java.util.LinkedList](jcore/collections/list/linked_list.md)
3. [java.util.Set](jcore/collections/set/intro.md)
    * [java.util.HashSet](jcore/collections/set/hash_set.md)
    * [java.util.LinkedHashSet](jcore/collections/set/linked_hash_set.md)
    * [java.util.TreeSet](jcore/collections/set/tree_set.md)
4. [java.util.Map](jcore/collections/map/intro.md)
    * [java.util.HashMap](jcore/collections/map/hash_map.md)
    * [java.util.LinkedHashMap](jcore/collections/map/linked_hash_map.md)
    * [java.util.TreeMap](jcore/collections/map/tree_map.md)
5. [Generics](jcore/collections/generics/generics.md)
6. Общие советы.
    * [Что делать, когда вам нужна пустая коллекция](jcore/collections/empty_collections.md)

## Concurrency

Многопоточность в `Java`.

1. [Введение в Concurrency Java](jcore/concurrency/intro.md)

## Сериализация

Сериализация в `Java`. Виды, использование, примеры.

## Загрузка классов

Все про загрузчики, порядок инициализации полей при загрузке и т.д.

1. [Загрузчики классов](jcore/class_loading.md)
2. [Порядок инициализации полей класса](jcore/beginner/order_of_loading.md)  

## Системы сборки проекта

  Системы сборки проекта в мире `Java`, структура и использование.

  1. [Работа с ресурсами приложения](build/resources.md)

## Надо знать или иметь представление

  1. [Ссылки в Java](jcore/reference.md)
  2. [Overloading и Overriding](jcore/over-load-ride.md) - // todo ПЕРЕПИСАТЬ

## Паттерны

Паттерны в `Java` и их использование.

### Порождающие

* [Builder](patterns/creational/builder.md)
* [Factory Method](patterns/creational/factory_method.md)
* [Abstract Factory](patterns/creational/abstract_factory.md)
* [Singleton](patterns/creational/singleton.md)

### Поведенческие

### Структурные

## Дата и время в Java

1. [Введение](jcore/time/intro.md)
2. [java.util.Date и java.util.Calendar](jcore/time/date_and_calendar.md)

    Старое `Time Api`. Входит в состав `JDK`. Из-за большого количества недочетов рекомендуется использовать либо новое `Time Api`, либо сторонние реализации, например, `joda-time`.
3. [Java 8 DateTime API](jcore/time/java_8_time_api.md)

    Новое `Time Api`. Входит в в состав `JDK`. Рекомендуется для использования.
4. [joda-time](jcore/time/joda_time.md)

    Одна из самых распространенных сторонних библиотек в `Java` для работы с временем.

## Алгоритмы и Структуры данных

Все что касается алгоритмов и их реализации на `Java`, с подробным описанием.

### Алгоритмы поиска

* [Бинарный поиск](algorithms/search/binary.md)

### Алгоритмы сортировки

* [Сортировка пузырьком. Bubble Sort](algorithms/sorting/bubble.md)
* [Сортировка простыми вставками. Insertion Sort](algorithms/sorting/insertion.md)
* [Сортировка слиянием. Merge Sort](algorithms/sorting/insertion.md)

## Многопоточное программирование

* [Потоки и пулы потоков](../jcore/concurrency/Concurrency.md)

## Разное

* [Garbage Collector](jcore/garbage_collector.md)
* [Кодировки](other/encoding.md)
* [Логирование](logging/logging.md)
* [Аннотации](jcore/annotations.md)

## Задачи к собеседованиям

### Алгоритмы и структуры данных

* [Список задач](interview/algos/intro.md)

### Java Core

* [Список задач](interview/core/intro.md)

### SQL

* [Список задач](interview/sql/intro.md)
