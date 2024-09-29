# Sliding window с подсчетом минимального элемента в окне

## Условие

Дан следующий класс:

```java
/**
 * Класс, используемый для подсчёта наименьшего значения в окне размером w.
 */
public class SlidingWindowMin {

    public SlidingWindowMin(int w) {
        // TODO
    }

    /**
     * Добавляет новое значение в окно. В случае, если размер окна превышает w, 
     * удаляет значение из начала окна (т.е. удаляется самый старый элемент из окна, а value добавляется в окно).
     */
    public void add(int value) {
        // TODO
    }

    /**
     * Возвращает наименьшее значение в окне.
     */
    public int getMin() {
        // TODO
    }

    public static void main(String[] args) {
        SlidingWindowMin window = new SlidingWindowMin(3);
        for (int i = 4; i < 14; i++) {
            int curValue = 34 % i;
            
            window.add(curValue);

            System.out.format("curValue = %2d , minValue = %2d\n", curValue, window.getMin());
        }
    }
}
```

Он описывыает sliding window: при создании задается размер окна через w.

Окно должно обладать следующими свойствами:

* В случае, если окно переполнено и происходит добавление еще одного элемента удаляется самый старый элемент в окне, а новый (текущий) добавляется.
* Можно получить минимальный элемент, который находится сейчас в окне

Необходимо реализовать окно в соответствиями с требованиями.

Для проверки можно запустить `main`, результат должен быть:

```java
curValue =  2 , minValue =  2
curValue =  4 , minValue =  2
curValue =  4 , minValue =  2
curValue =  6 , minValue =  4
curValue =  2 , minValue =  2
curValue =  7 , minValue =  2
curValue =  4 , minValue =  2
curValue =  1 , minValue =  1
curValue = 10 , minValue =  1
curValue =  8 , minValue =  1
```

## Решение

### Перебор

По сути в задаче решается две проблемы: это хранение данных в окне (и удаление более старых при переполнении окна) и поиск минимального элемента в окне.

Добавление и удаление элементов в окне - это очередь элементов, удаление наиболее старого элемента по сути - это удаление с конца.

В `Java` очереди представлены интерфейсом `java.util.Deque` и его реализациями, в качестве реализации возьмем `java.util.ArrayDeque`.

С хранением разобрались, теперь надо понять как искать минимальный элемент.

Прямолинейное решение 'в лоб' - это просто в момент вызова `getMin` перебирать все элементы и возвращать минимальный.

```java
import java.util.ArrayDeque;
import java.util.Deque;

/**
 * Класс, используемый для подсчёта наименьшего значения в окне размером w.
 */
public class SlidingWindowMin {
    private final int windowSize;
    private final Deque<Integer> queue;

    public SlidingWindowMin(int w) {
        if (w <= 0) {
            throw new IllegalStateException("Wrong window size");
        }

        this.windowSize = w;
        this.queue = new ArrayDeque<>();
    }

    /**
     * Добавляет новое значение в окно. В случае, если размер окна превышает w,
     * удаляет значение из начала окна.
     */
    public void add(int value) {
        if (queue.size() == windowSize) {
            queue.removeFirst();
        }

        queue.add(value);
    }

    /**
     * Возвращает наименьшее значение в окне.
     */
    public int getMin() {
        return queue.stream()
                .min(Integer::compareTo)
                .orElseThrow(IllegalStateException::new);
    }

    public static void main(String[] args) {
        SlidingWindowMin window = new SlidingWindowMin(3);
        for (int i = 4; i < 14; i++) {
            int curValue = 34 % i;
            window.add(curValue);

            System.out.format("curValue = %2d , minValue = %2d\n", curValue, window.getMin());

            // curValue =  2 , minValue =  2
            // curValue =  4 , minValue =  2
            // curValue =  4 , minValue =  2
            // curValue =  6 , minValue =  4
            // curValue =  2 , minValue =  2
            // curValue =  7 , minValue =  2
            // curValue =  4 , minValue =  2
            // curValue =  1 , minValue =  1
            // curValue = 10 , minValue =  1
            // curValue =  8 , minValue =  1
        }
    }
}
```

Производительность такого решения:

* Метод `getMin`: O(N)
* Метод `add`: O(1)

### Кеш для хранения минимума

Как избавиться от линейной сложности?

Для быстрого нахождения минимума его необходимо закешировать, просто завести поле на уровне класса `int minimum` не получится: при добавлении в заполненное окно наш минимальный элемент может быть удален, в окне могут быть повторяющиеся элементы.

Поэтому нам нужна структура данных кеша для хранения данных в отсортированном порядке, при этом с быстрым доступом до элемента.

Для этого может подойти `java.util.TreeMap`.

```java
import java.util.Deque;
import java.util.LinkedList;
import java.util.TreeMap;

/**
 * Класс, используемый для подсчёта наименьшего значения в окне размером w.
 */
public class SlidingWindowMin {
    private final int windowSize;
    private final Deque<Integer> queue;
    private final TreeMap<Integer, Integer> cache;

    public SlidingWindowMin(int w) {
        if (w <= 0) {
            throw new IllegalStateException("Wrong window size");
        }

        this.windowSize = w;
        this.queue = new LinkedList<>();
        this.cache = new TreeMap<>();
    }

    /**
     * Добавляет новое значение в окно. В случае, если размер окна превышает w, 
     * удаляет значение из начала окна.
     */
    public void add(int value) {
        int count = cache.getOrDefault(value, 0);
        if (queue.size() == windowSize) {
            queue.removeLast();
            if (count != 0) {
                cache.put(value, count--);
            } else {
                cache.remove(value);
            }
        }

        queue.add(value);

        count += 1;
        cache.put(value, count);
    }

    /**
     * Возвращает наименьшее значение в окне.
     */
    public int getMin() {
        if (queue.size() == 0) {
            throw new IllegalStateException("Empty queue");
        }

        return cache.firstKey();
    }

    public static void main(String[] args) {
        SlidingWindowMin window = new SlidingWindowMin(3);
        for (int i = 4; i < 14; i++) {
            int curValue = 34 % i;
            window.add(curValue);

            System.out.format("curValue = %2d , minValue = %2d\n", curValue, window.getMin());
        }
    }
}
```

Производительность такого решения:

* Метод `getMin`: O(log(N))
* Метод `add`: O(log(N))
