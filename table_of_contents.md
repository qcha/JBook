# Путеводитель по JBook

- [JCore](#jcore)
- [Логирование](#логирование)
- [Системы сборки](#системы-сборки)
- [Паттерны](#паттерны)
- [Сериализация](#сериализация)
- [Web и все-все-все](#web-и-все-все-все)
- [Алгоритмы и структуры данных](#алгоритмы-и-структуры-данных)
- [Общая информация и кругозор](#общая-информация-и-кругозор)
- [Подготовка к собеседованиям](#подготовка-к-собеседованиям)

## JCore

### Надо знать новичку

Советы и заметки для тех, кто только начинает работать с `Java`.

0. [Общие советы](jcore/beginner/common_advices.md)
1. [Когда надо и когда не надо использовать `static`](jcore/beginner/static_java.md)
2. [Как надо и как не надо писать код](jcore/beginner/code_style.md)
3. [Как оформить класс для хранения константных значений](jcore/beginner/classes_for_static.md)
4. [Ключевое слово final](jcore/beginner/final.md)
5. [Война с null](jcore/beginner/null_war.md)
6. [Что такое Optional](jcore/beginner/optional.md)
7. [Проверки и assert](jcore/beginner/assertions.md)
8. [Подробно о Enum в Java](jcore/beginner/enum.md)
9. [Comparable и Comparator](jcore/beginner/comparable_comparator.md)

### Объектно-ориентированное программирование

Заметки про `ООП`, зачем нужно, что включает в себя и как это использовать.

#### java.lang.Object

Раздел о `java.lang.Object` - корне иерархии классов в `Java`.

Все о главном классе в `Java` и его методах.

1. [java.lang.Object](jcore/object/intro.md)
2. [toString](jcore/object/to_string.md)
3. [equals](jcore/object/equals.md)
4. [hashcode](jcore/object/hashcode.md)
5. [clone](jcore/object/clone.md)
6. [finalize](jcore/object/finalize.md)
7. [getClass](jcore/object/get_class.md)

#### ООП в Java

1. [Введение в ООП](jcore/oop/intro.md)
2. [Инкапсуляция](jcore/oop/encapsulation.md)
3. [Наследование](jcore/oop/inheritance.md)
4. [Понятие интерфейса](jcore/oop/interface.md)
5. [Понятие абстрактного класса](jcore/oop/abstract_class.md)
6. [Абстрактные классы и интерфейсы](jcore/oop/abstract_vs_interface.md)
7. [Подробно о this и super в Java](jcore/oop/this_super.md)

### Исключения в Java

Важнейшая тема при работе с ЯП `Java`.

Исключения и все о работе с ними.

1. [Исключения в Java](jcore/exceptions/exceptions.md)
2. [Вопросы для проверки по теме исключений](jcore/exceptions/questions.md)

### Коллекции в Java

Все про коллекции в `Java` и `Generics`.

1. [Введение](jcore/collections/intro.md)
2. [java.util.List](jcore/collections/list/intro.md)
    - [java.util.ArrayList](jcore/collections/list/array_list.md)
    - [java.util.LinkedList](jcore/collections/list/linked_list.md)
3. [java.util.Set](jcore/collections/set/intro.md)
    - [java.util.HashSet](jcore/collections/set/hash_set.md)
    - [java.util.LinkedHashSet](jcore/collections/set/linked_hash_set.md)
    - [java.util.TreeSet](jcore/collections/set/tree_set.md)
4. [java.util.Map](jcore/collections/map/intro.md)
    - [java.util.HashMap](jcore/collections/map/hash_map.md)
    - [java.util.LinkedHashMap](jcore/collections/map/linked_hash_map.md)
    - [java.util.TreeMap](jcore/collections/map/tree_map.md)
5. [Generics](jcore/collections/generics/generics.md)

### Java Reflection

- [Аннотации](jcore/reflection/annotations.md)

### GC

- [Garbage Collector](jcore/garbage_collector.md)

### Загрузка классов

Все про загрузчики, порядок инициализации полей при загрузке и т.д.

1. [Загрузчики классов](jcore/class_loading.md)
2. [Порядок инициализации полей класса](jcore/beginner/order_of_loading.md)

### Ссылки в Java

  1. [Ссылки в Java](jcore/reference.md)

### Java Concurrency

Многопоточность в `Java`.

1. [Введение в Concurrency Java](jcore/concurrency/intro.md)

## Логирование

- [Общая информация про логирование в Java](logging/logging.md)

## Системы сборки

  Системы сборки в мире `Java`, структура и использования.

  1. [Работа с ресурсами приложения](build/resources.md)

## Паттерны

### Общие паттерны

- [SOLID](patterns/SOLID.md)

### Архитектурные и дизайн систем

Архитектурные паттерны про дизайн систем.

--

### Архитектура проектов и модулей

Организация кодовой базы на уровне модуля/проекта.

--

### Программирования

Паттерны программирования (GoF и подобные).

#### Порождающие

- [Builder](patterns/programming/creational/builder.md)
- [Factory Method](patterns/programming/creational/factory_method.md)
- [Abstract Factory](patterns/programming/creational/abstract_factory.md)
- [Singleton](patterns/programming/creational/singleton.md)

#### Поведенческие

--

#### Структурные

--

## Сериализация

Сериализация - что это и с чем ее едят в `Java`.

- [Введение](serialization/intro.md)
- [Binary Serialization in Java](serialization/binary/binary.md)

## Web и все-все-все

Все про web-разработку и все что с ней связано.

- [Про HTTP 1.1](web/http/http_11.md)

## Алгоритмы и структуры данных

Все что касается алгоритмов и их реализаций на `Java`.

### Алгоритмы поиска

- [Бинарный поиск](algorithms/search/binary.md)

### Алгоритмы сортировки

- [Сортировка пузырьком. Bubble Sort](algorithms/sorting/bubble.md)
- [Сортировка простыми вставками. Insertion Sort](algorithms/sorting/insertion.md)
- [Сортировка слиянием. Merge Sort](algorithms/sorting/insertion.md)

## Общая информация и кругозор

- [Кодировки](other/encoding.md)

## Подготовка к собеседованиям

- [Секция 'Алгоритмы и структуры данных'](interview/algorithms/intro.md)
- [Секция Core Programming](interview/core/intro.md)
- [Секция SQL Interview](interview/sql/intro.md)
- [Секция Design Interview](interview/design_interview/intro.md)
- [Секция Code Review](interview/code_review/intro.md)
- [Секция Live Coding](interview/live_coding/intro.md)
