# Valid Anagram

Задача с [LeetCode](https://leetcode.com/problems/valid-anagram/description/).

## Условие

Даны две строки s и t, вернуть true, если t является анаграммой s, и false в противном случае.

Анаграмма — это слово или фраза, образованные путем перестановки букв другого слова или фразы, при этом все исходные буквы используются ровно один раз.

### Примеры

```text
Example 1:

Input: s = "anagram", t = "nagaram"
Output: true

Example 2:

Input: s = "rat", t = "car"
Output: false
```

## Решение

Везде сразу поставим защиту от 'дурака':

```java
if (s.length() != t.length()) {
    return false;
}
```

Ведь если строки не равны по длине, то это не может быть анограмма.

### Sorting

Самое простое решение - это взять обе строки, получить из них массивы char и отсортировать. Если после этого массивы будут равны - эти строки будут анограммами.

```java
    public boolean isAnagram(String s, String t) {
        if (s.length() != t.length()) {
            return false;
        }

        char[] sChars = s.toCharArray();
        char[] tChars = t.toCharArray();
        
        Arrays.sort(sChars);
        Arrays.sort(tChars);
        
        return Arrays.equals(sChars, tChars);
    }
```

Временная сложность алгоритма будет зависет от сортировки и в `Java` это будет O(N * log(N)).

#### Асимптотика решения

Время работы: O(N * log(N)), где N - длина строки

Память: O(1)

### Кэш

Чтобы улучшить показатель времени работы можно завести кэш. Пройдемся по первой строке и посчитаем количество каждой из букв в ней. Далее пройдемся по второй, но будем количество уже уменьшать. Если в конце получили пустой кэш - значит анограммы.

```java
    public boolean isAnagram(String s, String t) {
        if (s.length() != t.length()) {
            return false;
        }

        Map<Character, Integer> cache = new HashMap<>();
        for (Character ch : s.toCharArray()) {
            cache.compute(ch, (character, integer) -> {
                if (integer == null) {
                    return 1;
                }

                return ++integer;
            });
        }

        for (Character ch : t.toCharArray()) {
            Integer count = cache.computeIfPresent(ch, (character, integer) -> --integer);
            if (count != null && count == 0) {
                cache.remove(ch);
            }
        }

        return cache.isEmpty();
    }
```

#### Асимптотика решения

Время работы: O(N), где N - длина строки

Память: O(N)

### Считаем алфавит алфавита

По идее нам не обязательно хранить в кэше все буквы, ведь мы работаем со словами - а значит с алфавитом.

Заведем массив, отвечающий за то, как часто каждая из букв алфавита встречается в строке. Индекс такого массива и будет самим char-ом, ведь по ASCII у нас A - это 65, B - это 66, ну и так далее. Размера 128 у массива нам хватит на весь алфавит a-zA-Z.

Пройдемся по каждой из строки и будем инкрементить по индексу char-а значение. В конце в массиве должны остаться либо все 0 - тогда это анограммы, либо если нет - вернем false.

```java
    public boolean isAnagram(String s, String t) {
        if (s.length() != t.length()) {
            return false;
        }

        var freq = new int[128];

        for (char c : s.toCharArray()) {
            freq[c]++;
        }

        for (char c : t.toCharArray()) {
            freq[c]--;
        }

        for (int i : freq) {
            if (i != 0) {
                return false;
            }
        }

        return true;
    }
```

#### Асимптотика решения

Время работы: O(N), где N - длина строки

Память: O(128) = O(1) - константа
