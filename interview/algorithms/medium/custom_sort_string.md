# Custom Sort String

Задача с [LeetCode](https://leetcode.com/problems/custom-sort-string/description/).

## Условие

Даны две строки: order и input.
Все символы в order являются уникальными и отсортированными в некотором пользовательском порядке ранее.

Переставьте символы input так, чтобы они соответствовали порядку, в котором был отсортирован order. Т.е., если символ x встречается перед символом y в order, то x должен встречаться перед y в переставленной строке.

Верните любую перестановку input, которая удовлетворяет этому свойству.

### Примеры

```text
Input: order = "cba", input = "abcd"

Output: "cbad"

Input: order = "bcafg", input = "abcd"

Output: "bcad"

Input: order = "dacfe", input = "dafcaefw"

Output: "daacffew"
```

## Решение

Так как строка input может содержать повторяющиеся символы, то нам необходимо посчитать количество вхождений каждого символа в input. Для этого заведем cache и проходя по input будем увеличивать счетчик вхождений символа.

После этого нам нужно пройтись по order и каждый символ из order повторить столько раз, сколько он встречается в input, для этого мы обращаемся в cache, где есть эта информация.

Таким образом мы получаем отсортированную как order строку, но без символов, которые есть в input, но не встречаются в order. По услових задачи их можно просто добавить в конец, что мы и сделаем.

```java
    public static String sortByPattern(String order, String input) {
        Map<Character, Integer> cache = new HashMap<>();
        for (Character ch : input.toCharArray()) {
            cache.compute(
                    ch,
                    (key, counter) -> counter == null ? 1 : counter + 1);
        }

        StringBuilder sb = new StringBuilder();
        for (Character ch : order.toCharArray()) {
            Integer count = cache.remove(ch);
            if (count != null) {
                sb.append(
                        String.valueOf(ch).repeat(Math.max(0, count))
                );
            }
        }

        for (Map.Entry<Character, Integer> pair: cache.entrySet()) {
            sb.append(
                    String.valueOf(pair.getKey()).repeat(Math.max(0, pair.getValue()))
            );
        }

        return sb.toString();
    }
```

Время работы: O(N + M), где N - это длина input, а M - это длина order

Память: O(N)
