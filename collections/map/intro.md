# Интерфейс java.util.Map

## Введение

Как уже было сказано во [введении](../intro.md), интерфейс `java.util.Map` не связан с `java.util.Collection`, однако формально является коллекцией.

По сути, это коллекция пар 'ключ -> значение', а `java.util.Map` - это интерфейс ассоциативного массива.
Каждому ключу соответствует значение.

Реализации `java.util.Map` также могут называть **мапами** или **хэш-таблицами**, а в `Python` это называется **словарь**.

---

**Вопрос**:

Почему бы тогда не назвать просто `словарь`? Как это сделано в `Python`?

**Ответ**:

На самом деле в `Java` существует абстрактный класс `java.util.Dictionary` еще с версии `1.0`.
Cейчас этот класс является устаревшим, о чем свидетельствует его `java doc`:

> This class is obsolete. New implementations should implement the Map interface, rather than extending this class.

Из-за минусов использования наследования новую иерархию начали строить с интерфейсов.

Интерфейс `java.util.Map` появился в `Java` начиная с версии `1.2`.

---

Название `Map` появилось как сокращение слова `mapping`, что значит отображение/соответствие.

Объявление `java.util.Map` выглядит как:

```java
public interface Map<K, V> {
    // опустим для краткости код
}
```

Здесь `K` - это параметризация ключа, а `V`, соответственно, значения.

Основные методы, которые предоставляет интерфейс `java.util.Map`:

* `V get(Object key)`
* `V put(K key, V value)`
* `V remove(Object key)`
* `Set<K> keySet()`
* `Collection<V> values()`
* `Set<Map.Entry<K, V>> entrySet()`

Это значит, что все реализации интерфейса `java.util.Map` позволяют доставать, добавлять и удалять элементы по ключам, а также предоставлять множество ключей и коллекцию хранимых значений.

Отдельного рассмотрения заслуживает последний метод: `Set<Map.Entry<K, V>> entrySet()`.

## Интерфейс java.util.Map#Entry

Как уже было сказано выше, мы работаем с набором пар ключ-значение.

Интерфейс, описывающий поведение такой пары, называется `Entry` и объявлен внутри `java.util.Map`.

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
    
    // hashCode, equals и т.д.
}
```

Поведение стандартное: можно получить ключ или значение пары, либо установить значение пары.

При этом обратите внимание, что `setValue`, в отличии от канонических `setter-`ов, возвращает старое замененное значение.

Помимо всего прочего предоставляются также компараторы для сравнения пар по ключу и значению.

### Реализации java.util.Map#Entry

У каждого класса, реализующего интерфейс `java.util.Map` своя реализация `java.util.Map#Entry`, но большинство из них основано на стандартной реализации, которая объявлена у `java.util.HashMap`:

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
Ее использование ограничено в рамках пакета, в классе которого она объявлена, т.е. `java.util`.

Стандартная реализация, доступная для использования везде, объявлена у `java.util.AbstractMap` и называется `SimpleEntry`:

```java
 public static class SimpleEntry<K,V>
        implements Entry<K,V>, java.io.Serializable {

        private final K key;
        private V value;

        // some code
}

```

Там же объявлена и неизменяемая (immutable) реализация `SimpleImmutableEntry`.

Еще существуют сторонние реализации `java.util.Map#Entry`, например, в библиотеке `Apache Commons`.

## Реализации java.util.Map

Иерархия классов выглядит следующим образом:

![Class hierarchy](../../images/collections/map/map_hierarchy.png)

Абстрактный класс `java.util.AbstractMap` предоставляет заготовку для последующих реализаций.

В нем уже определены некоторые методы, достаточные для неизменяемой структуры данных, но такие методы как `put` кидают исключение `java.lang.UnsupportedOperationException`. Это значит, что операция не поддерживется и ее надо либо не использовать, либо переопределить метод.

Наиболее известные реализации `java.util.Map`:

* [java.util.HashMap](./hash_map.md) - основана на хэш-таблицах.
* [java.util.LinkedHashMap](./linked_hash_map.md) - расширение предыдущей реализации на основе двусвязных списков.
* [java.util.TreeMap](./tree_map.md) - основана на красно-черном дереве.

Помимо этого, существует также давно устаревшая реализация `java.util.HashTable`, использование которой не рекомендуется.

## Выбор реализации

Когда и какую реализацию выбрать?

Если порядок хранения элементов не важен, то выбор `java.util.HashMap` более чем оправдан.

Данная реализация предоставляет отличную производительность работы с базовыми методами(добавление, поиск, удаление): `O(1)`, но при условии отсутствия коллизий, т.е хорошо определенной хэш-функции добавляемых элементов, в `Java` за это отвечает метод [hashCode](../../object/hashcode.md).

В случае, если порядок добавления элементов важен, то стоит рассмотреть `java.util.LinkedHashMap`. Понятно, что за сохранение порядка надо платить, поэтому данная реализация работает медленнее, чем `java.util.HashMap`.

Минусом может также являться то, что и `java.util.HashMap`, и `java.util.LinkedHashMap` занимают в памяти больше места, чем хранят элементов, в отличии от `java.util.TreeMap`.

Если необходимо, чтобы элементы были отсортированы, то следует присмотреться к `java.util.TreeMap`. Однако в таком случае добавляемые элементы должны либо реализовывать интерфейс `java.lang.Comparable`, либо необходимо написать свой собственный компаратор.

Помните, что `java.util.TreeMap` не поддерживает работу с `null` ключами.

| Структура данных  | Производительность (basic ops)  | Память                 | Отсортированность элементов | Работа с null                                            |
| ------------------|---------------------------------| ---------------------- |---------------------------- |----------------------------------------------------------|
| Treemap           | O(log(N))                       | Без издержек           | В естественном порядке      | Недопустимы null ключи, без ограничений на null значения |
| HashMap           | O(1)                            | С издержками           | Неотсортирован              | Допустим null ключ, без ограничений на null значения     |
| Linked HashMap    | O(1) (но медленнее HashMap)     | Также как и в HashMap  | В порядке добавления        | Допустим null ключ, без ограничений на null значения     |

## Итерирование

Реализации `java.util.Map` имеют встроенные итераторы. Получить можно как список всех ключей `keySet()` или всех значений `values()`, так и все пары ключ-значение `entrySet()`.

```java
// объявление и заполнение мапы
 Map<String, String> map = new HashMap<>();
 map.put("Hello", "World");
 map.put("Hello2", "World2");
 map.put("2", "World3");

for (Map.Entry<String, String> entry: map.entrySet())
    System.out.println(entry.getKey() + " = " + entry.getValue());

for (String key: map.keySet())
    System.out.println(map.get(key));

Iterator<Map.Entry<String, String>> itr = map.entrySet().iterator();
while (itr.hasNext()) {
    System.out.println(itr.next());
}

// итерирование по мапе
 for (Map.Entry<String, String> entry : map.entrySet()) {
         System.out.println(entry);
 }
```

При этом следует помнить, что порядок гарантируеутся не всеми реализациями, как например в примере выше.

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
```
