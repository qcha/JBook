# Иерархия исключений

Вопрос с mock-собеседования [Гриша Скобелев, Саша Бармин: Публичное собеседование Senior Software Engineer](https://www.youtube.com/watch?v=ajU9HZP6q8c).

## Условие

Задание на понимание исключений в Java.

Итак, что будет выведено на экран?

```java
public static void main(String[] args) {
    try {
        int i = 1 / 0;
        System.out.println(1);
    } catch (ArithmeticException e) {
        System.out.println(2);
        throw new RuntimeException();
    } catch (Exception e) {
        System.out.println(3);
    } finally {
        System.out.println(4);
    }

    System.out.println(5);
}
```

## Решение

При деление на 0 будет выброшено исключение `ArithmeticException`, поэтому 1 не будет выведена на экран.
Первый более подходящий блок catch словит исключение и напечатает 2.

После этого выкинет `RuntimeException`.

Так как блок `finally` отработает всегда, то 4 будет также напечатано. Но при этом надо помнить, что выше уже было кинуто, то оно вылетит дальше и метод `main` завершится с ним.

Будет напечатаны: 2 и 4.
