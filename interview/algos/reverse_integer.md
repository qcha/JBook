# Развернуть число

## Условие

На вход подается число, необходимо развернуть его.

Если число, которое получится после разворачивания, будет больше или меньше максимально допустимых значений (Integer.MAX_VALUE и Integer.MIN_VALUE), то вернуть 0.

### Примеры

```java
1122

Ответ: 2211
```

```java
121

Ответ: 121

```java
900

Ответ: 009
```

## Решение

### Преобразование к строке и перебор

Первое, что приходит в голову, это преобразовать число к строке, и развернуть строку простым копированием.
При этом запомнить, что если число отрицательное - надо добавить знак.

```java
    public static int reverseInt(int num) {
        if (num == Integer.MAX_VALUE || num == Integer.MIN_VALUE) {
            return 0;
        }

        boolean isNegative = num < 0;
        num = Math.abs(num);

        char[] chars = Integer.toString(num).toCharArray();
        char[] result;
        int resultIndex = 0;

        if (isNegative) {
            result = new char[chars.length + 1];
            result[resultIndex++] = '-';
        } else {
            result = new char[chars.length];
        }


        int lastCharIndex = chars.length - 1;
        while (lastCharIndex >= 0) {
            result[resultIndex++] = chars[lastCharIndex--];
        }

        long answer = Long.parseLong(String.valueOf(result));
        if (answer > Integer.MAX_VALUE || answer < Integer.MIN_VALUE) {
            return  0;
        }

        return (int) answer;
    }
```

Решение крайне спорное, сложное и долгое по работе. При этом, выделяется дополнительная память, проверка на переполнение int-а только в конце.

### Деление числа

Вспомним, что если взять остаток от деления на 10 у числа, то получим последнюю цифру в числе.
А разделив число на 10 мы 'обрежем' число на последнюю цифру.

Т.е:

```text
result = 0

123 % 10 = 3
123 / 10 = 12

12 % 10 = 2
12 / 10 = 1

1 % 10 = 1
1 / 10 = 0
```

Таким образом, получаем формулу разворота числа:

```text
result = result * 10 + num % 10

Пример:

result = 0 * 10 + 123 % 10
result = 3 * 10 + 12 % 10
result = 30 * 10 + 1 % 10

result = 321
```

В решении надо предусмотреть проверки на переполнение int-а (как только мы вышли за рамки можно сразу возвращать 0 по условию задачи):

```java
    public static int reverseInt(int num) {
        long result = 0;
        boolean isNegative = num < 0;
        num = Math.abs(num);

        while (num > 0) {
            result = result * 10 + num % 10;
            num = num / 10;

            if (result > Integer.MAX_VALUE || result < Integer.MIN_VALUE) {
                return  0;
            }
        }

        if (isNegative) {
            result = result * (-1);
        }

        return (int) result;
    }
```
