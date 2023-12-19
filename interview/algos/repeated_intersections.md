# Пересечение массивов с учетом повторов элементов

## Условие

Дано два списка целых чисел.
Необходимо вернуть пересечение списков, но при этом учесть количество повторов элементов в списках. Порядок неважен.

### Пример

```java
Input: [4, 5, 6, 7, 0, 1, 2], [4, 4, 4, 6, 0, 2]
Output: [4, 6, 0, 2]

Input: [4, 5, 3, 1, 5, 7], [4, 5, 5, 7]
Output: [4, 5, 5, 7]
```

## Решение

Для быстрого доступа к элементу оба списка переведем в множества. Таким образом доббиваемся `O(1)` на доступ к элементу и убираем дубли (сокращаем количество проходов). Как только нашли элемент, который содержится в обоих списках, посчитаем сколько таких элементов в первом и втором списке, после чего возьмем наименьшее - это и будет то количество пересечений этого элемента в списках.

```java
    static List<Integer> intersection(List<Integer> arr, List<Integer> arr2) {
        List<Integer> result = new ArrayList<>();

        if (arr.isEmpty() || arr2.isEmpty()) {
            return result;
        }

        Set<Integer> first = new HashSet<>(arr);
        Set<Integer> second = new HashSet<>(arr2);

        for (Integer num : first) {
            if (second.contains(num)) {
                long count = Math.min(
                        Collections.frequency(arr, num),
                        Collections.frequency(arr2, num)
                );

                for (int i = 0; i < count; i++) {
                    result.add(num);
                }
            }
        }

        return result;
    }
```
