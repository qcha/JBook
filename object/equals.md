# java.lang.Object#equals

## Введение

В основном мы оперируем объектами, и логично было бы уметь как-то сравнивать их между собой.

В `Java` существует два вида сравнения:

* По ссылке
* По значению

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

Но чаще всего это не то, что требуется, так как обычно при сравнении необходимо, чтобы сравнивались не ссылки, а `состояния` объектов.
При этом, не всегда требуется, чтобы сравнивались все `свойства` объектов. Ведь логика сравнения может быть своя. Например, мы можем сказать, что две машины равны, если они одинаково стоят. Но при этом у машины есть еще другие свойства, такие как цвет, тип кузова и т.д.

Для этого и существует сравнение по значению, за которое отвечает метод `equals`.
Метод наследуется от класса `java.lang.Object`, а значит присутствует у каждого класса.

Данный метод сравнивает два объекта и возвращает `true` в случае, если объекты равны, в противном случае будет возвращено `false`.

По умолчанию объявление метода в `java.lang.Object` выглядит так:

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

Т.е по-умолчанию производится сравнение **по ссылке**.

Поэтому данный метод всегда переопределяют для классов, объекты которого необходимо уметь сравнивать **по состоянию**.

Так как же правильно определить `equals`?

## Переопределение

### Требования

Требования из описания из `JavaDoc`:

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
}
```

Итак, нам необходимо переопределить `equals`, напомним как выглядит сигнатура метода:

```java
@Override
public boolean equals(Object obj) {
    
}
```

В качестве аргумента метода передается `java.lang.Object`, это ссылочный тип, а значит может быть передан `null`.
Поэтому сначала необходимо проверить: а не является ли `obj` `null`-ом, чтобы обезопасить себя от `java.lang.NullPointerException`, если да, то возвращаем `false`.

После этого, хорошо бы сделать проверку, а не ссылается ли объект по ссылке `obj`, на наш текущий, на `this`?
Ведь если так, то объекты равны и никаких дальнейших сравнений делать не надо.

Напишем все вышесказанное:

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

Теперь ответим на вопрос: а может ли объект по ссылке `obj` другого класса? Может, ведь ссылка у нас `Object`!
А значит, надо проверить: мы сравниваем объекты одного класса?

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

В итоге после всего вышесказанного наш класс будет выглядеть так:

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
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Person person = (Person) o;

        if (age != person.age) return false;
        if (number != person.number) return false;
        if (Double.compare(person.salary, salary) != 0) return false;
        if (name != null ? !name.equals(person.name) : person.name != null) return false;
        return carKey != null ? carKey.equals(person.carKey) : person.carKey == null;
    }
  }
```

С `Java 7+` добавили вспомогательные классы и методы, используя которые можно переписать наш `equals` более коротко:

```java
@Override
public boolean equals(Object obj) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;

    Person person = (Person) o;

    if (age != person.age) return false;
    if (number != person.number) return false;
    if (Double.compare(person.salary, salary) != 0) return false;
    if (!Objects.equals(name, person.name)) return false;
    return Objects.equals(carKey, person.carKey);
}
```

Однако, в `Java` есть еще один способ проверить принадлежность к классу: через оператор `instanceOf`.

### getClass vs instanceOf

Как работает [getClass](./getClass.md)?
Если коротко - то метод возвращает класс объекта.

```java
class Person {
    // ...
}

Object p = new Person();
System.out.println("Class is: " + p.getClass()); // -> Class is: Person
```

Что делает оператор `instanceOf`?
Оператор проверяет то, является ли объект инстансом конкретного класса или одним из его родителей.

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

В чем разница?
При использовании `getClass` вы можете проверить **только** принадлежность к определенному классу и не более.
При использовании `instanceOf` вы можете проверить как принадлежность к классу, так и к родительским классам.

Когда что использовать?
Зависит от ответа на вопрос: будет ли ваш класс участвовать в сравнении с родительскими классами?

И как только появляется необходимость сравнения с родительскими классами, начинается проблема.
Почему?

#### Проблема

Итак, рассмотрим следующий пример:

```java
class A {
    int field1;
    
    A(int field1) {
      this.field1 = field1;
    }

    public boolean equals(Object other) {
      return (other != null && other instanceof A && ((A) other).field1 == field1);
    }
}

class B extends A {
    int field2;
    
    B(int field1, int field2) {
        super(field1);
        this.field2 = field2;
    }

    public boolean equals(Object other) {
        return (other != null && other instanceof B && ((B) other).field2 == field2 && super.equals(other));
    }
}    
```

