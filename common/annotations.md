# Аннотации

- [Аннотации](#Аннотации)
    - [Введение](#Введение)
    - [Объявление аннотаций](#Объявление-аннотаций)
    - [Аннотации - это особые интерфейсы](#Аннотации---это-особые-интерфейсы)
    - [Методы аннотаций](#Методы-аннотаций)
        - [Значения по умолчанию](#Значения-по-умолчанию)
        - [Возвращаемые значения методов](#Возвращаемые-значения-методов)
            - [Типы](#Типы)
            - [Ограничения на значения](#Ограничения-на-значения)
    - [Мета-аннотации, тесно связанные с языком Java](#Мета-аннотации-тесно-связанные-с-языком-java)
        - [`@Retention` и доступность аннотаций](#retention-и-доступность-аннотаций)
        - [Сравнение RetentionPolicy](#Сравнение-retentionpolicy)
            - [Мини-эксперимент](#Мини-эксперимент)
            - [Что в `class` файле?](#Что-в-class-файле)
        - [Места использования и `@Target`](#Места-использования-и-target)
        - [Наследование аннотаций и `@Inherited`](#Наследование-аннотаций-и-inherited)
        - [Повторяемые аннотации и `@Repeatable`](#Повторяемые-аннотации-и-repeatable)
    - [Работа с аннотациями во время исполнения](#Работа-с-аннотациями-во-время-исполнения)
    - [Заключение](#Заключение)
    - [Полезные ссылки](#Полезные-ссылки)


## Введение

**Аннотации** в Java предназначены для добавления метаинформации в код.

В качестве примера, рассмотрим аннотацию, с которой, наверняка, сталкивался каждый Java-разработчик - [`@Override`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Override.html).


Она сообщает компилятору, что происходит [переопределение](over-load-ride.md) метода супер-класса или интерфейса.
Её отсутствие не вызовет ошибки компиляции, однако её присутствие у метода, не переопределяющего метод предка, ошибку компиляции _всё-таки вызовет_. Пример:

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

_Разработчик ошибся и в качестве типа параметра `equals` указал `Person`, хотя [`equals`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html#equals(java.lang.Object)) принимает `Object`._

---

**Вопрос**:

Где ещё аннотации могут быть полезны?

**Ответ**:

Аннотации могут выступать некоторым промежуточным звеном между определённым _инструментом, библиотекой или фреймворком_ и **прикладным разработчиком**:
* С одной стороны: программист **декларативно** указывает, что он хочет получить в месте использования аннотации.
* С другой: обработчик считывает примененную аннотацию и выполняет что-то со своей стороны.
  
Это может быть:

* _Статическая проверка кода_ на этапе _компиляции_, подобная `@Override`.
* _Проверка данных_ с помощью [Bean Validation](https://beanvalidation.org/):
    ```java
    public class User {
        @Pattern("[a-z]{6,30}")
        private String username;
    }
    ```
    * Библиотека проверит соответствие имени пользователя указанному регулярному выражению.
* Декларативное _кэширование_ с помощью [`@Cacheable`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/annotation/Cacheable.html).
* Явное указание имени _json-свойства_ с помощью [`@JsonProperty`](https://stackoverflow.com/questions/12583638/when-is-the-jsonproperty-property-used-and-what-is-it-used-for) из Jackson.

---

## Объявление аннотаций

Для объявления аннотаций используется комбинация `@` и ключевого слова `interface` (а в Kotlin для этого есть ключевое слово - [`annotation`](https://kotlinlang.org/docs/reference/annotations.html)):

```java
public @interface Wow {
}
```
В примере выше объявлена аннотация `Wow`, использоваться она может примерно так:

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

Здесь проаннотированы и класс, и поле, и конструктор, и его параметр.

_Если доводить использование аннотаций до абсурда, то может получиться [что-то такое](https://twitter.com/lukaseder/status/711612663202238464):_

![Передозировка аннотациями](../images/annotation_overdose.png)

## Аннотации - это особые интерфейсы

Об этом написано в [§ 9.6 JLS - спецификации языка Java](https://docs.oracle.com/javase/specs/jls/se11/html/jls-9.html#jls-9.6):

> An annotation type declaration specifies a new annotation type, **a special kind of interface type.**

То есть, как ни странно, их можно реализовывать, используя ключевое слово `implements`:

```java
public class DefaultFoo implements Wow {
    @Override
    public Class<? extends Annotation> annotationType() {
        return Wow.class;
    }
}
```

_при этом, компилятор попросит переопределить `annotationType` - метод из интерфейса [`Annotation`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Annotation.html), возвращающий тип аннотации (в данном случае необходимо вернуть `Wow.class`)._

---

**Вопрос**:

Имеет ли смысл _реализовывать (`implements`)_ аннотацию?

**Ответ**:

Скорее **нет**, чем да.

Intellij IDEA имеет инспекцию "Class extends annotation interface":

> Reports any classes declared as implementing or extending an annotation interface. While it is legal to extend an annotation interface, it is often done by accident, and the result won't be usable as an annotation.

Реализовывать аннотации разрешено, но часто это происходит по ошибке, и IDEA ругается на это:

![Intellij IDEA ругается на класс, реализующий аннотацию Wow](../images/class_implements_annotation.png)

---

**Вопрос**:

Если аннотация - интерфейс, то кто её реализует?

**Ответ**:

В общем случае это должно быть не так важно, но если вдаться в детали реализации OpenJDK, то это делается динамически с помощью класса [`Proxy`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/Proxy.html):

```java
@Wow
public class Main {
    @Retention(RetentionPolicy.RUNTIME)
    public @interface Wow {
    }

    public static void main(String[] args) {
        Wow wow = Main.class.getAnnotation(Wow.class);
        System.out.println(wow.getClass());
    }
}
```

Вывод:

```text
class com.sun.proxy.$Proxy1
```

---

## Методы аннотаций

У аннотаций могут быть _методы_.

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

У метода может быть значение _по умолчанию_, чтобы его задать используется ключевое слово `default` (оно же может быть использовано для определения тела метода по умолчанию в **интерфейсе**, начиная с _Java 8_):

```java
public @interface Scheduled {
    int delayMillis();

    int rateMillis() default 1000;
}
```

Тогда `rateMillis` указывать будет **необязательно**.
_Вместо этого будет использоваться значение по умолчанию - `1000`_:

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

_Если попробовать посмотреть на это с другой стороны, то аннотации добавляются к элементам класса, а *не* к их экземплярам_

---

**Вопрос**:

Аннотации **неизменяемы**, однако их _методы могут возвращать массивы_, которые, в свою очередь, **изменяемы**.
Что будет, если такой массив изменить?

**Ответ**:

Изменится _возвращенный массив_.
При этом _последующий вызов_ такого метода вернёт массив, в точности соответствующий исходному.
Реализация для этого [производит **копирование**](https://stackoverflow.com/questions/53436794/how-do-annotations-prevent-mutations-of-an-array-parameter/53436966#53436966).

---

## Мета-аннотации, тесно связанные с языком Java

Аннотация, находящаяся над аннотацией, называется **мета-аннотацией**.
Некоторые мета-аннотации особенно важны, т.к. они позволяют контролировать некоторые аспекты применения _аннотаций_.

Рассмотрим их чуть подробнее.

### `@Retention` и доступность аннотаций

Обработка аннотаций может происходить:

1. На этапе **компиляции**. Так делают: [Dagger](https://dagger.dev/), [Micronaut](https://micronaut.io/)
1. **Во время исполнения**.
Так делают: [Jackson](https://github.com/FasterXML/jackson), [Gson](https://github.com/google/gson), [Retrofit](https://square.github.io/retrofit/)

В связи с этим различием введено понятие **retention**.
Он определяет, _на каком этапе аннотация будет отброшена_.
Всего предусмотрено три таких этапа: `SOURCE`, `CLASS`, `RUNTIME`, они перечислены в [`RetentionPolicy`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/RetentionPolicy.html).

### Сравнение RetentionPolicy


| RetentionPolicy | Описание | Остается в класс-файле | Доступна во время исполнения|
|-----------------|----------|------------------------|-----------------------------|
| `SOURCE` | Отбрасываются после компиляции | Нет | Нет |
| `CLASS` | Отбрасывается на этапе _загрузки класса_ | Да | _Условно: если найти класс-файл и прочитать его_ |
| `RUNTIME` | Аннотация всегда доступна через reflection *во время исполнения* | Да | Да |

**Значение по умолчанию - `RetentionPolicy.CLASS`**

#### Мини-эксперимент

```java
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
* Над классом `Annotations` находятся все три
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
    ru.misc.annotations.RetainedAtRuntime
RuntimeInvisibleAnnotations:
  0: #34()
    ru.misc.annotations.RetainedInClass
```

`javap` говорит, что в `.class` файле есть:

* Аннотация, видимая во время исполнения, - `RetainedAtRuntime`
* Аннотация, которую во время исполнения не видно, это - `RetainedInClass`
* `RetainedInSource` не упоминается в контексте _использования_ аннотации в качестве аннотации

### Места использования и `@Target`

Аннотация [`@Target`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Target.html) - позволяет конкретно указать, где именно аннотация, _аннотированная аннотацией `@Target`,_ может быть использована.

Допустим, у нас есть аннотация, которую имеет смысл ставить только над полями - `@MakesSenseOnlyAtFields`:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface MakesSenseOnlyAtFields {
}
```

Указав `@Target(ElementType.FIELD)`, мы сможем ограничить её использование, так чтобы следующий код проходил компиляцию:

```java
public class User {
    @MakesSenseOnlyAtFields
    private final String username;

    public User(String username) {
        this.username = username;
    }
}
```

А при попытке добавить её к классу:

```java
@MakesSenseOnlyAtFields
public class User {
}
```

Мы получали ошибку:

```text
java: annotation type not applicable to this kind of declaration
```

Исчерпывающий список мест применения, как и всегда в подобных случаях, приводится в JLS - спецификации языка Java, [§ 9.6.4.1](https://docs.oracle.com/javase/specs/jls/se11/html/jls-9.html#jls-9.6.4.1). Наиболее распространенные из них - объявления:

1. Типов (`class`, `interface`, `enum`, аннотации. `ElementType.TYPE`)
1. Методов (`ElementType.METHOD`)
1. Конструкторов (`ElementType.CONSTRUCTOR`)
1. Полей (`ElementType.FIELD`)

Это соответствует перечислению [`ElementType`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/ElementType.html)

---

**Вопрос:**

Можно ли одновременно указать несколько `@Target`'ов?

**Ответ:**

Да, конечно: `value` у `@Target` принимает массив `ElementType`. Пример:

```java
@Target({ElementType.METHOD, ElementType.FIELD})
@interface Lax {
}

public class Main {
    @Lax
    private int bar;
    
    @Lax
    public void foo() {
    }
}
```

---

### Наследование аннотаций и `@Inherited`

Аннотация [`@Inherited`](https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/index.html) показывает, что она должна быть **унаследована**.
Т.е. при запросе аннотации у `class`'а будут проверены все супер-классы

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

* `1`: аннотация помечена как _наследуемая_
* `2`: `AbstractEntity` в свою очередь помечена как `@Persistable`
* `3`: `Task extends AbstractEntity` без добавления аннотации
* `4`: Запрос аннотации через `Class#getAnnotation`

Вывод:

```text
@ru.misc.annotations.Persistable()
```

Вывод без `@Inherited`:

```text
null
```

---

**Вопрос**:

А интерфейсы? Что будет, если такую аннотацию поставить над ним?

**Ответ**:

Она будет проигнорирована:

```java
@Persistable
interface Identifiable {
}

public abstract class AbstractEntity implements Identifiable {
}

public class Main {
    public static void main(String[] args) {
        Persistable persistable = AbstractEntity.class.getAnnotation(Persistable.class); // 4
        System.out.println(persistable);
    }
}
```

Вывод:

```text
null
```

[Javadoc `@Inherited`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Inherited.html) сообщает, что в этом нет смысла. Аннотация наследуется только от **супер-класса**:

> Note that this meta-annotation type has **no effect** if the annotated type is used to annotate **anything other than a class**. Note also that this meta-annotation only causes annotations to be inherited from superclasses; annotations on implemented interfaces have no effect.

---

### Повторяемые аннотации и `@Repeatable`

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

Однако, при компиляции класса `Cron` возникнет ошибка:

```text
java: ru.misc.annotations.RunEveryDayAt is not a repeatable annotation type
```

Чтобы это заработало необходимо:

1. объявить другую аннотацию, которая:
    1. имеет метод, который:
        1. возвращает массив _исходных_ аннотаций
        1. назван `value`
    1. не имеет других методов без указания `default` значений
1. пометить исходную аннотацию как [`@Repeatable`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/annotation/Repeatable.html) указав в ней аннотацию, полученную на предыдущем шаге

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

* Аннотация `RunEveryDayAts` - аннотация, полученная на первом шаге
* `1`: Метод `value` возвращает массив _исходных_ аннотаций.
* `2`: Исходная аннотация помечена как `Repeatable` с указанием контейнерной аннотации `RunEveryDayAts`

Тогда следующая программа:

```java
public class Main {
    public static void main(String[] args) throws NoSuchMethodException, SecurityException {
        Method method = Cron.class.getMethod("compactSpace");
        System.out.println(Arrays.toString(method.getAnnotationsByType(RunEveryDayAt.class)));
    }
}
```

Выведет:

```text
[@ru.misc.annotations.RunEveryDayAt(hours=11), @ru.misc.annotations.RunEveryDayAt(hours=23)]
```

**`getAnnotation(RunEveryDayAt.class)` в данном случае вернёт `null`**

## Работа с аннотациями во время исполнения

Аннотации сами по себе чаще всего ничего не значат, их должен кто-то обрабатывать.

* _Это важно понимать, когда вы столкнётесь с `@Transactional` / `@Cacheable` или `@OneToMany`_
* _Аннотации `@Target`, `@Retention` и подобные - *особые*, т.к. они тесно связаны с самим языком_

Обработка аннотаций в общем случае зависит от логики, которую необходимо добавить по предоставленной с помощью аннотаций информации.

Это может выглядеть следующим образом:

1. Получить `Class<?>` объекта, который нужно обработать.
1. Прочитать аннотации.
1. Выполнить необходимую логику

Для чтения аннотаций через reflection можно использовать методы интерфейса [`AnnotatedElement`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/AnnotatedElement.html).

Его реализуют `Class`, `Constructor`, `Field`, `Method` и другие.

Пример чтения аннотаций, находящихся над методами:

```java
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

_Вместо вывода может быть добавлена любая другая логика, например: отправка выбранных методов на исполнение в [`ScheduledThreadPoolExecutor`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ScheduledThreadPoolExecutor.html)._

См. также:
* [Java Reflection API](https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/index.html)

## Заключение

* Аннотация - _элемент языка_, позволяющий в определенном формате добавить **мета-информацию** в код.
* _Виртуальная машина_ и _язык Java_ предоставляют широкие возможности по работе с ними.
* Аннотации могут быть обработаны как на этапе **компиляции**, так и на во время **исполнения**.

## Полезные ссылки

* [Awesome Annotation Processing](https://github.com/gunnarmorling/awesome-annotation-processing) - агрегатор ссылок по теме Annotation Processing API - программного интерфейса обработки аннотаций на этапе компиляции.
* [Lesson: Annotations](https://docs.oracle.com/javase/tutorial/java/annotations/) - руководство по аннотациям от Oracle.
* [Lombok](https://projectlombok.org/) - Annotation Processor, способный избавить Java-разработчика от написания геттеров, сеттеров, `equals`'ов и `hashCode`'ов.
* [How does lombok work?](https://stackoverflow.com/questions/6107197/how-does-lombok-work) - вопрос на StackOverflow, где рассказывается, как работает Lombok.