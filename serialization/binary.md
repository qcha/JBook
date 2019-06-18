# Binary Serialization in Java

## Введение

Одним из серьезных преимуществ `Java` является поддержка сериализации объекта 'из коробки'.

Все необходимое уже есть в `JDK`.

> Все это актуально как минимум до `JDK 11`.
>
> `Oracle` задумывается о том, чтобы пределать механизм сериализации в последующих релизах, но точных данных о дате, когда изменения вступят в силу нет>
> 
> Подробнее об этом [здесь](https://www.infoworld.com/article/3275924/java/oracle-plans-to-dump-risky-java-serialization.html).

Так как умение объекта сериализовываться - это некоторое поведение, а за поведение в `Java` отвечают интерфейсы, то в `Java` существует целых два интерфейса, связанных с сериализацией:

* `java.io.Serializable`
* `java.io.Externalizable`

### java.io.Serializable

Универсальный способ сделать объект сериализуемым - это реализовать интерфейс-маркер `java.io.Serializable`.

Это наиболее часто встречаемый и используемый вариант, можно сказать, что это стандартный способ сериализации.

В `Java` он работает через `Reflection`: класс раскладывается на поля и метаданные, после чего уже пишется в выходной поток.

За эту универсальность, разумеется, надо платить и плата как раз заключается в использовании `Reflection`, который не дает максимальную производительность.

> В двух словах:
> 
> С помощью `Reflection` берется класс сериализуемого объекта, у него берется список полей, по всем полям проверяются различные условия, например, помечено ли поле модификатором `transient`(об этом будет речь ниже, если поле является объектом, то надо проверить сериализуемое ли оно и т.д, значения пишутся в поток, причем достаются из полей тоже через `Reflection`.

Продемонстрируем это, создав класс `ExampleSerializable` и попытавшись сериализовать и десериализовать его:

```java
public class Main {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        ExampleSerializable example = new ExampleSerializable(14, "Hello");

        System.out.println("Before Serialization: " + example);
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(baos);

        oos.writeObject(example);

        ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bais);

        ExampleSerializable from = (ExampleSerializable) ois.readObject();

        System.out.println("After Deserialization: " + from);
    }
}

class ExampleSerializable implements Serializable {
    private int a;
    private String hello;

    public ExampleSerializable(int a, String hello) {
        System.out.println("Constructor with all parameters");
        this.a = a;
        this.hello = hello;
    }

    @Override
    public String toString() {
        return "ExampleSerializable{" +
                "a=" + a +
                ", hello='" + hello + '\'' +
                '}';
    }
}
```

> Работа с ресурсами и их закрытие специально сделана наиболее просто(и не совсем верно), чтобы не переусложнять пример.

Выполнение кода даст нам:

```java
Constructor with all parameters
Before Serialization: ExampleSerializable{a=14, hello='Hello'}
After Deserialization: ExampleSerializable{a=14, hello='Hello'}
```

Т.е конструктор в этом коде вызвался только один раз - когда мы впервые создали объект, а при десериализации вызова конструктора не было.

> При использовании `java.io.Serializable`, при десериализации объекта **не вызывается** конструктор класса!

Следующий момент, который интересен - это как стандартная сериализация работает с наследованием.

---

**Вопрос**:

Является ли сериализуемым `дочерний` класс, если `родительский` класс реализует `java.io.Serializable`?

**Ответ**:

Да! Является!

Поэтому, если у вас иерархия классов, то можно реализовать интерфейс-маркер у одного общего родителя и таким образом сделать всю иерархию сериализуемой.

---

**Вопрос**:

Является ли `родительский` класс сериализуемым, если `наследник` реализует интерфейс `java.io.Serializable`, т.е:

```java
class Parent {
    protected int a;

    public Parent(int a) {
        System.out.println("Calling parent constructor");
        this.a = a;
    }

 

    @Override
    public String toString() {
        return "Parent: a= " + a;
    }
}

class Child extends Parent implements Serializable {
    private String test;

    public Child(int a, String test) {
        super(a);
        this.test = test;
    }

    @Override
    public String toString() {
        return "Child: a= " + a + ", test= " + test;
    }
}
```

Пример записи будет ровно тот же, что выше был с `ExampleSerializable`.

**Ответ**:

Нет, не будет!

При этом будет выброшено исключение: `java.io.InvalidClassException`.

А вот если добавить конструктор без параметров в родительский класс, то исключения уже не возникнет.

И вывод программы будет следующий:

```java
Calling parent constructor
Before Serialization: Child: a= 14, test= Hello
Calling parent default constructor
After Deserialization: Child: a= 0, test= Hello
```

Внимательный читатель сразу должен заметить, что нечто странное произошло с полем, которое принадлежит родителю.

А самый внимательный сразу поймет, что при дессериализации вызывается конструктор без параметров!

Это значит, что если родительский класс не реализует `java.io.Serializable`, то данные, которые принадлежат ему не сериализуются, а при десериализации используется конструктор без параметров.

При этом, если реализовать интерфейс `java.io.Serializable` у родительского класса, то вывод будет 'ожидаемым':

```java
Calling parent constructor
Before Serialization: Child: a= 14, test= Hello
After Deserialization: Child: a= 14, test= Hello
```

> Конструктор по-умолчанию должен иметь модификатор доступа `public` либо `protected`.

---

**Вопрос**:

А что насчет статических полей? С ними есть какие-то подводные камни?

**Ответ**:

Как известно, статические поля принадлежат классу, а не объекту.
В то же время сериализуем мы именно объект, задача сериализации - это работа с текущим состоянием объекта.

Поэтому при использовании стандартной сериализации с помощью `java.io.Serializable`, сериализуется только состояние объекта.

Статические поля не сериализуются.

---

### Transient

В некоторых случаях состоянием объекта являются поля, которые сериализовать, по той или иной причине, не надо.

За исключение полей из сериализации отвечает ключевое слово `transient`, т.е временный, скоротечный.

При десериализации поля, помеченные как `transient`, инициализируются значениями по умолчанию.

> Это может привести к `NPE` исключениям, если вы не объявили для ссылочных полей значения по умолчанию. Или же инициализировать примитивные поля значениями, которые не являются в данном контексте верными, что может привести к трудноуловимым ошибкам.
> 
> Поэтому старайтесь явно присваивать `transient` полям значения по умолчанию.

### java.io.Externalizable

Второй способ сериализовать объект в `Java` - это реализация интерфейса `java.io.Externalizable`.

Данный способ используется гораздо реже, но знать о нем нужно.

Интерфейс `java.io.Externalizable`, в отличии от `java.io.Serializable`, не является интерфейсом-маркером, его объявление выглядит следующим образом:

```java
public interface Externalizable extends java.io.Serializable {

    void writeExternal(ObjectOutput out) throws IOException;

    void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
}
```

Теперь, как видно из описания методов, то, как будет сериализоваться и десериализоваться объект - ответственность разработчика.

Это более гибкий способ. Грубо говоря используя этот способ мы создаем протокол сериализации для объектов класса, в котором реализуем `java.io.Externalizable` интерфейс.

И в отличии от `java.io.Serializable` в данном случае никаких метаданных не пишется! 

Поэтому для использования `java.io.Externalizable` необходимо, чтобы у всех классов, в том числе и дочерних, были конструкторы по умолчанию, так как с его помощью будет создан объект, а на нем уже будет вызван метод из `java.io.Externalizable`, который и восстановит состояние объекта.

Разберем как это выглядит:

```java
class ExampleExternalizable implements Externalizable {
    private int a;
    private String msg;

    public ExampleExternalizable() {
        System.out.println("Default Constructor called");
    }

    public ExampleExternalizable(int a, String message) {
        System.out.println("Constructor called");
        this.a = a;
        this.msg = message;
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeInt(a);
        out.writeUTF(msg);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException {
        a = in.readInt();
        msg = in.readUTF();
    }

    @Override
    public String toString() {
        return "ExampleExternalizable a= " + a + ", msg= " + msg;
    }
}
```

Код проверки будет примерно тот же, что и в предыдущих примерах.

Вывод результата:

```java
Constructor called
Before Serialization: ExampleExternalizable a= 14, msg= Hello
Default Constructor called
After Deserialization: ExampleExternalizable a= 14, msg= Hello
```

Видим, что при десериализации вызывается конструктор без параметров, таким образом создается объект и далее происходит инициализация полей в `readExternal`.

> При желании можно добавить в эти методы отладочную информацию и убедиться еще раз.

---

**Вопрос**:

Получается для такого подхода можно сериализовать и `transient` поле даже?

**Ответ**:

Совершенно верно!

При использовании `java.io.Externalizable` никто не мешает сериализовать и десериализовать такое поле.

---

**Вопрос**:

Выше было сказано, что при использовании `java.io.Externalizable` необходимо наличие конструкторов по умолчанию. А что делать с `final` полями?

**Ответ**:

При необходимости в сериализации/десериализации `final` полей использовать `java.io.Externalizable` не выйдет. Все дело в том, что `final` поля должны быть инициализированы в конструкторе, а при `java.io.Externalizable` инициализация полей происходит в `readExternal`.

Поэтому при необходимости в сериализации/десериализации `final` полей использовать можно только `java.io.Serializable`.

---

#### Кастомизация java.io.Serializable

При этом существует возможность кастомизировать даже стандартную сериализацию. Для этого в класс, реализующий `java.io.Serializable`, добавляются методы, отвечающие за сериализацию/десериализацияю класса:

```java
 private void writeObject(java.io.ObjectOutputStream out) throws IOException
 private void readObject(java.io.ObjectInputStream in) throws IOException, ClassNotFoundException;
 private void readObjectNoData() throws ObjectStreamException;
```

Этим методы будут вызваны при сериализации/десериализации и в поток данные попадут уже с помощью них, поэтому использовать эти методы надо только в случае, если нам необходимо четко и гибко контроллировать весь процесс сериализации/дессериализации. 

Чтобы вызвать `default` механизм сериализации/десериализации существуют методы у `java.io.ObjectOutputStream` и `java.io.ObjectInputStream` соответственно:

```java
public void defaultWriteObject() throws IOException
public void defaultReadObject()  throws IOException, ClassNotFoundException
```

Эти методы могут быть вызваны только в `writeObject` и `readObject`, иначе произойдет исключение `java.io.NotActiveException`.

Их задача - записать `transient` и нестатические поля в поток, что и делает стандартная сериализация.

Эти методы бывают полезны, если, например, если вы хотите использовать стандартную сериализацию, но вам дополнительно надо добавить некоторые поля в поток.

```java
private void writeObject(java.io.ObjectOutputStream stream) throws IOException {
    // default serialization
    stream.defaultWriteObject();
    // our special serialization
}

private void readObject(java.io.ObjectInputStream stream) throws IOException, ClassNotFoundException {
    // default deserialization
    stream.defaultReadObject();
    // our special deserialization and using setters
}
```

---

**Вопрос**:

А почему эти методы `private`?

**Ответ**:

Это вполне логично, так как написанная сериализаццию должна быть только для текущего конкретного объекта класса и ни для кого более.

---

Также если мы отнаследовались от класса, который сериализуемый, но по каким-то причинам не хотим, чтобы наш класс был серализуемый - мы можем сделать вот так в нашем классе:

```java
private void writeObject(ObjectOutputStream out) throws IOException {
    throw new NotSerializableException("Uuups!");
}
private void readObject(ObjectInputStream in) throws IOException {
    throw new NotSerializableException("Uuups!");
}
```

### Serial Version UID

При работе с стандартной сериализацией через `java.io.Serializable` есть еще один важный момент.

Существует специальное поле, которое содержит уникальный идентификатор версии сериализованного класса.

Это поле выглядит следующим образом:

```java
private static final long serialVersionUID
```

И добавляется в класс на этапе компиляции.

Это поле записывается в поток сериализации, а при десериализации объекта мы сравниваем его с тем, которое добавлено на этапе компиляции в класс(т.е у класса, загруженного в `JVM`) и если эти значения не совпадают, то кидается исключение.

Если какое-то поле будет исключено или изменен тип этого поля, то изменится `serialVersionUID`, а значит объекты класса, сериализованные по старому образцу не могут быть десериализованы по новому.

Однако бывают случаи, когда поля и их порядок определены, а методы и интерфейс меняются.

Вспоминаем, что сериализация - это работа с состоянием объекта, а значит изменение методов и интерфейсов не угрожают правильной работе, но в таком случае изменяется `serialVersionUID`.

Это требует либо полной перекомпиляции классов(везде, где будет использоваться такой класс при сериалиацзии/десериализации), что довольно накладно и неудобно. А иногда и вовсе не выполнимо.

Поэтому существует способ обойти это ограничение и явно задать это поле вручную.

Значение этого поля может быть абсолютно любым, некоторые часто ставят значение `1L`.
Можно воспользоваться утилитой `serialver`, и вычислить `serialVersionUID` стандартным способом.

Вообще, с версии `Java 5+` рекомендуется это поле объявлять в явном виде, так как вычисление значения `serialVersionUID` крайне чувствительно к деталям структуры класса, что может вызвать `java.io.InvalidClassException` на разынх `JVM`.

Модификатор доступа к такому полю не оговорен явно, но так как это поле принадлежит **только** классу, в котором объявлено, и больше никому, то, по признаку `мое и только мое`, можно ставить `private`.

## Работа с Singleton

Так как при десериализации мы получаем другой объект, но с тем же состоянием, что и сериализованный, то возникает некоторая проблема с [singleton](../patterns/singleton.md).

Ведь поулчается, что создается еще один объект `singleton`.

Для решения этой проблемы в классе объявляется следующий метод:

```java
private Object readResolve() throws ObjectStreamException
```

> Модификатор доступа может быть любой, но я использовал `private`, по той же причине, что и выще.

Суть метода заключается в том, чтобы возвращать уже существующий экземпляр класса, соответствующий внутреннему состоянию десериализованного объекта.

Пример я взял [отсюда](http://www.skipy.ru/technics/serialization.html#singleton).

```java
public class Answer implements Serializable{

    private static final String STR_YES = "Yes";
    private static final String STR_NO = "No";

    public static final Answer YES = new Answer(STR_YES);
    public static final Answer NO = new Answer(STR_NO);

    private String answer = null;

    private Answer(String answer){
        this.answer = answer;
    }

    private Object readResolve() throws ObjectStreamException{
        if (STR_YES.equals(answer))
            return YES;
        if (STR_NO.equals(answer))
            return NO;
        throw new InvalidObjectException("Unknown value: " + answer);
    }
}
```

Для `enum` в `Java` сделано нечто подобное, но внутри `JVM`, поэтому можно не переживать за сериализацию/десериализацию `enum`.

## Заключение

Если нет необходимости в кастомизации процесса сериализации/десериализации и производительность устраивает, то я не вижу смысла в использовании `java.io.Externalizable`.

Поэтому, обычным выбором, а потому и наиболее часто встречаемым, является стандартная сериализация через `java.io.Serializable`.

Необходимо помнить, что если родительский класс реализует интерфейс-маркер `java.io.Serializable`, то все дочерние классы также являются сериализуемыми.

Важным моментом является еще и то, что, если дочерний класс реализует `java.io.Serializable`, а родительский нет, то поля родительского конструктора будут инициализированы в конструкторе по умолчанию(у родительского класса, разумеется), если же конструктора по умолчанию нет, то будет выброшено исключение.

При необходимости можно также добавить кастомизированную логику по сериализации в стандартную реализацию `java.io.Serializable`, для этого следует добавить методы:

```java
 private void writeObject(java.io.ObjectOutputStream out) throws IOException
 private void readObject(java.io.ObjectInputStream in) throws IOException, ClassNotFoundException;
 private void readObjectNoData() throws ObjectStreamException;
```

В таком случае логика сериализации будет взята оттуда.

Статические поля не сериализуются.

При сериализации стандартным способом в поток попадут метаданные класса и родительских классов, состояния объекта класса, который сериализуется и родительских.

Основным минусом может стандартной сериализации может являться не самая быстрая производительность, по сравнению с другими способами.

Подход с реализацией `java.io.Externalizable` более гибок и производителен, однако вся логика и ответственность по сериализации объекта теперь ложиться на разработчика.

При грамотной реализации `java.io.Externalizable` можно получить ощутимый выигрыш в производительности, вот тут пишут про [выигрыш более чем в 10 раз](http://www.skipy.ru/technics/serialization.html#performance).

Однако учтите, что преждевременные оптимизации могут принести много горя, поэтому я еще раз повторю, если вас устраивает производительность стандартной сериализации, или вы вообще о ней не задумывались еще, то не стоит переходить на `java.io.Externalizable`.

Это может вызвать дополнительные трудности и сложности, а также ошибки, поэтому необходимо четко понимать, что вы получаете как плюсы подхода и что как минусы.

При этом, если класс реализует оба интерфейса - и `java.io.Externalizable`, и `java.io.Serializable`, то более высокий приоритет у `java.io.Externalizable`.

При `java.io.Externalizable` в поток не пишутся метаданные классов, также с его помощью можно сериализовать и статические поля, и поля, отмеченные как `transient`.

Однако, использование `final` полей будет невозможно при таком подходе, так как сериализовать их не составит труда, а вот десериализовать уже будет нельзя.

## Полезные ссылки

Рекомендую ознакомиться:

* [Сериализация от А до Я](http://www.skipy.ru/technics/serialization.html#performance) - просто **must read**.
* [Коротко о сериализации](https://www.baeldung.com/java-externalizable)
* [Официальная документация](https://docs.oracle.com/javase/8/docs/api/java/io/Serializable.html)
* [Transient vs Static](https://javabeginnerstutorial.com/core-java-tutorial/transient-vs-static-variable-java/)
