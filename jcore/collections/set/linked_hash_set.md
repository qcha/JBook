# java.util.LinkedHashSet

## Введение

Класс `java.util.LinkedHashSet` является наследником `java.util.HashSet`, но, в отличии от последнего, основана не на `java.util.HashMap`, а на `java.util.LinkedHashMap`.

Благодаря `java.util.LinkedHashMap` сохраняется порядок добавления элементов.

Реализация может динамически менять размер и позволяет хранить `null` значения.

## Реализация

> Прежде чем приступить необходимо ознакомитья с тем, как реализована [java.util.LinkedHashMap](../map/linked_hash_map.md).

Благодаря тому, что основную работу берет на себя родительский класс, серьезных отличий в реализации `java.util.LinkedHashSet` и `java.util.HashSet` не много.

Для инициализации `java.util.LinkedHashSet` используется конструктор родительского класса `java.util.HashSet`:

```java
    /**
     * Constructs a new, empty linked hash set.  (This package private
     * constructor is only used by LinkedHashSet.) The backing
     * HashMap instance is a LinkedHashMap with the specified initial
     * capacity and the specified load factor.
     *
     * @param      initialCapacity   the initial capacity of the hash map
     * @param      loadFactor        the load factor of the hash map
     * @param      dummy             ignored (distinguishes this
     *             constructor from other int, float constructor.)
     * @throws     IllegalArgumentException if the initial capacity is less
     *             than zero, or if the load factor is nonpositive
     */
    HashSet(int initialCapacity, float loadFactor, boolean dummy) {
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
    }
```

Обратите внимание на модификатор доступа `package` у конструктора.

Как уже было сказано выше, порядок элементов при обходе множества является идентичным порядку добавления элементов.

В остальном все сказанное в адрес `java.util.HashSet` справедливо и для `java.util.LinkedHashSet`.

Также, как и для `java.util.HashSet`, для элементов, хранимых в множестве, важно определить [hashCode](../../object/hashcode.md) и [equals](../../object/equals.md).

## Многопоточность

Все методы `java.util.LinkedHashSet` - не синхронизированы.

Поэтому добавление из различных потоков в такой список **строго** не рекомендуется.

## Производительность

Как и `java.util.HashSet`, реализация `java.util.LinkedHashSet` предоставляет константное время выполнения для добавления элемента: `O(1)`.

Для удаления и проверки на содержание элемента в большинстве случаев будет константное время: `O(1)`.

Но только при условии отсутствия коллизий, т.е хорошо определенной хэш-функции добавляемых элементов, в `Java` за это отвечает метод `hashCode`.

Данная реализация чуть медленнее родительского класса `java.util.HashSet` из-за дополнительных затрат на поддержание двусвязного списка.

Для хранения элементов и их связей также понадобится больше места в памяти.

## Заключение

Реализация `java.util.LinkedHashSet` позволяет быстро добавлять, удалять и проверять на содержание элемента в множестве, хоть и не так быстро, как родительский класс `java.util.HashSet`.

В отличии от `java.util.HashSet` в данной реализации сохраняется порядок добавления элементов и для хранения элементов и их связей понадобится больше места в памяти.

Не является потокобезопасной реализацией множества.