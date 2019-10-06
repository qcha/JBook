# java.util.ArrayList

## Введение

Класс `java.util.ArrayList` является одной из самых популярных и часто используемых реализаций интерфейса `java.util.List`.

Данная реализация основана на массиве, но, в отличии от массивов в `Java`, `java.util.ArrayList` может динамически менять размер, а также хранить `null` значения.

## Реализация

Прежде всего обратим внимание на поля класса и выделим наиболее значимые:

```java
    /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
     * will be expanded to DEFAULT_CAPACITY when the first element is added.
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     */
    private int size;
```

Количество элементов в списке хранится в переменной `size`.

Важно понимать, что `size` - это не размер массива, это именно количество элементов.

> Если представить массив как шкаф с ящиками, то размер массива - это количество ящиков,
 а количество элементов массива - это количество занятых ящиков.

Как было сказано выше, в основе реализации лежит массив.

За это отвечает поле `elementData`.

Массив имеет фиксированный размер, а значит при создании списка его надо как-то инициализировать.

Если не указать размер массива при создании объекта `java.util.ArrayList` будет создан массив размером `10`.

Это значение по-умолчанию, которое хранится в константе `DEFAULT_CAPACITY`.

Можно указать размер при создании объекта списка воспользовавшись конструктором и передав туда значение желаемого размера:

```java
public ArrayList(int initialCapacity) {
    // ...
}
```

Более того, если вы знаете, что в список будет добавлено большое количество элементов, лучше сразу задать необходимый `initialCapacity`.

---

**Вопрос**:

Элементы `java.util.ArrayList` хранятся в `elementData`, как уже было отмечено, однако тип массива `java.lang.Object`.

А где же параметризация?

**Ответ**:

Приведение к параметризованному типу происходит при обращении, т.е при попытке достать элемент.

Это связано с тем, как устроена параметризация(`Generic`-и) в `Java`.

// todo generics link
Об этом можно прочитать здесь.

---

Теперь разберем то, как происходит добавление элементов в список и за счет чего достигается динамичность размера.

## Добавление элемента

В `java.util.ArrayList` существует несколько методов добавить элемент, для начала разберем наиболее часто используемый.

### Добавление в конец

За добавление элемента в конец у `java.util.ArrayList` отвечает метод:

```java
    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```

Перед тем, как вставить добавляемый элемент в список происходит проверка: достаточно ли места для вставки? 

За это отвечает метод `ensureCapacityInternal`.

Т.е сначала проверяется хватает ли места для добавления нового элемента и если хватает, то происходит добавление элемента в конец, при этом возвращается `true`, так как список изменяется.

Что происходит в случае, если массив, хранящий элементы, переполнен, т.е для добавления элемента в список нет места?

Ответ прост: список **увеличивается** в размере.

Расширение списка происходит следующим образом:

* Создается новый массив, большего размера
* Вызывается `System.arrayCopy`, который копирует старый массив в новый.
* Добавляемый элемент вставляется в новый, только что созданный и заполненный список.
  
Для наглядности приведем пример:

У вас есть шкаф с 10 ячейками, но со временем этого количества стало не хватать, так как вы купили 11-ю вещь. Вы покупаете новый шкаф, с большим количеством ячеек, перекладываете туда свои старые вещи и 11-ю покупку. Старый же шкаф отдается `GarbageCollector`-у.

Теперь пришла пора разобрать алгоритм увеличения размера списка.

#### Алгоримт увеличения размера списка

Новый массив создается по следующему правилу:

```java
int oldCapacity = elementData.length;
int newCapacity = oldCapacity + (oldCapacity >> 1);
```

Это означает, что размер будет увеличен в **полтора раза**.

Теперь рассмотрим добавлние элемент в середину или начало списка.

### Добавление в середину или начало

За добавление элемента в конкретную ячейку по индексу у `java.util.ArrayList` отвечает метод:

```java
    /**
     * Inserts the specified element at the specified position in this
     * list. Shifts the element currently at that position (if any) and
     * any subsequent elements to the right (adds one to their indices).
     *
     * @param index index at which the specified element is to be inserted
     * @param element element to be inserted
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }
```

Добавление происходит в несколько этапов:

1. Идет проверка на то, что индекс, по которому происходит вставка, не выходит за границы списка:

```java
    /**
     * A version of rangeCheck used by add and addAll.
     */
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```

2. После этого идет стандартная проверка на то, достаточно ли места в массиве для вставки новго элемента, точно та же самая, как и при вставке в конец списка.
При необходимости происходит расширение списка:
   
```java
ensureCapacityInternal(size + 1);
```

3. Далее массив подготоваливается для вставки:

```java
System.arraycopy(elementData, index, elementData, index + 1, size - index);
```

Подготовка происходит следующим образом: все элементы, находящиеся **правее** места вставки, включая само место вставки, сдвигается правее. Достигается это копированием правой части массива.

4. Ну и наконец происходит непосредственно вставка значения по указанному индексу:

```java
elementData[index] = element;
size++;
```

Описанный процесс изображен на рисунке:

![Добавление элемента в список](../../images/collections/array_add_element.png)

Если места для вставки нет, то `System.arrayCopy` будет вызван дважды: первый раз при расширении, т.е создании нового массива, а второй раз уже непосредственно при вставке нового элемента.

## Удаление элемента

Удаление элемента из `java.util.ArrayList` возможно двумя способами:

* `public E remove(int index)` - по индексу
* `public boolean remove(Object o)` - по значению

### Удаление по индексу

За удаление по индексу у `java.util.ArrayList` отвечает метод:

