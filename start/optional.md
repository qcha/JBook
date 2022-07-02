# Optional

- [Optional](#optional)
    - [Введение](#введение)
    - [Реализация](#реализация)
    - [Работа с Optional](#работа-с-optional)
        - [Создание](#создание)
        - [Использование](#использование)
            - [isPresent](#ispresent)
            - [ifPresent](#ifpresent)
            - [orElse](#orelse)
            - [orElseGet](#orelseget)
            - [orElseThrow](#orelsethrow)
        - [Методы преобразования](#методы-преобразования)
            - [filter](#filter)
            - [map](#map)
            - [flatMap](#flatmap)
            - [or](#or)
        - [Комбинирование методов](#комбинирование-методов)
        - [Примитивы](#примитивы)
    - [Когда не надо использовать Optional](#когда-не-надо-использовать-optional)
    - [Заключение](#заключение)
    - [Полезные ссылки](#полезные-ссылки)

## Введение

При разработке на `Java` вы так или иначе будете сталкиваться с `null`-ами.
Они будут везде! И это целая война.

Подробнее об [этом](./null_war.md).

Но вернемся непосредственно к герою данной заметки - `java.util.Optional`.

Для примера, пусть необходим метод, который в телефонной книге ищет пользователя по имени и фамилии (как уникальным идентификаторам пользователя в нашей реализации):

```java
public Person findByNameAndSurname(final String name, final String surname) {
    // some code
}
```

Очевидно, что может возникнуть ситуация, когда будет произведен поиск несуществующего пользователя (например, не зарегестрированного у нас).

Что делать в таком случае?

Одним из вариантов может быть выброс исключения, например, собственного `PersonNotFoundException`.
Но обрекать себя и пользователей нашего кода на постоянные проверки в `try/catch` не хочется.
Это сделает использование такого кода громоздким и опасным, ведь один раз забыв про `try/catch` над вызовом `findByNameAndSurname` может сломать всю программу.
Да и сама ситуация отсутствия по фамилии/отчеству человека в телефонной книге не такого рода, чтобы ломать поток выполнения программы исключением.

Другой вариант явно вернуть `null`, при отсутствии значения.
Поток выполнения не ломает, перехватывать и обрабатывать исключения на каждый вызов не надо.

Но все еще есть существенный минус: теперь надо не забыть проверить возвращаемое значение на `null`.
А это уже проблема, потому что сигнатура метода явно ничего не говорит о том, может ли быть возвращен `null` или нет.

Поэтому нам нужен вариант, когда мы и в сигнатуре метода явно покажем, что может быть отсутствие значения, и поток выполнения не сломаем исключением.
И для этих целей в `Java 8+` есть `java.util.Optional`!

## Реализация

Для понимания внутреннего устройства рассмотрим часть кода класса из `Java 8`:

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
    /**
     * Common instance for {@code empty()}.
     */
    private static final Optional<?> EMPTY = new Optional<>();

    /**
     * If non-null, the value; if null, indicates no value is present
     */
    private final T value;

    // code code code
}
```

Как видно из описания, `java.util.Optional` - это контейнер, хранящий в себе значение.
Класс параметризуется и позволяет хранить любые ссылочные типы.

По сути, этот класс - это обертка над значением.
Если значения нет -  контейнер будет пустой, если есть - оно будет внутри контейнера.

При этом, использование контейнера-обертки явно показывает, что значения может не быть и **обязует** программиста делать явные проверки.

Для этого внутри `java.util.Optional` существует большое количество вспомогательных методов.

## Работа с Optional

### Создание

Создание `Optional` происходит через фабричные методы.

1. `Optional.of`.

    Используется тогда, когда вы помещаете в `Optional` значение, которое совершенно точно не является `null`.
    При этом, если передать `null` будет выброшено `NPE` исключение.

    ```java

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
    ```

2. `Optional.ofNullable()`

    Используется тогда, когда вы не гарантируете то, что помещаемое значение не является `null`.

    ```java

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

3. `Optional.empty()`

    Используется тогда, когда вам надо вернуть пустой контейнер.

    ```java
        public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }
    ```

### Использование

Итак, какие методы предоставляет `java.util.Optional` для работы с ним.

Предположим, что у нас уже есть метод:

```java
public Optional<Person> findByNameAndSurname(final String name, final String surname) {
    // some code
}
```

Как можно заметить, теперь мы оперируем уже контейнером `Optional`, в котором либо есть значение, либо его нет и контейнер пуст.

Программист может явно взять значение из контейнера с помощью метода `get`:

```java

    /**
     * If a value is present in this {@code Optional}, returns the value,
     * otherwise throws {@code NoSuchElementException}.
     *
     * @return the non-null value held by this {@code Optional}
     * @throws NoSuchElementException if there is no value present
     *
     * @see Optional#isPresent()
     */
    public T get() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }
```

Как видно, просто взять значение из контейнера небезопасно, так как контейнера может быть пуст, в таком случае будет сгенерировано исключение `NoSuchElementException`.

Да и такое использование `Optional` бессмысленно, ведь пропадает основная идея, почему он вводился - показать программисту возможное отсутствие значения, обязать явно проверять и реагировать на это отсутствие.

И в `Optional` есть все необходимое!

#### isPresent

Существует метод, явно проверяющий существует обёрнутый объект или нет: `isPresent()`.

```java

    /**
     * Return {@code true} if there is a value present, otherwise {@code false}.
     *
     * @return {@code true} if there is a value present, otherwise {@code false}
     */
    public boolean isPresent() {
        return value != null;
    }
```

Метод `isPresent` - это по сути обычная проверка, как если бы мы писали `if (value != null )`:

```java
Optional<Person> optPerson = findByNameAndSurname("Aleksandr", "Kuchuk");

if (optPerson.isPresent()) {
    Person person = optPerson.get();

    // some code
}
```

Как видно, здесь возвращаемое значение из `findByNameAndSurname` - это `Optional`, что обязует разработчика сначала проверить на наличие значения, а уже после безопасно его извлечь.

Однако, если вы пользуетесь `Optional` только так, то это верный признак, что вы не используете возможности класса в полной мере!

#### ifPresent

Помимо `isPresent`, существует еще метод `ifPresent`, куда можно передать функцию, которая будет выполнена над объектом внутри контейнера, при условии, что он там есть:

```java
    /**
     * If a value is present, invoke the specified consumer with the value,
     * otherwise do nothing.
     *
     * @param consumer block to be executed if a value is present
     * @throws NullPointerException if value is present and {@code consumer} is
     * null
     */
    public void ifPresent(Consumer<? super T> consumer) {
        if (value != null)
            consumer.accept(value);
    }
```

Пример:

```java
Optional<Person> optPerson = findByNameAndSurname("Aleksandr", "Kuchuk");

optPerson.ifPresent(person -> System.out.println(person));
```

#### ifPresentOrElse

Зачастую бывает необходимо выполнить какую-то логику, при отсутствии значения, а не просто проигнорировать эту ситуацию, в таком случае пригодится `ifPresentOrElse`:

```java
 /**
 * If a value is present, performs the given action with the value,
 * otherwise performs the given empty-based action.
 *
 * @param action the action to be performed, if a value is present
 * @param emptyAction the empty-based action to be performed, if no value is
 *        present
 * @throws NullPointerException if a value is present and the given action
 *         is {@code null}, or no value is present and the given empty-based
 *         action is {@code null}.
 * @since 9
 */
public void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction) {
    if (value != null) {
        action.accept(value);
    } else {
        emptyAction.run();
    }
}
```

Пример:

```java
Optional<Person> optPerson = findByNameAndSurname("Aleksandr", "Kuchuk");

optPerson.ifPresentOrElse(
    person -> System.out.println(person),
    () -> System.out.println("Unknown user")
);
```

Если нашли пользователя - распечатали его в консоль, не нашли - выполнили печать "Unknown user".

Теперь разберемся, что делать, если надо не выполнить действие, а вернуть или значение из контейнера, или значение по-умолчанию.
В таком случае у `Optional` есть методы: `orElse`, `orElseGet` и `orElseThrow`.

#### orElse

Метод `orElse`:

```java
    /**
     * Return the value if present, otherwise return {@code other}.
     *
     * @param other the value to be returned if there is no value present, may
     * be null
     * @return the value, if present, otherwise {@code other}
     */
    public T orElse(T other) {
        return value != null ? value : other;
    }
```

Метод простой и делает ровно то, о чем мы говорили выше: или возвращает значение в контейнере, или значение по-умолчанию, которое вы указали.

Пример использования:

```java
config.get("timeout").orElse(5000);
```

Когда используется: когда при отсутствии значения ваша реакция - это возвращение значения по-умолчанию.

#### orElseGet

Метод `orElseGet`:

```java

    /**
     * Return the value if present, otherwise invoke {@code other} and return
     * the result of that invocation.
     *
     * @param other a {@code Supplier} whose result is returned if no value
     * is present
     * @return the value if present otherwise the result of {@code other.get()}
     * @throws NullPointerException if value is not present and {@code other} is
     * null
     */
    public T orElseGet(Supplier<? extends T> other) {
        return value != null ? value : other.get();
    }
```

Метод очень похож на предыдущий, только вместо конкретного значения по-умолчанию, при отсутствии значения в контейнере, он выполнит функцию (`Supplier`).

Пример использования:

```java
config.get("timeout").orElseGet(() -> {
    logger.warn("Can't find timeout property, use default value");

    5000
});
```

Когда используется: когда при отсутствии значения вы хотите выполнить действие (Suplier), результатом которого будет возвращаемое значение.

#### orElseThrow

Метод `orElseThrow` либо вернет значение в контейнере, либо выкинет исключение (которое вы сгенерируете внутри `Supplier`):

```java

    /**
     * Return the contained value, if present, otherwise throw an exception
     * to be created by the provided supplier.
     *
     * @apiNote A method reference to the exception constructor with an empty
     * argument list can be used as the supplier. For example,
     * {@code IllegalStateException::new}
     *
     * @param <X> Type of the exception to be thrown
     * @param exceptionSupplier The supplier which will return the exception to
     * be thrown
     * @return the present value
     * @throws X if there is no value present
     * @throws NullPointerException if no value is present and
     * {@code exceptionSupplier} is null
     */
    public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
        if (value != null) {
            return value;
        } else {
            throw exceptionSupplier.get();
        }
    }
```

Пример использования:

```java
Optional<Person> optPerson = findByNameAndSurname("Aleksandr", "Kuchuk");

optPerson.orElseThrow(() -> PersonNotFoundException::new);
```

Когда используется: когда при отсутствии значения вы хотите выполнить действие (`Suplier`), результатом которого будет генерация исключения.

### Методы преобразования

Но самая главная сила `Optional` в том, что у него есть методы, знакомые нам по `Stream`-ам: `map`, `filter` и `flatMap`.

#### filter

Как работает `filter`: если внутри контейнера есть значение и оно удовлетворяет переданному условию (`Predicate`), то будет возвращен контейнер с этим значением, иначе будет возвращен пустой контейнер.

```java

    /**
     * If a value is present, and the value matches the given predicate,
     * return an {@code Optional} describing the value, otherwise return an
     * empty {@code Optional}.
     *
     * @param predicate a predicate to apply to the value, if present
     * @return an {@code Optional} describing the value of this {@code Optional}
     * if a value is present and the value matches the given predicate,
     * otherwise an empty {@code Optional}
     * @throws NullPointerException if the predicate is null
     */
    public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
    }
```

Пример использования:

```java
// либо в контейнере Кучук Александр старше 30, либо пустой контейнер.
Optional<Person> optPerson = findByNameAndSurname("Aleksandr", "Kuchuk").filter(p -> p.age > 30);
```

Когда используется: когда необходимо проверить удовлетворение значения в контейнере условию.

#### map

Как работает `map`: если внутри контейнера есть значение, то к значению применяется переданная функция, результат оборачивается в `Optional` и возвращается, в случае отсутствия значения будет возвращен пустой контейнер.

```java
    /**
     * If a value is present, apply the provided mapping function to it,
     * and if the result is non-null, return an {@code Optional} describing the
     * result.  Otherwise return an empty {@code Optional}.
     *
     * @apiNote This method supports post-processing on optional values, without
     * the need to explicitly check for a return status.  For example, the
     * following code traverses a stream of file names, selects one that has
     * not yet been processed, and then opens that file, returning an
     * {@code Optional<FileInputStream>}:
     *
     * <pre>{@code
     *     Optional<FileInputStream> fis =
     *         names.stream().filter(name -> !isProcessedYet(name))
     *                       .findFirst()
     *                       .map(name -> new FileInputStream(name));
     * }</pre>
     *
     * Here, {@code findFirst} returns an {@code Optional<String>}, and then
     * {@code map} returns an {@code Optional<FileInputStream>} for the desired
     * file if one exists.
     *
     * @param <U> The type of the result of the mapping function
     * @param mapper a mapping function to apply to the value, if present
     * @return an {@code Optional} describing the result of applying a mapping
     * function to the value of this {@code Optional}, if a value is present,
     * otherwise an empty {@code Optional}
     * @throws NullPointerException if the mapping function is null
     */
    public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }
```

Пример использования:

```java
// либо в контейнере телефон Кучука Александра, либо пустой контейнер.
Optional<String> optPhone = findByNameAndSurname("Aleksandr", "Kuchuk").map(kuchuk -> kuchuk.getPhone());
```

Когда используется: когда необходимо применить функцию к значению в контейнере.

#### flatMap

Как уже было сказано, `map` применяет функцию и возвращает результат работы, завернутый в `Optional`.
Но что делать, если результат работы нашей функции уже и так возвращает `Optional`?

Например, телефона может не быть у пользователя и мы явно оборачиваем в `Optional` это.
В таком случае, результат работы `map` будет `Optional` завернутый в `Optional`.

Ведь мы применили функцию, которая возвращает `Optional` и `map` результат работы функции всегда оборачивает в `Optional`:

```java
Optional<Optional<String>> optPhone = findByNameAndSurname("Aleksandr", "Kuchuk")
                                                .map(kuchuk -> Optional.ofNullable(kuchuk.getPhone()));
```

Это зачастую неудобно, ведь не очень удобно работать с дважды запакованным значением.

Для этого и нужен `flatMap`!

Как работает `flatMap`: работает также как `map`, но в качестве возвращаемого значения функции требует `Optional`.

```java
    /**
     * If a value is present, apply the provided {@code Optional}-bearing
     * mapping function to it, return that result, otherwise return an empty
     * {@code Optional}.  This method is similar to {@link #map(Function)},
     * but the provided mapper is one whose result is already an {@code Optional},
     * and if invoked, {@code flatMap} does not wrap it with an additional
     * {@code Optional}.
     *
     * @param <U> The type parameter to the {@code Optional} returned by
     * @param mapper a mapping function to apply to the value, if present
     *           the mapping function
     * @return the result of applying an {@code Optional}-bearing mapping
     * function to the value of this {@code Optional}, if a value is present,
     * otherwise an empty {@code Optional}
     * @throws NullPointerException if the mapping function is null or returns
     * a null result
     */
    public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())
            return empty();
        else {
            return Objects.requireNonNull(mapper.apply(value));
        }
    }
```

Пример использования:

```java
// либо в контейнере телефон Кучука Александра, либо пустой контейнер.
Optional<String> optPhone = findByNameAndSurname("Aleksandr", "Kuchuk").flatMap(kuchuk -> Optional.ofNullable(kuchuk.getPhone()));
```

Когда используется: когда необходимо применить функцию, уже возвращающую `Optional`, к значению в контейнере.

#### or

Начиная с `Java 9+` в `Optional` добавлен еще один метод: `or`.

Как работает `or`: метод позволяет преобразовать пустой `Optional` в непустой.

```java
/**
 * If a value is present, returns an {@code Optional} describing the value,
 * otherwise returns an {@code Optional} produced by the supplying function.
 *
 * @param supplier the supplying function that produces an {@code Optional}
 *        to be returned
 * @return returns an {@code Optional} describing the value of this
 *         {@code Optional}, if a value is present, otherwise an
 *         {@code Optional} produced by the supplying function.
 * @throws NullPointerException if the supplying function is {@code null} or
 *         produces a {@code null} result
 * @since 9
 */
public Optional<T> or(Supplier<? extends Optional<? extends T>> supplier) {
    Objects.requireNonNull(supplier);
    if (isPresent()) {
        return this;
    } else {
        @SuppressWarnings("unchecked")
        Optional<T> r = (Optional<T>) supplier.get();
        return Objects.requireNonNull(r);
    }
}
```

Пример использования:

```java
Department department = Optional.ofNullable(employee)
        .map(Employee::getDepartment)
        .or(() -> Optional.of(new Department()))
        .get();
```

Когда используется: когда необходимо пустой `Optional` преобразовать в непустой.

### Комбинирование методов

Все методы преобразования прекрасно комбинируются друг с другом, позволяя создавать цепочки логики и не захламлять код лишними проверками:

```java
Instant time = Optional.ofNullable(req.getCreationTime())
    .map(x -> LocalDateTime.parse(x, DateTimeFormatter.ISO_DATE_TIME))
    .orElse(LocalDate.now().minusDays(1).atStartOfDay())
    .atZone(ZoneId.systemDefault())
    .toInstant()
```

И по сути, в этом и заключается основное удобство использования `Optional`.
В комбинации методов преобразования и явной проверки.

### Примитивы

Для работы с примитивами есть `java.util.OptionalDouble`, `java.util.OptionalInt` и `java.util.OptionalLong`.

Все эти классы похожи на `Optional`, но не имеют методов преобразования.
В них доступны только: `get`, `orElse`, `orElseGet`, `orElseThrow`, `ifPresent` и `isPresent`.

Используются они крайне редко.

## Когда не надо использовать Optional

Прежде всего надо уяснить, что тип `Optional` ввели для возвращаемых значений из метода!

> Optional was designed to provide a limited mechanism for library method return types where there needed to be a clear way to represent "no result".

Это значит, что **не надо** использовать его для объявления свойств класса:

```java
'Optional<String>' used as type for field 'zipCode'

Inspection info: Reports any uses of java.util.Optional<T>, java.util.OptionalDouble, java.util.OptionalInt, java.util.OptionalLong or com.google.common.base.Optional as the type for a field or a parameter. Optional was designed to provide a limited mechanism for library method return types where there needed to be a clear way to represent "no result". Using a field with type java.util.Optional is also problematic if the class needs to be Serializable, which java.util.Optional is not.
```

Использование `Optional` в качестве типа для поля класса имеет ряд проблем: не сериализуется, использование его как поле класса не всегда хорошо работает в `Spring`, `Hibernate` и некоторых других популярных фреймворках.

Не оборачивайте коллекции в `Optional`!

Такая запись:

```java
public Optional<List<String>> getPhones() {
    // code
}
```

Не имеет никакого смысла.
Любая коллекция является контейнером сама по себе. Для того чтобы показать отсутствие элементов в ней, не требуется использовать дополнительные обертки.
Нет значения - верните пустую коллекцию.

## Заключение

Класс `Optional` не решает проблему `NullPointerException` полностью, но при правильном применении позволяет снизить количество ошибок, сделать код более читабельным и компактным.
Использование `Optional` не всегда уместно, но для возвращаемых значений из методов он подходит отлично.

Несмотря на некоторые спорные решения класса, как, например, фабричный метод `of`, использование `Optional` обезопасит ваш код, так как явно будет требовать от разработчика проверять возвращаемое значение. При этом вам не потребуется ломать поток выполнения программы исключениями.

## Полезные ссылки

1. [Java Optional не такой уж очевидный](https://habr.com/ru/post/540080/)
2. [Optional или как избавиться от NPE в Java / жизнь без Null Pointer Exception реальна?](https://youtu.be/fbEnhHjEX3M)
3. [Java Optional — попытка избежать NullPoinerException.](https://youtu.be/WRRHDl0cwOg)
4. [Блог Косарева Александра: Введение в Optional](https://alexkosarev.name/2019/03/15/optional-in-java/)
