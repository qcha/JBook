# java.lang.Object#equals

## Введение

В основном мы оперируем объектами, и логично было бы уметь как-то сравниватьэти объекты между собой.

В `Java` существует два вида сравнения:

* Сравнение по ссылке
* Сравнение по значению

Сравнение по ссылке происходит тогда, когда вы используете оператор `==`.

Для демонстрации давайте посмотрим на следующий пример:

```java
public class Test {
    public static void main(String[] args) {
        Car my = new Car();
        Car your = new Car();
        Car link = my;

        System.out.println(my == your);
        System.out.println(my == link);
    }
}

class Car {}
```

Ссылки `my` и `your` указывают на разные объекты, в то время как `link` ссылается на тот же объект, на который ссылается `my`.

Результат выполнения кода:

```java
false
true
```

Из примера выше видно, что `==` сравнивает не свойства объектов, не состояние, а ссылки на объекты. Такой способ сравнения подходит, если мы сравниваем уникальные объекты, существующие в одном экземпляре.

Но чаще всего это не то, что нужно, так как обычно при сравнении необходимо, чтобы сравнивались не ссылки, а `состояния` объектов.
При этом, не всегда требуется, чтобы сравнивались все `свойства` объектов. Ведь логика сравнения всегда своя. Например, мы можем сказать, что две машины равны, если они одинаково стоят. Но при этом у машины есть еще другие свойства, такие как цвет, тип кузова и т.д.

Для этого и существует сравнение по значению, за которое отвечает метод `equals`.

Данный метод сравнивает два объекта и возвращает `true` в случае, если объекты равны, в противном случае будет возвращено `false`.

По умолчанию реализация этого метода выглядит так:

```java
public boolean equals(Object obj) {
        return (this == obj);
    }
```

Т.е по умолчанию данный метод производит сравнение по ссылке.

Поэтому данный метод всегда переопределяют для классов, объекты которого необходимо уметь сравнивать по состоянию.

Так как же правильно определить `equals`?

## Переопределяем equals

Напомним, что объявление метода в `java.lang.Object` выглядит так:

```java
    /**
     * Indicates whether some other object is "equal to" this one.
     * <p>
     * The {@code equals} method implements an equivalence relation
     * on non-null object references:
     * <ul>
     * <li>It is <i>reflexive</i>: for any non-null reference value
     *     {@code x}, {@code x.equals(x)} should return
     *     {@code true}.
     * <li>It is <i>symmetric</i>: for any non-null reference values
     *     {@code x} and {@code y}, {@code x.equals(y)}
     *     should return {@code true} if and only if
     *     {@code y.equals(x)} returns {@code true}.
     * <li>It is <i>transitive</i>: for any non-null reference values
     *     {@code x}, {@code y}, and {@code z}, if
     *     {@code x.equals(y)} returns {@code true} and
     *     {@code y.equals(z)} returns {@code true}, then
     *     {@code x.equals(z)} should return {@code true}.
     * <li>It is <i>consistent</i>: for any non-null reference values
     *     {@code x} and {@code y}, multiple invocations of
     *     {@code x.equals(y)} consistently return {@code true}
     *     or consistently return {@code false}, provided no
     *     information used in {@code equals} comparisons on the
     *     objects is modified.
     * <li>For any non-null reference value {@code x},
     *     {@code x.equals(null)} should return {@code false}.
     * </ul>
     * <p>
     * The {@code equals} method for class {@code Object} implements
     * the most discriminating possible equivalence relation on objects;
     * that is, for any non-null reference values {@code x} and
     * {@code y}, this method returns {@code true} if and only
     * if {@code x} and {@code y} refer to the same object
     * ({@code x == y} has the value {@code true}).
     * <p>
     * Note that it is generally necessary to override the {@code hashCode}
     * method whenever this method is overridden, so as to maintain the
     * general contract for the {@code hashCode} method, which states
     * that equal objects must have equal hash codes.
     *
     * @param   obj   the reference object with which to compare.
     * @return  {@code true} if this object is the same as the obj
     *          argument; {@code false} otherwise.
     * @see     #hashCode()
     * @see     java.util.HashMap
     */
    public boolean equals(Object obj) {
        return (this == obj);
    }
```

### Требования к equals

Требования из описания:

* Рефлективность
    
    Для любой ссылки на значение `х` выражение `х.equals(x)` должно возвращать `true`.
    
* Симметричность 
  
  Для любых ссылок на значения `х` и `у` выражение `х.equals(y)` должно возвращать `tгue` тогда и только тогда, когда `y.equals(x)` возвращает `true`.
  
* Транзитивность
    Для любых ссылок на значения `х`, `у` и `z`, если `x.equals(y)` возвращает `true` и `y.equals(z)` возвращает `true`, то и выражение `х.equals(z)` должно возвращать `true`.

* Непротиворечивость или Согласованность
    
    Для любых ссылок на значения `х` и `у`, если несколько раз вызвать `х.equals(y)`, постоянно будет возвращаться значение `true`, либо постоянно будет возвращаться значение `false` при условии, что никакая информация, используемая при сравнении объектов, не поменялась.

* Для любой ненулевой ссылки на значение `х` выражение `х.equals(null)` должно возвращать `false`.

* При переопределении `equals` необходимо переопределить и [hashCode](./hashcode.md).

### Как переопределить equals

* Проверить на то, является ли `obj` `null`-ом, чтобы обезопасить себя от `java.lang.NullPointerException`, если да, то возвращаем `false`.