```java
    /**
     * Removes the element at the specified position in this list.
     * Shifts any subsequent elements to the left (subtracts one from their
     * indices).
     *
     * @param index the index of the element to be removed
     * @return the element that was removed from the list
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index, numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```

В начале идет привычная проверка на то, не выходит ли индекс за рамки списка.

После чего идет смещение всех элементов **влево**:

1. Определяется количество элементов, которые необходимо скопировать.

```java
int numMoved = size - index - 1;
```

2. Происходит копирование с помощью уже знакомого `System.arrayCopy`:

```java
System.arraycopy(elementData, index+1, elementData, index, numMoved);
```

3. Благодаря этому получается 'сдвиг' влево, а значит, последний элемент уже не нужен, поэтому его отдаем на 'съедение' `GarbageCollectror`-у.
   
```java
elementData[--size] = null; // clear to let GC do its work
```

Описанный процесс изображен на рисунке:

![Удаление элемента из списка](../../images/collections/remove_element.png)

### Удаление по значению

За удаление по значению у `java.util.ArrayList` отвечает метод:

```java
    /**
     * Removes the first occurrence of the specified element from this list,
     * if it is present.  If the list does not contain the element, it is
     * unchanged.  More formally, removes the element with the lowest index
     * <tt>i</tt> such that
     * <tt>(o==null&nbsp;?&nbsp;get(i)==null&nbsp;:&nbsp;o.equals(get(i)))</tt>
     * (if such an element exists).  Returns <tt>true</tt> if this list
     * contained the specified element (or equivalently, if this list
     * changed as a result of the call).
     *
     * @param o element to be removed from this list, if present
     * @return <tt>true</tt> if this list contained the specified element
     */
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
```

При удалении по значению происходит поиск удаляемого элемента.
Идет перебор массива поэлементно, пока не будет найдет необходимый.

После чего происходит то же самое, что и при удалении по индексу.

При удалении по значению **очень** важно переопределенный метод `equals`, так как удаляемый элемент ищется с его помощью:

```java
o.equals(elementData[index])
```

[Подробнее про equals](../../object/equals.md).

Так как списки могут содержать `null` значения, то за их удаление отвечает первый цикл.

Если в списке присутствуют дубли, то удален будет **первый найденный** элемент!

## Важно знать

Еще одним важным моментом, о котором стоило бы поговорить, является то, что при удалении элементов размер массива `elementData`, хранящего наши элементы, не изменяется. Т.е значение `capacity` остается прежним.

Разберем это на примере:

Создадим список и добавим в него 1000 элементов, после чего удалим 999 из них.

Так вот, несмотря на то, что элементы из списка мы удалили, размер списка, т.е количество доступных ячеек в массиве не изменится.

Если снова обратиться к примеру со шкафом, то представьте, что вы из своего шкафа выбросили часть вещей. Размер шкафа и его вместимость при этом остались теми же, просто часть ячеек не заняты.

И если в реальной жизни в этом нет ничего страшного, то в программировании это может привести к утечкам памяти.

Для того, чтобы держать массив в 'актуальном' состоянии и 'ужать' его размер после удаления большого количества элементов существует метод `trimToSize()`.

```java
    /**
     * Trims the capacity of this <tt>ArrayList</tt> instance to be the
     * list's current size.  An application can use this operation to minimize
     * the storage of an <tt>ArrayList</tt> instance.
     */
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }
```

Данный метод урежет размер массива до количества элементов.

В примере с удалением 999 элементов и вызове `trimToSize()` мы поулучим размер массива равный `1`.

Помните, что если вы заполнили список и вы не планируете, что этот список будет больше расширяться, то можно на таком списке вызвать `trimToSize()`  и ужать неиспользованные 'ячейки'.

Но это не значит, что нужно постоянно вызывать этот метод, особенно, если вы постоянно добавляете и удаляете элементы из списка.

## Многопоточность

Все методы `java.util.ArrayList` - не синхронизированы.

Поэтому добавление из различных потоков в такой список **строго** не рекомендуется.

Если необходима `concurrent`-реализация `java.util.ArrayList`, то стоит воспользоваться `java.util.concurrent.CopyOnWriteArrayList`.

## Производительность

Благодаря тому, что в основе реализации лежит массив, `java.util.ArrayList` предоставляет доступ к элементам `по индексу` за константное время: `O(1)`.

В то время как доступ к элементу `по значению` требует перебор списка до нахождения искомого, что выражается в линейном времени доступа: `O(N)`.

## Заключение

Реализация `java.util.ArrayList` из стандартной библиотеки `Java` предоставляет быстрый доступ к элементам по индексу и линейное время доступа к элементам по значению.

Благодаря использованию `native`-метода `System.arrayCopy` реализация `java.util.ArrayList` является предпочтительным выбором при необходимости использования списка.

> Ключевое слово `native` означает, что метод реализован в платформенно-зависимом коде, чаще всего на `C/C++`, и скомпонован в виде динамической библиотеки.
>
> Эта реализация зависит от `JVM`.

> Возможно, вас сейчас это напугало, но на самом деле достаточно просто понимать, что 
> `native` означает лишь то, что вызываемый код, реализован не на `Java`.

Данная реализация позволяет хранить любые значения, в том числе дубликаты и `null`.

Не синхронизована, поэтому доступ из разных потоков не рекомендуется.

Если количество добавляемых элементов в список велико, то имеет смысл сразу задать должное `capacity`, что увеличит скорость работы и сэкономит ресурсы.

Данная реализация динамически меняет размер при добавлении элементов, но не уменьшает размер внутреннего массива при удалении.

Для того, чтобы контролировать размер массива при удалении элементов стоит воспользоваться методом `trimToSize()`.