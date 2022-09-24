# Паттерн Builder

## Введение

Представим ситуацию, в которой нам надо работать с классом, содержащим большое количество параметров.
Нашей задачей является присвоение объекту некоторого состояния наиболее удобным и безопасным путем.

Давайте разберем: какие возможные варианты есть для работы с такими классами в `Java`?

Для примера будем оперировать некоторой сущностью: класс `Telephone` содержит серийный номер, имя телефона, марку и т.д.

С первым способом мы уже давно знакомы: это работа с конструктором класса (иногда его сокращенно называют `ctor`).

## Telescoping constructor pattern

В данном подходе мы объявляем конструкторы, в которые передаем значения и присваиваем их полям класса:

```java
public class Telephone {
    private int serialnumber;
    private String name;
    private String screenName;
    private String mark;

    public Telephone(int serialnumber, String name, String screenName, String mark) {
        this.serialnumber = serialnumber;
        this.name = name;
        this.screenName = screenName;
        this.mark = mark;
    }
}
```

Подход прост, потому красив, но хорош только в ситуациях, когда количество параметров, передаваемых в конструктор, не превышает разумные пределы.
Обычно считается, что более **пяти** параметров это уже перебор.

Однако, в ситуациях, когда передаваемых аргументов много данный подход **не рекомендуется**.

Почему?

Так как мы должны передать большое количество аргументов, то код становится тяжело читать, сложнее разбираться в передаваемых параметрах, становится легче ошибиться и передать не тот параметр и т.д.

Еще одной ситуацией, когда подобный подход может оказаться неудобным в использовании - это несколько конструкторов у класса.
Представьте, что одно или несколько полей могут быть не заданы, в таком случае появляются еще конструкторы:

```java
public class Telephone {
    private int serialnumber;
    private String name;
    private String screenName;
    private String mark;

    public Telephone(int serialnumber, String name, String screenName, String mark) {
        this.serialnumber = serialnumber;
        this.name = name;
        this.screenName = screenName;
        this.mark = mark;
    }

    public Telephone(int serialnumber, String screenName, String mark) {
        this(serialnumber, "DEFAULT_NAME", screenName, mark);
    }
}
```

А если вам не повезло и под это никто не создал второй конструктор, то вам придется пользоваться одним большим конструктором и самому заполнять необязательные поля значениями по-умолчанию:

А что если может быть необязательным еще какое-то поле? В таком случае снова приходится брать и писать еще один конструктор!

И это хорошо, если программист сможет все эти конструкторы вызвать через `this`, чтобы не создать совсем уж хаос, когда у вас экран текса, где почти вся логика - это конструкции вида `this.field = field`:

```java
    public Telephone(int serialnumber, String name, String screenName, String mark) {
        this.serialnumber = serialnumber;
        this.name = name;
        this.screenName = screenName;
        this.mark = mark;
    }

    public Telephone(int serialnumber, String screenName, String mark) {
        this.serialnumber = serialnumber;
        this.name = "DEFAULT";
        this.screenName = screenName;
        this.mark = mark;
    }

    public Telephone(int serialnumber, String screenName) {
        this.serialnumber = serialnumber;
        this.name = "DEFAULT_FOR_MARK";
        this.screenName = screenName;
        this.mark = "DEFAULT_MARK";
    }
```

И это на простом примере с четырьмя свойствами и тремя значениями по-умолчанию!

Как видите, минусов более чем достаточно, чтобы задуматься о более простом и безопасном способе.

## Java Beans pattern

Как выше описано, конструкторы в большом количестве неудобны, особенно, когда приходиться работать с большим количеством параметров для создания объекта.

Одним из решений, что называется, в лоб, может быть переложить ответственность на `set`-ры.

Этот способ основывается на том, что сначала создается объект (с помощью пустого конструктора или конструктора, принимающего самые необходимые и обязательные параметры), а уже после этого происходит *наполнение* уже созданного объекта данными с помощью `set`-ов:

```java
public class Telephone {
    private int serialnumber;
    private String name;
    private String screenName;
    private String mark;

    public void setSerialnumber(int serialnumber) {
        this.serialnumber = serialnumber;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setScreenName(String screenName) {
        this.screenName = screenName;
    }

    public void setMark(String mark) {
        this.mark = mark;
    }
}
```

