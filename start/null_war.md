# Борьба с null

Здесь будет описано и рассказано про `null` с точки зрения разработчика на `Java`.

- [Борьба с null](#борьба-с-null)
  - [Да будет null](#да-будет-null)
  - [Ближе к Java](#ближе-к-java)
  - [Техника безопасности](#техника-безопасности)
    - [Не доверяй и проверяй](#не-доверяй-и-проверяй)
    - [Проверяй](#проверяй)
      - [Аккуратность в проверках](#аккуратность-в-проверках)
    - [Примитивы не так уж и примитивны](#примитивы-не-так-уж-и-примитивны)
    - [Аккуратнее со строками](#аккуратнее-со-строками)
    - [Не плоди null](#не-плоди-null)
    - [Отсутствие значения не всегда null](#отсутствие-значения-не-всегда-null)
    - [Optional](#optional)
    - [Доверяйте, но с аннотациями](#доверяйте-но-с-аннотациями)
  - [Заключение](#заключение)
  - [Полезные ссылки](#полезные-ссылки)
  - [Отдельная благодарность](#отдельная-благодарность)

## Да будет null

Для начала надо понять, как к `null` пришло человечество.

Во время написания кода в [объектно-ориентированной парадигме](../oop/intro.md) вы представляете свою программу в виде совокупности и взаимодействия объектов, каждый из которых является экземпляром определённого класса.

И всё бы ничего, но что делать, если вам необходимо обозначить отсутствие объекта?
Например, отсутствие пользователя, какого-то ресурса и т.д.

Представьте, что вы написали класс, описывающий пользователя, одним из полей класса является адрес электронной почты - `email`.
Но не у всех пользователей есть `email`: кто-то вообще его не использует, кто-то не хочет указывать его или собирается это сделать позже.

Т.е значения этого поля на *данный момент* нет, но в дальнейшем вполне возможно, что оно появится: например, пользователь его добавит в личном кабинете, или укажет в отделени банка сотруднику.

Вам необходимо обозначить *отсутствие* объекта, и вот тут-то на сцену и выходит `null`.

Казалось бы, всё и на этом тему можно закрывать, ведь все довольны и счастливы! Не совсем.

Если вы попытаетесь вызвать любой метод на `null`, то неизбежно получите `java.lang.NullPointerException`. А если исключение не будет обработано, то это неизбежно послужит тому, что ваше приложение аварийно завершится.

В этом и состоит основная проблема использования `null`: это потенциальный источник `java.lang.NullPointerException`.

Недаром `null` называют [ошибкой стоимостью в миллиард долларов](https://www.youtube.com/watch?v=ybrQvs4x0Ps).

## Ближе к Java

Что такое `null` в `Java`?

> "There is also a special null type, the type of the expression null, which has no name. Because the null type has no name,
> it is impossible to declare a variable of the null type or to cast to the null type. The null reference is the only
> possible value of an expression of null type. The null reference can always be cast to any reference type. In practice, the
> programmer can ignore the null type and just pretend that null is merely a special literal that can be of any reference type."
> [JLS 4.1](https://docs.oracle.com/javase/specs/jls/se8/html/jls-4.html#jls-4.1)

Из чего следует, что `null` в `Java` — это особое значение, оно не ассоциируется ни с каким типом (оператор `instanceOf` возвращает `false`, если в качестве параметра указать любую ссылочную переменную со значением `null` или `null` сам по себе) и может быть присвоено любой ссылочной переменной, а также возвращено из метода.

Это значит, что:

1. Каждое возвращаемое значение ссылочного типа может быть `null`.
2. Каждое значение параметра ссылочного типа может также быть `null`.

Чувствуете масштаб проблемы? Добавьте сюда ещё и тот факт, что `null` является значением по умолчанию для ссылочных типов, чтобы получить полное представление о ситуации с `null`-ами!

![NPE](../images/start/null.jpg)

Однако, будет неверным считать, что `null` - это всегда зло, что он неуместен только из-за того, что его неаккуратное использование может привести к ошибкам. И ножом можно нанести повреждения, но это не делает нож плохим инструментом.

В итоге `null` стал жертвой того, что, благодаря возможности присвоить любому ссылочному значению или вернуть из метода, его стали использовать неправильно. И возненавидели.

Но, как и в ситуации с ножом, у `null` тоже есть правила безопасности и о них мы и поговорим.

## Техника безопасности

Рассмотрим следущий код:

```java
public String greeting() {
    if (/* no greeting */) {
        return null;
    }

    return "Hello World!"
}
```

Подобный способ довольно распространен, вы пишите метод, а когда нет возвращаемого значения - отдаете `null`.
А теперь представьте как вы будете пользоваться (и как обрекаете на это других) этим методом?

Вы уже не можете просто взять и сделать:

```java
String message = greeting();
int len = message.length();
```

Ведь у вас нет **гарантии** о том, что будет возвращено: значение или `null`. Вы теряете **доверие** к этому коду.
Это очень опасная и губительная практика: возвращение `null` из методов.

Но надо понимать, что нет ничего плохого в `null` как в значении, но `null` как `reference` - однозначное и чистое **зло**.

Это значит, что поле класса вполне может быть хранить внутри `null`, проблемы начинаются в момент, когда начинается работа с этим полем - использование его как ссылку.

Для примера рассмотрим старого доброго `Person`-а, который скоро уже в суд подаст на нас за преследования:

```java
public class Person {
    private int age;
    private String userName;
    private String email;

    public Person(final int age, final String userName) {
        this.age = age;
        this.userName = userName;
    }

    // getters and setters
}
```

Как уже было сказано не раз, `null` - это значение по умолчанию для ссылочных типов.
Соответственно, по умолчанию у объектов `Person` в поле `email` будет `null`.

В целом, подобный код часто встречается и это **не** плохо: у нас есть обязательные значения(`name` и `age`) и необязательные, которые **могут** отсутствовать - электронный адрес.

### Не доверяй и проверяй

Самая явная и очевидная проверка на `null` в `Java` выглядит следующим образом:

```java
if(person.getEmail() != null) {
    // do some work with email
}
```

В `Java 7+` появился вспомогательный класс `java.util.Objects`, который содержит вспомогательные методы проверки на `null`:

```java
     /**
     * Returns {@code true} if the provided reference is {@code null} otherwise
     * returns {@code false}.
     *
     * @apiNote This method exists to be used as a
     * {@link java.util.function.Predicate}, {@code filter(Objects::isNull)}
     *
     * @param obj a reference to be checked against {@code null}
     * @return {@code true} if the provided reference is {@code null} otherwise
     * {@code false}
     *
     * @see java.util.function.Predicate
     * @since 1.8
     */
    public static boolean isNull(Object obj) {
        return obj == null;
    }

    /**
     * Returns {@code true} if the provided reference is non-{@code null}
     * otherwise returns {@code false}.
     *
     * @apiNote This method exists to be used as a
     * {@link java.util.function.Predicate}, {@code filter(Objects::nonNull)}
     *
     * @param obj a reference to be checked against {@code null}
     * @return {@code true} if the provided reference is non-{@code null}
     * otherwise {@code false}
     *
     * @see java.util.function.Predicate
     * @since 1.8
     */
    public static boolean nonNull(Object obj) {
        return obj != null;
    }
```

Однако, эти методы были добавлены в основном для `Java Stream API`, для удобного использования в `filter`.
Да и на мой взгляд, обычная проверка более читабельна и явная, сравните:

```java
if(Objects.nonNull(person.getEmail())) {
    // do some work with email
}

// vs

if(person.getEmail() != null) {
    // do some work with email
}
```

### Проверяй

Несмотря на все наши усилия и договорённости, `null` может также просочиться в объект ещё на этапе его создания.

```java
final Person person = new Person(20, null);
```

Объект будет создан, несмотря на то, что *не ожидается*, что поле `userName` может быть `null`.
Логичным решением будет потребовать уже на этапе создания объекта невозможность присвоения `null` значения такому полю.

Т.е. перед инициализацией **проверить** допустимость значений, которые получены конструктором.

К счастью, для этого в уже знакомом `java.util.Objects` есть необходимые методы:

```java
public static <T> T requireNonNull(T obj) {
    if (obj == null)
        throw new NullPointerException();
    return obj;
}

public static <T> T requireNonNull(T obj, String message) {
    if (obj == null)
        throw new NullPointerException(message);
    return obj;
}
```

Благодаря чему, наш `Person` приобретает дополнительные проверки и вы **всегда** быстро поймёте какое поле было проинициализировано неправильно:

```java
public Person(final int age, final String userName) {
    this.age = age;
    this.userName = Objects.requireNonNull(userName, "userName can't be null");
}
```

---

**Вопрос**:

Постойте, ведь в `Java` давно есть `assert`, а что насчёт них?

**Ответ**:

Да, есть и их также можно использовать для валидации значений:

```java
public Person(final int age, final String userName) {
    assert userName != null;

    this.age = age;
    this.userName = userName;
}
```

Однако, я ими пользоваться не рекомендую и вот почему:

1. По умолчанию они отключены.
2. Такая проверка, в случае проблемы, кидает `java.lang.AssertionError`, что осложняет обработку исключений в проекте и дальнейшую отладку.

Более подробно о [проверках](./assertions.md).

---

#### Аккуратность в проверках

Помните, что вызов метода на `null` неизбежно породит `java.lang.NullPointerException`:

```java
public void checkTag(final String tag) {
    tag.equals("UNKNOWN"); // может быть NPE, если tag = null
    "UNKNOWN".equals(tag); // безопасно
}
```

Поэтому, при работе с константами, `enum`-ами и т.д. вызывайте методы *на* константах, как наиболее безопасном месте.

### Примитивы не так уж и примитивны

Будьте аккуратны с `boxing/unboxing`.

Как вы думаете, что будет при выполнении следующего кода:

```java
public void printInt(Integer num) {
    System.out.println((int) num);
}

printInt(null);
```

А будет уже знакомый и горячо любимый `java.lang.NullPointerException`, возникший как раз в `unboxing`-е.

Поэтому будьте аккуратнее с `boxing/unboxing` и `null`-ами, по возможности пользуйтесь примитивами.

### Аккуратнее со строками

Помните, что при конкатенации строк, если там затесалось `null` значение, не будет `java.lang.NullPointerException`, но и игнорирования `null` не будет:

```java
String s1 = "hello";
String s2 = null;
String s3 = s1 + s2;
String s4 = new StringBuilder(s1).append(s2).toString();

System.out.println(s3);
System.out.println(s4);
```

В итоге в обоих случаях будет: `hello null`!

Человечество, кому повезло не быть разработчиками, ещё не готово к таким наскальным надписям, поэтому, дабы не удивляться `null`-ам в `UI` и логах, используйте `java.util.Objects`, [Google Guava](https://mvnrepository.com/artifact/com.google.guava/guava) или [Apache Commons](https://mvnrepository.com/artifact/org.apache.commons/commons-lang3) библиотеки.

Например, как это сделано в `java.util.Objects`:

```java

    /**
     * Returns the result of calling {@code toString} on the first
     * argument if the first argument is not {@code null} and returns
     * the second argument otherwise.
     *
     * @param o an object
     * @param nullDefault string to return if the first argument is
     *        {@code null}
     * @return the result of calling {@code toString} on the first
     * argument if it is not {@code null} and the second argument
     * otherwise.
     * @see Objects#toString(Object)
     */
    public static String toString(Object o, String nullDefault) {
        return (o != null) ? o.toString() : nullDefault;
    }
```

В таком случае:

```java
String s3 = Objects.toString(s1, "") + Objects.toString(s2, "");
```

Но я рекомендую использовать [Apache Commons](https://mvnrepository.com/artifact/org.apache.commons/commons-lang3) и класс [StringUtils](http://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html). Удобная и безопасная работа со строками с учётом `null`-ов.

### Не плоди null

В случае написания функциональности, где не все параметры всегда предоставляются, обычно велик соблазн написать один метод и далее использовать его, передавая при необходимости `null` в параметры, которые могут отсутствовать.

Например, обновление статуса с дополнительной информацией (по возможности):

```java
public void updateStatus(final String status, final String details) {
    // some code
}
```

При этом, `details` вполне может отсутствовать и зачастую использование сводится к:

```java
updateStatus("NEW", "Creating new order status");
updateStatus("NEW", null);  // при отсутствии details
```

В таком случае, лучше перегрузить метод и сделать:

```java
public void updateStatus(final String status, final String details) {
    // some code
}

public void updateStatus(final String status) {
    // some code
}
```

Старайтесь минимизировать явную передачу `null` в методы.

Во-первых, `null` в методе делает его менее читабельным, особенно, если `null` значений несколько, например:

```java
updateStatus("NEW", null, null, 200)
```

Во-вторых, закладывая возможность передачи в метод несколько `null` значений вы увеличиваете шанс появления ошибки, потому что следить за разрастающимся количеством `null`-ов тяжело, появляется возможность случайно передать его в метод.

Перегрузка является единственным способом борьбы с такой проблемой, так как в `Java`, к сожалению, нет поддержки значений по умолчанию.

### Отсутствие значения не всегда null

Одной из самых популярных ошибок в использовании `null` является возвращение его там, где ожидается коллекция данных.

Разберём пример: вы написали телефонную книгу, где есть возможность получить список номеров по имени абонента:

```java
public List<Person> findByName(final String name) {
    if(book.contains(name)) {
        return persons;
    }

    return null;
}
```

Т.е происходит простое ветвление логики на случай, если в телефонной книге есть записи с таким именем, и на случай, если нет, возвращается то самое отсутствие значения, наш любимый `null`.

Казалось бы, всё правильно, но нет!

Там, где контракт метода говорит о том, что будет возвращена коллекция данных по какому-то фильтру (в нашем случае - имени)**всегда** возвращайте пустую коллекцию, при отсутствии данных.

Точно также, если вы разрабатываете функционал, где одним из параметров является *необязательная* коллекция данных: **не делайте** возможность передачи вместо неё `null`:

```java
public List<Person> findByNameAndTags(final String name, final Set<Tag> tags) {
    // some code
}
```

Если `tags` необязательное значение для поиска, то при отсутствии значения передавайте пустое множество:

```java
findByNameAndTags("Kuchuk", ImmutableSet.of())
```

Либо сделайте два метода: с `tags` в сигнатуре и без.

```java
// когда tags обязательно
public List<Person> findByNameAndTags(final String name, final Set<Tag> tags) {
    // some code
}

// когда tags необязательно
public List<Person> findByNameAndTags(final String name) {
    // some code
}
```

Точно тот же совет, когда коллекция - это поле класса.

Например, вам необходимо множество кодов ошибок в валидаторе значений:

```java
public class Validator {
    private Set<Integer> codes = new HashSet<>();

    // some code
}
```

Если нет множества(не инициализировали, отсутствует) - сделайте его пустым множеством. Но не делайте его `null`!

Как вариант можно возвращать ещё итератор.

Отстутствие значения при работе с коллекциями - это пустая коллекция.
При работе с коллекциями и отношением данных `one-to-many` отсутствие значения - это и есть пустая коллекция.

### Optional

Начиная с `Java 8+` для борьбы с `null` был добавлен класс `java.util.Optional`.
Класс был добавлен для того, чтобы дать возможность разработчикам **явно** показывать, что значения **может** не быть.

Если объяснять на пальцах, то `java.util.Optional` - это просто контейнер, в который вы оборачиваете значение. При отсутствии значения у вас пустой контейнер, при существовании значения - у вас контейнер со значением.

```java
/**
 * A container object which may or may not contain a non-null value.
 * If a value is present, {@code isPresent()} will return {@code true} and
 * {@code get()} will return the value.
 *
 * <p>Additional methods that depend on the presence or absence of a contained
 * value are provided, such as {@link #orElse(java.lang.Object) orElse()}
 * (return a default value if value not present) and
 * {@link #ifPresent(java.util.function.Consumer) ifPresent()} (execute a block
 * of code if the value is present).
 *
 * <p>This is a <a href="../lang/doc-files/ValueBased.html">value-based</a>
 * class; use of identity-sensitive operations (including reference equality
 * ({@code ==}), identity hash code, or synchronization) on instances of
 * {@code Optional} may have unpredictable results and should be avoided.
 *
 * @since 1.8
 */
public final class Optional<T> { 
    // ...
}
```

Для работы `Optional` с примитивами существует: `java.util.OptionalInt`, `java.util.OptionalLong` и `java.util.OptionalDouble`.

Для примера, пусть необходим метод, который в телефонной книге ищет пользователя по имени и фамилии (как уникальным идентификаторам пользователя в нашей реализации):

```java
public Person findByNameAndSurname(final String name, final String surname) {
    // some code
}
```

Очевидно, что может возникнуть ситуация, когда будет произведен поиск несуществующего пользователя (например, не зарегестрированного у нас).

Что делать в таком случае?

Варианта, на самом деле, три:

1. Кинуть исключение
2. Вернуть `null`
3. Вернуть `Optional`

Из всех этих вариантов предпочтимее всего в данном случае вернуть `Optional`. Т.е. обернуть возвращаемое значение в контейнер и **явно** показать этим, что по таким параметрам поиска(имени и фамилии) пользователя может не быть.

```java
public Optional<Person> findByNameAndSurname(final String name, final String surname) {
    // some code
}
```

В таком случае, использование может быть в виде:

```java
Optional<Person> person = findByNameAndSurname("Aleksandr", "Kuchuk");

if(person.isPresent()) {
    // нашли пользователя
} else {
    // не нашли
}
```

Помимо примитивной проверки в `if`-е `Optional` можно(и нужно) использовать в `Java Stream API`:

```java
// ifPresent - совершение действия при существовании значения
final Optional.ofNullable(person.getEmail).ifPresent(email -> System.out.println(email));

// map и orElse - при существовании применить функцию к значению, если нет - вернуть unknown 
final String email = Optional.ofNullable(p.getEmail()).map(String::toLowerCase).orElse("unknown");
```

Однако, `Optional`, на мой взгляд, имеет смысл использовать **только** в качестве возвращаемых значений.
Определенно **не стоит** его использовать в качестве параметров метода, поскольку при использовании `Optional` в качестве параметров метода теряется читабельность, код становится более громоздким, проверят на `null` придеться всё равно и поэтому лучше предоставить действительное значение или сделать перегрузку метода в случаях, когда параметр необязателен.

Также и с полями класса. Нет смысла делать поля класса `Optional`, так как это будет не `Java Bean`, также `Optional` является не сериализуемым классом и т.д.

Есть три способа создать `Optional`:

```java

    /**
     * Returns an empty {@code Optional} instance.  No value is present for this
     * Optional.
     *
     * @apiNote Though it may be tempting to do so, avoid testing if an object
     * is empty by comparing with {@code ==} against instances returned by
     * {@code Option.empty()}. There is no guarantee that it is a singleton.
     * Instead, use {@link #isPresent()}.
     *
     * @param <T> Type of the non-existent value
     * @return an empty {@code Optional}
     */
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }

    /**
     * Returns an {@code Optional} with the specified present non-null value.
     *
     * @param <T> the class of the value
     * @param value the value to be present, which must be non-null
     * @return an {@code Optional} with the value present
     * @throws NullPointerException if value is null
     */
    public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }

    /**
     * Returns an {@code Optional} describing the specified value, if non-null,
     * otherwise returns an empty {@code Optional}.
     *
     * @param <T> the class of the value
     * @param value the possibly-null value to describe
     * @return an {@code Optional} with a present value if the specified value
     * is non-null, otherwise an empty {@code Optional}
     */
    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? empty() : of(value);
    }
```

Используйте:

1. `Optional.empty()` в случае, если вы уверены, что необходимо вернуть пустой контейнер.
2. `Optional.of(value)` в случае, если вы уверены, что значение, `value`, которое вы собираетесь положить в контейнер **совершенно точно** не `null`.
3. В противном случае используйте `Optional.ofNullable(value)`.

Никогда не смешивайте `null` и `Optional`:

```java
Optional<Object> optional = null; // Плохо!
Optional<Object> optional = Optional.of(null); // exception
```

Это путь в ад и к ненависти.

Подробно о [Optional](./optional.md).

### Доверяйте, но с аннотациями

Но как быть тому, кто будет использовать ваш код, понять, какие поля вы какие поля *могут* по нашей задумке быть `null`, а какие - нет, и это *явная* ошибка?

Ведь это нам, как разработчикам этого кода, прозрачно и понятно, что `email`-а может не быть и это нормальное поведение кода.
Другим разработчикам, да и нам самим через неделю, это абсолютно не ясно - и из этого возникает мысль, что было бы удобно как-то *разметить* наш класс, показать, где ожидаем `null` и это заложено, а где не ожидаем и это ошибка на миллион долларов.

Может ли поле быть с `null` или нет - это уже метаинформация о поле. А, значит, логично воспользоваться аннотациями.

К сожалению, в `Java` нет какого-то общего стандарта, и существует несколько видов аннотаций для этого, например:

1. [javax.validation.constraints.NotNull](https://jcp.org/en/jsr/detail?id=380)
2. [javax.annotation.Nonnull](https://jcp.org/en/jsr/detail?id=305)
3. [org.jetbrains.annotations.NotNull](https://www.jetbrains.com/help/idea/nullable-and-notnull-annotations.html)
4. [lombok.NonNull](https://projectlombok.org/)
5. [org.eclipse.jdt.annotation.NonNull](http://help.eclipse.org/oxygen/topic/org.eclipse.jdt.doc.user/tasks/task-improve_code_quality.htm)

В целом, я уверен, существуют еще несколько, но я выделил наиболее популярные (исключая `Android` специфичные) на данный момент.

Какую выбрать и как использовать?

На самом деле, вопрос сложный, но я в своих проектах пользуюсь тем, что предоставляет [JSR-305](https://jcp.org/en/jsr/detail?id=305), т.е `javax.annotation.Nonnull` и т.д.

Почему?
Дело в том, что я стараюсь избегать ссылок на `IDE` специфичные аннотации, поэтому `jetbrains`, `lombok` и `eclipse` отпадают.
Остается только `javax.annotation` и `javax.validation.constraints`.
Я сделал выбор в пользу первой как более простой, наглядной и распространённой.
Вообще, это довольно холиварный вопрос, но, если вы пишите на `Java 8`, то `JSR-305` будет к месту.

> Подробнее об этом [тут](https://dzone.com/articles/when-to-use-jsr-305-for-nullability-in-java) и [тут](https://stackoverflow.com/questions/4963300/which-notnull-java-annotation-should-i-use).

Благодаря [плагину](https://plugins.jetbrains.com/plugin/13665-effective-inner-builder) и аннотациям из [JSR-305](https://jcp.org/en/jsr/detail?id=305) получается разметить и сгенерировать код с билдерами и проверками на `null`:

```java
import javax.annotation.Nonnull;
import javax.annotation.Nullable;
import javax.annotation.ParametersAreNonnullByDefault;
import java.util.Objects;

@ParametersAreNonnullByDefault
public class Person {
    @Nonnull
    private final String name;

    @Nullable
    private final String email;

    private final int age;


    private Person(Builder builder) {
        this.name = Objects.requireNonNull(builder.name, "name");
        this.age = Objects.requireNonNull(builder.age, "age");
        this.email = builder.email;
    }

    public static Builder builder() {
        return new Builder();
    }

    @Nonnull
    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    @Nullable
    public String getEmail() {
        return email;
    }

    public static class Builder {
        private String name;
        private Integer age;
        private String email;

        private Builder() {
        }

        public Builder setName(String name) {
            this.name = name;
            return this;
        }

        public Builder setAge(int age) {
            this.age = age;
            return this;
        }

        public Builder setEmail(@Nullable String email) {
            this.email = email;
            return this;
        }

        public Builder of(Person person) {
            this.name = person.name;
            this.age = person.age;
            this.email = person.email;
            return this;
        }

        public Person build() {
            return new Person(this);
        }
    }
}
```

Аннотация `@ParametersAreNonnullByDefault` на классе говорит о том, что все параметры в методах по умолчанию `@Nonnull`.

> Но не возвращаемые значения!
> Возвращаемые значения надо размечать вручную самому и явно.

Это довольно удобно и практично.

Однако, надо понимать, что эти аннотации - не более, чем разметка, чтобы облегчить и структурировать себе и потомкам понимание логики.

Грубо говоря, это как разметка на дороге, вы понимаете, где может быть опасное место, а где в целом безопасный отрезок трассы, но это не защитит вас от грузовика по встречной полосе, если кто-то на нём вырулит:

```java
public void printStatus(@Nonnull String status) {
    // some code
}

printStatus(null);  // тот самый грузовик по встречной полосе
```

Аннотации - не более, чем рекомендации и описание правил, которые **в идеале** должны соблюдать все участники проекта.
Но они могут и не соблюдать их!

## Заключение

При разработке не бойтесь `null`, но старайтесь минимизировать его использование.
Нет ничего плохого в `null` как в значении, но `null` как `reference` - однозначное и чистое **зло**.

По возможности, избегайте использование `null` в качестве возвращаемых значений, предпочитая `Optional` или исключение.
Не задействуйте `null` для указания ошибок: лучше выбрасывайте явное исключение.

Помните, что отсутствие значения у коллекции - это чаще всего пустая коллекция или пустой итератор.

По возможности, явно размечайте аннотациями код, чтобы показать разработчикам, использующим ваш код, где `null` заложен в логике, а где - нет.

Старайтесь использовать `pre-conditions` с помощью `Objects.requireNonNull` - чем раньше вы поймёте где проблема и с чем, тем лучше.

Будьте аккуратнее со строками и `boxing/unboxing`!

## Полезные ссылки

1. [Null pointer в разных языках программирования](https://en.wikipedia.org/wiki/Null_pointer)
2. [Спецификация Java про null](https://docs.oracle.com/javase/specs/jls/se8/html/jls-4.html#jls-4.1)
3. [Про null](https://www.mindprod.com/jgloss/null.html)
4. [Когда использовать NonNull аннотации](https://dzone.com/articles/when-to-use-jsr-305-for-nullability-in-java)
5. [Про проверки на null](https://www.baeldung.com/java-avoid-null-check)
6. [Какие аннотации для null использовать](https://stackoverflow.com/questions/4963300/which-notnull-java-annotation-should-i-use)
7. [Про Optional](https://blog.joda.org/2014/11/optional-in-java-se-8.html)
8. [Java Optional — попытка избежать NullPoinerException](https://www.youtube.com/watch?v=WRRHDl0cwOg)
9. [Ещё про Optional](https://habr.com/ru/post/346782/)
10. [Про Null.Часть 1](https://medium.com/destinationaarhus-techblog/avoiding-null-pointer-exceptions-in-a-modern-java-application-c048ba872f7e)
11. [Про Null.Часть 2](https://medium.com/destinationaarhus-techblog/avoiding-null-pointer-exceptions-in-a-modern-java-application-5e54fab6e3b)
12. [Stop Abusing Nihil](https://rocket-science.ru/hacking/2019/03/21/stop-abusing-nihil)
13. [Java. Проблема с null. Null safety.](https://www.youtube.com/watch?v=QIfIoXW1SHA&list=PL5BhKu-LkR_3zYHvQZ-AjkmcYciv65QVB)

## Отдельная благодарность

Отдельную благодарность за ревью и помощь автор хочет выразить следующим людям из твиттера:

1. [@norrittmobile](https://twitter.com/norrittmobile)
2. [@thecoldwine](https://twitter.com/thecoldwine)
3. [@mudasobwa](https://twitter.com/mudasobwa)
4. [@dmi3vim](https://twitter.com/dmi3vim)
5. [@ihavenoclue12](https://twitter.com/ihavenoclue12)
