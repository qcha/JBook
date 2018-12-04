# java.util.LinkedList

## Введение

Класс `java.util.LinkedList` является второй, после `java.util.ArrayList`, популярной реализацией интерфейса `java.util.List`.

Данная реализация основана на двусвязном списке, каждый элемент содержит ссылку на следующий и предыдущий элементы.

Реализация полностю написана на `Java` и не использует никаких `native` методов.

Позволяет хранить любые значения, в том числе и  `null` значения.

## Реализация

Прежде всего обратим внимание на поля класса и выделим наиболее значимые:

```java
    transient int size = 0;

    /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;

    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

В классе `java.util.LinkedList` объявляется вложенный класс `Node`.

Класс `Node` является оберткой, в которую 'заворачиваются' все добавляемые элементы, он необходим для того, чтобы объявить ссылки на близлежащие элементы списка.

Представьте себе цепь, каждое звено которой сцеплено с предыдущим и следующим звеном. Так вот каждое звено - это и есть объект класса `Node`.

Проиллюстрируем это:

![Двусвязный список](../../images/collections/linked_list_example.png)

Количество элементов в списке хранится в переменной `size`, точно также как и в `java.util.ArrayList`.

Класс `java.util.LinkedList` хранит ссылку на первый и последний элемент списка. Благодрая чему осуществляется быстрая вставка в начало и в конец.

Теперь разберем то, как происходит добавление элементов в список.

## Добавление элемента



