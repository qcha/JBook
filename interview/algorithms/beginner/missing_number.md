# Сбалансированность скобок

Задача с [LeetCode](https://leetcode.com/problems/missing-number/description/).

## Условие

Дан массив `nums`, содержащий `n` различных чисел в диапазоне `[0, n]`. Найдите единственное число в этом диапазоне, отсутствующее в массиве.

Ограничения:

* 0 <= nums[i] <= n
* 1 <= n <= 104
* n == nums.length
* Числа уникальны

### Примеры

```text
Input: nums = [3,0,1]

Output: 2

Input: nums = [0,1]

Output: 2

Input: nums = [9,6,4,2,3,5,7,0,1]

Output: 8
```

## Решение

Вспомним, что сумма первых натуральных чисел - это n * (n + 1) / 2, а также увидев, что у нас всего n чисел в массиве, то можно найти эту сумму.
Это будет сумма, которая была бы, если бы число не было пропущно.

Теперь из этой суммы вычтем сумму всех чисел из массива - получим искомое число.

```java
    public int missingNumber(int[] nums) {
        if (nums.length == 0) {
            throw new IllegalArgumentException();
        }

        int n = nums.length;

        long expectedSum = (long) n * (n + 1) / 2;

        long actualSum = 0;

        for (int num : nums) {
            actualSum += num;
        }

        return (int)(expectedSum - actualSum);
    }
```
