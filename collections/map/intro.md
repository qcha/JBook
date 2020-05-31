# Интерфейс java.util.Map

## Введение

Как уже было сказано во [введении](../intro.md), интерфес `java.util.Map` не имеет отношения к интерфейсу `java.util.Collection`, однако формально является коллекцией.

По сути, это коллекция `пар` ключ -> значение, а `java.util.Map` это интерфейс ассоциативного массива. Каждому ключу соответствует некоторое значение.
Иногда реализации `java.util.Map` называют 'мапами', хэш-таблицами, в `Python` же это и вовсе `словари`.

Название `Map` появилось как сокращение слова `mapping`, что значит `отображение`, `соответствие`.

В интерфейсе `java.util.Map` параметризуются два типа, это ключ и значение.

Объявление `java.util.Map` выглядит как:

```java
public interface Map<K,V> {
    // ...
}
```

При этом стоит отметить, что не может быть повторяющихся ключей, что следует из названия и смысла `Map`: каждому ключу соответствует значение.

---

**Вопрос**:

Почему бы тогда не назвать просто `словарь`, как в `Python`?

**Ответ**:

На самом деле в `Java` существует абстрактный класс `java.util.Dictionary` еще с версии `1.0`, который сейчасс уже давно не используется.

И до этого иерархия как раз и была основана на классе `java.util.Dictionary`, например, как одна из старых реализаций хэш-таблицы - `java.util.HashTable`.

Класс `java.util.Dictionary` является полностью абтрактным, без какого-либо состояния.

Но благодаря `WORA` в `Java` просто так ничего не меняют и не удаляют, поэтому старые классы остались, а новую иерархию начали строить с интерфейсов.

Интерфейс `java.util.Map` появился в `Java` начиная с версии `1.2`.

---

Основные методы, которые предоставляет интерфейс `java.util.Map`:

* `V get(Object key)`
* `V put(K key, V value)`
* `int indexOf(Object element)`
* `V remove(Object key)`
* `Set<K> keySet()`
* `Collection<V> values()`
* `Set<Map.Entry<K, V>> entrySet()`

Это значит, что все реализации интерфейса `java.util.Map` позволяют доставать, добавлять и удалять элементы по ключам, а также предоставлять множество ключей и коллекцию хранимых значений.

Отдельного рассмотрения заслуживает последний метод: `Set<Map.Entry<K, V>> entrySet()`.

## Интерфейс java.util.Map#Entry

Как уже было сказано выше, `Map` это набор пар ключ-значение.

Так вот интерфейс, описывающий поведение такой пары, называется `Entry`.

Объявление `java.util.Map#Entry` выглядит как:

```java
interface Entry<K,V> {
    /**
     * Returns the key corresponding to this entry.
     *
     * @return the key corresponding to this entry
     * @throws IllegalStateException implementations may, but are not
     *         required to, throw this exception if the entry has been
     *         removed from the backing map.
     */
    K getKey();

    /**
     * Returns the value corresponding to this entry.  If the mapping
     * has been removed from the backing map (by the iterator's
     * <tt>remove</tt> operation), the results of this call are undefined.
     *
     * @return the value corresponding to this entry
     * @throws IllegalStateException implementations may, but are not
     *         required to, throw this exception if the entry has been
     *         removed from the backing map.
     */
    V getValue();

    /**
     * Replaces the value corresponding to this entry with the specified
     * value (optional operation).  (Writes through to the map.)  The
     * behavior of this call is undefined if the mapping has already been
     * removed from the map (by the iterator's <tt>remove</tt> operation).
     *
     * @param value new value to be stored in this entry
     * @return old value corresponding to the entry
     * @throws UnsupportedOperationException if the <tt>put</tt> operation
     *         is not supported by the backing map
     * @throws ClassCastException if the class of the specified value
     *         prevents it from being stored in the backing map
     * @throws NullPointerException if the backing map does not permit
     *         null values, and the specified value is null
     * @throws IllegalArgumentException if some property of this value
     *         prevents it from being stored in the backing map
     * @throws IllegalStateException implementations may, but are not
     *         required to, throw this exception if the entry has been
     *         removed from the backing map.
     */
    V setValue(V value);
    
    // ...
}
```

И предоставляет методы, позволяющие получить/установить ключ и значение.

Единственное, что может быть здесь интересно - это то, что `setValue` в отличии от канонических `setter-`ов возвращает старое замененное значение.

Помимо всего прочего предоставляются также компараторы для сравнения пар по ключу и значению.

### Реализации java.util.Map#Entry

У каждого класса, реализующего интерфейс `java.util.Map` своя реализация `java.util.Map#Entry`, но большинство из них основано на стандартной реализации, которая объявлена у `java.util.HashMap`.

Объявление выглядит следующим образом:

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    // some code
}
```

В этой реализации нет каких-то подводных камней и сложностей, кроме того, что она объявлена с модификатором доступа `package`.
Ее использование ограничено в рамках пакета, в классе которого она объявлена, т.е `java.util`.

Стандартная реализация, доступная для использования везде, объявлена у `java.util.AbstractMap` и называется `SimpleEntry`.

Объявление выглядит следующим образом:

```java
 public static class SimpleEntry<K,V>
        implements Entry<K,V>, java.io.Serializable {

        private final K key;
        private V value;

        // some code
}

