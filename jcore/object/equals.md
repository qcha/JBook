# java.lang.Object#equals

- [java.lang.Object#equals](#javalangobjectequals)
    - [Введение](#введение)
    - [Переопределение](#переопределение)
        - [Требования](#требования)
        - [Пример переопределения equals](#пример-переопределения-equals)
        - [Наследование и equals](#наследование-и-equals)
        - [getClass vs instanceOf](#getclass-vs-instanceof)
    - [Частые ошибки](#частые-ошибки)
        - [Нарушение согласованности](#нарушение-согласованности)
        - [Overload vs Override](#overload-vs-override)
        - [Массивы](#массивы)
        - [NaN и Infinity](#nan-и-infinity)
    - [Заключение](#заключение)
    - [Полезные ссылки](#полезные-ссылки)

## Введение

При операциях над объектами логично иметь инструмент для сравнения их между собой.

В `Java` существует два вида сравнения:

* По ссылке

    В этом случае каждый экземпляр класса будет равен только самому себе.

* По значению

    В этом случае каждый экземпляр класса будет равен только при равенстве полей.

Для сравнения по ссылке существует оператор `==`.

Рассмотрим следующий пример:

```java
public class Test {
    public static void main(String[] args) {
        Car my = new Car(20);
        Car your = new Car(40);
        Car link = my;

        System.out.println(my == your);
        System.out.println(my == link);
    }
}

class Car {
    private int price;

    public Car(int price) {
        this.price = price;
    }
}
```

Ссылки `my` и `your` указывают на разные объекты (каждый из которых породили через `new`), в то время как `link` ссылается на тот же объект, на который ссылается `my`.

Результат выполнения кода:

```java
false
true
```

Из примера выше видно, что `==` сравнивает не состояние объекта, а ссылки.

Но чаще всего в задачах необходимо сравнивать именно состояние объектов.

При этом не всегда требуется, чтобы сравнение происходило по абсолютно всем полям объектов, все зависит от логики сравнения. Например, мы можем сказать, что две машины равны, если они одинаково стоят. Но при этом у машины есть еще другие свойства: такие как цвет, тип кузова и т.д.

Для этого и существует сравнение по значению за которое отвечает метод `equals`.

Метод наследуется от класса `java.lang.Object`, а значит присутствует у каждого класса.

Данный метод сравнивает два объекта и возвращает `true` в случае если объекты равны, иначе будет возвращено `false`.

Объявление метода выглядит как:

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

По-умолчанию, как видим, сравнение производится **по ссылке**.

И поэтому данный метод обычно переопределяют.

Но как правильно переопределить `equals` и какие подводные камни могут здесь быть? Давайте разбираться.

## Переопределение

Для начала из `JavaDoc` соберем требования по контракту метода.

### Требования

Итак, метод должен выполнять следующие требования:

* Рефлексивность

    Для любой ненулевой ссылки на `х` выражение `х.equals(x)` должно возвращать `true`.

* Симметричность
  
    Для любых ненулевых ссылок на `х` и `у` выражение `х.equals(y)` должно возвращать `tгue` тогда и только тогда, когда `y.equals(x)` возвращает `true`.
  
* Транзитивность

    Для любых ссылок на `х`, `у` и `z` выполняется условие: если `x.equals(y)` возвращает `true` и `y.equals(z)` возвращает `true`, то и выражение `х.equals(z)` должно возвращать `true`.

* Непротиворечивость или Согласованность

    Для любых ссылок на `х` и `у` вызов `х.equals(y)` несколько раз, при условии, что никакая информация, используемая при сравнении объектов, не поменялась, то будет возвращаться одно и то же значение: либо `true`, либо `false`.

* Для любой ненулевой ссылки `х` выражение `х.equals(null)` должно возвращать `false`.

* При переопределении `equals` необходимо переопределить и [hashCode](./hashcode.md).

### Пример переопределения equals

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

    // ...
}
```

Давайте попробуем переопределить `equals` для этого класса.

Еще раз посмотрим на сигнатуру метода:

```java
@Override
public boolean equals(Object obj) {
    
}
```

В качестве аргумента метода передается `java.lang.Object` - это ссылочный тип, а значит может быть передан `null`.

Поэтому для начала необходимо проверить: а не является ли переданная ссылка `null`-ом, и если да, то возвращаем `false`.

Далее необходимо сделать проверку на то, что не ссылается ли объект по ссылке `obj` на себя самого? По сути - проверить рефлексивность.

Добавим эти проверки:

```java
@Override
public boolean equals(Object obj) {
        if (obj == null) {
            return false;
        }

        if (obj == this) {
            return true;
        }

        // to be continued
}
```

Теперь ответим на вопрос: а может ли объект по ссылке `obj` быть объектом другого класса? Может.

Значит надо проверить: одного ли типа (класса) объекты мы сравниваем?

Для этого существует метод `getClass`.

Как работает [getClass](./get_class.md)?
Если коротко - то метод возвращает класс объекта.

```java
class Person {
    // ...
}

Object p = new Person();
System.out.println("Class is: " + p.getClass()); // -> Class is: Person
```

Итак, добавим необходимые проверки:

```java
@Override
public boolean equals(Object obj) {
        if (obj == null) {
            return false;
        }

        if (obj == this) {
            return true;
        }

        if (getClass() != o.getClass()) {
            return false;
        }

        // to be continued
}
```

После этого ссылку `obj` можно безопасно скастовать к нашему классу и сделать сравнение полей:

```java
    @Override
    public boolean equals(Object obj) {
        if (obj == null) {
            return false;
        }

        if (obj == this) {
            return true;
        }

        if (getClass() != o.getClass()) {
            return false;
        }

        Person that = (Person) obj;  // тот самый 'каст' obj к Person

        if (age != person.age) return false;
        if (number != person.number) return false;
        if (Double.compare(person.salary, salary) != 0) return false;
        if (name != null ? !name.equals(person.name) : person.name != null) return false;

        return carKey != null ? carKey.equals(person.carKey) : person.carKey == null;
    }
```

### Наследование и equals

В нашем примере выше был единственный класс, без наследников и родителей, поэтому переопределить `equals` не являлось проблемой.

Но что если добавить наследников?

Для этого давайте рассмотрим следующий пример:

```java
public class Point {
    private int x;
    private int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (o == null || getClass() != o.getClass()) return false;

        Point point = (Point) o;

        return x == point.x && Objects.equals(y, point.y);
    }
}
```

Теперь создадим класс-наследник, подкласс привносит немного информации, оказывающей влияние на процедуру сравнения:

```java
public class CounteredPoint extends Point {
    private  static AtomicInteger counter = new AtomicInteger();

    public CounteredPoint(int x, int y) {
        super(x, y);
        counter.incrementAndGet();
    }

    @Override
    public boolean equals(Object o) {
        return super.equals(o);
    }
}
```

В обоих случаях переопределен метод `equals`, как это уже делали выше. У класса `CounteredPoint` переопределение вызывает родительский метод, так как новых свойств он не добавляет для сравнения.

Предположим, что мы хотим написать метод определяющий, является ли точка целого числа частью единичной окружности:

```java
public class Main {
    private static final List<Point> unitCircle;
    static {
        unitCircle = new ArrayList<>();
        unitCircle.add(new Point( 1, 0));
        unitCircle.add(new Point(0, 1));
        unitCircle.add(new Point(-1, 0));
        unitCircle.add(new Point( 0, -1));
    }

    public static boolean onUnitCircle(Point p) {
        return unitCircle.contains(p);
    }

    public static void main(String[] args) {
        Point point = new Point(1, 0);
        CounteredPoint counteredPoint = new CounteredPoint(1, 0);

        System.out.println(onUnitCircle(point));
        System.out.println(onUnitCircle(counteredPoint));
    }
}
```

Запустим наш код и посмотрим вывод:

```java
true
false
```

Несмотря на то, что обе точки по сути своей находятся в списке `unitCircle`, но counteredPoint была не найдена.

Получается, что текущая реализация нарушает [Liskov substitution principle](../../patterns/SOLID.md#liskov-substitution-principle):

> Объекты могут быть заменены их наследниками без изменения свойств программы.

Для того, чтобы этот принцип выполнялся нам необходимо 'научить' наш `equals` работать с потомками `Point`.

Для того, чтобы проверить является ли объект инстансом конкретного класса или одним из его родителей существует оператор
`instanceOf`:

```java
class Person {
    // ...
}

class Student extends Person {
    // ...
}

Person p = new Person();
Student ps = new Student();

System.out.println(ps instanceof Student); // -> true
System.out.println(ps instanceof Person); // -> true
```

В чем разница с `getClass`?

При использовании `getClass` вы можете проверить **только** принадлежность к определенному классу и не более.
При использовании `instanceOf` вы можете проверить как принадлежность к определенному классу, так и к родительским классам.

Давайте переопределим `equals` с учетом вводных:

```java
class public class Point {
    // ...

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point point)) return false;

        return x == point.x && y == point.y;
    }
}
```

Снова запустим наш код в `Main`:

```java
true
true
```

Ура! Кажется, что вот оно - решение.

Не совсем.

Теперь создадим класс-наследник, который привносит немного информации, оказывающей влияние на процедуру сравнения:

```java
public class ColorPoint extends Point {

    private final String color;

    public ColorPoint(int x, int y, String color) {
        super(x, y);
        this.color = color;
    }
}
```

Здесь нам уже необходимо учесть уже цвет:

```java
@Override
public final boolean equals(Object o) {
    if (!(o instanceof ColorPoint that)) {
        return false;
    }

    if (!super.equals(o)) {
        return false;
    }

    return Objects.equals(color, that.color);
}
```

Теперь запустим следующий код:

```java
    public static void main(String[] args) {
        Point point = new Point(1, 0);
        ColorPoint colorPoint = new ColorPoint(1, 0, "Green");

        System.out.println(point.equals(colorPoint));
        System.out.println(colorPoint.equals(point));
    }
```

Получаем:

```java
true
false
```

Получаем в итоге нарушение требования симметричности.

Не подходит.

Попробуем учесть эти 'смешанные' сравнения явно:

```java
    @Override
    public final boolean equals(Object o) {
        if (!(o instanceof Point)) {
            return false;
        }

        // если это обычная точка
        if (!(o instanceof ColorPoint that)) {
            return super.equals(o);
        }

        // если это цветная точка, то учитываем цвет
        return super.equals(o) && Objects.equals(color, that.color);
    }
```

Снова запускаем наш проверочный код:

```java
true
true
```

Ура? К сожалению, нет, так как нарушается требование транзитивности:

```java
    public static void main(String[] args) {
        Point point = new Point(1, 0);
        ColorPoint colorPoint = new ColorPoint(1, 0, "Green");
        ColorPoint colorPoint2 = new ColorPoint(1, 0, "Blue");

        System.out.println(point.equals(colorPoint));
        System.out.println(point.equals(colorPoint2));
        System.out.println(colorPoint.equals(colorPoint2));
    }
```

Вывод:

```java
true
true
false
```

Вывод: Не существует способа расширить класс, добавив к нему новое поле (участвующее в сравнении), сохранив при этом соглашения для метода `equals`.

Например, `java.sql.Timestamp` является подклассом класса `java.util.Date` и добавляет поле для наносекунд. Реализация метода
`equals` в `Timestamp` нарушает правило симметрии, и это может привести к странному поведению программы, если объекты Timestamp и Date использовать в одной коллекции.

В документации к классу `Timestamp` есть предупреждение, предостерегающее от смешивания объектов классов `Date` и `Timestamp`. Такое поведение не является правильным и подражать ему не надо.

### getClass vs instanceOf

Так что делать и когда использовать getClass, а когда instanceOf?

Выбор `getClass` или `instanceOf` в качестве способа для проверки принадлежности класса необходимо делать под конкретную задачу, осознавая к чему может привести ваше решение.

Если ваш класс будет участвовать в наследовании, то рассмтаривать надо вариант с `instanceOf`, так как `getClass` может привести к крайне неприятному поведению в `HashMap`-ах и прочих `hash`-структурах:

> The reason that I favor the instanceof approach is that when you use the getClass approach, you have the restriction that objects are only equal to other objects of the same class, the same run time type.
>
> If you extend a class and add a couple of innocuous methods to it, then check to see whether some object of the subclass is equal to an object of the super class, even if the objects are equal in all important aspects, you will get the surprising answer that they aren't equal.
>
> In fact, this violates a strict interpretation of the Liskov substitution principle, and can lead to very surprising behavior.
>
> In Java, it's particularly important because most of the collections (HashTable, etc.) are based on the equals method.
> If you put a member of the super class in a hash table as the key and then look it up using a subclass instance, you won't find it, because they are not equal.

Например, посмотрите как определен `equals` у `HashSet`:

```java
    /**
     * Compares the specified object with this set for equality.  Returns
     * <tt>true</tt> if the given object is also a set, the two sets have
     * the same size, and every member of the given set is contained in
     * this set.  This ensures that the <tt>equals</tt> method works
     * properly across different implementations of the <tt>Set</tt>
     * interface.<p>
     *
     * This implementation first checks if the specified object is this
     * set; if so it returns <tt>true</tt>.  Then, it checks if the
     * specified object is a set whose size is identical to the size of
     * this set; if not, it returns false.  If so, it returns
     * <tt>containsAll((Collection) o)</tt>.
     *
     * @param o object to be compared for equality with this set
     * @return <tt>true</tt> if the specified object is equal to this set
     */
    public boolean equals(Object o) {
        if (o == this)
            return true;

        if (!(o instanceof Set))
            return false;
        Collection<?> c = (Collection<?>) o;
        if (c.size() != size())
            return false;
        try {
            return containsAll(c);
        } catch (ClassCastException unused)   {
            return false;
        } catch (NullPointerException unused) {
            return false;
        }
    }
```

Здесь использован `instanceOf`, так как если все элементы одного множества содержатся в другом, то множества считаются равными, независимо от того, что это за множества: `HashSet` или `TreeSet`.

С другой стороны, неаккуратное использование `instanceOf` может привести к проблемам с симметричностью.

Поэтому для ответа на вопрос 'Что использовать?' необходмио понимать как и где вы будете использовать ваш класс.

А лучше вообще отказаться от наследования в пользу композиции.

Гораздо более правильнее было бы поступить так, что объявить класс в виде:

```java
public class ColorPoint {
    private final String color;
    private final Point point;

    public ColorPoint(int x, int y, String color) {
        point = new Point(x, y);
        this.color = color;
    }

    public Point asPoint() {
        return point;
    }

    @Override
    public boolean equals(Object o) {
        if (o == null || getClass() != o.getClass()) return false;

        ColorPoint that = (ColorPoint) o;
        return color.equals(that.color) && point.equals(that.point);
    }
}
```

Для логики работы с обычной точкой сделать метод, возвращающий точку, для работы с цветными точками использовать свое сравнение и не смешивающая классы.

## Частые ошибки

### Нарушение согласованности

Важно помнить требование согласованности.
Грубо говоря, оно гласит: если два объекта равны, то они должны быть равны все время, пока один из них (или оба) не будет изменен.

Не делайте в методе `equals` сравнение по непостоянным полям: данным, которые могут измениться без воздействия на объект (например по `ip` адресам).

Иначе это может привести к трудноуловимым проблемам.

### Overload vs Override

Обратите внимание, что тип аргумента в методе - это `java.lang.Object`:

```java
public boolean equals(Object obj)
```

Именно поэтому в реализации `equals` есть строки:

```java
    if (o == null || getClass() != o.getClass()) return false;

    Person that = (Person) obj; // тот самый 'каст' obj к Person
```

И часто у начинающих разработчиков возникает желание указать более конкретный аргумент в `equals`, чтобы избавиться от этой проверки и приведения к `Person`, поэтому некоторые пишут так:

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

Такое делать **не надо**! Потому что в случае выше вы не переопределили (Override) метод, а **перегрузили** (Overload).

В итоге получается два метода: `equals(Object obj)` и `equals(Person obj)`.

Про второй метод знаете только вы, поэтому во многих местах (например, в `hash`-таблицах), завязанных на контракт с `equals(Object obj)` будет использоваться реализация по умолчанию и это приведет к трудноуловимым ошибкам.

Поэтому надо переопределять такие методы, а не перегружать их.

Подробнее про [переопределение и перегрузку методов](./over-load-ride.md).

При этом надо отметить, что аннотация `@Override` предотвратит от такой ошибки и не даст скомпилировать код. Поэтому, когда переопределяете методы, не забывайте ставить эту аннотацию.

### Массивы

Массивы в `Java` - это объекты, а значит у них тоже есть `equals`.

Но у массивов этот метод не переопределен и выполняется сравнение ссылок.

```java
        int[] arr = {1, 2, 3, 4, 5};
        int[] arr2 = {1, 2, 3, 4, 5};
        System.out.println(arr.equals(arr2));
        System.out.println(arr == arr2);
```

Результат:

```java
false
false
```

Решением является использование статического метода из стандартной библиотеки для сравнения массивов: `java.util.Arrays.equals(...)`:

```java
        int[] arr = {1, 2, 3, 4, 5};
        System.out.println(Arrays.equals(arr));

        arr[0] = 100;
        System.out.println(Arrays.equals(arr));
```

Результат:

```java
true
false
```

### NaN и Infinity

Помните, что при работе с `java.lang.Float` и `java.lang.Double` существуют понятия `NaN` и `Infinity`:

```java
    /**
     * A constant holding the positive infinity of type
     * {@code float}. It is equal to the value returned by
     * {@code Float.intBitsToFloat(0x7f800000)}.
     */
    public static final float POSITIVE_INFINITY = 1.0f / 0.0f;

    /**
     * A constant holding the negative infinity of type
     * {@code float}. It is equal to the value returned by
     * {@code Float.intBitsToFloat(0xff800000)}.
     */
    public static final float NEGATIVE_INFINITY = -1.0f / 0.0f;

    /**
     * A constant holding a Not-a-Number (NaN) value of type
     * {@code float}.  It is equivalent to the value returned by
     * {@code Float.intBitsToFloat(0x7fc00000)}.
     */
    public static final float NaN = 0.0f / 0.0f;

```

Чтобы правильно обработать сравнение с типами данных `java.lang.Float` и `java.lang.Double` используйте `Float.compare` и `Double.compare`.

## Заключение

Для сравнения объектов по значению необходимо переопределять метод `equals`, при этом выполняя требования к методу: рефлексивности, симметричности, транзитивности и согласованности.

* Помните о разнице `instanceOf` и `getClass`, а также о подводных камнях при выборе того или иного способа проверки

* Учитывайте в сравнении только значимые поля

* После определения проверьте соответствие контракту и требованиям

* При работе с `float` и `double` помните о том, что существуют `NaN`, поэтому для сравнения таких типов используйте специальные методы `Float.compare` и `Double.compare`

* Для простых полей, кроме `double` и `float` используйте обычное сравнение через `==`

* Не забывайте, что очередность сравнения влияет на производительность, поэтому сначала сравниваем поля, которые чаще других могут быть различны

* Не забывайте, что метод принимает `java.lang.Object`, поэтому изменение сигнатуры метода - это не переопределение, а перегрузка

    Объявление `equals` в виде

    ```java
    public boolean equals(MyClass obj) {
        // some logic
    }
    ```

    Является ошибкой.

    Подробнее про [переопределение и перегрузку методов](./over-load-ride.md).

Всегда с `equals` переопределяйте еще и [hashCode](./hashcode.md).

Помните, что большинство IDE сейчас легко сгенерируют вам `equals`, чтобы вы не писали его вручную.

Также, существуют сторонние проекты, которые берут кодогенерацию на себя, например, проект [lombok](https://projectlombok.org/).
Существуют и сторонние библиотеки, помогающие в вычислении `equals`, например [apache commons](https://commons.apache.org/).

## Полезные ссылки

1. [Java equals() and hashCode() Contracts](https://www.baeldung.com/java-equals-hashcode-contracts)
2. [Java. Методы equals и hashCode.](https://www.youtube.com/watch?v=lWnzRILIEZ0)
3. [Что выбрать: getClass vs instanceOf](https://stackoverflow.com/questions/4989818/instanceof-vs-getclass)
4. [Liskov substitution principle vs Symmetric in equals](https://stackoverflow.com/questions/27581/what-issues-should-be-considered-when-overriding-equals-and-hashcode-in-java/32223#32223)
5. [Effective Java 2nd Edition, Item 8: Obey the general contract when override equals](https://www.amazon.com/dp/0321356683)