Пример использования:

```java
Telephone telephone = new Telephone();
telephone.setSerialnumber(1123);
telephone.setName("Samsung");
telephone.setScreenName("Screen Good");
telephone.setMark("Wow");
```

Проблем с количеством конструкторов, множеством параметров и прочего тут нет, уже неплохо.

Но у подхода есть огромный минус.
Из-за разделения вызовов наш объект, будет некоторое время находиться в неустойчивом состоянии! Ведь пока эти все `set`-ры выполняются объект находится в неправильном, незавершенном состоянии.

В худшем случае возможна ситуация, когда вы забудете вызвать `set` на важном поле, так у вас нет никаких гарантий и проверок, что вызваны все `set`-ры на нужные поля!

Добавим сюда еще и то, что такой объект не может быть неизменным, у него не может быть `final` полей.

В итоге получаем, что нужно искать другое решение проблемы.

## Builder Pattern

Как же соединить два эти подхода, но так, чтобы мы обходили недостатки каждого стороной?

Ответ прост, нам нужен новый класс, который будет собирать компоненты, по мере их поступления, а в конце уже строить объект.

Чтобы программист, будто в супермаркете с тележкой шел, накидывая ингридиенты, а после уже, когда все собрано, получал необходимый объект.
В качестве этой тележки и будет некоторый класс, который назовем `Builder`.

Итак, создаем внутренний класс, часто называемый `Builder`, через которого выставляем параметры, а после делаем `build()`. Т.е пока мы не вызвали `build` - мы работаем с промежуточным объектом.

Благодаря этому подходу мы можем конструировать наш объект в нужном порядке, более тонкий контроль над процессом конструирования, изолирует код, реализующий конструирование и представление.

Можно по разному сделать `Builder`:

### Реализация 1

```java
public class Telephone {
    private int serialnumber;
    private String name;
    private String screenName;
    private String mark;

    private Telephone() {

    }

    public class Builder {

        private Builder() {
        }

        public Builder name(String name) {
            Telephone.this.name = name;

            return this;
        }

        public Builder screenName(String screenName) {
            Telephone.this.screenName = screenName;

            return this;
        }

        public Builder mark(String mark) {
            Telephone.this.mark = mark;
            return this;

        }

        public Builder serialnumber(int serialnumber) {
            Telephone.this.serialnumber = serialnumber;

            return this;
        }

        public Telephone build() {
            return Telephone.this;
        }
    }

    public static Builder builder() {
        return new Telephone().new Builder();
    }
}
```

### Реализация 2

```java
public class Telephone {
    private final int serialnumber;
    private final String name;
    private final String screenName;
    private final String mark;

    private Telephone(int serialnumber, String name, String screenName, String mark) {
        this.serialnumber = serialnumber;
        this.name = name;
        this.screenName = screenName;
        this.mark = mark;
    }

    public static class Builder {
        private int serialnumber;
        private String name;
        private String screenName;
        private String mark;

        private Builder() {
        }

        public Builder name(String name) {
            this.name = name;

            return this;
        }

        public Builder screenName(String screenName) {
            this.screenName = screenName;

            return this;
        }

        public Builder mark(String mark) {
            this.mark = mark;
            return this;

        }

        public Builder serialnumber(int serialnumber) {
            this.serialnumber = serialnumber;

            return this;
        }

        public Telephone build() {
            return new Telephone(serialnumber, name, screenName, mark);
        }
    }

    public static Builder builder() {
        return new Builder();
    }
}
```

Примеры использования каждого подхода:

```java
public static void main(String[] args) {
    // example of telescoping pattern
    Telephone telephone1 = new Telephone(1, "Sony", "TFT", "X");

    // example of java beans pattern
    Telephone telephone2 = new Telephone();
    telephone2.setSerialnumber(2);
    telephone2.setName("Sony");
    telephone2.setScreenName("TFT");
    telephone2.setMark("X");

    //example of builder
    Telephone telephone3 = Telephone.builder()
                                    .name("Sony")
                                    .serialnumber(3)
                                    .mark("X")
                                    .screenName("TFT")
                                    .build();
}
```

Читать и работать с таким `Builder` гораздо проще, чем при использовании `Telescoping`, а также безопаснее, чем при использовании `Java Beans`.

