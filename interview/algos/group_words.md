# Группировка слов, состоящих из одних и тех же букв

## Условие

Дан список слов, необходимо вернуть сгруппированный список слов, содержащих один и тот же набор букв.

### Пример

```java
Input: ["eat", "ate", "hello", "llo"]
Output: [["eat", "ate"], ["hello"], ["llo"]]

Input: ["eat", "ate", "eta", "llo"]
Output: [["eat", "ate", "eta"], ["llo"]]
```

## Решение

Необходимо сгруппировать слова, где ключом будет именно последовательность букв слова. Для унифицирования просто будем сортировать набор букв из слова, таким образом получая один и тот же ключ из слов, содержащих одни и те же буквы.

Ну а далее просто вернем списки таких слов.

```java
    static List<List<String>> groupWords(List<String> words) {
        Map<String, List<String>> result = new HashMap<>();

        for (String word : words) {
            char[] rawKey = word.toCharArray();
            Arrays.sort(rawKey);

            String key = String.valueOf(rawKey);

            List<String> store = result.computeIfAbsent(key, k -> new ArrayList<>());
            store.add(word);
        }

        return new ArrayList<>(result.values());
    }
```
