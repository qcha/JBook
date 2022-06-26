# Сортировка простыми вставками

## Введение

Алгоритм сортировки простыми вставками является одним из самых простых алгоритмов как для понимания, так и для реализации.

Это алгоритм, в котором элементы входной последовательности просматриваются по одному, и каждый новый поступивший элемент размещается в подходящее место среди ранее упорядоченных элементов.

## Алгоритм

Идея заключается в том, что сортируемый массив мы условно делим на две части, левую и правую.
Левая часть будет считаться отсортированной, а правая нет.

Соответственно, мы из правой части берем элемент и "переносим" в левую часть, но не просто переносим, а вставляем в подходящее место среди ранее уже упорядоченных элементов.

![Insertion sort](../../images/algorithms/sorting/insertion/insertion_sort.gif)

Отсюда и название: сортировка вставками.

Алгоритм состоит из двух циклов, первый по правой части, по неотсортированной последовательности и второй по левой, где мы ищем нужное место для вставки очередного элемента, который переносим с правой части.

Если массив состоит из одного элемента, то он считается отсортированным.

## Реализация

Для простоты пример будет работать с массивами `int`-ов.

```java
    public static void insertionSort(int arr[]) {
        for (int i = 1; i < arr.length; i++) {
            int current = arr[i];
            int j = i;

            while (j > 0 && arr[j - 1] > current) {
                arr[j] = arr[j - 1];
                j--;
            }

            arr[j] = current; 
        }
    }
```

### В JDK

Данная сортировка реализована в стандартной библиотеке в классе `java.util.DualPivotQuicksort` в методе `insertionSort`:

```java
  /**
     * Sorts the specified range of the array using insertion sort.
     *
     * @param a the array to be sorted
     * @param low the index of the first element, inclusive, to be sorted
     * @param high the index of the last element, exclusive, to be sorted
     */
    private static void insertionSort(int[] a, int low, int high) {
        for (int i, k = low; ++k < high; ) {
            int ai = a[i = k];

            if (ai < a[i - 1]) {
                while (--i >= low && ai < a[i]) {
                    a[i + 1] = a[i];
                }
                a[i + 1] = ai;
            }
        }
    }
```

Где значения `low` - это 0 и `high` - это `a.length`.

## Производительность

Из-за двух вложенных циклов сортировка вставкой медленная, ее временная сложность в наихудшем случае: `О(N^2)`.

Наихудшим случаем является массив, отсортированный в порядке, обратном нужному. Так как каждый новый элемент сравнивается со всеми в отсортированной последовательности.

Однако, для небольших массивов, либо для почти упорядоченных массивов результат будет более чем приемлем: в таком случае сортировка вставкам, скорее всего, обгонит оптимизированный алгоритм какой-нибудь быстрой сортировки.

Этому способствует сама основная идея этого класса — переброска элементов из неотсортированной части массива в отсортированную. При близком расположении близких по величине данных место вставки обычно находится близко к краю отсортированной части, что позволяет вставлять с наименьшими накладными расходами.

Лучшая временная сложность сортировки вставками равна `O(N)`.

## Применение

В отличии от [сортировки пузырьком](./bubble.md), сортировка вставками применяется в реальной жизни и присутствует даже в `JDK`.
Встретить эту сортировку можно в `java.util.Arrays` в методе `sort`:

```java
    /**
     * Sorts the specified array into ascending numerical order.
     *
     * @implNote The sorting algorithm is a Dual-Pivot Quicksort
     * by Vladimir Yaroslavskiy, Jon Bentley, and Joshua Bloch. This algorithm
     * offers O(n log(n)) performance on all data sets, and is typically
     * faster than traditional (one-pivot) Quicksort implementations.
     *
     * @param a the array to be sorted
     */
    public static void sort(int[] a) {
        DualPivotQuicksort.sort(a, 0, 0, a.length);
    }
```

И если посмотреть куда ведёт `DualPivotQuicksort.sort`, то увидим вот такой код:

```java
    /**
     * Max array size to use mixed insertion sort.
     */
    private static final int MAX_MIXED_INSERTION_SORT_SIZE = 65;

    /**
     * Max array size to use insertion sort.
     */
    private static final int MAX_INSERTION_SORT_SIZE = 44;

    /**
     * Sorts the specified range of the array using parallel merge
     * sort and/or Dual-Pivot Quicksort.
     *
     * To balance the faster splitting and parallelism of merge sort
     * with the faster element partitioning of Quicksort, ranges are
     * subdivided in tiers such that, if there is enough parallelism,
     * the four-way parallel merge is started, still ensuring enough
     * parallelism to process the partitions.
     *
     * @param a the array to be sorted
     * @param parallelism the parallelism level
     * @param low the index of the first element, inclusive, to be sorted
     * @param high the index of the last element, exclusive, to be sorted
     */
    static void sort(int[] a, int parallelism, int low, int high) {
        int size = high - low;

        if (parallelism > 1 && size > MIN_PARALLEL_SORT_SIZE) {
            int depth = getDepth(parallelism, size >> 12);
            int[] b = depth == 0 ? null : new int[size];
            new Sorter(null, a, b, low, size, low, depth).invoke();
        } else {
            sort(null, a, 0, low, high);
        }
    }

    static void sort(Sorter sorter, int[] a, int bits, int low, int high) {
        while (true) {
            int end = high - 1, size = high - low;

            /*
             * Run mixed insertion sort on small non-leftmost parts.
             */
            if (size < MAX_MIXED_INSERTION_SORT_SIZE + bits && (bits & 1) > 0) {
                mixedInsertionSort(a, low, high - 3 * ((size >> 5) << 3), high);
                return;
            }

            /*
             * Invoke insertion sort on small leftmost part.
             */
            if (size < MAX_INSERTION_SORT_SIZE) {
                insertionSort(a, low, high);
                return;
            }
            
        // и так далее
     }
    }

    /**
     * Sorts the specified range of the array using insertion sort.
     *
     * @param a the array to be sorted
     * @param low the index of the first element, inclusive, to be sorted
     * @param high the index of the last element, exclusive, to be sorted
     */
    private static void insertionSort(int[] a, int low, int high) {
        for (int i, k = low; ++k < high; ) {
            int ai = a[i = k];

            if (ai < a[i - 1]) {
                while (--i >= low && ai < a[i]) {
                    a[i + 1] = a[i];
                }
                a[i + 1] = ai;
            }
        }
    }
```

Т.е. сортировка вставками будет применена при определенных длинах массива.

## Заключение

Сортировка простыми вставками, несмотря на свою простоту, находит свое применение, например, для почти отсортированных массивов, однако не является наилучшим выбором при больших или сильно перемешанных массивах.

Используется в `Oracle JDK 17` в классе `java.uti.Arrays#sort` для сортировки массивов с длинами менее 44.

## Полезные ссылки

1. [Сортировка вставками](https://ru.wikipedia.org/wiki/%D0%A1%D0%BE%D1%80%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%BA%D0%B0_%D0%B2%D1%81%D1%82%D0%B0%D0%B2%D0%BA%D0%B0%D0%BC%D0%B8)
2. [Java. Сортировка вставками.](https://www.youtube.com/watch?v=jywoZ2XaQoM)
3. [Сортировки вставками](https://habr.com/ru/post/415935/)
