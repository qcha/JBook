# Бинарный поиск


- [Введение](#введение)
- [Алгоритм](#алгоритм)
- [Реализация](#реализация)
    - [Рекурсивно](#рекурсивно)
    - [Итеративно](#итеративно)
    - [Через Generic](#через-generic)
    - [В JDK](#в-jdk)
- [Производительность](#производительность)
- [Заключение](#заключение)
- [Полезные ссылки](#полезные-ссылки)
- [Отдельная благодарность](#отдельная-благодарность)

## Введение

Бинарный поиск является одним из базовых алгоритмов поиска и часто встречается на собеседованиях.
В отличии от обычного линейного перебора, работает быстрее, но работает только с отсортированными данными.

## Алгоритм

Алгоритм состоит в том, чтобы взять серединный элемент массива, сравнить его с искомым, и, в зависимости от того, меньше он искомого или больше,
искать в левой или правой части соответственно.

Это продолжается до тех пор, пока искомый элемент не будет найден, либо пока массив не закончится.

![Binary Search](../../images/algorithms/search/binary/binary_search.gif)

Таким образом нам понадобится число операций равное тому, сколько раз нам нужно поделить массив размером `N` пополам.
Отсюда и временная сложность алгоритма двоичного поиска: `O(log2(n))`.

Из описания алгоритма понятно, что он работает **только** с отсортированными массивами.

## Реализация

Реализовать алгоритм бинарного поиска можно как итеративно, так и рекурсивно.

### Рекурсивно

```java
public class BinarySearch {
    public static int search(final int[] arr, final int element) {
        if (arr.length == 0) {
            return -1;
        }

        return search(arr, element, 0, arr.length - 1);
    }

    private static int search(final int[] arr, final int element, final int left, final int right) {
        if (left > right) {
            return -1;
        }

        final int mid = left + (right - left) / 2

        if (arr[mid] == element) {
            return mid;
        } else if (arr[mid] > element) {
            return search(arr, element, left, mid - 1);
        } else {
            return search(arr, element, mid + 1, right);
        }
    }
}
```

### Итеративно

```java
public class BinarySearch {
    public static int searchIter(final int[] arr, final int key) {
        if (arr.length == 0) {
            return -1;
        }

        int left = 0;
        int right = arr.length - 1;

        while (left <= right) {
            final int mid = left + (right - left) / 2

            if (arr[mid] == key) {
                return mid;
            } else if (arr[mid] < key) {
                left = mid + 1;
            } else if (arr[mid] > key) {
                right = mid - 1;
            }
        }

        return -1;
    }
}
```

### Через Generic

```java
import java.util.Comparator;
import java.util.List;

public class BinarySearch {
    public static <T extends Comparable<? super T>> int search(List<T> xs, T x) {
        return search(xs, x, T::compareTo);
    }

    public static <T> int search(List<T> arr, T element, Comparator<? super T> comparator) {
        int left = 0;
        int right = arr.size() - 1;

        while (left <= right) {
            int mid = left + (right - left) / 2;

            if (comparator.compare(arr.get(mid), element) == 0) {
                return mid;
            }

            if (comparator.compare(arr.get(mid), element) < 0) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }

        return -1;
    }
}
```

### В JDK

Данная сортировка также представлена в стандартной библиотеке `Java` в `java.util.Collections` через метод `binarySearch`.
Также существует реализация, умеющая работать с массивами, она доступна в `java.util.Arrays` через метод `binarySearch`.

## Производительность

Асимптотическая сложность: `O(log2(n))`.

## Заключение

Алгоритм бинарного поиска используется во многих библиотеках и используется с отсортированными структурами данных, в частности, в `Java` уже есть его реализация, досутпная для массивов в `java.util.Arrays` и в `java.util.Collections` для коллекций данных.

## Полезные ссылки

1. [The curious case of Binary Search — The famous bug that remained undetected for 20 years](https://thebittheories.com/the-curious-case-of-binary-search-the-famous-bug-that-remained-undetected-for-20-years-973e89fc212)
2. [Binary Search Algorithm in Java](https://www.baeldung.com/java-binary-search)
3. [Двоичный поиск](https://ru.wikipedia.org/wiki/%D0%94%D0%B2%D0%BE%D0%B8%D1%87%D0%BD%D1%8B%D0%B9_%D0%BF%D0%BE%D0%B8%D1%81%D0%BA)

## Отдельная благодарность

Отдельную благодарность за ревью и помощь автор хочет выразить:

1. [Alexander Lisianoi](https://github.com/alisianoi)
