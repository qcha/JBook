# Рисуем букву Z

## Условие

Напишите функцию, которая в консоли рисует квадратную букву `Z` заданного размера из зацикленной последовательности цифр 0-9:

### Пример

Input:

```java
print_z(5)
```

Output:

```text
01234
   5
  6
 7
89012
```

Высота и ширина буквы равна 5.

## Решение

В начале нам необходимо подумать о том, как мы будем получать числа для 'рисования'. Так как необходимо следить за 'переполнением' при инкриментах, то для этого заведем отдельный класс:

```java
class DigitCounter {
    private int currentDigit;

    DigitCounter() {
        this.currentDigit = 0;
    }

    public int getAndIncrement() {
        if (currentDigit > 9) {
            currentDigit = 0;
        }

        return currentDigit++;
    }
}
```

При объявлении метода `printZ` обработаем краевой случай: z можно нарисовать только, если введенное число больше или равно 3.

Аккумулятором значений будет выступать `StringBuilder`.

Сам же алгоритм строится на том, что первая и последняя строки всегда будут заполняться полностью, а между ними строка будет формироваться из пробелов, кроме места `number - y`, где `number` - это введенное число, ширина и длина 'полотна', а `y` - это ось, которая начинается от левого верхнего угла и направлена вниз.

Грубо говоря, наше 'полотно' для рисования будет работать со следующими осями:

```text
   x0 x1 x2 ... xN
y0
y1
y2
...
yN
```

Соответственно, для number = 4:

```text
   x0 x1 x2 x3
y0 0  1  2  3 
y1       4   
y2    5
y3 6  7  8  9
```

Конечный вариант в коде:

```java
public class Main {
    public static void main(String[] args) {
        printZ(5);
    }

    public static void printZ(int number) {
        if (number < 3) {
            throw new IllegalArgumentException("Can't draw z with depth less than 3");
        }

        DigitCounter digitCounter = new DigitCounter();
        StringBuilder sb = new StringBuilder();
        for (int y = 0; y < number; y++) {
            for (int i = 0; i < number; i++) {
                if (y == 0 || y == number - 1) {
                    sb.append(digitCounter.getAndIncrement());
                } else {
                    if (i == number - y - 1) {
                        sb.append(digitCounter.getAndIncrement());
                    } else {
                        sb.append(" ");
                    }
                }
            }

            sb.append("\n");
        }

        System.out.println(sb);
    }
}

class DigitCounter {
    private int currentDigit;

    DigitCounter() {
        this.currentDigit = 0;
    }

    public int getAndIncrement() {
        if (currentDigit > 9) {
            currentDigit = 0;
        }

        return currentDigit++;
    }
}
```
