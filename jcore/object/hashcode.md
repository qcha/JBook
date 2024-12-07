# java.lang.Object#hashCode

- [java.lang.Object#hashCode](#javalangobjecthashcode)
    - [Введение](#введение)
    - [Переопределение](#переопределение)
        - [Требования](#требования)
        - [Правила переопределения](#правила-переопределения)
        - [Пример переопределения hashCode](#пример-переопределения-hashcode)
    - [Использование hashCode в equals](#использование-hashcode-в-equals)
    - [Классы-обертки и hashCode](#классы-обертки-и-hashcode)
    - [Массивы и hashCode](#массивы-и-hashcode)
    - [Заключение](#заключение)
    - [Полезные ссылки](#полезные-ссылки)

## Введение

Название метода говорит о том, что работа будет происходить по получению некоего хэш-кода.

Но что такое хэшкод?

Это некоторое число, генерируемое на основе объекта и описывающее его состояние. Это число вычисляется на основе хэш-функции.  
В свою очередь метод `hashCode` является реализацией хэш-функции для `Java`-объекта.

Возвращаемое число считается хэш-кодом объекта.

Теперь взглянем на сам метод.

Объявление выглядит как:

```java
    /**
     * Returns a hash code value for the object. This method is
     * supported for the benefit of hash tables such as those provided by
     * {@link java.util.HashMap}.
     * <p>
     * The general contract of {@code hashCode} is:
     * <ul>
     * <li>Whenever it is invoked on the same object more than once during
     *     an execution of a Java application, the {@code hashCode} method
     *     must consistently return the same integer, provided no information
     *     used in {@code equals} comparisons on the object is modified.
     *     This integer need not remain consistent from one execution of an
     *     application to another execution of the same application.
     * <li>If two objects are equal according to the {@link
     *     equals(Object) equals} method, then calling the {@code
     *     hashCode} method on each of the two objects must produce the
     *     same integer result.
     * <li>It is <em>not</em> required that if two objects are unequal
     *     according to the {@link equals(Object) equals} method, then
     *     calling the {@code hashCode} method on each of the two objects
     *     must produce distinct integer results.  However, the programmer
     *     should be aware that producing distinct integer results for
     *     unequal objects may improve the performance of hash tables.
     * </ul>
     *
     * @implSpec
     * As far as is reasonably practical, the {@code hashCode} method defined
     * by class {@code Object} returns distinct integers for distinct objects.
     *
     * @return  a hash code value for this object.
     * @see     java.lang.Object#equals(java.lang.Object)
     * @see     java.lang.System#identityHashCode
     */
    @IntrinsicCandidate
    public native int hashCode();
```

Ключевое слово `native` означает, что метод реализован в платформенно-зависимом коде, чаще всего на `C/C++`, и скомпонован в виде динамической библиотеки.

Эта реализация зависит от `JVM`.

Возможно, вас сейчас это напугало, но на самом деле достаточно просто понимать, что `native` означает лишь то, что вызываемый код, реализован не на `Java`.

Данный метод играет ключевую роль при работе с любыми коллекциями, построенными на хэш-таблицах или ассоциативных массивах (их еще называют словари или мапы).

Почему?

Представим, что у нас есть массив пар ключ-значение.
Для поиска по ключу в таком массиве вам надо обойти весь массив, заглянуть в каждую ячейку и сравнить ключ.

Алгоритмичекая сложность такой операции `O(N)`, где N - это длина массива.

Как можно ускорить эту работу?

Представьте себе набор пронумерованных корзин, стоящих в ряд. В каждую корзину вы можете положить только одну вещь. Вы кладете обувь на в первую корзину, кофту во вторую и т.д. Грубо говоря, вы уже заранее знаете по какому индексу обращаться к элементу - именно поэтому вы быстро находите нужную вещь.

Теперь вспомним, что хэш - это по сути метаинформация о состоянии объекта, некоторое число.
А что если просто взять хэш и по нему понимать в какой ячейке хранится искомая пара, а потом просто взять ее оттуда, без перебора всего массива?

Ведь это будет гораздо быстрее.

Разумеется, для этого нужны уникальные значения хэшей, чтобы каждому хэшу в соответствие была своя ячейка.

Что происходит, если два разных ключа вернут одинаковое значение хэш-кода и как такие ситуации решаются можно прочитать в статье про устройство [HashMap](../collections/map/hash_map.md)

По сути, главное преимущество в быстродействии ассоциативных массивов основано на работе с `hashCode`.
Именно за счет хэша мы можем вставлять и получать данные за `O(1)`, то есть за время пропорциональное вычислению хэш-функции.

Поэтому очень важно переопределять `hashCode`.

## Переопределение

### Требования

Давайте взглянем на контракт метода:

```java
    /**
     * Returns a hash code value for the object. This method is
     * supported for the benefit of hash tables such as those provided by
     * {@link java.util.HashMap}.
     * <p>
     * The general contract of {@code hashCode} is:
     * <ul>
     * <li>Whenever it is invoked on the same object more than once during
     *     an execution of a Java application, the {@code hashCode} method
     *     must consistently return the same integer, provided no information
     *     used in {@code equals} comparisons on the object is modified.
     *     This integer need not remain consistent from one execution of an
     *     application to another execution of the same application.
     * <li>If two objects are equal according to the {@link
     *     equals(Object) equals} method, then calling the {@code
     *     hashCode} method on each of the two objects must produce the
     *     same integer result.
     * <li>It is <em>not</em> required that if two objects are unequal
     *     according to the {@link equals(Object) equals} method, then
     *     calling the {@code hashCode} method on each of the two objects
     *     must produce distinct integer results.  However, the programmer
     *     should be aware that producing distinct integer results for
     *     unequal objects may improve the performance of hash tables.
     * </ul>
     *
     * @implSpec
     * As far as is reasonably practical, the {@code hashCode} method defined
     * by class {@code Object} returns distinct integers for distinct objects.
     *
     * @return  a hash code value for this object.
     * @see     java.lang.Object#equals(java.lang.Object)
     * @see     java.lang.System#identityHashCode
     */
```

Контракт `hashCode` предъявляет следующие требования к реализации:

* Если вызвать метод `hashCode` на одном и том же объекте, состояние которого не меняли, то будет возвращено **одно и то же** значение.

* Если два объекта равны, то вызов `hashCode` для каждого **обязан** давать один и тот же результат.

    Равенство объектов проверяется через вызов метода [equals](equals.md).

* Если два объекта имеют один и тот же хэш-код, то это **не гарантирует** равенства объектов.

Проще говоря, разный хэш-код у двух объектов - это гарантия того, что объекты не равны, в то время как одинаковый не гарантирует равенства.

Ведь размер `int` ограничен (да и любого типа данных), реализации хэш-функции могут быть далеко не идеальны, поэтому обеспечить гарантию уникальности пар объект->хэш-код невозможно.

Cитуации, когда разные объекты имеют одинаковые хэш-код, называется коллизией.

Реализация метода `hashCode` по-умолчанию возвращает разные значения хэш-кодов для разных объектов:

```java
public class Test {
    public static void main(String[] args) {
        System.out.println(new Person(4).hashCode());
        System.out.println(new Person(4).hashCode());
    }
}

class Person {
    private int age;

    public Person(int age) {
        this.age = age;
    }
}
```

Вывод:

```java
664740647
804564176
```

Стандартная реализация этого метода в классе `java.lang.Object` генерирует хэш-код, основываясь на адресе памяти объекта, но точный алгоритм зависит от реализации `JVM`.

Пока можно не акцентировать внимание на том, как реализована хэш-функция по-умолчанию и запомнить, что для разных объетов будут всегда возвращены разные значения хэш-кодов, не взирая на состояние объекта.

Исходя из описания контракта метода становится понятно, что `hashCode`, наряду с `equals`, играет важную роль в сравнении объектов.
По сути он показывает изменилось ли состояние объекта, которое мы используем в `equals` для сравнения.

Именно поэтому, если вы переопределяете `equals`, то вы **обязаны** переопределить `hashCode`.

Самый очевидный и самый плохой пример как можно переопределить `hashCode` - это всегда возвращать константу:

```java
@Override
public int hashCode() {
    return 14;
}
```

Для равных объектов такой метод вернет одно и то же число, что удовлетворяет спецификации.

Но, делать так **категорически** не рекомендуется.

Если состояние объекта (с такой реализацией `hashCode`) будет изменено, то это никак не отразится на `hashCode`, а это в свою очередь гарантирует возникновение коллизий **всегда**.

Использовать такой подход - это лишить себя всех преимуществ, которые дает нам использование ассоциативных массивов.

Поэтому, давайте посмотрим на что надо ориентироваться при переопределении этого метода.

### Правила переопределения

Во-первых, необходимо исключить избыточные поля, которые не участвуют в `equals`. Так как `equals` и `hashCode` связаны, то набор полей в обоих методах должен быть одинаковым.

Далее необходимо выбрать базу: число, которое будет основной вычисления хэш-кода.

По историческим причинам за базу берут число `31`.

В разных источниках я читал разную информацию об этом выборе.
Где-то говорится, что это взято из-за близости к числу `32`, т.е. степени двойки `2^5 - 1`.
Ну а кто-то утверждает, что был проведен эксперимент и наиболее лучшей базой получились числа `31` и `33`, а `31` понравилось больше.

В целом, вы можете выбрать за базу любое число, но обычно выбирают `31`.
Многие `IDE` (например, `IDEA`) работают именно с такой базой.

Правила вычисления:

* Присваиваем переменной-аккумулятору (например, result) ненулевое значение - базу.
* Далее для каждого значимого поля в объекте вычисляем `hashCode`, по следующим правилам:

    1. Если поле `boolean`:

         ```java
         (f ? 1 : 0)
         ```

    2. Если поле `byte`, `char`, `short` или `int`:

         ```java
         (int) f
         ```

    3. Если поле `long`:

        ```java
        (int)(f ^ (f >>> 32))
        ```

    4. Если поле `float`:

        ```java
        Float.floatToIntBits(f);
        ```

    5. Если поле `double`:

        ```java
            Double.doubleToLongBits(f)
        ```

         А затем работать как с `long`

    6. Если поле это ссылка на другой объект, то рекурсивно вызываем `hashCode()` у объекта

    7. Если поле `null`, то возвращаем `0`

    8. Если поле - это массив, то обрабатываем так, как будто каждый элемент массива - это поле объекта

* После каждого обработанного поля объединяем его `hashCode` с текущим значением:

    ```java
        result = 31 * result + c; // c - это hashCode обработанного поля.
    ```

* Возвращаем результат

### Пример переопределения hashCode

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
    public int hashCode() {
      int result = 31;
      result = result * 17 + age;
      result = result * 17 + number;

      long lnum = Double.doubleToLongBits(number);
      result = result * 17 + (int)(lnum ^ (lnum >>> 32));

      result = result * 17 + name.hashCode();
      result = result * 17 + carKey.hashCode();

      return result;
    }

    // override equals
    // ...
}
```

Также, с версии `Java 8+`, в классе `java.util.Objects` есть вспомогательные методы для генерации `hashCode`:

```java
@Override
public int hashCode() {
    return Objects.hash(age, number, salary, name, carKey);
}
```

## Использование hashCode в equals

Так как `hashCode` допускает возникновение коллизий, то использовать его в `equals` нет смысла.

Методы идут в паре, но использование одного в другом не желательно и бесмыссленно, так как `hashCode` призван облегчить поиск объектов в каких-то структурах, например, ассоциативных массивах.

В свою очередь, `equals` - это уже непосредственно про сравнение объектов.

## Классы-обертки и hashCode

Помните, что типы-обертки, которые по размеру меньше или равны `int`, возвращают в качестве `hashCode` свое значение:

```java
public class Test {
    public static void main(String[] args) {
        Integer a = 5000;
        System.out.println(a.hashCode()); // 5000
    }
}
```

Так как `Long` превосходит `int`, то, разумеется, возвращать свое значение в качестве `hashCode` он не может.
Вместо этого используется более сложный алгоритм, который работает с значением `long` (как в примере `Person`-а).

## Массивы и hashCode

Помните, что хэш-код массива не зависит от хранимых в нем элементов, а присваивается при создании массива!

```java
        int[] arr = {1, 2, 3, 4, 5};
        System.out.println(arr.hashCode());

        // меняем элемент в массиве
        arr[0] = 100;
        // hashCode массива не изменяется
        System.out.println(arr.hashCode());
```

Результат:

```java
1580066828
1580066828
```

Решением является использование статического метода из стандартной библиотеки для рассчета `hashCode`: `java.util.Arrays.hashCode(...)`:

```java
        int[] arr = {1, 2, 3, 4, 5};
        System.out.println(Arrays.hashCode(arr));

        arr[0] = 100;
        System.out.println(Arrays.hashCode(arr));
```

Результат:

```java
29615266
121043845
```

## Заключение

Хэш-код - это некоторое число, генерируемое на основе объекта и описывающее его состояние.

Если планируется использовать объекты в качестве ключа в ассоциативном массиве, то необходимо переопределять `hashCode`.

Также, если в классе переопределяется [equals](./equals.md), то необходимо переопределять `hashCode` (и наоборот).

Эти методы всегда должны определяться парой!

Плохая реализация хэш-функции даст вам большое количество коллизий, что сведет на нет все преимущества использования ассоциативных массивов.

Помните, что большинство IDE сейчас легко сгенерируют вам `hashCode`, чтобы вы не писали его вручную.

Также, существуют сторонние проекты, которые берут кодогенерацию на себя, например, проект [lombok](https://projectlombok.org/).
Существуют и сторонние библиотеки, помогающие в вычислении `hashCode`, например [apache commons](https://commons.apache.org/).

## Полезные ссылки

1. [Владимир Долженко — Внутрь VM сквозь замочную скважину hashCode](https://www.youtube.com/watch?v=GS2YqQ1DNJU)
2. [Как работает hashCode() по умолчанию?](https://habr.com/ru/company/mailru/blog/321306/)
3. [Разбираемся с hashCode() и equals()](https://habr.com/ru/post/168195/)
4. [What role does hashCode play when comparing two objects](https://stackoverflow.com/questions/29949305/what-role-does-hashcode-play-when-comparing-two-objects)
5. [Java equals() and hashCode() Contracts](https://www.baeldung.com/java-equals-hashcode-contracts)
