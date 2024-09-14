# Является ли число палиндромом

## Условие

Определить: является ли переданное число палиндромом или нет.

[Что такое палиндром](https://ru.wikipedia.org/wiki/%D0%9F%D0%B0%D0%BB%D0%B8%D0%BD%D0%B4%D1%80%D0%BE%D0%BC)

### Примеры

```java
112233

Ответ: false
```

```java
121

Ответ: true

```java
010

Ответ: true
```

## Решение

### Преобразование к строке и перебор

Первое, что приходит в голову, это преобразовать число к строке и с помощью двух указателей (на начало и конец строки) сравнивать символы.

```java
    public static boolean isPalindrome(int num) {
        char[] asString = Integer.toString(num).toCharArray();
        int start = 0;
        int last = asString.length - 1;

        while (start < last) {
            if (asString[start] != asString[last]) {
                return false;
            }

            start++;
            last--;
        }

        return true;
    }
```

### Получение развернутого числа

```java
    public static boolean isPalindrome(int num) {
        if (num < 0) {
            return false;
        }
        
        // 100
        if (num != 0 && num % 10 == 0) {
            return false;
        }

        int reversed = 0;
        while (num > reversed) {
            reversed = reversed * 10 + num % 10;
            num = num / 10;
        }

        return num == reversed || num == reversed / 10;
    }
```

Время работы: O(1)
В `Java` тип `int` ограничен 4 байтами, т.е. длина числа будет конечна (`Integer.MAX_AVLUE` - это 2^32) (в отличии от `Python`), поэтому будет не более чем 32 * log10(2) операций цикла. А значит константное время.

Для `Python`, где число не ограничено типом так, как в `Java`, можно было бы сказать, что сложность была бы log10(num), так как каждой итерации мы сокращаем число в 10 раз и значит потребуется log10(num) операций цикла, чтобы он завершился.

Но тут спрятано то, как мы поделили (в том числе прочитали) число размером n меньше чем за n, поэтому истинная оценка будет O(N), где N - это длина числа (ввода).