* Проверить на то, является ли `obj` ссылкой на указанный объект, если да, то возвращаем `true`.

* Проверить на то, что объект имеет тот же тип с помощью `instanceOf`, если нет, то возвращаем `false`.
    
    Оператор `instanceOf` используется тогда, когда требуется проверить, к какому классу принадлежит объект. Это булев оператор, и выражение `foo instanceof Foo` истинно, если объект `foo` принадлежит классу `Foo` или **его наследнику** или реализует интерфейс `Foo`.

* После того, как мы убедились что `obj` не `null`, что это объект того же класса, с которым мы хотим его сравнивать, необходимо привести `obj` к одному типу для дальнейшего сравнения. Проще говоря, мы должны 'скастовать' к проверяемому типу `obj`. Ниже будет показано как это сделать.

* Проверить необходимые поля объектов на равенство и возвращаем `true` в случае полного равенства объектов, иначе - `false`.

Пример приведем с помощью многострадального класса `Person`:

```java
public class Person {
    private int age;
    private int number;
    private double salary;
    private String name;
    private CarKey carKey;

    public Person(int age, int number, String name, double salary, CarKey carKey) {
        this.age = age;
        this.number = number;
        this.name = name;
        this.salary = salary;
        this.carKey = carKey;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj == null)
            return false;

        if (obj == this)
            return true;

        if (!(obj instanceof Person))
            return false;

        Person that = (Person) obj;  // тот самый 'каст' obj к Person

        if (that.age != this.age ||
                !that.name.equals(this.name) ||
                that.number != this.number ||  //need to check for NPE
                (Double.compare(that.salary, this.salary) != 0) ||
                that.carKey.equals(carKey)) //need to check for NPE
              {
            return false;
        }

        return true;
    }
  }
```

С `Java 7+` добавили вспомогательные классы и методы, используя которые можно переписать наш `equals` более коротко:

```java
@Override
public boolean equals(Object obj) {
    if (obj == null)
        return false;

    if (obj == this)
        return true;

    if (!(obj instanceof Person))
        return false;

    Person that = (Person) obj; // тот самый 'каст' obj к Person

    return Objects.equals(age, that.age) && Objects.equals(number, that.number) &&
           Objects.equals(salary, that.salary) && Objects.equals(name, that.name) &&
           Objects.equals(carKey, that.carKey) && Objects.equals(carKey, that.carKey);
    }

    return true;
}
```

## Ошибки

Обратите внимание, что тип аргумента в методе - это `java.lang.Object`:

```java
public boolean equals(Object obj)
```

Именно поэтому в реализации `equals` есть строки:

```java
    if (!(obj instanceof Person))
        return false;

    Person that = (Person) obj; // тот самый 'каст' obj к Person
```

Поэтому часто у начинающих разработчиков возникает желание указать более конкретный аргумент в `equals`, чтобы избавиться от этой проверки и приведения к `Person`, поэтому некоторые пишут так:

```java
    public boolean equals(Person obj) {
        if (obj == null)
            return false;

        if (obj == this)
            return true;

        if (that.age != this.age ||
                !that.name.equals(this.name) ||
                that.number != this.number ||  //need to check for NPE
                (Double.compare(that.salary, this.salary) != 0) ||
                that.carKey.equals(carKey)) //need to check for NPE
              {
            return false;
        }

        return true;
    }
```

Разумеется, делать так не стоит, если ваша цель `переопределить` метод. Потому что в случае выше, вы не переопределили метод, вы его **перегрузили**.

В итоге у вас получается два метода: `equals(Object obj)` и `equals(Person obj)`. Про второй метод знаете только вы, ведь он не относится к `java.lang.Object`, поэтому везде будет также использоваться реализация по умолчанию `equals(Object obj)`, например, в хештаблицах, кроме ситуаций, где вы явно свой метод не вызываете. Делать так не стоит, так как это вносит путаницу, работает такое сравнение только там, где вы его явно вызываете, а в коллекциях будет для сравнения использоваться `equals(Object obj)`. Поэтому надо `переопределять` такие методы, а не `перегружать` их.

Подробнее про [переопределение и перегрузку методов](../common/over-load-ride.md)

> При этом надо отметить, что аннотация `@Override` предотвратит от такой ошибки не дав скомпилировать код, поэтому, когда переопределяете методы не забывайте ее ставить.

## Заключение

* При переопределении `hashCode` всегда переопределяйте `equals` и наоборот.

* При работе с `float` и `double` помните о том, что существуют `Float.NaN`, поэтому для сравнения таких типов используйте специальные методы `Float.compare` и `Double.compare`.

* Для простых полей, кроме `double` и `float` используйте обычное сравнение через `==`.

* Не забывайте, что очередность сравнения влияет на производительность, поэтому сначала сравниваем поля, которые чаще других могут быть различны.

* Не забывайте, что метод принимает `java.lang.Object`, поэтому изменение сигнатуры метода - это не переопределение и о таком методе будете знать только вы.

    Объявление `equals` в виде
    ```java
    public boolean equals(MyClass obj) {
    // some logic
    }
    ```
    Является ошибкой, так как этот метод **не переопределяет**(override) `equals` у `java.lang.Object`, а **перегружает**(overload) его.

    Подробнее про [переопределение и перегрузку методов](../common/over-load-ride.md)

> Помните, что большинство IDE сейчас легко сгенерируют вам `equals`, чтобы вы не писали его вручную.
>
> Также, существуют сторонние проекты, которые берут кодогенерацию на себя, например, проект [lombok](https://projectlombok.org/).
