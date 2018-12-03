# Интерфейс java.util.List

## Введение

Как уже было сказано во [введении](../intro.md), интерфес `java.util.List` расширяет `java.util.Collection`, добавляя туда новые методы.

Как и все интерфейсы в иерархии коллекций, `java.util.List` параметризуется.

Объявление выглядит как:

```java
public interface List<E> extends Collection<E> {
    // ...
}
```

Иногда реализации `java.util.List` называют списками или последовательностями.

Реализации `java.util.List` являются упорядоченными структурами данных, разрешающими дубликаты. 

Основные методы, которые добавляет интерфейс `java.util.List`:

* `E get(int index)`
* `void add(int index, E element)`
* `int indexOf(Object element)`
* `E remove(int index)`

Интерфейс позволяет доставать элементы списка по индексу(позиции в списке), добавлять элемент по определенному индексу и узнавать индекс элемента.

Другими словами, реализации `java.util.List` предоставляют пользователю контроль над позицией элемента в списке.

Нумерация индексов, как и у массивов, начинается с `0`.

### ListIterator

Интерфейс `java.util.ListIterator` предоставляет возможность вставлять и заменять элементы списка, а также производить перебор элементов в двух направлениях.

```java
public interface ListIterator<E> extends Iterator<E> {
    boolean hasNext();
    E next();
    boolean hasPrevious();
    E previous();
    int nextIndex();
    int previousIndex();
    void remove();
    void set(E e);
    void add(E e);
}
```

Это расширенный итератор, который позволяет двигаться еще и в обратном порядке.

## Реализации List

В `Java` существует 4 стандартных реализации `java.util.List`:

* `java.util.ArrayList` - на основе массива
* `java.util.LinkedList` - на основе двусвязного списка
* `java.util.Vector` - также построена на основе массива, но синхронизирована
* `java.util.Stack` - расширение `java.util.Vector`

Реализации `java.util.Vector` и `java.util.Stack` сейчас практически не используются.

Виной тому является то, что `java.util.Vector` синхронизована, из-за чего она является более медленной, чем `java.util.ArrayList` или `java.util.LinkedList`.

Вместо `java.util.Stack` же лучше использовать реализации `java.util.Dequeue`.

Поэтому сосредоточим свое внимание на первых двух:

* Подробнее про [ArrayList](./ArrayList.md)
* Подробнее про [LinkedList](./LinkedList.md)

> Забегая вперед можно скзаать, что сейчас с `Java 8+` советуют использовать только `java.util.ArrayList`.

Еще одним интересным моментом является то, что `java.util.List` добавляет специальный итератор по списку.