На первый взгляд, вроде бы все верно.
Однако, такая реализация нарушает требование **симметричности**:

```java
A a = new A(1);
B b = new B(1, 1);

a.equals(b) == true;
b.equals(a) == false;
```

При этом, если переопределить метод с помощью `getClass`, как это мы сделали ранее, то нарушается [Liskov substitution principle](../oop/SOLID.md):

> Объекты могут быть заменены их наследниками без изменения свойств программы.

А это может привести к крайне неприятному поведению в `HashMap`-ах и прочих `hash`-структурах:

> The reason that I favor the instanceof approach is that when you use the getClass approach, you have the restriction that objects are only equal to other objects of the same class, the same run time type. 
> If you extend a class and add a couple of innocuous methods to it, then check to see whether some object of the subclass is equal to an object of the super class, even if the objects are equal in all important aspects, you will get the surprising answer that they aren't equal. 
> In fact, this violates a strict interpretation of the Liskov substitution principle, and can lead to very surprising behavior. 
> In Java, it's particularly important because most of the collections (HashTable, etc.) are based on the equals method. 
> If you put a member of the super class in a hash table as the key and then look it up using a subclass instance, you won't find it, because they are not equal.

В итоге получается палка о двух концах, поэтому выбор `getClass` или `instanceOf` надо делать под конкретную задачу, осознавая к чему может привести ваше решение.

Возможно, стоит вообще отказаться от наследования.

Но в некоторых случаях, `instanceOf` вполне можно использовать и не переживать о нарушениях.
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

Здесь использован `instanceOf`, так как если все элементы одного множества содержатся в другом - множества считаются равными, независимо от того, это `HashSet` или `TreeSet`.

Забегая вперед скажу, что, хоть иногда и необходим явно `instanceOf`, в подавляющем большинстве случаев вы будете использовать `getClass` и горя не знать.

## Ошибки

Обратите внимание, что тип аргумента в методе - это `java.lang.Object`:

```java
public boolean equals(Object obj)
```

Именно поэтому в реализации `equals` есть строки:

```java
    if (o == null || getClass() != o.getClass()) return false;

    Person that = (Person) obj; // тот самый 'каст' obj к Person
```

И часто у начинающих разработчиков возникает желание указать более конкретный аргумент в `equals`, 
чтобы избавиться от этой проверки и приведения к `Person`, поэтому некоторые пишут так:

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

Разумеется, делать так не стоит, если ваша цель `переопределить` метод. Потому что в случае выше вы не переопределили метод, вы его **перегрузили**.

В итоге у вас получается два метода: `equals(Object obj)` и `equals(Person obj)`.
Про второй метод знаете только вы, ведь он не относится к `java.lang.Object`, поэтому везде будет также использоваться реализация по умолчанию `equals(Object obj)`, например, в `hash`-таблицах, кроме ситуаций, где вы явно свой метод не вызываете.

Делать так не стоит, так как это вносит путаницу, работает такое сравнение только там, где вы его явно вызываете, а в коллекциях будет использоваться `equals(Object obj)`.
Поэтому надо `переопределять` такие методы, а не `перегружать` их.

Подробнее про [переопределение и перегрузку методов](../common/over-load-ride.md)

> При этом надо отметить, что аннотация `@Override` предотвратит от такой ошибки не дав скомпилировать код, поэтому, когда переопределяете методы не забывайте ее ставить.

## Массивы и equals

У массивов не переопределен `equals` и выполняется сравнение ссылок.

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

Решением является использовать статический метод для сравнения массивов: `Arrays.equals(...)`.

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

## Заключение

Для сравнения объектов по значению необходимо переопределять метод `equals`, при этом выполняя требования к методу: рефлективность, симметричность, транзитивность, согласованность.
Всегда с `equals` переопределяйте еще и `hashCode`.

* Помните о разнице `instanceOf` и `getClass` и выбирайте использование под задачу.
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
> 
> Существуют и сторонние библиотеки, помогающие в вычислении `equals`, например [apache commons](https://commons.apache.org/).


## Полезные ссылки

1. [Java equals() and hashCode() Contracts](https://www.baeldung.com/java-equals-hashcode-contracts)
2. [Java. Методы equals и hashCode.](https://www.youtube.com/watch?v=lWnzRILIEZ0)
3. [Что выбрать: getClass vs instanceOf](https://stackoverflow.com/questions/4989818/instanceof-vs-getclass)
4. [Liskov substitution principle vs Symmetric in equals](https://stackoverflow.com/questions/27581/what-issues-should-be-considered-when-overriding-equals-and-hashcode-in-java/32223#32223)