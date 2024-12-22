# Сбалансированность скобок

Задача с [LeetCode](https://leetcode.com/problems/valid-parentheses/description/).

## Условие

Дана строка s, содержащая только символы '(', ')', '{', '}', '[' и ']', определите, является ли входная строка сбалансированной по скобкам.

Входная строка сбалансирована по скобкам, если:

* Открытые скобки должны быть закрыты скобками того же типа.

* Открытые скобки должны быть закрыты в правильном порядке.

* Каждой закрывающейся скобке соответствует открытая скобка того же типа.

### Примеры

```text
Input: s = "()"
Output: true

Input: s = "()[]{}"
Output: true

Input: s = "(]"
Output: false

Input: s = "([])"
Output: true
```

## Решение

### Очередь

Основное правило согласованности состоит в том, что открытые скобки должны быть закрыты в правильном порядке скобками того же типа.
Для обеспечения порядка нам нужна `LIFO` структура данных, в `Java` для этого можно использовать `java.util.ArrayDeque`.

Итак, если мы встречаем открытую скобку, то помещаем ее в стек, если скобка не является открывающей, то достаем из стека то, что находится на вершине и проверяем закрывает ли она ее. Если нет - то сбалансированность нарушена и можно сразу возвращать `false`.

```java
public static boolean isBracerBalanced(String line) {
        if (line == null) {
            throw new IllegalArgumentException("Input can't be null");
        }

        if (line.isBlank()) {
            return true;
        }

        Deque<Character> openBraces = new ArrayDeque<>();

        for (char ch : line.toCharArray()) {
            if (ch == '{' || ch == '[' || ch == '(') {
                openBraces.add(ch);
            } else {
                if (openBraces.isEmpty()) {
                    return false;
                }

                char popped = openBraces.pollLast();

                if (ch == ')') {
                    if (popped != '(') {
                        return false;
                    }
                }

                if (ch == ']') {
                    if (popped != '[') {
                        return false;
                    }
                }

                if (ch == '}') {
                    if (popped != '{') {
                        return false;
                    }
                }
            }
        }

        return openBraces.isEmpty();
    }
```

### От обратного

Можно подойти немного иначе и при встрече с открывающей скобкой добавлять ее закрывающий аналог.

В таком случае, можно избавиться от череды `if`-ов с `popped` проверкой, так как, если встречается не открывающая скобка, то из стека мы достанем либо ее же, либо это будет не сбалансированная строка.

```java
    public static boolean isBracerBalanced(String line) {
        if (line == null) {
            throw new IllegalArgumentException("Input can't be null");
        }

        if (line.isBlank()) {
            return true;
        }

        Deque<Character> stack = new ArrayDeque<>();

        for (char ch : line.toCharArray()) {
            if (ch == '{') {
                stack.add('}');
            } else if (ch == '[') {
                stack.add(']');
            } else if (ch == '(') {
                stack.add(')');
            } else if (stack.isEmpty() || stack.pollLast() != ch) {
                return false;
            }
        }

        return stack.isEmpty();
    }
```
