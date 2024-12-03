# java.lang.Object#toString

- [java.lang.Object#toString](#javalangobjecttostring)
    - [Введение](#введение)
    - [Переопределение toString](#переопределение-tostring)
        - [Подумайте о формате](#подумайте-о-формате)
    - [Подводные камни](#подводные-камни)
        - [Циклический вызов](#циклический-вызов)
        - [Логика на toString](#логика-на-tostring)
        - [Чувствительные данные](#чувствительные-данные)
    - [Заключение](#заключение)
    - [Полезные ссылки](#полезные-ссылки)

## Введение

Данный метод позволяет получить некоторое строковое представление объекта.

Объявление метода выглядит как:

```java
    /**
     * Returns a string representation of the object.
     * @apiNote
     * In general, the
     * {@code toString} method returns a string that
     * "textually represents" this object. The result should
     * be a concise but informative representation that is easy for a
     * person to read.
     * It is recommended that all subclasses override this method.
     * The string output is not necessarily stable over time or across
     * JVM invocations.
     * @implSpec
     * The {@code toString} method for class {@code Object}
     * returns a string consisting of the name of the class of which the
     * object is an instance, the at-sign character `{@code @}', and
     * the unsigned hexadecimal representation of the hash code of the
     * object. In other words, this method returns a string equal to the
     * value of:
     * <blockquote>
     * <pre>
     * getClass().getName() + '@' + Integer.toHexString(hashCode())
     * </pre></blockquote>
     *
     * @return  a string representation of the object.
     */
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```

Т.е по-умолчанию результатом работы метода будет имя класса и его `hashCode` в hexadecimal представлении, разделенные символом `@`.

В большинстве случаев это не то, что хочет видеть пользователь и именно поэтому в `JavaDoc` рекомендуется этот метод переопределять: так как это не информативно, а также не отражает состояние объекта.

Хорошо реализованный `toString` помогает при отладке: так как в [логе](https://habr.com/ru/articles/795445/) печатаются легкочитаемые и информативные строки, показывающие что это за объект и что у него было за состояние на момент вызова.

Помните, что если вы переопределяете метод `toString`, то возвращаемая строка должна содержать всю **значимую** (полезную) информацию объекта.

Относитесь к выбору того, что попадет в строковое предствление, очень внимательно.

## Переопределение toString

Для примера рассмотрим следующий класс и переопределим метод `toString`:

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
    public String toString() {
      return String
            .format("Person name: %s, age: %d, number: %s, salary: %4.2f, carKey: %s",
                    name, age, number, salary, carKey);
    }
}

class CarKey {
    private int key;

    public CarKey(int key) {
        this.key = key;
    }
}

```

Создадим объект и распечатаем его в консоль:

```java
public static void main(String[] args) {
    System.out.println(new Person(27, 8, "Aleksandr", 200000, new CarKey(14)));
}
```

Полученный результат:

```java
Person name: Aleksandr, age: 27, number: 8, salary: 200000.00, carKey: examples.CarKey@2f92e0f4
```

Без переопределения `toString` у класса `CarKey` его объект снова выведет нечеловекочитаемую информацию, что должно навести на мысль: а так ли нужен вывод `CarKey`?

Если да, мы понимаем, что `CarKey` обязателен - это значимая информация для строкового представления класса `Person`, то необходимо либо переопределить `toString` у `CarKey`, либо вручную, например, с помощью `get` методов, сформировать строковое представление объекта класса `CarKey`.

Еще один важный момент - это наличие и отсутствие `get` методов для полей, которые входят в вывод `toString`.
Если вы включаете какое-либо поле объекта в `toString`, то правильно было бы проконтролировать то, что у такого поля имеется `get`-метод.

И действительно, если мы включаем поле в `toString`, который является публичным методом, то такое поле как минимум логично сделать доступным на чтение: ведь его значение все равно попадает в результат `toString`.

---

**Вопрос**:

Как вы думаете, определен ли и если определен, то как метод `toString` у классов-оберток в `Java`? Например, `java.lang.Integer`?

**Ответ**:

Метод `toString` у классов-оберток переопределен и всегда возвращает строковое представление значения примитива:

```java
System.out.println(Integer.valueOf(10)); // 10
```

---

### Подумайте о формате

Также полезно заранее подумать о формате, в котором будет строковое представление объекта.

Например, заранее решить, что вы будете представлять все в `json` формате. При использовании библиотеки `commons-lang3` от `Apache`, можно воспользоваться для этой цели можно воспользоваться билдером:

```java
    @Override
    public String toString() {
        return new ToStringBuilder(this, ToStringStyle.JSON_STYLE)
                .append("state", state)
                .append("name", name)
                .toString();
    }
```

Точно также заранее следует подумать и зафиксировать формат дат, сложных бизнес данных, чтобы заранее было удобно и читаемо.

Обязательно следите и фиксируйте формат: иначе каждый разработчик начнет формировать строковое представление в любимом формате и в какой-то момент ваши логи превратятся в кашу, где уже ничего не разобрать.

## Подводные камни

### Циклический вызов

Разберем следующий пример:

```java
public class Exmaple {
    public static void main(String[] args) {
        Test2 test2 = new Test2();
        Test1 test1 = new Test1(test2);
        
        test2.setTest1(test1);

        System.out.println(test1);
    }
}

class Test1 {
    Test2 test2;

    public Test1(Test2 test2) {
        this.test2 = test2;
    }

    @Override
    public String toString() {
        return "Test1{ test2=" + test2 + '}';
    }
}

class Test2 {
    Test1 test1;

    public Test2() {

    }

    public void setTest1(Test1 test1) {
        this.test1 = test1;
    }

    @Override
    public String toString() {
        return "Test2{ test1=" + test1 + '}';
    }
}
```

У нас есть два класса, каждый из которых содержит ссылку на другой.
Мы переопределяем `toStirng` так, как показано выше.

Как вы думаете, что получится?

А получится:

```java
java.lang.StackOverflowError
```

Как это произошло: `System.out.println` вызывает у объекта `test1` метод `toString`, в методе `toString` у `test1` происходит вызов `toString` у объекта `test2`, внутри которого уже снова идет обращение к `toString` у `test1`. В результате мы получаем зацикленность - мы ходим по кругу, вызывая `toString`, пока стек вызовов не переполнится.

Змей Уроборос снова укусил себя за хвост.

### Логика на toString

В моей практике встречался код, который был завязан на то, что возвращает `toString` того или иного объекта. Грубо говоря, происходил анализ-парсинг того, что возвращал метод у объекта.

Старайтесь избегать написания кода, который завязан на результат работы `toString`. Метод не дает никаких гарантий о том, в каком формате и виде будет сформирована строка, строить свою логику вокруг этого не самая лучшая идея.

Даже разные версии `IDE` могут генерировать разное представление `toString` для одного и того же объекта. Это же касается и разных библиотек, которые берут на себя часть работы по кодогенерации, например, [lombok](https://projectlombok.org/).

Исключением из этого правила может быть разве что работа с примитивами и классами-обертками.

### Чувствительные данные

Обязательно следите за тем, чтобы в `toString` не попадали чувствительные данные: пароли и важные данные о пользователе, проводимой операции. Это может привести к утечке данных, так как часто логируется весь объект:

```java
logger.debug("Person after update: {}", person);
```

А значит вызывается `toString` и при наличии в нем чувствительных данных они утекут в логи, т.е. будут скомпрометированы.

Вот, например, подобный класс (из реального проекта):

```java
public class AuthCredentials {

    private String clientId;
    private String password;

    public AuthCredentials(String clientId, String password) {
        this.clientId = clientId;
        this.password = password;
    }

    public String getClientId() {
        return clientId;
    }

    public void setClientId(String clientId) {
        this.clientId = clientId;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public String toString() {
        return "AuthCredentials{" +
                "clientId='" + clientId + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```

Не трудно догадаться, что один не очень аккуратный вызов `toString` там, где не надо и ваш пароль станет достоянием всех, кто умеет читать логи.

## Заключение

Если планируется использовать строковое представление класса, то необходимо переопределить метод `toString`.

Помните, что важно включать в такую реализацию только **необходимую** и **достаточную** информацию об объекте, убирая избыточную информацию.

Плохим тоном считается создание огромных строковых представлений, в которых часть информации не имеет значения или является секретной, например, пароль пользователя.

Если у поля нет `get`-метода (или любого другого метода на получение поля), то, задайтесь вопросом: а так ли нужно включать такое поле в строковое представление объекта?

При включении в `toString` поля, принадлежащего к ссылочному типа, убедитесь, что у этого типа также переопределен `toString`, иначе вам придется вручную формировать строковое представление объекта. А лучше и вовсе задуматься о том, чтобы отказаться от включения его в `toString` реализацию класса.

Контролируйте то, что вы включаете в реализацию `toString`, помните о возможности циклического вызова, который неизбежно приведет к `java.lang.StackOverflowError`.

Старайтесь не строить свою логику и работу программы на результате вызова `toStirng`.

Помните, что большинство IDE сейчас легко сгенерируют вам `toString`, чтобы вы не писали его вручную.
Также, существуют сторонние проекты, которые берут кодогенерацию на себя, например, проект [lombok](https://projectlombok.org/).

## Полезные ссылки

1. [Effective Java 2nd Edition, Item 10: Always override toString](https://www.amazon.com/dp/0321356683)