## Подробнее про Builder

Кто-то может сказать, что в других языках программирования это решается через именованные аргументы.
Но, надо отметить, что именованные аргументы — лишь частный (и очень простой случай) `Builder`-а.

Сила паттерна раскрывается не в простых, а в более сложных случаях: когда у нас действительно сложный объект, с множеством полей, каждое из которых также представляет собой сложный объект, когда одни поля влияют на то, как будут инициализироваться другие, например наличие одного аргумента приводит к тому, что становится невозможно задать (или наоборот, возможно только в этом случае) другие аргументы, т.е. на выходе метода билдера — какой-то класс, разный, с разным набором полей и т.д. И вот это и есть применение паттерна.

В таком случае, у нас может быть вообще несколько реализаций нашего `Builder`-а! А где несколько реализаций - дело пахнет интерфейсом. А раз у нас объект сложный, то и *порядок* вызова конструирующих методов может быть у каждой реализации `Builder`-а быть разным, за порядок вызова конструирующих методов тоже должен кто-то отвечать, некий супервизор.

Поэтому давайте разберем паттерн с этого ракурса и выделим следующие важные компоненты:

1.`Product` (Продукт) - создаваемый объект, это класс, который определяет наш сложный объект, который мы пытаемся собрать.
2.`Builder` (Строитель) - это абстрактный класс/интерфейс стро­и­те­ля, который определяет этапы/шаги кон­стру­и­ро­ва­ния про­дук­тов, общие для всех реализаций строителей.
3.`ConcreteBuilder` (Конкретный строитель) - уже непосредственные реализации интерфейсов (`Builder`-а), которые предоставляют конкретные шаги для построения продукта. Может быть несколько разных `ConcreteBuilder`-классов, каждый из которых реализует различный способ создания продукта.
4.`Director` (Директор/Супервизор) - супервизионный класс, опре­де­ля­ет поря­док вызо­ва стро­и­тель­ных шагов (этапов) для про­из­вод­ства того или иного продукта. Обычно получает на вход реализацию строителя.

Если коротко, то: билдер предоставляет *методы* для построения объекта, директор знает в каком *порядке* и с какими *значениями* надо вызывать эти методы, чтобы получить *продукт*.

Представьте себе, что вы работаете в кафе и у вас представлены комбо-наборы, так вот комбо-набор является конечным продуктом, билдер - это этапы сборки нашего заказа (положить первое, второе, компот и т.д.), а какой комбо-набор можно выбрать и как его собрать знает официант - наш супервизор, директор.

В целом, если все шаги (сбор значений для полей) не являются обязательными и могут выполняться в любом порядке, то директор опускается и объект собирается только билдером.

Для примера можно рассмотреть облетевший интернет `Builder` машин:

Итак, наш конечный продукт - это собранная машина:

```java
public class Car {
    private String chassis;
    private String body;
    private String paint;
    private String interior;

    public Car(String chassis, String body, String paint, String interior) {
        this.chassis = chassis;
        this.body = body;
        this.paint = paint;
        this.interior = interior;

        validate();
    }

    public String getChassis() {
        return chassis;
    }

    public void setChassis(String chassis) {
        this.chassis = chassis;
    }

    public String getBody() {
        return body;
    }

    public void setBody(String body) {
        this.body = body;
    }

    public String getPaint() {
        return paint;
    }

    public void setPaint(String paint) {
        this.paint = paint;
    }

    public String getInterior() {
        return interior;
    }

    public void setInterior(String interior) {
        this.interior = interior;
    }

    private void validate() {
        if ((chassis == null || chassis.trim().isEmpty()) || (body == null || body.trim().isEmpty())
                || (paint == null || paint.trim().isEmpty()) || (interior == null || interior.trim().isEmpty())) {
                    throw new IllegalStateException("Car assembly is incomplete. Can't deliver!");
                }
    }

    @Override
    public String toString() {
        // StringBuilder class also uses Builder Design Pattern with implementation of java.lang.Appendable interface
        StringBuilder builder = new StringBuilder();
        builder.append("Car [chassis=").append(chassis).append(", body=").append(body).append(", paint=").append(paint)
        return builder.toString();
    }
}
```

Для сборки различных автомобилей выделяем интерфейс билдера:

