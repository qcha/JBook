# Найти число в отсортированном массиве, развернутом относительно какого-то элемента

## Условие

Дан отсортированный массив уникальных чисел развернутый относительно какой-то точки.
Найти индекс (номер) заданного число в массиве или вернуть -1.

### Пример

```java
Input: nums = [4, 5, 6, 7, 0, 1, 2], target = 0
Output: 4

Input: nums = [4, 5, 6, 7, 0, 1, 2], target = 3
Output: -1

Input: nums = [1], target = 0
Output: -1
```

## Решение

Суть решения в том, чтобы сначала найти точку излома, ту самую точку, относительно которой происходит разворот.
Например: `4, 5, 6, 7, 0, 1, 2`, здесь точка излома - это как раз 4-й элемент.

Найдя точку излома наш массив делится по сути на два отсортированных массива, где можно применить [бинарный поиск](../../algorithms/search/binary.md).

```java
    public static int indexOf(int[] nums, int target) {
        if (nums == null || nums.length == 0) {
            return -1;
        }

        // Найдем точку перелома массива
        int rotationIndex = findRotationIndex(nums, 0, nums.length - 1);

        int left = 0;
        int right = nums.length - 1;

        // теперь найдем ту часть массива, в которой должен находиться искомый элемент
        if (target >= nums[rotationIndex] && target <= nums[right]) {
            left = rotationIndex;
        } else {
            right = rotationIndex;
        }

        // Ищем бинарным поиском элемент
        while (left <= right) {
            int middle = left + (right - left) / 2;
            if (nums[middle] == target) {
                return middle;
            } else if (target < nums[middle]) {
                right = middle - 1;
            } else {
                left = middle + 1;
            }
        }

        return -1;
    }

    // Поиск точки излома
    static int findRotationIndex(int[] arr, int start, int end) {
        if (arr[start] < arr[end]) {
            return 0;
        } else {
            if (start + 1 >= end) {
                return end;
            }

            int length = end - start;
            int result = findRotationIndex(arr, start, start + length / 2);

            if (result != 0) {
                return result;
            }

            return findRotationIndex(arr, start + length / 2, end);
        }
    }
```

Поиск точки излома похож на бинарный поиск, где точка выхода из рекурсии будет тогда, когда мы найдем соседний элемент, который нарушает сортированный порядок.

## Полезные ссылки

* [LeetCode Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/)
