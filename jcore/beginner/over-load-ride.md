# Override и Overload

## Введение

Говоря про наследование мы всегда говорим в том числе и про наследование методов родительского класса, его поведения.

```java
class Person {
    public void greeting() {
        System.out.println("Person greeting");
    }
}

class Employee extends Person {
}

public class Main {
    public static void main(String[] args) {
        Person p = new Person();
        Employee e = new Employee();

        p.greeting();
        e.greeting();
    }

}
```

Однако не всегда то поведение, что мы унаследовали от родительского класса подходит: в примере выше, я думаю, логичнее было бы, чтобы Employee имел свое собственное приветствие.

Т.е. нам необходимо переопределить поведение метода.

## Override

Переопределение - это по сути замена реализации метода из родительского класса.

Реализация метода в подклассе переопределяет (заменяет) его реализацию в суперклассе, описывая метод с тем же названием, что и у метода суперкласса, а также у нового метода подкласса должны быть те же параметры или сигнатура, тип возвращаемого результата, что и у метода родительского класса.

Сигнатура метода — это имя метода плюс параметры (причем порядок параметров имеет значение).

Грубо говоря, мы 'подменяем' унаследованный метод другим, с 'виду' точно таким же, как унаследованный, но реализованным по другому.

Теперь, исправим пример выше:

```java
class Person {
    public void greeting() {
        System.out.println("Person greeting");
    }
}

class Employee extends Person {
    public void greeting() {
        System.out.println("Employee greeting");
    }
}

public class Main {
    public static void main(String[] args) {
        Person p = new Person();
        Employee e = new Employee();

        p.greeting();
        e.greeting();
    }
}
```

### Требования

* У переопределенного метода должны быть те же аргументы, что и у метода родителя.

* У переопределенного метода должен быть тот же тип возвращаемого значения, что и у метода родителя.

* Модификатор доступа у переопределенного метода может расширять область видимости, но не сужать его.

### @Override

Аннотация `@Override` существует для того, чтобы подсветить сам факт переопределния.

Объявление аннотации:

```java
/**
 * Indicates that a method declaration is intended to override a
 * method declaration in a supertype. If a method is annotated with
 * this annotation type compilers are required to generate an error
 * message unless at least one of the following conditions hold:
 *
 * <ul><li>
 * The method does override or implement a method declared in a
 * supertype.
 * </li><li>
 * The method has a signature that is override-equivalent to that of
 * any public method declared in {@linkplain Object}.
 * </li></ul>
 *
 * @author  Peter von der Ah&eacute;
 * @author  Joshua Bloch
 * @jls 8.4.8 Inheritance, Overriding, and Hiding
 * @jls 9.4.1 Inheritance and Overriding
 * @jls 9.6.4.4 @Override
 * @since 1.5
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

Можно не вдаваться сейчас в подробности объявления, нас интересует именно `JavaDoc`, а именно следующие строки:

```text
If a method is annotated with this annotation type compilers are required to generate an error message unless at least one of the following conditions hold:

* The method does override or implement a method declared in a supertype.
* The method has a signature that is override-equivalent to that of any public method declared in {@linkplain Object}.
```

Что означает, что компилятор должен сгенерировать ошибку, если хотя бы одно из условий не будет выполнено:

* Метод переопределяет или реализует метод, объявленный в супертипе (в родительском классе).

* Метод имеет сигнатуру, которая переопределяет эквивалент любого открытого метода, объявленного в `java.lang.Object`.

Методы, объявленные в `java.lang.Object` подробнее рассматриваются [здесь](../object/intro.md).

Будет ли работать ваш код без этой аннотации? Да, будет, что демонстрирует пример выше.

Но ставить аннотацию при переопределении методов необходимо, так как это подсказка как для компилятора, так и для разработчика.

Почему это подсказка для компилятора?

Компилятор, видя данную аннотацию у метода, понимает, что это именно переопределение метода и значит сигнатуру метода можно найти у родительского класса или интерфейса, проверив ее:

```java
class Employee {
    // Ошибка компиляции - Method does not override method from its superclass
    @Override
    public void hello() {
        
    }
}
```

Почему это подсказка для разработчика?

Если разработчик, переопределяя метод, случайно изменит сигнатуру, то данная аннотация подсветит ему эту ошибку:

```java
class Employee extends Person {
    // Ошибка компиляции - Method does not override method from its superclass
    @Override
    public void greeting(String greeting) {
        System.out.println("Employee greeting");
    }
}
```

### Область видимости

Важно знать, что при переопределении допускаются расширять область видимости, но не сужать:

```java
class Person {
    protected void greeting() {
        System.out.println("Person greeting");
    }
}