```java
public interface CarBuilder {
    // возможные этапы сборки
    public CarBuilder fixChassis();
    public CarBuilder fixBody();
    public CarBuilder paint();
    public CarBuilder fixInterior();
    
    // Получение конкретного продукта
    public Car build();
}
```

И реализация конкретного билдера:

```java
public class ModernCarBuilder implements CarBuilder {

    private String chassis;
    private String body;
    private String paint;
    private String interior;

    @Override
    public CarBuilder fixChassis() {
        this.chassis = "Modern Chassis";
        return this;
    }

    @Override
    public CarBuilder fixBody() {
        this.body = "Modern Body";
        return this;
    }
  
    @Override
    public CarBuilder paint() {
        this.paint = "Modern Black Paint";
        return this;
    }

    @Override
    public CarBuilder fixInterior() {
        this.interior = "Modern interior";
        return this;
    }

    @Override
    public Car build() {
        return new Car(chassis, body, paint, interior);
    }
}
```

И директор:

```java
public class AutomotiveEngineer {
    private CarBuilder builder;

    public AutomotiveEngineer(CarBuilder builder) {
        this.builder = builder;
    }

    public Car manufactureCar() {
        return builder.fixChassis().fixBody().paint().fixInterior().build();
    }
}
```

Заметьте, что директор вызывает этапы сборки автомобиля в нужном порядке (он же инженер, понимает как собирать автомобили).

Использование кода:

```java
public class Main {

    public static void main(String[] args) {
        CarBuilder builder = new ModernCarBuilder();
        AutomotiveEngineer engineer = new AutomotiveEngineer(builder);
        Car car = engineer.manufactureCar();
        
        System.out.println("Below car delievered: ");
        System.out.println("======================================================================");
        System.out.println(car);
        System.out.println("======================================================================");
    }
}
```

Пример абстрактный, но суть передает.

Также можно еще больше накрутить ваш билдер: добавлять лямбды, отдельный билдер для каждого аргумента, писать функции вроде `initAfter`/`initBefore`, валидацию объектов при сборке и т.д.

Вот, например, реальный пример с конфигурированием нейронной сети:

```java
MultiLayerNetwork conf = new NeuralNetConfiguration.Builder()
    .seed(rngSeed)
    .optimizationAlgo(OptimizationAlgorithm.STOCHASTIC_GRADIENT_DESCENT)
    .updater(new Adam())
    .l2(1e-4)
    .list()
    .layer(new DenseLayer.Builder()
        .nIn(numRows * numColumns) // Number of input datapoints.
        .nOut(1000) // Number of output datapoints.
        .activation(Activation.RELU) // Activation function.
        .weightInit(WeightInit.XAVIER) // Weight initialization.
        .build())
    .layer(new OutputLayer.Builder(LossFunctions.LossFunction.NEGATIVELOGLIKELIHOOD)
        .nIn(1000)
        .nOut(outputNum)
        .activation(Activation.SOFTMAX)
        .weightInit(WeightInit.XAVIER)
        .build())
    .pretrain(false)
    .backprop(true)
    .build()
```

Разумеется, не всегда надо разделять на интерфейс, выделять директора и т.д.
Как уже было сказано выше, если сборка вашего объекта не зависит от последовательности шагов и будет единственная реализация билдера, то использовать паттерн можно и так, как в начале статьи, без наворотов. Все зависит от задачи и сложности объекта (продукта), с которым вы работаете.

## Где встречается

Паттерн `builder` широко применяется в `Java`, особенно в случаях, когда вам не нужны множественные реализации билдеров, директоры и прочее.

