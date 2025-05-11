# Оглавление по разделу 'Java Core'

- [Оглавление по разделу 'Java Core'](#оглавление-по-разделу-java-core)
    - [Надо знать новичку](#надо-знать-новичку)
    - [Объектно-ориентированное программирование](#объектно-ориентированное-программирование)
        - [java.lang.Object](#javalangobject)
        - [ООП в Java](#ооп-в-java)
    - [Исключения в Java](#исключения-в-java)
    - [Коллекции в Java](#коллекции-в-java)
    - [Java Reflection](#java-reflection)
    - [GC](#gc)
    - [Загрузка классов](#загрузка-классов)
    - [Ссылки в Java](#ссылки-в-java)
    - [Java Concurrency](#java-concurrency)

## Надо знать новичку

Советы и заметки для тех, кто только начинает работать с `Java`.

0. [Общие советы](./beginner/common_advices.md)
1. [Как надо и как не надо писать код](./beginner/code_style.md)
2. [Когда надо и когда не надо использовать `static`](./beginner/static_java.md)
3. [Override/Overload в Java](./beginner/over-load-ride.md)
4. [Связывание методов](./beginner/early_late_binding.md)
5. [Ключевое слово final](./beginner/final.md)
6. [Война с null](./beginner/null_war.md)
7. [Что такое Optional](./beginner/optional.md)
8. [Проверки и assert](./beginner/assertions.md)
9. [Подробно о Enum в Java](./beginner/enum.md)
10. [Comparable и Comparator](./beginner/comparable_comparator.md)

## Объектно-ориентированное программирование

Заметки про `ООП`, зачем нужно, что включает в себя и как это использовать.

### java.lang.Object

Раздел о `java.lang.Object` - корне иерархии классов в `Java`.

Все о главном классе в `Java` и его методах.

1. [java.lang.Object](./object/intro.md)
2. [toString](./object/to_string.md)
3. [equals](./object/equals.md)
4. [hashcode](./object/hashcode.md)
5. [clone](./object/clone.md)
6. [finalize](./object/finalize.md)
7. [getClass](./object/get_class.md)

### ООП в Java

1. [Введение в ООП](./oop/intro.md)
2. [Инкапсуляция](./oop/encapsulation.md)
3. [Наследование](./oop/inheritance.md)
4. [Понятие интерфейса](./oop/interface.md)
5. [Понятие абстрактного класса](./oop/abstract_class.md)
6. [Абстрактные классы и интерфейсы](./oop/abstract_vs_interface.md)
7. [Подробно о this и super в Java](./oop/this_super.md)

## Исключения в Java

Важнейшая тема при работе с ЯП `Java`.

Исключения и все о работе с ними.

1. [Исключения в Java](./exceptions/exceptions.md)
2. [Вопросы для проверки по теме исключений](./exceptions/questions.md)

## Коллекции в Java

Все про коллекции в `Java` и `Generics`.

1. [Введение](./collections/intro.md)
2. [java.util.List](./collections/list/intro.md)
    - [java.util.ArrayList](./collections/list/array_list.md)
    - [java.util.LinkedList](./collections/list/linked_list.md)
3. [java.util.Set](./collections/set/intro.md)
    - [java.util.HashSet](./collections/set/hash_set.md)
    - [java.util.LinkedHashSet](./collections/set/linked_hash_set.md)
    - [java.util.TreeSet](./collections/set/tree_set.md)
4. [java.util.Map](./collections/map/intro.md)
    - [java.util.HashMap](./collections/map/hash_map.md)
    - [java.util.LinkedHashMap](./collections/map/linked_hash_map.md)
    - [java.util.TreeMap](./collections/map/tree_map.md)
5. [Generics](./collections/generics/generics.md)

## Java Reflection

- [Аннотации](./reflection/annotations.md)

## GC

- [Garbage Collector](./garbage_collector.md)

## Загрузка классов

Все про загрузчики, порядок инициализации полей при загрузке и т.д.

1. [Загрузчики классов](./class_loading.md)
2. [Порядок инициализации полей класса](./beginner/order_of_loading.md)

## Ссылки в Java

  1. [Ссылки в Java](./reference.md)

## Java Concurrency

Многопоточность в `Java`.

1. [Введение в Concurrency Java](./concurrency/intro.md)
