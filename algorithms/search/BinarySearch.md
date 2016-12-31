## Бинарный поиск
Одним из основных алгоритмов поиска является бинарный поиск.
Он работает только с отсортированными данными, за счет этого достигается большая
 скорость поиска.

### Алгоритм
Идея алгоритма состоит в том, чтобы найти серединный элемент массива, сраванить
его с искомым элементом, и, в зависимости от того, меньше он искомого или больше,
искать в левой или правой части соответственно. Это продолжается до тех пор, пока
искомый элемент не будет найден, либо пока массив не закончится.

Разумеется, это работает **только с отсортированными массивами**.

### Производительность
Сложность - `O(log(n))`.

### Реализации
```java
public class BinarySearch {
    public static boolean search(int[] arr, int element) {
        if (arr.length == 0) {
            return false;
        }
        return search(arr, element, 0, arr.length - 1);
    }

    private static boolean search(int[] arr, int element, int left, int right) {
        if (left > right) {
            return false;
        }

        final int mid = (left + right) / 2;

        if (arr[mid] == element) {
            return true;
        } else if (arr[mid] > element) {
            return search(arr, element, left, right = mid - 1);
        } else {
            return search(arr, element, mid + 1, right);
        }
    }
}
```
