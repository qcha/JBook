# Assertions

Во время написания кода зачастую необходимо осуществлять валидацию значений параметров.

И о том, какие инструменты предоставляет для этого `Java` пойдёт речь!

- [Assertions](#assertions)
  - [Проблема](#проблема)
  - [Проверки](#проверки)
    - [Assert](#assert)
      - [Механизм `assert` отключён по умолчанию](#механизм-assert-отключён-по-умолчанию)
      - [Тип исключения](#тип-исключения)
    - [Explicit checks](#explicit-checks)
      - [Guava](#guava)
      - [Apache Commons](#apache-commons)
  - [Советы по использованию](#советы-по-использованию)
  - [Заключение](#заключение)
  - [Полезные ссылки](#полезные-ссылки)
  - [Отдельная благодарность](#отдельная-благодарность)

## Проблема

Далеко ходить не будем и попросим подняться на эту сцену нашего `Person`-а:

```java
public class Person {
    private int age;
    private String userName;
    private String email;

    public void setAge(int age) {
        this.age = age;
    }

    // getters and setters
}
```

Допустим, совершенно случайно, мы ответственно относимся к данным и всё ещё живём в мире, где отрицательный возраст невозможен.
Но `int` в `Java`, как мы помним, может принимать значения от `-2147483648` до `2147483647`. В `Java` нет возможности объявить беззнаковое число.

Что делать?

![sherlok](../images/start/assert/sherlok.jpg)

Знатоки берут минуту на размышление, но мы с вами ответим быстрее: необходимо **проверять** получаемые значения прежде чем работать с ними.

Такие проверки, обеспечивающие консистентность данных в методах/классах называются `preconditions`.

> In computer programming, a precondition is a condition or predicate that must always be true just prior to the execution of some section of code or before an operation in a formal specification.
>
> If a precondition is violated, the effect of the section of code becomes undefined and thus may or may not carry out its intended work.

Существуют также и `postconditions`:

> In computer programming, a postcondition is a condition or predicate that must always be true just after the execution of some section of code or after an operation in a formal specification.
>
> Postconditions are sometimes tested using assertions within the code itself. Often, postconditions are simply included in the documentation of the affected section of code.

Более лаконичное определение:

> The precondition statement indicates what
must be true before the function is called.
>
> The postcondition statement indicates what
will be true when the function finishes its
work.

Теперь, когда мы пришли к тому, что проверки нужны, ответим на вопрос: как проверять?

## Проверки

### Assert

В `Java`, начиная с версии `1.4`, существует ключевое слово `assert`, являющиеся унарным оператором.
Данный оператор проверяет истинность выражения и, при не выполнении условия, бросает исключение `java.lang.AssertionError`.

```java
public void connect(Connection connection) {
    assert connection != null;

    // some code
}
```

Соответственно, если будет передан `null`, то будет выброшено исключение.

Разумеется, `assert` можно применять не только для проверки на `null`, а вообще для любого логического выражения:

```java
public void setAge(int age) {
    assert age >= 0 && age < 150;
    this.age = age;
}
```

Также, `assert` можно использовать с сообщением описания проблемы:

```java
public Person(final int age, final String userName) {
    assert age >= 0 : "Age can't be less then zero";
    this.age = age;
    this.userName = Objects.requireNonNull(userName, "userName can't be null");
}
```

Соответственно при вызове конструктора `Person` с отрицательным значением в графе возраст приведёт к выбрасыванию исключения:

```text
Exception in thread "main" java.lang.AssertionError: Age can't be less then zero
...
```

Таким образом, достигается гарантия, что не валидные данные присвоить будет нельзя.

Однако, я **не** рекомендую использовать `assert`-ы для `preconditions` или `postconditions` в своей работе.

И вот почему:

#### Механизм `assert` отключён по умолчанию

По умолчанию `assert`-ы в `Java` отключены. Это значит, что пока вы не включите их специальным флагом при запуске приложения - они будут игнорироваться.

Включается это флагом `-ea` или `-enableassertions`. Также можно указывать конкретные классы/пакеты для которых необходимо включить механизм проверок. Есть антипод: флаг `-da` или `-disableassertions`, эти флаги можно использовать друг с другом для гибкой настройки.

Теоретически, возможно было бы удобно `assert`-ы оставлять включенными во время разработки и тестирования кода, но отключать в релиз-версии. Однако, как по мне, лучше, если проверки работают всегда.

Вообще, до версии `Java 1.4` слово `assert` не являлось ключевым и могло быть использовано для наименования переменных, методов и так далее.

То, что `assert`-ы отключены по умолчанию для меня является минусом.

Необходимость постоянно следить за тем включил ты их или нет, как минимум неприятна. По сути, при использовании `assert`-ов, один флаг отделяет вас от того будут ли проверки консистентности данных или нет.
Как по мне - слишком опасно и лишние проблемы на пустом месте.

#### Тип исключения

Как уже было сказано ранее, `assert`, при невыполнении условия, бросает `java.lang.AssertionError`:

```java
/**
 * Thrown to indicate that an assertion has failed.
 *
 * <p>The seven one-argument public constructors provided by this
 * class ensure that the assertion error returned by the invocation:
 * <pre>
 *     new AssertionError(<i>expression</i>)
 * </pre>
 * has as its detail message the <i>string conversion</i> of
 * <i>expression</i> (as defined in section 15.18.1.1 of
 * <cite>The Java&trade; Language Specification</cite>),
 * regardless of the type of <i>expression</i>.
 *
 * @since   1.4
 */
public class AssertionError extends Error {
    // ...
}
```

И, как видите, это наследник `Error`!

Вспоминаем гигантскую картинку из раздела про [исключения](../exceptions/exceptions.md):

![exceptions](../images/exception/exceptions.png)

И представляем иерархию. Понятно, что обработка исключений при использовании `assert` усложняется в разы.
Просто потому, что отреагировать на такое специфическое исключение - сложно.

Из этого следует, что использовать `assert`-ы в проверках (например, на `null` или валидацию входных значений в метод) - плохая задумка, потому что:

* Если передали неправильные аргументы, то существует `java.lang.IllegalArgumentException`:
    ```java
    /**
     * Thrown to indicate that a method has been passed an illegal or
     * inappropriate argument.
     *
     * @author  unascribed
     * @since   JDK1.0
     */
    public class IllegalArgumentException extends RuntimeException {
        // ...
    }
    ```
* Если метод вызван не вовремя, например у `Iterator`'а вызвали метод `remove` без вызова `next`, то JDK на этот случай предлагает `java.lang.IllegalStateException`:
    ```java
    /**
    * Signals that a method has been invoked at an illegal or
    * inappropriate time.  In other words, the Java environment or
    * Java application is not in an appropriate state for the requested
    * operation.
    *
    * @author  Jonni Kanerva
    * @since   1.1
    */
    public class IllegalStateException extends RuntimeException {
    ```

Это ещё серьёзный камень в огород `assert`-ов.

Получается, что случайно не включить `assert`-ы и лишиться **всех** проверок просто, перехватывать исключения и восстанавливать поток выполнения программы (в случае когда это возможно) после непрохождения проверок сложно.

Исходя из этого, я не использую `assert`-ы в своём коде с `conditions`-проверками. Они **не** должны использоваться как `pre/postconditions` к публичным `api`.

---

**Вопрос**:

Так где же всё-таки можно использовать `assert`-ы?

**Ответ**:

Исходя из всего вышесказанного `assert`-ы крайне редко встречаются, однако бывают ситуации, когда ими удобно воспользоваться.
Во-первых, `assert` может пригодиться там, где вы хотите проверить инвариант, а условие проверки тяжёлое. Либо сообщение об ошибке не ленивое и тяжёлое. И каждый раз проверять это - нагрузка, терять в производительности здесь не хочется, поэтому имеет смысл использовать `assert` и включить только в тестах.

Также имеет смысл использовать `assert` для внутренних инвариантов алгоритма, например так делается в `JDK`:

![assert_example](../images/start/assert/assert_example.jpeg)

Польза в том, что если найти тестовые данные, на которых инварианты нарушаются, то при запуске тестов мы сразу найдём проблемное место. А разбить всё на минимальные методы, чтобы каждый из них покрыть отдельным юнит-тестом бывает иногда плохо: можно потерять и читаемость, и скорость.

---

### Explicit checks

При работе с `pre/postconditions` я отдаю предпочтение **явным** проверкам:

```java
public void setAge(int age) {
    if (age < 0 || age > 150) {
        throw new IllegalArgumentException("Age can't be less then 0 or higher 150");
    }

    this.age = age;
}
```

Для меня плюсы подобного подхода:

1. Более явно и нельзя по ошибке отключить/не включить/указать не тот пакет и т.д.
2. Я сам выбираю тип исключения, что позволяет мне лучше контролировать поток выполнения программы: проще писать обработчики ошибок, так как `Exception` и потомков легче перехватывать, в отличие от `Error`.
3. Можно реагировать не только бросанием исключения, а, например, присваиванием `default` значения:

    ```java
    // ...
    public static int DEFAULT_TIMEOUT = 0;
    
    // ...
    public void runWithTimeout(int timeout) {
        if (timeout < 0) {
            run(DEFAULT_TIMEOUT);
        }
    }
    ```

Также, в `Java 7+` существует вспомогательный класс `java.util.Objects`, который содержит вспомогательные методы проверки на `null`:

```java
    /**
     * Checks that the specified object reference is not {@code null}. This
     * method is designed primarily for doing parameter validation in methods
     * and constructors, as demonstrated below:
     * <blockquote><pre>
     * public Foo(Bar bar) {
     *     this.bar = Objects.requireNonNull(bar);
     * }
     * </pre></blockquote>
     *
     * @param obj the object reference to check for nullity
     * @param <T> the type of the reference
     * @return {@code obj} if not {@code null}
     * @throws NullPointerException if {@code obj} is {@code null}
     */
    public static <T> T requireNonNull(T obj) {
        if (obj == null)
            throw new NullPointerException();
        return obj;
    }

    /**
     * Checks that the specified object reference is not {@code null} and
     * throws a customized {@link NullPointerException} if it is. This method
     * is designed primarily for doing parameter validation in methods and
     * constructors with multiple parameters, as demonstrated below:
     * <blockquote><pre>
     * public Foo(Bar bar, Baz baz) {
     *     this.bar = Objects.requireNonNull(bar, "bar must not be null");
     *     this.baz = Objects.requireNonNull(baz, "baz must not be null");
     * }
     * </pre></blockquote>
     *
     * @param obj     the object reference to check for nullity
     * @param message detail message to be used in the event that a {@code
     *                NullPointerException} is thrown
     * @param <T> the type of the reference
     * @return {@code obj} if not {@code null}
     * @throws NullPointerException if {@code obj} is {@code null}
     */
    public static <T> T requireNonNull(T obj, String message) {
        if (obj == null)
            throw new NullPointerException(message);
        return obj;
    }
```

#### Guava

Многословность явных проверок зачастую утомляет и иногда делает код более тяжелым к пониманию/прочтению/написанию, было бы наивно ожидать, что не будет написано что-то вспомогательное. Поэтому `preconditions` является частью [Google Guava](https://mvnrepository.com/artifact/com.google.guava/guava).

Поэтому код с явной проверкой:

```java
public static double sqrt(double value) {
    if (value < 0) {
        throw new IllegalArgumentException("input is negative: " + value);
    }
    // calculate square root
}
```

Может быть переписан более компактно, при использовании `Guava`:

```java
public static double sqrt(double value) {
    Precondtions.checkArgument(value >= 0, "input is negative: %s", value);
    // calculate square root
}
```

Данная библиотека содержит наиболее часто встречаемые проверки, например:

1. `checkArgument` - проверка аргумента на выполнение условия. При невыполнении бросается `java.lang.IllegalArgumentException`.
2. `checkElementIndex` - проверка на валидность индекса массива/списка/строки. В противном случае бросается `java.lang.IndexOutOfBoundsException`.
3. `checkNotNull` - проверка на наш любимый `null`. В противном случае бросается `java.lang.NullPointerException`.
4. `checkState` - проверка состояния. Если аргумент - `false` кинет `java.lang.IllegalStateException`.

`Precondtions.checkNotNull` является аналогом `Objects.requireNonNull` из стандартной библиотеки.

Использовать `Preconditions` из `Guava` или нет зачастую зависит от команды и проекта, кому-то кажется, что это делает код более лаконичным, с другой стороны тащить только ради этого целую библиотеку в проект спорно (если до этого у вас не было `Guava` в проекте).

Лично мне кажется более читабельным вариант с явными проверками, которые я пишу сам.

#### Apache Commons

Помимо `Google Guava` есть ещё [apache-commons-lang3](https://mvnrepository.com/artifact/org.apache.commons/commons-lang3), также содержащий утилитные классы для `preconditions`:

Пример выше с использованием `apache-commons-lang3`:

```java
public static double sqrt(double value) {
    Validate.isTrue(value >= 0, "input is negative: %s", value);
    // calculate square root
}
```

Данная библиотека также содержит наиболее часто встречаемые проверки, например:

1. `isTrue` - проверка аргумента на выполнение условия. При невыполнении бросается `java.lang.IllegalArgumentException`.
2. `validIndex` - проверка на валидность индекса массива/списка/строки. В противном случае бросается `java.lang.IndexOutOfBoundsException`.
3. `notNull` - проверка на наш любимый `null`. В противном случае бросается `java.lang.NullPointerException`.
4. `validState` - проверка состояния. Кинет `java.lang.IllegalStateException`, если первый аргумент - `false`.

И так далее.

В обеих библиотеках содержится большое количество вспомогательных методов для работы со строками, `null`, проверок аргументов и т.д.

Выбор библиотеки скорее дело вкуса.

## Советы по использованию

И тут, на самом деле, возникает главный вопрос: где проверять?

Разумеется, всё зависит от задачи. Важно понимать, что надо выдержать баланс. Чрезмерное количество проверок во всех возможных и невозможных местах захламляют кодовую базу и код становится тяжелее понимать, читать и поддерживать (так как изменения параметров, например, добавление новых, надо также будет отображать в валидации). Полное отсутствие валидации приведёт к тому, что у вас будут **неконсистентные** данные.

Проверять данные лучше всего в моменте их получения и присваивания. Если вы получаете данные с какого-то источника и **можете** их провалидировать - сделайте это сразу же, чтобы **дальше** была гарантия консистентности данных, с которыми вы будете работать.

Все `precondition` проверки (такие как проверка аргументов, на `null` и т.д.) должны быть написаны в **начале** метода. Не должно быть никаких проверок аргументов в середине кода метода или конструктора. Ведь иначе не будет гарантии консистентности тех данных, с которыми мы работаем, правда?

Существуют ситуации, когда имеет смысл поставить проверку в конце метода, так называемые `postconditions`.
Например, при вычислении факториала результат **должен** быть больше или равен `1`.

Однако, в реальной жизни я крайне редко вижу использование `postconditions` в коде проекта, а не тестах на код.

Использовать `assert` имеет смысл только для проверки внутренних инвариантов и тестах. Не для `public api` с `preconditions` или `postconditions`.

## Заключение

При написании кода необходимо осуществлять проверки данных, дабы обеспечить консистентность и обезопасить код.
Для этого в `Java` существует несколько способов:

1. Ключевое слово `assert`.
2. Явные проверки.
3. Библиотеки, предоставляющие наиболее распространённые проверки.

Из всего этого я точно не рекомендовал бы использовать `assert`-ы для `pre/postconditions`, по причине того, что они не включены по умолчанию, при невыполнении условия выбрасывают `Error` и неудобны в перехвате исключений.

Использовать же `assert`-ы допустимо в тестах и для проверки сложных инвариантов.

## Полезные ссылки

1. [Assert in java](https://www.baeldung.com/java-assert)
2. [Preconditions and Postconditions](https://medium.com/@mlbors/preconditions-and-postconditions-5913fc0fcdaf)
3. [Preconditions, Postconditions, and Class Invariants](https://docs.oracle.com/cd/E19683-01/806-7930/assert-13/index.html)
4. [Assert. Что это?](https://habr.com/ru/post/141080/)

## Отдельная благодарность

Отдельную благодарность за ревью и помощь автор хочет выразить следующим людям из твиттера:

1. [@tagir_valeev](https://twitter.com/tagir_valeev)
2. [@dolzhenko](https://twitter.com/dolzhenko)
