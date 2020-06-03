# Интерфейс java.util.List

- [Интерфейс java.util.List](#интерфейс-javautillist)
  - [Введение](#введение)
    - [Интерфейс java.util.ListIterator](#интерфейс-javautillistiterator)
  - [Реализации java.util.List](#реализации-javautillist)
  - [Выбор реализации](#выбор-реализации)
  - [Полезные ссылки](#полезные-ссылки)

## Введение

Как уже было сказано во [введении](../intro.md), интерфес `java.util.List` расширяет `java.util.Collection`, добавляя туда новые методы.
Как и все интерфейсы в иерархии коллекций, `java.util.List` параметризуется.
Иногда реализации `java.util.List` называют списками или последовательностями.

Объявление `java.util.List` выглядит как:

```java
public interface List<E> extends Collection<E> {
    // ...
}
```

Реализации `java.util.List` являются упорядоченными структурами данных, разрешающими дубликаты. 

Основные методы, которые добавляет интерфейс `java.util.List`:

* `E get(int index)`
* `void add(int index, E element)`
* `int indexOf(Object element)`
* `E remove(int index)`

Интерфейс позволяет извлекать элементы списка по индексу(позиции в списке), добавлять элемент по определенному индексу и узнавать индекс элемента.

Другими словами, реализации `java.util.List` предоставляют пользователю контроль над позицией элемента в списке.

Нумерация индексов, как и у массивов, начинается с `0`.

---

**Вопрос**:

Индекс в сигнатуре метода `E get(int index)` имеет тип `int`.

Как вы думаете: с чем связано это ограничение?
Ведь существует тип `long`, который имеет более широкий диапазон принимаемых значений, однако создатели интерфейса выбрали именно `int`.

**Ответ**:

В `Java` тип `int` занимает 32 бита и максимальное возможное положительное значение - это `2147483647`.

Предположим, что вы создали список такого размера, что вам не хватает типа `int` для индекса.
Давайте навскидку прикинем размер такого списка.

В 32-х битной `JVM` размер `Integer`-а составляет `16 байт`.

При полном заполнении списка этими значениями: `16 * 2147483647 = 34359738352 байт` или `34,359738352 гигабайта`.

Получается, что даже если полностью заполнить список объектами `Integer` памяти может не хватить.

Не говоря уже о том, что итерироваться по такому размеру крайне долго и затруднительно.

Поэтому типа `int` для значений индекса у списка вполне хватает, можно даже сказать, что с запасом. 

Например, максимальный размер `java.util.ArrayList`:

```java
    /**
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```

---

Еще одним интересным моментом является то, что `java.util.List` добавляет специальный итератор по списку.

### Интерфейс java.util.ListIterator

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

## Реализации java.util.List

Иерархия классов выглядит следующим образом:

<img src="../../images/collections/list_hierarchy.png">

Абстрактный класс `java.util.AbstractList` предоставляет заготовку для последующих реализаций.
В нем уже определены некоторые методы, но большинство наиболее значимых, таких как `get` и `add`, кидает исключение `java.lang.UnsupportedOperationException`.
Что говорит о том, что операция не поддерживется и ее надо либо не использовать, либо переопределить метод.

> Сейчас кажется немного странным такое поведение, но класс очень старый, поэтому не стоит удивляться.

Абстрактный класс `java.util.AbstractSequentialList` предоставляет похожую реализацию заготовки, но для списков с последовательным доступом, таким, как `java.util.LinkedList`.

Также из рисунка видно, что в `Java` существует четыре наиболее популярных стандартных реализации `java.util.List`:

* `java.util.ArrayList` - на основе массива
* `java.util.LinkedList` - на основе двусвязного списка
* `java.util.Vector` - также построена на основе массива, но синхронизирована
* `java.util.Stack` - расширение `java.util.Vector`

Реализации `java.util.Vector` и `java.util.Stack` сейчас практически не используются.

Виной тому является то, что `java.util.Vector` синхронизована, из-за чего она является более медленной, чем `java.util.ArrayList` или `java.util.LinkedList`.

Вместо `java.util.Stack` же лучше использовать реализации `java.util.Dequeue`.

Поэтому сосредоточим свое внимание на первых двух:

* Подробнее про [java.util.ArrayList](./array_list.md)
* Подробнее про [java.util.LinkedList](./linked_list.md)

## Выбор реализации

Забегая вперед скажу, что сейчас с `Java 8+` советуют использовать только `java.util.ArrayList`.

Основное техническое отличие `java.util.ArrayList` от `java.util.LinkedList` состоит в том, что `java.util.ArrayList` использует `native` методы для увеличения размера, в то время как `java.util.LinkedList` полностью написана на `Java`.

> Ключевое слово `native` означает, что метод реализован в платформенно-зависимом коде, чаще всего на `C/C++`, и скомпонован в виде динамической библиотеки.
>
> Эта реализация зависит от `JVM`.

> Возможно, вас сейчас это напугало, но на самом деле достаточно просто понимать, что 
> `native` означает лишь то, что вызываемый код, реализован не на `Java`.

Из-за этого реализация `java.util.ArrayList` сейчас более быстродейственна и рекомендуется использовать именно `java.util.ArrayList` при выборе реализации `java.util.List`.

Плюс ко всему, важно ещё понимать что элементы `java.util.ArrayList` хранятся массивом, т.е. линейно в памяти, и можно довольно удачно загрузить кусок массива в кэш процессора, что также благоприятно скажется на [производительности](https://kjellkod.wordpress.com/2012/08/08/java-galore-linkedlist-vs-arraylist-vs-dynamicintarray/).

Также стоит отметить, что `java.util.ArrayList` предоставляет более низкие затраты по хранению объектов, в отличии от `java.util.LinkedList`.

Связано это с тем, что `java.util.LinkedList` хранит объекты в классе-обертке `Node`, который помимо хранимого объекта имеет еще ссылки на следующий и предыдущий элемент:

```java
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

Это накладывает дополнительные расходы на хранение объектов.

Сложность алгоритмов основных операций для реализаций:

| Метод              | java.util.ArrayList | java.util.LinkedList     |
|:------------------:|:-------------------:|:------------------------:|
|      get(index)    |     O(1)            |            O(N)          |
|      add(E)        |     O(N)            |            O(1)          |
|      add(E, index) |     O(N)            |            O(N)          |
|      remove(index) |     O(N)            |            O(N)          |

После ознакомления с реализациями списков в `Java` предлагаю разобрать классическую задачу с собеседований.

---

**Вопрос**:

Перед вами стоит задача: необходимо добавить в список большое(действительно большое) количество объектов.

Какую реализацию вы выберете и почему?

**Ответ**:

В более ранних версиях `Java` действительно имел место выбор между реализациями.

Главный вопрос, после которого можно выбрать реализацию: куда мы добавляем все элементы?

Если добавление идет либо в конец, либо в начало, то разумным выбором будет `java.util.LinkedList`, в то время как при добавлении в середину быстрее будет `java.util.ArrayList`.

Однако, как уже было сказано выше, в настоящий момент предпочтительнее всегда выбирать `java.util.ArrayList`.

---

Еще одной интересной задачей является распространенная проблема, когда вы изменяете список, по которому итерируетесь.

Ее также иногда спрашивают на собеседованиях и знать этот подводный камень необходимо, поэтому разберем ее.

---

**Вопрос**:

Посмотрите на следующий код:

```java
import java.util.ArrayList;
import java.util.List;

public class Test {
    public static void main(String[] args) {
        List<String> lst = new ArrayList<>();
        lst.add("Hello");
        lst.add("Hello2");
        lst.add("Hello3");
        lst.add("Hello4");
        lst.add("World");
        lst.add("World1");
        lst.add("World12");

        for (String line : lst) {
            if (line.equals("World")) {
                lst.remove(line);
            }
        }

        System.out.println(lst);
    }
}
```

Будет ли он работать?

**Ответ**:

Нет! Не будет!

> Это также справедливо и для `java.util.LinkedList`.

При запуске нашего кода выбрасывается исключение:

```java
Exception in thread "main" java.util.ConcurrentModificationException
	at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:909)
	at java.util.ArrayList$Itr.next(ArrayList.java:859)
	at examples.Test.main(Test.java:18)
```

Дело в том, что список хранит переменную `modCount`, которая отвечает за количество раз, которое список был изменен.

В момент, когда создается итератор на список это число присваивается переменной итератора `expectedModCount`.

И если в момент итерирования список изменяется, то `modCount` также изменяется и становится не таким же, как у итератора.

Отсюда и возникает исключение, так как итератор ничего не знает о том, что список был изменен.

Как избежать этой проблемы?

Существует несколько способов:

1. Использование `for`-цикла:
   ```java
    String line;
    for (int i = 0; i < lst.size() - 1; i++) {
        line = lst.get(i);
        if (line.equals("World")) {
            lst.remove(line);
        }
    }
   ``` 
2. Удалять элемент явно с помощью итератора:
    ```java
    Iterator<String> iterator = lst.iterator();

    String line;
    while (iterator.hasNext()) {
        line = iterator.next();
        if (line.equals("World")) {
            iterator.remove();
        }
    }
    ```

## Полезные ссылки

1. [Java Battle Royal: LinkedList vs ArrayList vs DynamicIntArray](https://kjellkod.wordpress.com/2012/08/08/java-galore-linkedlist-vs-arraylist-vs-dynamicintarray/)
2. [Baeldung Java ArrayList vs LinkedList](https://www.baeldung.com/java-arraylist-linkedlist)
3. [Dzone ArrayList vs. LinkedList vs. Vector](https://dzone.com/articles/arraylist-vs-linkedlist-vs)