# Бинарный поиск

## Введение

Бинарный поиск является одним из базовых алгоритмов поиска и часто встречается на собеседованиях.
В отличии от обычного линейного перебора, работает быстрее, но работает только с отсортированными данными.

## Алгоритм

Алгоритм состоит в том, чтобы взять серединный элемент массива, сравнить его с искомым, и, в зависимости от того, меньше он искомого или больше,
искать в левой или правой части соответственно.

Это продолжается до тех пор, пока искомый элемент не будет найден, либо пока массив не закончится.

![Binary Search](../../images/algorithms/search/binary/binary_search.gif)

Таким образом нам понадобится число операций равное тому, сколько раз нам нужно поделить массив размером `N` пополам.
Отсюда и временная сложность алгоритма двоичного поиска: `O(log(n))`.

Из описания алгоритма понятно, что он работает **только** с отсортированными массивами.

## Реализация

Реализовать алгоритм бинарного поиска можно как итеративно, так и рекурсивно.

Рекурсивно:

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

        final int mid = (left + right) / 2;

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

Итеративно:

```java
public class BinarySearch {
    public static int searchIter(final int[] arr, final int key) {
        if (arr.length == 0) {
            return -1;
        }

        int left = 0;
        int right = arr.length - 1;

        while (left <= right) {
            int mid = (left + right) / 2;

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

## Производительность

Временная сложность: `O(log(n))`.

## Заключение

Алгоритм бинарного поиска используется во многих библиотеках и используется с отсортированными структурами данных, в частности, в `Java` уже есть его реализация, досутпная в `java.util.Arrays` через метод `binarySearch`.
