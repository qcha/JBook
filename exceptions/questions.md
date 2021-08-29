# Опросник

Здесь собраны вопросы для закрепления материала по теме [исключений](./exceptions.md) и для подготовки к собеседованиям.

## Что надо знать

### Иерархия

Обязательно надо помнить иерархию исключений.

![Exception Hierarchy](../images/exception/exceptions.png)

Также могут попросить привести пару примеров исключений каждого типа.
Необходимо помнить какие из исключений проверяемые, а какие не проверяемые.

### Методы и поля исключения

Помнить, что каждое у каждого исключения есть причина, которая хранится в поле `cause`, получить ее можно через соответствующий метод:

```java
/**
 * Returns the cause of this throwable or {@code null} if the
 * cause is nonexistent or unknown.  (The cause is the throwable that
 * caused this throwable to get thrown.)
 *
 * <p>This implementation returns the cause that was supplied via one of
 * the constructors requiring a {@code Throwable}, or that was set after
 * creation with the {@link #initCause(Throwable)} method.  While it is
 * typically unnecessary to override this method, a subclass can override
 * it to return a cause set by some other means.  This is appropriate for
 * a "legacy chained throwable" that predates the addition of chained
 * exceptions to {@code Throwable}.  Note that it is <i>not</i>
 * necessary to override any of the {@code PrintStackTrace} methods,
 * all of which invoke the {@code getCause} method to determine the
 * cause of a throwable.
 *
 * @return  the cause of this throwable or {@code null} if the
 *          cause is nonexistent or unknown.
 * @since 1.4
 */
public synchronized Throwable getCause() {
    return (cause==this ? null : cause);
}
```

Помимо причины у исключения может быть еще и сообщение, которое хранится в поле `detailMessage`, получить его можно также через соответствующий метод:

```java
/**
 * Returns the detail message string of this throwable.
 *
 * @return  the detail message string of this {@code Throwable} instance
 *          (which may be {@code null}).
 */
public String getMessage() {
    return detailMessage;
}
```

## Вопросы

Общие вопросы на понимание.

---

**Вопрос**:

Скомпилируется ли данный код:

```java
public class ExceptionExample {
    public static void main(String[] args) {
        String line; // (1)
        try {
            line = "hello"; // (2)
        } catch (Exception e) {
            System.err.println(e);
        }

        System.out.println(line); // (3)
    }
}
```

**Ответ**:

Нет.

Java-программист должен проинициализировать переменную, чтобы ею воспользоваться.
Компилятор выдаст ошибку, если это не сделать.

В примере выше программист:

1. Объявляет переменную `line` - (1)
2. Присваивает `line` значение `hello` - (2)
3. Вызывает `System.out.println` с аргументом `line` - (3)

Если в блоке `try` вылетит исключение, то:

1. В блоке `catch` никто значение не присвоит
2. На третьей строке переменная окажется неинициализированной

Поэтому компилятор считает переменную неинициализированной и выдаёт ошибку на строке (3):

```
variable line might not have been initialized
        System.out.println(line); // (3)
```

---

**Вопрос**:

Видите ли вы проблемы в данном коде:

```java
/**
 * Parse date from string to java.util.Date.
 * @param date as string 
 * @return Date object.
 */
public static Date from(String date) {
    try {
        DateFormat format = new SimpleDateFormat("MMMM d, yyyy", Locale.ENGLISH);
        return format.parse(date);
    }  catch (ParseException e) {
        return new Date();
    }
}
```

**Ответ**:

Самая серьезная проблема заключается в том, что так реагировать на исключение **категорически нельзя**.

Потому что происходит подмена результата. Контракт метода говорит о том, что метод преобразовывает дату из строки в объект класса `Date`.
Однако, в ситуации, когда в строке содержится невалидные данные, например, "12:22:aa:cc", исключение будет перехвачено и будет возвращен объект с текущей датой (на данные момент). А это абсолютно не то, что ожидается по контракту метода.

Представьте, что я прошу вас преобразовать "Hello World" в дату, а вы, вместо того, чтобы мне явно сказать о невозможности этой операции, говорите мне текущую дату.