```

Там же объявлена и неизменяемая реализация `SimpleImmutableEntry`.
Также существуют сторонние реализации `java.util.Map#Entry`, например, в библиотеке `Apache Commons`.

## Реализации java.util.Map

Иерархия классов выглядит следующим образом:

<img src="../../images/collections/map/map_hierarchy.png">

Абстрактный класс `java.util.AbstractMap` предоставляет заготовку для последующих реализаций.

В нем уже определены некоторые методы, достаточные для неизменяемой структуры данных, но такие методы как `put` кидают исключение `java.lang.UnsupportedOperationException`.

Что говорит о том, что операция не поддерживется и ее надо либо не использовать, либо переопределить метод.

Наиболее известные реализации `java.util.Map`:

* [java.util.HashMap](./hash_map.md) - основана на хэш-таблицах.
* [java.util.LinkedHashMap](./linked_hash_map.md) - расширение предыдущей реализации на основе двусвязных списков.
* [java.util.TreeMap](./tree_map.md) - основана на красно-черном дереве.

Существует также реализация `java.util.HashTable`, но она уже давно не используется, во многом благодаря тому, что большинство методов в ней является `synchronized`, что губительо сказывается на производительности.

## Выбор реализации

Когда и какую реализацию выбрать?

Если порядок хранения элементов не важен, то выбор `java.util.HashMap` более чем оправдан.

Данная реализация предоставляет быстрый доступ до элемента, но при условии отсутствия коллизий, т.е хорошо определенной хэш-функции добавляемых элементов, в `Java` за это отвечает метод [hashCode](../../object/hashcode.md).

В случае, если порядок добавления элементов важен стоит рассмотреть `java.util.LinkedHashMap`. Понятно, что за сохранение порядка надо платить, поэтому данная реализация работает медленнее, чем `java.util.HashMap`.

Если необходимо, чтобы элементы были отсортированы, то следует присмотреться к `java.util.TreeMap`. Однако в таком случае добавляемые элементы должны либо реализовывать интерфейс `java.lang.Comparable`, либо необходимо написать свой собственный компаратор.

## Итерирование

Реализации `java.util.Map` имеют встроенные итераторы. Получить можно как список всех ключей `keySet()` или всех значений `values()`, так и все пары ключ-значение `entrySet()`.

```java
for (Map.Entry<String, String> entry: hashmap.entrySet())
    System.out.println(entry.getKey() + " = " + entry.getValue());

for (String key: hashmap.keySet())
    System.out.println(hashmap.get(key));

Iterator<Map.Entry<String, String>> itr = hashmap.entrySet().iterator();
while (itr.hasNext())
    System.out.println(itr.next());
```

---

**Вопрос**:

Как проитерироваться по `Map`? Ведь `java.util.Map` не является `java.lang.Iterable`.

**Ответ**:

Самым простым способом будет взять множество пар и пройтись по этому множеству.

```java
 // объявление и заполнение мапы
 Map<String, String> map = new HashMap<>();
 map.put("Hello", "World");
 map.put("Hello2", "World2");
 map.put("2", "World3");

// итерирование по мапе
 for (Map.Entry<String, String> entry : map.entrySet()) {
         System.out.println(entry);
 }
```

При этом следует помнить, что порядок гарантируеутся не всеми реализациями.
Например, в примере выше первая напечатанная пара будет `2=World3`.

Другой возможный вариант, это получить множество ключей с помощью `Set<K> keySet()` и доставать элементы по ключам, однако предыдущий вариант мне кажется наиболее предпочтительным.

Стоит помнить, что, как и в случае с итерированием [сипсков](../list/intro.md), если в ходе работы итератора структура данных была изменена (без использования методов итератора), то будет выброшено исключение:

```java
public class ExampleApplication {
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<>();

        map.put("key1", "value1");
        map.put("key13", "value1");
        map.put("key12", "value1");

        System.out.println(map);

        Iterator<Map.Entry<String, String>> itr = map.entrySet().iterator();
        while (itr.hasNext())
        {
            Map.Entry<String, String> next = itr.next();
            System.out.println(next);
            if (next.getKey().equals("key1")) {
                map.remove(itr.next().getKey());
            }
        }
    }
}
```

Результатом будет:

```java
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.HashMap$HashIterator.nextNode(HashMap.java:1442)
	at java.util.HashMap$EntryIterator.next(HashMap.java:1476)
	at java.util.HashMap$EntryIterator.next(HashMap.java:1474)
```

Избежать этого можно удаляя элемент с помощью итератора, с которым происходит работа:

```java
        Iterator<Map.Entry<String, String>> itr = map.entrySet().iterator();
        while (itr.hasNext())
        {
            Map.Entry<String, String> next = itr.next();
            System.out.println(next);
            if (next.getKey().equals("key1")) {
                itr.remove(); // удаление с помощью метода итератора
            }
        }
    }
```
---
