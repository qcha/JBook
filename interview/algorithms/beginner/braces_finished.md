# Все ли скобки закрыты

## Условие

Дана произвольная строка, в которой могут встречаться разные виды скобок “()[]{}“

Необходимо проверить, что количество открытых скобок равно количеству закрытых.

### Примеры

```text
Input: Hello{World}
Output: True

Input: Hel{World][
Output: False

Input: {}[]))((}
Output: False
```

## Решение

В данной задаче главное не забыть краевые условия, в остальном же просто заведем три переменные, отвечающие за количество открытых скобок каждого вида. При встрече с открытой скобки будем делать инкремент соответсвующей переменной, при нахождении закрытой - декремент.

```java
    public static boolean isBracerFinished(String line) {
        if (line == null) {
            throw new IllegalArgumentException("Input can't be null");
        }

        if (line.isBlank()) {
            return true;
        }

        int openSquareBraces = 0;
        int openCurlyBraces = 0;
        int openParentheses = 0;

        for (char ch : line.toCharArray()) {
            switch (ch) {
                case '{':
                    openCurlyBraces++;
                    break;
                case '}':
                    openCurlyBraces--;
                    break;
                case '[':
                    openSquareBraces++;
                    break;
                case ']':
                    openSquareBraces--;
                    break;
                case '(':
                    openParentheses++;
                    break;
                case ')':
                    openParentheses--;
                    break;
            }
        }

        int result = openSquareBraces + openCurlyBraces + openParentheses;

        return result == 0;
    }
```
