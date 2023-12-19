# Количество повторов символов в строке

## Условие

Дана строка, необходимо заменить повторяющиеся подряд символы числом повторений. Если повторов нет, то символ трогать не надо.

### Пример

```java
Input: AAA
Output: A3

Input: AAABB
Output: A3B2

Input: AAABBCDDE
Output: A3B2CD2E
```

## Решение

Заводим переменные, хранящую символ и количество его повторов. Для начала положим туда первый символ из строки.
Если следующий символ тот же, то просто увеличиваем счетчик. Если нет, то записываем символ и количество повторов (если повторов нет, то кладем просто символ). При этом новый символ записываем в нашу переменную и сбрасываем количество повторов.

```java
    static String countLetters(String line) {
        char currentLetter = line.charAt(0);
        int count = 1;

        StringBuilder result = new StringBuilder();
        for (char ch : line.substring(1).toCharArray()) {
            if (currentLetter == ch) {
                count++;
            } else {
                result.append(currentLetter);

                if (count > 1) {
                    result.append(count);
                }

                currentLetter = ch;
                count = 1;
            }
        }

        result.append(currentLetter);

        if (count > 1) {
            result.append(count);
        }

        return result.toString();
    }
```
