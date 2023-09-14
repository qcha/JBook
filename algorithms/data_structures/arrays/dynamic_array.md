# Динамический массив

## Определение

**Массив** - последовательность элементов одного типа, доступ к которым осуществляется по индексу.

**Динамический массив** - массив, который может менять свой размер в зависимости от количества элементов в нем.

## Зачем

Динамические массивы используются в случае, когда заранее неизвестен необходимый размер памяти. 

Помимо доступа по индексу и изменения произвольного элемента у динамических массив добавляются следующие операции:
* Добавление элемента в произвольную позицию
* Добавление элемента в конец
* Удаление элемента с произвольной позиции
* Удаление элемента с конца
* Найти первый элемент равный переданному
* Удалить первый элемент равный переданному
* Узнать размер массива

Стоит заметить, что амортизированная сложность добавления в конец, удаления с конца и получения размера массива должна быть О(1).

## Реализация 

В языке Java уже есть хорошо оптимизированная реализация динамического массива. Это `ArrayList` из `java.util.ArrayList`. Подробнее про нее можно почитать [здесь](../../../jcore/collections/list/intro.md) и [здесь](../../../jcore/collections/list/array_list.md), но для полного понимания процессов рекомендуется написать свою версию.

В случае с обычными массивами мы работаем с ссылкой на непрерывный кусок памяти, размер которого нельзя поменять после создания. Но это меняется в случае с динамическим массивом - мы храним ссылку на массив, его вместимость (текущий размер массива), а также текущее число элементов.

Если у нас кончится память в текущем массиве (например, при добавлении), то нам необходимо создать новый массив большего размера, скопировать туда элементы из старого массива и удалить старый массив.

Полная реализация:

```java
public class DynamicArray<T> {
    private T[] array;
    private int size = 0;
    private int capacity = 4;

    public DynamicArray() {
        this(4);
    }

    public DynamicArray(int capacity) {
        if (capacity > 4) {
            this.capacity = capacity;
        }

        array = (T[]) new Object[capacity];
    }

    public int size() {
        return size;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    public boolean contains(T value) {
        for (int i = 0; i < size; i++) {
            if (array[i].equals(value)) {
                return true;
            }
        }

        return false;
    }

    public T get(int index) {
        if (index < 0 || index > size - 1) {
            throw new IndexOutOfBoundsException("Invalid index.");
        }

        return array[index];
    }

    public void add(int index, T value) {
        if (index < 0 || index > size - 1) {
            throw new IndexOutOfBoundsException("Invalid insert index.");
        }

        if (size == capacity) {
            reserve(capacity * 2);
        }

        for (int i = size; i > index; i--) {
            array[i] = array[i - 1];
        }

        array[index] = value;
        ++size;
    }

    public T remove(int index) {
        if (index < 0 || index > size - 1) {
            throw new IndexOutOfBoundsException("Invalid remove index.");
        }

        T value = array[index];
        for (int i = index; i < size; ++i) {
            array[i] = array[i + 1];
        }

        --size;
        return value;
    }

    public int indexOf(T value) {
        for (int i = 0; i < size; i++) {
            if (array[i].equals(value)) {
                return i;
            }
        }

        return -1;
    }

    public boolean add(T value) {
        if (size == capacity) {
            reserve(capacity * 2);
        }

        array[size] = value;
        ++size;

        return true;
    }

    public boolean remove(T value) {
        if (value == null) {
            throw new NullPointerException("This list doesn't allow null elements.");
        }

        int index = indexOf(value);
        if (index == -1) {
            return false;
        }

        remove(index);
        return true;
    }

    private void reserve(int newCapacity) {
        if (capacity >= newCapacity) {
            return;
        }

        T[] newArray = (T[]) new Object[newCapacity];
        for (int i = 0; i < size && i < newCapacity; i++) {
            newArray[i] = array[i];
        }

        size = Math.min(size, newCapacity);
        capacity = newCapacity;
        array = newArray;
    }
}
```