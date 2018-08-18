## java.lang.Object#equals

### Введение

В `Java` существует два вида сравнения объектов:

* Сравнение по ссылке
* Сравнение по значению

Сравнение по ссылке происходит тогда, когда вы используете оператор `==`.

Для демонстарции давайте посмотрим на следующий пример:

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

Ссылки `my` и `your` - указывают на разные объект, в то время как `link` - указывает на тот же объект, на который ссылается `my`.

Поэтому результат вызова этого кода будет:

```java
false
true
```

Однако чаще всего необходимо сравнение именно по значению, поэтому и ввели метод `equals`.

Данный метод сравнивает два объекта и возвращает `true` в случае их равенства, а `false` в случае различия.

По умолчанию реализация этого метода выглядит так:

```java
public boolean equals(Object obj) {
        return (this == obj);
    }
```

Т.е по умолчанию данный метод проверяет на то, что проверяемая ссылка ссылается на этот же объект - происходит проверка равенства по ссылке.

Именно поэтому данный метод советуют переопределять - иначе это будет то же самое, что и сравнение по ссылке.

Важно понимать, что так как в `Java` мы оперируем не только объектами, но и примитивными типами данных, такими как `int`, `long` и т.д, то сравнивать примитивные типы данных нужно через `==`.
Что на самом деле логично, учитывая как минимум то, что у примитивных типов данных нет никакого `equals` метода.


Так как же правильно определить `equals`?

### Переопределяем equals

Всопмним объявление метода в `java.lang.Object`:

```java
public boolean equals(Object obj);
```

И определим основные правила:

* Проверяем на то, является ли `obj` `null`-ом, чтобы обезопасить себя от `NullPointerException`, если да - то возвращаем `false`.

* Проверяем на то, что является ли `obj` ссылкой на указанный объект, если да - возвращаем `true`.

* Проверяем на то, что объект имеет тот же тип с помощью `instanceOf`, если нет - возвращаем `false`.

* После того, как мы убедились что `obj` - это не `null`, что это объект того же класса, с которым мы хотим его сравнивать - мы должны привести `obj` к одному типу для дальнейшего сравнения. Проще говоря, мы должны 'скастовать' к проверяемому типу `obj`. Ниже будет показано как это сделать.

* Проверяем необходимые поля объектов на равенство, если все равно - то `true`, иначе  -`false`.

Пример приведем с помощью многострадального `Person`:

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

        Person that = (Person) obj;

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

С приходом `Java` версии 7 и выше добавили вспомогательные методы, используя которые можно переписать наш `equals` более коротко:

```java
@Override
public boolean equals(Object obj) {
    if (obj == null)
        return false;

    if (obj == this)
        return true;

    if (!(obj instanceof Person))
        return false;

    Person that = (Person) obj;

    return Objects.equals(age, that.age) && Objects.equals(number, that.number) &&
            Objects.equals(salary, that.salary) && Objects.equals(name, that.name) &&
            Objects.equals(carKey, that.carKey) && Objects.equals(carKey, that.carKey);
    }

    return true;
}
```

### Требования к equals

После этого проверим метод на транзитивность, симметричность и противоречивость, написав тесты.

// todo об этом

### Заключение

* При переопределении `hashCode` всегда переопределяйте `equals` и наоборот.

* При работе с `flaot` и `double` помните о том, что существуют `Float.NaN`, поэтому для сравнения таких типво используйте специальные методы `Float.compare` и `Double.compare`.

* Для простых полей, кроме `double` и `float` используйте обычное сравнение через `==`.

* Не забывайте, что очередность сравнения влияет на производительность, поэтому сначала сравниваем поля, которые чаще других могут быть различны.

* Не забывайте, что метод принимает `java.lang.Object`, поэтому изменение сигнатуры метода - это не переопределение и о таком методе будете знать только вы.

    Объявление `equals` в виде
    ```java
    public boolean equals(MyClass obj) {
    // some logic
    }
    ```
    Является ошибкой, так как этот метод *не переопределяет*(override) `equals` у `java.lang.Object`, а *перегружает*(overload) его.

    Подробнее о override/overload: // статья о overload/override

> Помните, что большинство IDE сейчас легко сгенерируют вам `equals`, чтобы вы не писали его вручную.
>
> Также, существуют сторонние проекты, которые берут кодогенерацию на себя, например, проект [lombok](https://projectlombok.org/).