class Employee extends Person {
    @Override
    public void greeting() {
        System.out.println("Employee greeting");
    }
}
```

Здесь при переопределении мы расширили область видимости метода с `protected` до `public`.

А вот сузить уже нельзя:

```java
class Person {
    public void greeting() {
        System.out.println("Person greeting");
    }
}

class Employee extends Person {
    // Ошибка компиляции 'greeting()' in 'Employee' clashes with 'greeting()' in 'Person'; attempting to assign weaker access privileges ('protected'); was 'public'
    @Override
    protected void greeting() {
         System.out.println("Employee greeting");
    }
}
```

### Исключения

Важно отметить, что выбрасываемые из метода исключения, как `unchecked`, так и `checked`, указанные через `throws`, не являются частью сигнатуры метода, а значит не накладывают никаких требований на переопределение:

```java
class Person {
    public void greeting() throws Exception {
        System.out.println("Person greeting");
    }
}

class Employee extends Person {
    @Override
    public void greeting() {
        System.out.println("Employee greeting");
    }
}
```

Несмотря на то, что родительский метод явно декларирует `throws Exception`, переопределенный метод у `Employee` убирает это объявление.

## Overload

Мы рассмотрели, что у переопределенного метода должна быть та же сигнатура, что и у метода родителя, иначе это не считается переопределением.

Ситуация, когда объявляется несколько методов с одним и тем же именем, но отличающихся по параметрам, типам параметров и т.д. называется перегрузкой метода или `overload`:

```java
public class Main {
    public static void main(String[] args) {
        Person p = new Person();
        Employee e = new Employee();

        p.greeting();
        e.greeting("Hello");
    }

}

class Person {
    public void greeting() {
        System.out.println("Person greeting");
    }
}

class Employee extends Person {
    public void greeting(String greeting) {
        System.out.println(greeting);
    }
}
```

В таком случае говорится, что метод `greeting` перегружен.

Как раз от ошибочных ситуация с перегрузкой и призвана защитить аннотация `@Override`.

Здесь необходимо добавить, что имена параметров не имеют значения как в ситуации с перегрузкой, так и с переопределением.

Следующее объявление ошибочно:

```java
public double sum(double x1, double x2) {
    return x1 + x2;
}
 
public double sum(double y1, double y2) {
    return y1 + y2;
}
```

С точки зрения компилятора объявлено два метода с одинаковой сигнатурой, что является ошибкой.

## Статические методы

В `Java` также существует возможность сделать метод статическим, т.е. принадлежащим классу, а не объекту класса.

Об этом речь идет [тут](./static_java.md).

Итак, давайте посмотрим на то, можно ли переопределять или перегружать статические методы.

Может ли статический метод быть перегужен?

Да, конечно, ровно как и обычный метод:

```java
class Person {
    public static void greeting() {
        System.out.println("Person greeting");
    }

    public static void greeting(String greeting) {
        System.out.println("Person greeting: " + greeting);
    }
}
```

Может ли статический метод быть переопределен?

Нет!

Несмотря на то, что мы можем создать и у родителя, и у потомка метод, который удовлетворяет всем требованиям переопределения, метод не будет переопределен, он будет скрыт!

Подробнее об этом разговор идет [здесь](./static_java.md#hiding)

## Заключение

Язык программирования `Java` предоставляет возможность как переопределять методы, так и перегружать их.

Переопределение накладывает ряд требований: метод должен быть с тем же названием, что и у метода суперкласса, а также у нового метода подкласса должны быть те же параметры или сигнатура, тип возвращаемого результата, что и у метода родительского класса.

В свою очередь, методы, имеющие одинаковые имена, но разный набор параметров и их типов, являются перегруженными.

Для того, чтобы явно показать то, что метод переопределен используйте аннотацию `@Override`.