Еще одной проблемой можно назвать создание `SimpleDateFormat` на каждый вызов метода, хотя формат преобразования даты один и тот же. Такие моменты лучше выносить в поля класса.

Третьей проблемой можно назвать то, что явно никак не обрабатывается и не проверяется ситуация с `null`.
Хотя это и не обязательно, но я предпочитаю явно прописывать проверки по `null`.

---

**Вопрос**:

Может ли возникнуть ситуация, при которой блок `finally` **не** выполнится?

**Ответ**:

Да, может!
Блок `finally` не будет выполнен, если, в код включен предшествующий блоку finally системный выход.
Или произойдет завершение работы `JVM`.

```java
try { 
    System.exit(0); 
} catch(Exception e) { 
    System.err.println("Catch"); 
} finally {
    System.err.println("Finally"); 
}
```

---

**Вопрос**:

Какой результат выполнения вызова метода:

```java
public int example() {
    try { 
        throw new IllegalArgumentException();
    } catch(Exception e) { 
        System.err.println("Catch");
        return 14;
    } finally {
        return -1;
    }
}
```

**Ответ**:

Так как блок `finally` выполняется всегда, то результатом будет `-1`.

---

**Вопрос**:

Какой результат выполнения вызова метода:

```java
public int example() {
    try { 
        return 10;
    } finally {
        throw new RuntimeException();
    }
}
```

**Ответ**:

Даже несмотря на `return` из метода, блок `finally` будет выполнен и результатом будет выбрасывание `RuntimeException` исключения.

---

**Вопрос**:

Можем ли мы бросить `java.lang.Throwable` исключение?
А `java.lang.OutOfMemoryError`?

**Ответ**:

Да, как и любое другое исключение.

---

**Вопрос**:

Скомпилируется ли данный код:

```java
try {
    throw new FileNotFoundException("Ooops");
} catch (FileNotFoundException | IOException e) {
    System.err.println("Catch"); 
}
```

**Ответ**:

Нет, данный код не скомпилируется.
Причина в том, что `IOException` является родительским классом для `FileNotFoundException`, а значит уже на этапе компиляции будет понятно, что можно обойтись одним `IOException`, поэтому будет ошибка:

```java
Types in multi-catch must be disjoint: 'java.io.FileNotFoundException' is a subclass of 'java.io.IOException'
```

---

**Вопрос**:

Перепишем пример выше в виде:

```java
try {
    throw new FileNotFoundException("Ooops");
} catch (IOException e) {
    System.err.println("Catch");
} catch (FileNotFoundException e) {
    System.err.println("Catch 2");
}
```

Что теперь?

**Ответ**:

Скомпилировано также не будет, так как блок с обработкой  `FileNotFoundException` недостижим, ведь блок выше, перехватывающий `IOException` (родительский для `FileNotFoundException`), перехватит все `FileNotFoundException`.

---

**Вопрос**:

Как исправить ситуацию выше?

**Ответ**:

Если обработка для `FileNotFoundException` отличается от `IOException`, то поменять `catch`-блоки местами.
Если обработка для обоих исключений одна, то оставить обработку только более общего (в данном случае это`IOException`).

---

**Вопрос**:

Коронный вопрос на всех собеседованиях на `junior`:

В чем отличия между `final`, `finally` и `finalize`?

**Ответ**:

Блок `finally` необходим для гарантированного действия при обработке исключения.

Метод `finalize` был нужен для описания действий при уничтожении объекта сборщиком мусора, но сейчас практически не используется и [вот почему](../object/finalize.md).

Ключевое слово `finally` применяется для того, чтобы явно сказать, что метод/переменная/класс являются законченными и не должны быть переопределены/перезаписаны/участвовать в наследовании.
Об этом [тут](../start/final.md).

---

## Полезные ссылки

1. [Вопросы для интервью по исключениям](https://www.baeldung.com/java-exceptions-interview-questions)
2. [Вопросы для интервью по исключениям 2](https://javaconceptoftheday.com/java-exception-handling-interview-questions-and-answers/)