Его можно встретить в `java` реализации работы с [gRPC](https://grpc.io/docs/languages/java/). Где вы декларативно описываете сообщения и с помощью кодогенерации получаете билдеры работы с этими сообщениями в `java`: [пример](https://grpc.io/docs/languages/java/quickstart/).

Я часто использую его в своей работе при помощи [плагина Effective Inner Builder](https://plugins.jetbrains.com/plugin/13665-effective-inner-builder), который умеет работать с аннотациям из [JSR-305](https://jcp.org/en/jsr/detail?id=305), благодаря этому получается разметить и сгенерировать код с билдерами и проверками на `null`:

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

Подробнее об этом вот [тут](../../start/null_war.md#доверяйте-но-с-аннотациями).

В библиотеке [apache commons](https://commons.apache.org/) широко используется этот паттерн, начиная с работы [конкатенации строк](https://commons.apache.org/proper/commons-lang/javadocs/api-3.1/org/apache/commons/lang3/builder/ToStringBuilder.html) и заканчивая работой с [hashCode](https://commons.apache.org/proper/commons-lang/javadocs/api-3.1/org/apache/commons/lang3/builder/HashCodeBuilder.html) и [equals](https://commons.apache.org/proper/commons-lang/javadocs/api-3.1/org/apache/commons/lang3/builder/EqualsBuilder.html), более того, это один из варинтов их генерации, предлагаемых [IntelliJ IDEA](https://www.jetbrains.com/ru-ru/idea/).

> Напоминание:
>
> Зачем нужен [hashCode](../../object/hashcode.md)
>
> Зачем нужен [equals](../../object/equals.md)

В этой библиотеке также предоставляется еще и [интерфейс](https://commons.apache.org/proper/commons-lang/javadocs/api-3.1/org/apache/commons/lang3/builder/Builder.html) для более простой работы с паттерном.

Cуществуют и другие сторонние проекты, которые берут кодогенерацию `builder`-ов на себя, например, проект [lombok](https://projectlombok.org/).

Этот паттерн очень часто встречается там, где надо что-то настроить, собрать конфиг (см. наш пример выше по конфигурации нейросети).

## Заключение

Паттерн `Builder` или `Строитель` используется для упрощения и контроля построения объекта, при этом контроль может осуществляться как конечного результата, так и порядка вызовов методов-этапов.

Широко представлен в мире `Java` как уже готовыми классами-билдерами, так и кодогенераторами.

Чаще всего применяется именно для борьбы с большим количеством конструкторов и аргументов у классов.
Отсюда и основная критика паттерна: отсутствие именнованных аргументов в `Java`.

Но надо понимать, что это - только частный случай работы с паттерном.

Cила паттерна раскрывается в сложных случаях: когда разработчик/тестировщик оперирует действительно сложными объектами, с множеством полей, каждое из которых также представляет собой сложный объект, в которых одни поля влияют на инициализацию других, например наличие одного аргумента приводит к невозможности задания других, например: `spark.read.csv()` и дальше разные параметры чтения из CSV, типа разделителя, заголовков и `spark.read.jdbc()` и параметры JDBC соединения, совершенно другие.

В заключении просто представьте ситуацию, что вы работаете на проекте, где у вас клиенты, которые хотят получать данные по какому-то набору инструментов (бонды, суверенные бонды, корп. векселя и т.п) для каких-то конкретных компаний, при этом они хотят не всё подряд, а только поля КОГДА, ДЕНЬ ВЫПЛАТЫ, ЦЕНА НА СЕГОДНЯ, геолокация клиентов разная (отсюда и работа с датой разная, например, в США - это MM-DD-YYYY), у кого-то точность цены до центов, а у кого-то до целого (евро/доллар и т.п).

И вот чтобы собрать такой объект-ответ для клиента и не ошибиться во всем этом вам необходима и гибкость, и контроль, и порядок вызовов.
Вот тут-то  на сцену и выходит паттерн `Builder`.

## Полезные ссылки

1. [Элегантный Builder на Java](https://habr.com/ru/post/244521/)
2. [Еще один пример Builder](https://github.com/iluwatar/java-design-patterns/tree/master/builder)
3. [Builder Design Pattern In Java](https://dzone.com/articles/builder-design-pattern-in-java)
4. [Перевод Builder Design Pattern In Java на Хабре](https://habr.com/ru/company/otus/blog/552412/)
5. [Design patterns a quick guide to builder pattern](https://medium.com/@andreaspoyias/design-patterns-a-quick-guide-to-builder-pattern-a834d7cacead)
6. [Builder Pattern more than one director](https://stackoverflow.com/questions/39717862/builder-pattern-more-than-one-director)
7. [The Builder Design Pattern in Java](https://stackabuse.com/the-builder-design-pattern-in-java/)
8. [An Introduction to Apache Commons Lang 3](https://www.baeldung.com/java-commons-lang-3)
9. [Книга 'Погружение в Паттерны проектирования'](https://refactoring.guru/ru/design-patterns/book)
