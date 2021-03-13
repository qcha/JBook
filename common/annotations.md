# Аннотации

Представим, что у нас в проекте каждый тест, исправляющий дефект должен быть связан с некоторым баг-репортом во внешней системе. Естественным желанием было бы указывать ссылку на этот баг-репорт в коде, например, в комментариях (пример из [Spring Data MongoDB)](https://github.com/spring-projects/spring-data-mongodb/blob/a88748d79809ac969567cbeb840437ec44e81fe2/spring-data-mongodb/src/test/java/org/springframework/data/mongodb/gridfs/GridFsTemplateIntegrationTests.java#L316):

```java
@Test // DATAMONGO-765
public void considersSkipLimitWhenQueryingFiles() {
    // тело теста
}
```

подобный комментарий, по сути, несет в себе некоторую структурированную мета-информацию.

В Java для выражения подобного существует специальный тип - **аннотация**.


**Аннотация** в Java - элемент языка, позволяющий прикрепить некоторую мета-информацию к _другим_ элементам языка.

В качестве примера, рассмотрим аннотацию, с которой, наверняка, сталкивался каждый Java-разработчик - [`@Override`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Override.html).


Она сообщает компилятору, что происходит переопределение метода супер-класса или интерфейса.
Её отсутствие не вызовет ошибки компиляции, однако её присутствие у метода, не переопределяющего метод предка, вызовет ошибку компиляции.

Здесь можно вспомнить тот самый пример с `equals`:

```java
public class Person {
    @Override
    public boolean equals(Person person) {
        //                ^^^^^^
        //               Должен быть Object
        return super.equals(obj);
    }
}
```

Среди других аннотаций пакета `java.lang` находятся:

* [`@Deprecated`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Deprecated.html) - предупреждает разработчика о том, что _элемент_ (например: метод, класс) устарел и не должен использоваться
* [`@FunctionalInterface`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/FunctionalInterface.html) выражает намерение программиста сделать из интерфейса _функциональный интерфейс_ (тот, у которого один `abstract` ный метод - _аналог_ функций в Java), в противном случае - выдать ошибку компиляции
* [`@SafeVarargs`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/SafeVarargs.html) - сигнал компилятору о том, что тело метода *не* будет выполнять небезопасных операций с vararg типами параметров (это связано с generic'ами).
См. пример в javadoc + [Java SafeVarargs annotation, does a standard or best practice exist?](https://stackoverflow.com/questions/14231037/java-safevarargs-annotation-does-a-standard-or-best-practice-exist)
* [`@SuppressWarnings`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/SuppressWarnings.html) - сообщает компилятору, что предупреждение необходимо подавить.
Наиболее известный частный случай - [`@SuppressWarnings("unchecked")`](https://stackoverflow.com/questions/1129795/what-is-suppresswarnings-unchecked-in-java)

## Объявление аннотаций

Для объявления аннотаций используется комбинация `@` и ключевого слова `interface` (а в Kotlin для этого есть ключевое слово [`annotation`](https://kotlinlang.org/docs/reference/annotations.html)):

```java
public @interface Wow {
}
```

[_Аннотации - на самом деле интерфейсы_](https://docs.oracle.com/javase/specs/jls/se11/html/jls-9.html#jls-9.6)

* В примере кода выше объявлена аннотация `Wow`.
* Она _практически_ не будет видна во время исполнения (см. [retention](#)) // todo
* Она может быть добавлена _практически_ (<<targetDefaultValue, Target по умолчанию>>) к любому элементу, например:

Пример:

```java
@Wow
public class Something {
    @Wow
    private final Set<String> strings;

    @Wow
    public Something(@Wow Set<String> stringsSet) {
        this.strings = stringsSet;
    }
}
```

В примере выше проаннотированы и класс, и поле, и конструктор, и его параметр.

_Если доводить их использование до абсурда, то может получиться [что-то такое](https://twitter.com/lukaseder/status/711612663202238464)_

## Методы аннотаций

У аннотаций могут быть _методы_ (ведь, аннотации -- интерфейсы).

Пример:

```java
public @interface Scheduled {
    int delayMillis();

    int rateMillis();
}
```

Использоваться эти методы могут примерно следующим образом:

```java
public class ScheduledUsage {
    @Scheduled(delayMillis = 100, rateMillis = 1000)
    public void scheduledMethod() {
    }
}
```

### Значения по умолчанию

У метода может быть значение по умолчанию, чтобы его задать используется ключевое слово `default`:

```java
public @interface Scheduled {
    int delayMillis();

    int rateMillis() default 1000;
}
```

Тогда `rateMillis` указывать будет необязательно.
Вместо этого будет использоваться значение по умолчанию - `1000`:

```java
public class ScheduledUsage {
    @Scheduled(delayMillis = 100)
    public void scheduledMethod() {
    }
}
```

### Возвращаемые значения методов

#### Типы

Методы аннотаций могут иметь *только* один из следующих *типов* возвращаемого значения:

* Аннотация
* Примитив (`int`, `long`, `float` и т.д.)
* `java.lang.Class`
* `enum`
* `java.lang.String`
* Массивы того, что перечислено _выше_

#### Ограничения на значения

Возвращаемые значения могут быть только *константами*, т.е. следующий пример *не* скомпилируется:

```java
public class ScheduledUsage {
    private final int delay;
    private final int rate;

    public ScheduledUsage(int delay, int rate) {
        this.delay = delay;
        this.rate = rate;
    }

    @Scheduled(delayMillis = delay, rateMillis = rate)
    // java: element value must be a constant expression
    public void scheduledMethod() {
    }
}
```

_Если попробовать посмотреть на это с другой стороны, то: аннотации добавляются к элементам класса, а *не* к их экземплярам_

### `java.lang.annotation`

`java.lang.annotation` - пакет, добавляющий поддержку аннотаций в сам язык.
Как ни странно, часть этой поддержки тоже добавляется через аннотации.

В этом пакете находятся:

* `interface` [`Annotation`](https://stackoverflow.com/questions/1129795/what-is-suppresswarnings-unchecked-in-java).
Все аннотации его реализуют
* Ошибки / исключения: [`AnnotationTypeMismatchException`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/AnnotationTypeMismatchException.html), [`IncompleteAnnotationException`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/IncompleteAnnotationException.html), [`AnnotationFormatError`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/AnnotationFormatError.html)
* Другие аннотации:
  * [`@Documented`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Documented.html)
  * [`@Inherited`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Inherited.html)
  * [`@Native`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Native.html)
  * [`@Repeatable`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Repeatable.html)
  * [*`@Retention`*](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Retention.html) - аннотация, позволяющая указать, "до какого этапа жизни" аннотации она должна присутствовать (Подробнее - в [Retention](#Retention))
  * [*`@Target`*](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Target.html) - аннотация, ставящаяся над другой аннотацией.
Она позволяет уточнить / ограничить места, где данная _аннотированная_ аннотация может быть добавлена

_Наиболее важные элементы в списке выше *выделены*._

## Retention

Аннотации могут быть использованы для разных целей:

1. Генерация исходного `java`-кода на этапе компиляции (так делает, например, [Dagger](https://dagger.dev/), [Micronaut](https://micronaut.io/))
1. Использование информации *во время исполнения*.
Так делают [Jackson](https://github.com/FasterXML/jackson), [Gson](https://github.com/google/gson), [Retrofit](https://square.github.io/retrofit/)

* Пункт 1 соответствует `RetentionPolicy.SOURCE`.
* Второй - `RetentionPolicy.RUNTIME`.
* Кроме этого, есть нечто посередине - `RetentionPolicy.CLASS`.
Это - значение *по умолчанию*

_См. [`RetentionPolicy`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/RetentionPolicy.html)_

TIP: Для генерации исходного java-кода на этапе компиляции входная точка - [Annotation Processing API](https://docs.oracle.com/javase/8/docs/api/javax/annotation/processing/package-summary.html).
См также: [Awesome Java Annotation Processing](https://github.com/gunnarmorling/awesome-annotation-processing)

### Сравнение RetentionPolicy


| RetentionPolicy | Описание | Остается в класс-файле | Доступна во время исполнения|
|-----------------|----------|------------------------|-----------------------------|
| `SOURCE` | Отбрасываются после компиляции | icon:times[] | icon:times[] |
| `CLASS` | Отбрасывается на этапе _загрузки класса_ | icon:check[] | Условно: если найти класс-файл и прочитать его |
| `RUNTIME` | Аннотация всегда доступна через reflection *во время исполнения* | icon:check[] | icon:times[] |

#### Мини-эксперимент

```java
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.CLASS)
@interface RetainedInClass {
}

@Retention(RetentionPolicy.SOURCE)
@interface RetainedInSource {
}

@Retention(RetentionPolicy.RUNTIME)
@interface RetainedAtRuntime {
}

@RetainedInClass
@RetainedAtRuntime
@RetainedInSource
public class Annotations {
    public static void main(String[] args) {
        boolean retainedInClassIsVisible = Annotations.class.getAnnotation(RetainedInClass.class) != null;
        boolean retainedInSourceIsVisible = Annotations.class.getAnnotation(RetainedInSource.class) != null;
        boolean retainedAtRuntime = Annotations.class.getAnnotation(RetainedAtRuntime.class) != null;
        System.out.println("RetainedInClass is visible? " + retainedInClassIsVisible);
        System.out.println("RetainedInSource is visible? " + retainedInSourceIsVisible);
        System.out.println("RetainedAtRuntime is visible? " + retainedAtRuntime);
    }
}
```

* Аннотация `RetainedInClass` имеет `retention = CLASS`, `RetainedInSource` - `SOURCE`, `RetainedAtRuntime`
* Над классом `Annotations` висят все три
* В методе `main` производится попытка получить аннотации через reflection api (см. [`Class#getAnnotation`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Class.html#getAnnotation(java.lang.Class)))

Результат:

```
RetainedInClass is visible? false
RetainedInSource is visible? false
RetainedAtRuntime is visible? true
```

_То есть, как и ожидалось, во время исполнения видна только аннотация с `RetentionPolicy.RUNTIME`._

#### Что в `class` файле?

Для просмотра класс файла можно воспользоваться утилитой [`javap`](https://docs.oracle.com/en/java/javase/11/tools/javap.html), поставляемой вместе с jdk.

```shell
$ javap -v Annotations.class
```

* `-v` - выводить подробно
* `Annotations.class` - файл, полученный после компиляции

Часть вывода:

```text
RuntimeVisibleAnnotations:
  0: #32()
    ru.hse.annotations.RetainedAtRuntime
RuntimeInvisibleAnnotations:
  0: #34()
    ru.hse.annotations.RetainedInClass
```

`javap` говорит, что в `.class` файле есть:

* Аннотация, видимая во время исполнения, - `RetainedAtRuntime`
* Аннотация, которую во время исполнения не видно, это - `RetainedInClass`
* `RetainedInSource` не упоминается в контексте _использования_ аннотации в качестве аннотации

## Места использования

Аннотации могут быть использованы в совершенно различных местах.
Исчерпывающий список, как и всегда в подобных случаях, приводится в JLS - спецификации языка Java, [§ 9.6.4.1. `@Target`](https://docs.oracle.com/javase/specs/jls/se11/html/jls-9.html#jls-9.6.4.1):

1. Объявления модулей (`module`, `ElementType.MODULE`)
1. Объявление пакетов (`package`, `ElementType.PACKAGE`)
1. Объявления типов (`class`, `interface`, `enum`, аннотации. `ElementType.TYPE` + для аннотаций `ElementType.ANNOTATION_TYPE`)
1. Объявления методов (`ElementType.METHOD`)
1. Объявления конструкторов (`ElementType.CONSTRUCTOR`)
1. Объявления generic классов, интерфейсов, методов, конструкторов (`ElementType.TYPE_PARAMETER`)
1. Объявления полей (`ElementType.FIELD`)
1. Формальные параметры и параметры объявления исключений (TODO: что это?) (`ElementType.PARAMETER`)
1. Объявления локальных переменных (`ElementType.LOCAL_VARIABLE`)

### Пример использования аннотации для локальной переменной

```java
public class Main {
    public static void main(String[] args){
        List<String> stringList = Collections.emptyList();
        @SuppressWarnings({"RawTypeCanBeGeneric", "rawtypes"})
        List l = stringList;
    }
}
```

TIP: В стандартной библиотеке это соответствует перечислению [`ElementType`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/ElementType.html)

### [`@Target`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Target.html)

Аннотация `@Target` - одна из особых аннотаций языка.
Она позволяет ограничить, где именно аннотация, _аннотированная аннотацией `@Target`,_ может быть использована.

Пример:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELDS)
public @interface MakesSenseOnlyAtFields {
}
```

```java
public class User {
    @MakesSenseOnlyAtFields
    private final String username;

    public User(String username) {
        this.username = username;
    }
}
```

* `@Target` аннотации - `FIELDS`
* Она добавлена к полю, поэтому код успешно комплируется
* При попытке добавить её к классу, т.е.:

```java
@MakesSenseOnlyAtFields
public class User {
}
```

возникает ошибка компиляции:

```text
java: annotation type not applicable to this kind of declaration
```

[#targetDefaultValue]
### `target` по умолчанию

В случае, если `@Target` не будет задан, то будет использоваться значение по умолчанию - везде, кроме параметров типов и type contexts (TODO: что это?).

* Параметры типов - это про generic'и:

```java
Set<@Wow String> strings;
//  ^^^
```

* Type contexts - непонятно (TODO)

## Работа с аннотациями во время исполнения

Аннотации сами по себе чаще всего ничего не значат, их должен кто-то обрабатывать.

* _Это важно понимать, когда вы столкнётесь с `@Transactional` / `@Cacheable` или `@OneToMany`_
* _Аннотации, упомянутые до этого, - *особые*, т.к. они тесно связаны с самим языком_

Обработка аннотаций в общем случае зависит от логики, которую необходимо добавить по предоставленной с помощью аннотаций информации.

Это может выглядеть следующим образом:

1. Получить `Class<?>` объекта, который нужно обработать.
1. Прочитать аннотации.
1. Выполнить необходимую логику (Ниже будет пример).

Для чтения аннотаций через reflection можно использовать интерфейс [`AnnotatedElement`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/AnnotatedElement.html).

Его реализуют `Class`, `Constructor`, `Field`, `Method` и другие.

Пример чтения аннотаций со всех методов:

```java
package ru.hse.annotations;

import java.lang.reflect.Method;
import java.util.Arrays;
import java.util.List;
import java.util.Objects;
import java.util.stream.Collectors;

public class ScheduledScanner {
    @Scheduled(delayMillis = 100, rateMillis = 100) // 1
    public void scheduled1() {
    }

    @Scheduled(delayMillis = 200, rateMillis = 200) // 2
    public void scheduled2() {
    }

    @Scheduled(delayMillis = 300, rateMillis = 300) // 3
    public void scheduled3() {
    }

    private static List<Scheduled> getSchedules(Object o) { // 4
        Class<?> clazz = o.getClass(); // 5
        Method[] methods = clazz.getMethods(); // 6
        return Arrays.stream(methods)
                .map(method -> method.getAnnotation(Scheduled.class))// 7
                .filter(Objects::nonNull) // 8
                .collect(Collectors.toList());
    }

    public static void main(String[] args) {
        ScheduledScanner scheduledScanner = new ScheduledScanner();
        List<Scheduled> schedules = getSchedules(scheduledScanner);
        schedules.forEach(scheduled -> System.out.println("@Scheduled (delay = " // 9
                + scheduled.delayMillis() + ", rate = " + scheduled.delayMillis() + ")"));
    }
}
```

* Методы `1`, `2`, `3` помечены аннотацией `@Scheduled`
* `4`: метод принимает *`Object`* и возвращает список аннотаций
* `5`: `getClass` возвращает класс `o`
* `6`: возвращает все методы
* `7`: `method.getAnnotation` вернет аннотацию или `null`, если её нет
* `8`: если был метод без аннотации, то полученный `null` нужно пропустить
* `9`: вывод полученных значений

Вывод:

```
@Scheduled (delay = 100, rate = 100)
@Scheduled (delay = 200, rate = 200)
@Scheduled (delay = 300, rate = 300)
```

См. также: [Java Reflection API](https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/index.html)

## Наследование аннотаций и [`@Inherited`](https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/index.html)

Аннотация [`@Inherited`](https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/index.html) показывает, что она должна быть унаследована.
Т.е. при запросе аннотации у `class` 'а будут проверены все супер-классы

А интерфейсы?
TODO

Пример:

```java
@Retention(RetentionPolicy.RUNTIME)
@Inherited // 1
public @interface Persistable {
}

@Persistable // 2
public abstract class AbstractEntity {
}

public class Task extends AbstractEntity { // 3
}

public class InheritedDemo {
    public static void main(String[] args) {
        Persistable persistable = Task.class.getAnnotation(Persistable.class); // 4
        System.out.println(persistable);
    }
}
```

* `1`: аннотация помечена, как _наследуемая_
* `2`: `AbstractEntity` в свою очередь помечена, как `@Persistable`
* `3`: `Task extends AbstractEntity` без добавления аннотации
* `4`: Запрос аннотации через `Class#getAnnotation`

Вывод:

```text
@ru.hse.annotations.Persistable()
```

Без `@Inherited` вывод был следующим:

```text
null
```

## Повторяемые аннотации и [`@Repeatable`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Repeatable.html)

Иногда может быть полезно применить одну и ту же аннотацию несколько раз, например:

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface RunEveryDayAt {
    int hours() default 0;
}

public class Cron {
    @RunEveryDayAt(hours = 11)
    @RunEveryDayAt(hours = 23)
    public void compactSpace() {
    }
}
```

Однако, класс `Cron` просто не скомпилируется с ошибкой:

```text
java: ru.hse.annotations.RunEveryDayAt is not a repeatable annotation type
```

Чтобы это заработало необходимо:

1. объявить другую аннотацию, которая:
    1. имеет метод, который:
        1. возвращает массив _исходных_ аннотаций
        1. назван <<methodValue, `value`>>
    1. не имеет других методов без указания `default` значений
1. пометить исходную аннотацию как `@Repeatable` указав в ней аннотацию, полученную на предыдущем шаге

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface RunEveryDayAts {
    RunEveryDayAt[] value() default {}; // 1
}

@Retention(RetentionPolicy.RUNTIME)
@Repeatable(RunEveryDayAts.class) // 2
public @interface RunEveryDayAt {
    int hours() default 0;
}
```

* Аннотация `RunEveryDayAts` - аннотация, получанная на первом шаге
* `1`: Метод `value` возвращает массив _исходных_ аннотаций.
* `2`: Исходная аннотация помечена, как `Repeatable` с указанием контейнерной аннотации `RunEveryDayAts`

Тогда следующий код:

```java
public class Main {
    public static void main(String[] args) throws NoSuchMethodException, SecurityException {
        Method method = Cron.class.getMethod("compactSpace");
        System.out.println(Arrays.toString(method.getAnnotationsByType(RunEveryDayAt.class)));
    }
}
```

выведет:

```text
[@ru.hse.annotations.RunEveryDayAt(hours=11), @ru.hse.annotations.RunEveryDayAt(hours=23)]
```

**`getAnnotation(RunEveryDayAt.class)` в данном случае вернёт `null`**

## `@Documented`

Мета-аннотация, которая даёт _подсказку_ инструментам, генерирующим документацию, что использование аннотации с `@Documented` должно быть задокументировано.

См. также:

* [`@Documented` annotation in java](https://stackoverflow.com/q/5592703/6486622)

## `@Native`

Аннотация, сообщающая что _поле_ может быть использовано в `native` коде.

См. также:

* [Why would someone use `@Native` annotations?](https://softwareengineering.stackexchange.com/questions/218538/why-would-someone-use-native-annotations)

[#methodValue]
## Метод `value`

## Особенности синтаксиса

[#metaAnnotations]
## Мета-аннотации

Мета-аннотация - аннотация, которая ставится над другой аннотацией.

Все аннотации, у которых `target = ElementType.ANNOTATION_TYPE` - мета-аннотации.
TODO

Мета-аннотации из JDK:

* [`@Target`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Target.html)
* [`@Inherited`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Inherited.html)
