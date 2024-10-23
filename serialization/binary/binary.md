# Binary Serialization in Java

## Введение

Одним из серьезных преимуществ `Java` является поддержка сериализации объектов 'из коробки'.

У этого способа есть ряд существенных минусов (как и у любого способа сериализации данных), подробнее об этом можно прочесть:

* [тут](https://openjdk.org/jeps/154)
* [тут](https://stackoverflow.com/questions/39078841/what-are-the-security-risks-in-the-serialization-of-a-lambda-expression).

В свое время `Oracle` даже задумывалась о том, чтобы пределать механизм сериализации в последующих релизах, но воз и ныне тут, поэтому разберем существующий механизм подробнее.

Возможность сериализовать объект - это некоторое поведение, а за поведение в `Java` отвечают интерфейсы.
Поэтому в `Java` существует целых два интерфейса, связанных с сериализацией:

* `java.io.Serializable`
* `java.io.Externalizable`

Взглянем на них подробнее.

## java.io.Serializable

Для начала взглянем на объявление интерфейса:

```java
package java.io;

/**
 * Serializability of a class is enabled by the class implementing the
 * java.io.Serializable interface.
 *
 * <p><strong>Warning: Deserialization of untrusted data is inherently dangerous
 * and should be avoided. Untrusted data should be carefully validated according to the
 * "Serialization and Deserialization" section of the
 * {@extLink secure_coding_guidelines_javase Secure Coding Guidelines for Java SE}.
 * {@extLink serialization_filter_guide Serialization Filtering} describes best
 * practices for defensive use of serial filters.
 * </strong></p>
 *
 * Classes that do not implement this
 * interface will not have any of their state serialized or
 * deserialized.  All subtypes of a serializable class are themselves
 * serializable.  The serialization interface has no methods or fields
 * and serves only to identify the semantics of being serializable. <p>
 *
 * It is possible for subtypes of non-serializable classes to be serialized
 * and deserialized. During serialization, no data will be written for the
 * fields of non-serializable superclasses. During deserialization, the fields of non-serializable
 * superclasses will be initialized using the no-arg constructor of the first (bottommost)
 * non-serializable superclass. This constructor must be accessible to the subclass that is being
 * deserialized. It is an error to declare a class Serializable if this is not
 * the case; the error will be detected at runtime. A serializable subtype may
 * assume responsibility for saving and restoring the state of a non-serializable
 * supertype's public, protected, and (if accessible) package-access fields. See
 * the <a href="{@docRoot}/../specs/serialization/input.html#the-objectinputstream-class">
 * <cite>Java Object Serialization Specification,</cite></a> section 3.1, for
 * a detailed specification of the deserialization process, including handling of
 * serializable and non-serializable classes. <p>
 *
 * When traversing a graph, an object may be encountered that does not
 * support the Serializable interface. In this case the
 * NotSerializableException will be thrown and will identify the class
 * of the non-serializable object. <p>
 *
 * Classes that require special handling during the serialization and
 * deserialization process must implement special methods with these exact
 * signatures:
 *
 * <PRE>
 * private void writeObject(java.io.ObjectOutputStream out)
 *     throws IOException
 * private void readObject(java.io.ObjectInputStream in)
 *     throws IOException, ClassNotFoundException;
 * private void readObjectNoData()
 *     throws ObjectStreamException;
 * </PRE>
 *
 * <p>The writeObject method is responsible for writing the state of the
 * object for its particular class so that the corresponding
 * readObject method can restore it.  The default mechanism for saving
 * the Object's fields can be invoked by calling
 * out.defaultWriteObject. The method does not need to concern
 * itself with the state belonging to its superclasses or subclasses.
 * State is saved by writing the individual fields to the
 * ObjectOutputStream using the writeObject method or by using the
 * methods for primitive data types supported by DataOutput.
 *
 * <p>The readObject method is responsible for reading from the stream and
 * restoring the classes fields. It may call in.defaultReadObject to invoke
 * the default mechanism for restoring the object's non-static and
 * non-transient fields.  The defaultReadObject method uses information in
 * the stream to assign the fields of the object saved in the stream with the
 * correspondingly named fields in the current object.  This handles the case
 * when the class has evolved to add new fields. The method does not need to
 * concern itself with the state belonging to its superclasses or subclasses.
 * State is restored by reading data from the ObjectInputStream for
 * the individual fields and making assignments to the appropriate fields
 * of the object. Reading primitive data types is supported by DataInput.
 *
 * <p>The readObjectNoData method is responsible for initializing the state of
 * the object for its particular class in the event that the serialization
 * stream does not list the given class as a superclass of the object being
 * deserialized.  This may occur in cases where the receiving party uses a
 * different version of the deserialized instance's class than the sending
 * party, and the receiver's version extends classes that are not extended by
 * the sender's version.  This may also occur if the serialization stream has
 * been tampered; hence, readObjectNoData is useful for initializing
 * deserialized objects properly despite a "hostile" or incomplete source
 * stream.
 *
 * <p>Serializable classes that need to designate an alternative object to be
 * used when writing an object to the stream should implement this
 * special method with the exact signature:
 *
 * <PRE>
 * ANY-ACCESS-MODIFIER Object writeReplace() throws ObjectStreamException;
 * </PRE><p>
 *
 * This writeReplace method is invoked by serialization if the method
 * exists and it would be accessible from a method defined within the
 * class of the object being serialized. Thus, the method can have private,
 * protected and package-private access. Subclass access to this method
 * follows java accessibility rules. <p>
 *
 * Classes that need to designate a replacement when an instance of it
 * is read from the stream should implement this special method with the
 * exact signature.
 *
 * <PRE>
 * ANY-ACCESS-MODIFIER Object readResolve() throws ObjectStreamException;
 * </PRE><p>
 *
 * This readResolve method follows the same invocation rules and
 * accessibility rules as writeReplace.<p>
 *
 * Enum types are all serializable and receive treatment defined by
 * the <a href="{@docRoot}/../specs/serialization/index.html"><cite>
 * Java Object Serialization Specification</cite></a> during
 * serialization and deserialization. Any declarations of the special
 * handling methods discussed above are ignored for enum types.<p>
 *
 * Record classes can implement {@code Serializable} and receive treatment defined
 * by the <a href="{@docRoot}/../specs/serialization/serial-arch.html#serialization-of-records">
 * <cite>Java Object Serialization Specification,</cite> Section 1.13,
 * "Serialization of Records"</a>. Any declarations of the special
 * handling methods discussed above are ignored for record types.<p>
 *
 * The serialization runtime associates with each serializable class a version
 * number, called a serialVersionUID, which is used during deserialization to
 * verify that the sender and receiver of a serialized object have loaded
 * classes for that object that are compatible with respect to serialization.
 * If the receiver has loaded a class for the object that has a different
 * serialVersionUID than that of the corresponding sender's class, then
 * deserialization will result in an {@link InvalidClassException}.  A
 * serializable class can declare its own serialVersionUID explicitly by
 * declaring a field named {@code "serialVersionUID"} that must be static,
 * final, and of type {@code long}:
 *
 * <PRE>
 * ANY-ACCESS-MODIFIER static final long serialVersionUID = 42L;
 * </PRE>
 *
 * If a serializable class does not explicitly declare a serialVersionUID, then
 * the serialization runtime will calculate a default serialVersionUID value
 * for that class based on various aspects of the class, as described in the
 * <a href="{@docRoot}/../specs/serialization/index.html"><cite>Java Object Serialization
 * Specification.</cite></a> This specification defines the
 * serialVersionUID of an enum type to be 0L. However, it is <em>strongly
 * recommended</em> that all serializable classes other than enum types explicitly declare
 * serialVersionUID values, since the default serialVersionUID computation is
 * highly sensitive to class details that may vary depending on compiler
 * implementations, and can thus result in unexpected
 * {@code InvalidClassException}s during deserialization.  Therefore, to
 * guarantee a consistent serialVersionUID value across different java compiler
 * implementations, a serializable class must declare an explicit
 * serialVersionUID value.  It is also strongly advised that explicit
 * serialVersionUID declarations use the {@code private} modifier where
 * possible, since such declarations apply only to the immediately declaring
 * class--serialVersionUID fields are not useful as inherited members. Array
 * classes cannot declare an explicit serialVersionUID, so they always have
 * the default computed value, but the requirement for matching
 * serialVersionUID values is waived for array classes.
 *
 * @see java.io.ObjectOutputStream
 * @see java.io.ObjectInputStream
 * @see java.io.ObjectOutput
 * @see java.io.ObjectInput
 * @see java.io.Externalizable
 * @see <a href="{@docRoot}/../specs/serialization/index.html">
 *      <cite>Java Object Serialization Specification</cite></a>
 * @since   1.1
 */
public interface Serializable {
}
```

Как видно из объявления - это интерфейс-маркер, который 'включает' сериализацию у классов:

```text
* Serializability of a class is enabled by the class implementing the
* java.io.Serializable interface.
```

Раз 'включает' как маркер, то значит работает с помощью `Reflection`: класс раскладывается на поля и метаданные, после чего уже пишется в выходной поток.

Если постараться коротко описать как это происходит: с помощью `Reflection` берется класс сериализуемого объекта, далее берется список полей, проверяются различные условия, например, если поле - это объект, то надо проверить можно ли его сериализовать, после чего значения пишутся в поток.

Продемонстрируем сериализацию и десериализацию.

Создадим класс `Entity`:

```java
public class Entity implements Serializable {
    private int a;
    private String hello;

    public Entity(int a, String hello) {
        System.out.println("Constructor Entity with all parameters");

        this.a = a;
        this.hello = hello;
    }

    // C 17-й Java, если у вас версия ниже - используйте StringBuilder
    @Override
    public String toString() {
        return new StringJoiner(", ", Entity.class.getSimpleName() + "[", "]")
                .add("a=" + a)
                .add("hello='" + hello + "'")
                .toString();
    }
}
```

Попробуем сериализовать объект и десериализовать его.

Для демонстрации того как (и как часто) вызывается конструктор добавим строчку с `System.out.println`, также добавим строчки до сериализации и после десериализации:

```java
public class Main {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Entity example = new Entity(14, "Hello");

        System.out.println("Before Serialization: " + example);

        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(baos);

        oos.writeObject(example);

        ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bais);

        Entity from = (Entity) ois.readObject();

        System.out.println("After Deserialization: " + from);
    }
}
```

В данном примере намеренно опускается все, что касается работы с ошибками, закрытия ресурсов и т.д. - для простоты.

Запустим наш пример:

```java
Constructor Entity with all parameters
Before Serialization: Entity[a=14, hello='Hello']
After Deserialization: Entity[a=14, hello='Hello']
```

Обратите внимание, что конструктор в этом коде вызвался только один раз: когда впервые создали объект, при десериализации вызова конструктора не было!

Здесь мы можем подвести первый итог: при использовании `java.io.Serializable` во время десериализации объекта **не вызывается** конструктор класса! Это необходимо учитывать, особенно, если в конструкторе расположена дополнительная логика.

Следующий момент, который интересен - это как стандартная сериализация работает с наследованием.

---

**Вопрос**:

Что будет при запуске кода, написанного выше, но при условии, что мы уберем реализацию `java.io.Serializable` у `Entity`?

**Ответ**:

Будет выброшено исключения **в момент** попытки сериализовать объект:

```java
Constructor Entity with all parameters
Before Serialization: Entity{a=14, hello='Hello'}
Exception in thread "main" java.io.NotSerializableException: Entity
    at java.base/java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1197)
    at java.base/java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:354)
    at Main4.main(Main4.java:16)
```

### Наследование и java.io.Serializable

Теперь представим ситуацию, что существует некоторая иерархия наследования. Для этого сделаем класс `Customer` и просто отнаследуемся от нашего класса `Entity`

```java
public class Customer extends Entity {
    private String name;

    public Customer(int a, String hello, String name) {
        super(a, hello);
        this.name = name;

        System.out.println("Constructor Customer with all parameters");
    }

    @Override
    public String toString() {
        return new StringJoiner(", ", Customer.class.getSimpleName() + "[", "]")
                .add("super='" + super.toString() + "'")
                .add("name='" + name + "'")
                .toString();
    }
}
```

Теперь попробуем сериализовать/десериализовать `Customer`:

```java
public class Main {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Customer example = new Customer(14, "Hello", "Aleksandr");

        System.out.println("Before Serialization: " + example);

        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(baos);

        oos.writeObject(example);

        ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bais);

        Customer from = (Customer) ois.readObject();

        System.out.println("After Deserialization: " + from);
    }
}
```

Запустим код:

```text
Constructor Entity with all parameters
Constructor Customer with all parameters
Before Serialization: Customer[super='Entity[a=14, hello='Hello']', name='Aleksandr']
After Deserialization: Customer[super='Entity[a=14, hello='Hello']', name='Aleksandr']
```

Отсюда можно сделать еще один вывод: реализация интерфейса-маркера у родительского класса делает всю иерархию классов сериализуемой.

Теперь посмотрим что будет, если дочерний класс реализует интерфейс `java.io.Serializable`, а родительский нет?

```java
class Parent {
    protected int a;

    public Parent(int a) {
        System.out.println("Parent constructor");
        this.a = a;
    }

    @Override
    public String toString() {
        return new StringJoiner(", ", Parent.class.getSimpleName() + "[", "]")
                .add("a=" + a)
                .toString();
    }
}

class Child extends Parent implements Serializable {
    private String test;

    public Child(int a, String test) {
        super(a);
        this.test = test;

        System.out.println("Child constructor");
    }

    @Override
    public String toString() {
        return new StringJoiner(", ", Child.class.getSimpleName() + "[", "]")
                .add("test='" + test + "'")
                .add("a=" + a)
                .toString();
    }
}
```

Попробуем сериализовать `Child`:

```java
public class Main {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Child example = new Child(14, "Hello");

        System.out.println("Before Serialization: " + example);

        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(baos);

        oos.writeObject(example);

        ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bais);

        Child from = (Child) ois.readObject();

        System.out.println("After Deserialization: " + from);
    }
}
```

В итоге сериализовать **НЕ** получится!
При этом будет выброшено исключение: `java.io.InvalidClassException`:

```text
Parent constructor
Child constructor
Before Serialization: Child[test='Hello', a=14]
Exception in thread "main" java.io.InvalidClassException: Child; no valid constructor
```

Давайте посмотрим повнимательнее на исключение:

```java
/**
 * Thrown when the Serialization runtime detects one of the following
 * problems with a Class.
 * <UL>
 * <LI> The serial version of the class does not match that of the class
 *      descriptor read from the stream
 * <LI> The class contains unknown datatypes
 * <LI> The class does not have an accessible no-arg constructor
 * <LI> The ObjectStreamClass of an enum constant does not represent
 *      an enum type
 * <LI> Other conditions given in the <cite>Java Object Serialization
 *      Specification</cite>
 * </UL>
 *
 * @since   1.1
 */
public class InvalidClassException extends ObjectStreamException {
    // code
}
```

Т.е. причина исключения: `The class does not have an accessible no-arg constructor`.

Добавим конструктор без параметров в родительский класс `Person`:

```java
public Parent() {
    System.out.println("Parent default constructor");
}
```

И снова запустим код, вывод будет следующий:

```java
Parent constructor
Child constructor
Before Serialization: Child[test='Hello', a=14]
Parent default constructor
After Deserialization: Child[test='Hello', a=0]
```

Получаем следующий тезис: если родительский класс не реализует `java.io.Serializable`, то данные, которые принадлежат ему не сериализуются, а при десериализации используется конструктор без параметров.

И еще: конструктор по-умолчанию должен иметь модификатор доступа `public` либо `protected`.

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

Не всегда нужно сериализовать все поля класса и в некоторых случаях состоянием объекта являются поля, которые необходимо исключить, не сериализовывать их.

За исключение полей из сериализации отвечает ключевое слово `transient`, т.е временный, скоротечный.

Добавим `transient` к нашему `Entity` классу к полю `hello`:

```java
public class Entity implements Serializable {
    private int a;
    private transient  String hello;

    public Entity(int a, String hello) {
        System.out.println("Constructor with all parameters");

        this.a = a;
        this.hello = hello;
    }

    @Override
    public String toString() {
        return new StringJoiner(", ", Entity.class.getSimpleName() + "[", "]")
                .add("a=" + a)
                .add("hello='" + hello + "'")
                .toString();
    }
}
```

Попробуем (ровно как в самом начале это делали, запустить тот же `Main` можно) сериализовать это и десериализовать:

```text
Constructor with all parameters
Before Serialization: Entity[a=14, hello='Hello']
After Deserialization: Entity[a=14, hello='null']
```

Отсюда видим, что значение поля не сериализовалось, а при десериализации поля, инициализировалось значением по умолчанию.

Это может привести к `NPE` исключениям, если вы не объявили для ссылочных полей значения по умолчанию. Или же привести к трудноуловимым ошибкам, когда примитивное поле, отмеченное `transient` будет при десериализации инициализировано значением, которое не является в данном контексте верным.

Поэтому старайтесь явно присваивать `transient` полям значения по умолчанию.

### Serial Version UID

Представим ситуацию, что объект класса `Entity` сериализовали в байтовое представление, сохранили на диск и через каоке-то время решили десериализовать обратно.

Однако, за это время класс поменялся, добавили новое поле (или удалили уже ранее существовашвее).

В таком случае, десериализовать объект не получится и будет выброшено исключение:

```java
java.io.InvalidClassException: Entity; local class incompatible: stream classdesc serialVersionUID = 2463605269252763973, local class serialVersionUID = -5068758618598267145
```

Это значит, что существует специальное поле, которое содержит уникальный идентификатор версии сериализованного класса (для проверки совместимости).

Поле выглядит следующим образом:

```java
private static final long serialVersionUID
```

И добавляется в класс на этапе компиляции (при условии, что класс реализует интерфейс `java.io.Serializable`).

Поле записывается в поток сериализации, а при десериализации объекта мы сравниваем его с тем, которое добавлено на этапе компиляции в класс (т.е у класса, загруженного в `JVM`) и если эти значения не совпадают, то бросается исключение.

Если какое-то поле будет удалено или изменено (например тип этого поля), будут изменены методы, то изменится `serialVersionUID`, а значит объекты класса, сериализованные по старому образцу не могут быть десериализованы по новому.

По сути, это такой идентификатор, как хеш, который показывает: были ли изменения у этого класса, по сравнению с той версией, которую мы сериализовали (помните, что пишется метадата в поток?).

Значение поля высчитывается автоматически и крайне чувствительно к деталям структуры класса, даже если вы поменяете местами объявления полей класса - это уже приведет к тому, что `serialVersionUID` изменится.

А раз так, то это потребует полной перекомпиляции классов (везде, где будет использоваться такой класс при сериалиацзии/десериализации), что довольно накладно и неудобно. А иногда и вовсе не выполнимо.

Поэтому **рекомендуют** это поле объявлять в явном виде:

```java
public class Entity implements Serializable {
    private static final long serialVersionUID = 4058626046366605816L;
    // ...
}
```

Таким образом вы будете явно контролировать совместимость текущего класса с другими версиями и при необходимости (при внесении ломающих совместимость изменений) инкрементировать версию.

Значение этого поля может быть абсолютно любым, некоторые часто ставят значение `1L`, хотя я бы не рекомендовал так делать.
Можно также воспользоваться утилитой `serialver` и вычислить `serialVersionUID`.

## java.io.Externalizable

Второй способ сериализовать объект в `Java` - это реализация интерфейса `java.io.Externalizable`.

Данный способ используется гораздо реже.

Интерфейс `java.io.Externalizable`, в отличии от `java.io.Serializable`, не является интерфейсом-маркером, его объявление выглядит следующим образом:

```java
/**
 * Only the identity of the class of an Externalizable instance is
 * written in the serialization stream and it is the responsibility
 * of the class to save and restore the contents of its instances.
 *
 * The writeExternal and readExternal methods of the Externalizable
 * interface are implemented by a class to give the class complete
 * control over the format and contents of the stream for an object
 * and its supertypes. These methods must explicitly
 * coordinate with the supertype to save its state. These methods supersede
 * customized implementations of writeObject and readObject methods.<br>
 *
 * Object Serialization uses the Serializable and Externalizable
 * interfaces.  Object persistence mechanisms can use them as well.  Each
 * object to be stored is tested for the Externalizable interface. If
 * the object supports Externalizable, the writeExternal method is called. If the
 * object does not support Externalizable and does implement
 * Serializable, the object is saved using
 * ObjectOutputStream. <br> When an Externalizable object is
 * reconstructed, an instance is created using the public no-arg
 * constructor, then the readExternal method called.  Serializable
 * objects are restored by reading them from an ObjectInputStream.<br>
 *
 * An Externalizable instance can designate a substitution object via
 * the writeReplace and readResolve methods documented in the Serializable
 * interface.<br>
 *
 * @see java.io.ObjectOutputStream
 * @see java.io.ObjectInputStream
 * @see java.io.ObjectOutput
 * @see java.io.ObjectInput
 * @see java.io.Serializable
 * @since   1.1
 */
public interface Externalizable extends java.io.Serializable {
    /**
     * The object implements the writeExternal method to save its contents
     * by calling the methods of DataOutput for its primitive values or
     * calling the writeObject method of ObjectOutput for objects, strings,
     * and arrays.
     *
     * @serialData Overriding methods should use this tag to describe
     *             the data layout of this Externalizable object.
     *             List the sequence of element types and, if possible,
     *             relate the element to a public/protected field and/or
     *             method of this Externalizable class.
     *
     * @param     out the stream to write the object to
     * @throws    IOException Includes any I/O exceptions that may occur
     */
    void writeExternal(ObjectOutput out) throws IOException;

    /**
     * The object implements the readExternal method to restore its
     * contents by calling the methods of DataInput for primitive
     * types and readObject for objects, strings and arrays.  The
     * readExternal method must read the values in the same sequence
     * and with the same types as were written by writeExternal.
     *
     * @param     in the stream to read data from in order to restore the object
     * @throws    IOException if I/O errors occur
     * @throws    ClassNotFoundException If the class for an object being
     *            restored cannot be found.
     */
    void readExternal(ObjectInput in) throws IOException, ClassNotFoundException;
}
```

Грубо говоря, это описание своего протокола сериализации для объектов класса.

В отличии от `java.io.Serializable` никаких метаданных не пишется, поэтому для использования `java.io.Externalizable` необходимо, чтобы у всех классов, в том числе и дочерних, были конструкторы по умолчанию.

Так как с его помощью будет создан объект, а на нем уже будет вызван метод из `java.io.Externalizable`, который и восстановит состояние объекта.

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

---

**Вопрос**:

Можно ли в таком подходе сериализовать `transient` поле?

**Ответ**:

Да, это возможно!

При использовании `java.io.Externalizable` вы сами управляете тем что сериализуется и как инициализируется при десериализации, поэтому никто не мешает сериализовать и десериализовать такое поле.

---

**Вопрос**:

Что делать с `final` полями?

**Ответ**:

При необходимости в сериализации/десериализации `final` полей использовать `java.io.Externalizable` не выйдет. Все дело в том, что `final` поля должны быть инициализированы в конструкторе, а при `java.io.Externalizable` инициализация полей происходит в `readExternal`.

Поэтому при необходимости в сериализации/десериализации `final` полей использовать можно только `java.io.Serializable`.

---

## Кастомизация java.io.Serializable

При этом существует возможность кастомизировать даже стандартную сериализацию. Для этого в класс, реализующий `java.io.Serializable`, добавляются методы, отвечающие за сериализацию/десериализацияю класса (внимательно с сигнатурами):

```java
 private void writeObject(java.io.ObjectOutputStream out) throws IOException
 private void readObject(java.io.ObjectInputStream in) throws IOException, ClassNotFoundException;
 private void readObjectNoData() throws ObjectStreamException;
```

Этим методы будут вызваны при сериализации/десериализации и в поток данные попадут уже с помощью них. По сути, это добавление гибкости к стандартному механизму `java.io.Serializable`.

```java
public class Entity implements Serializable {
    private int a;
    private String hello;

    public Entity(int a, String hello) {
        System.out.println("Constructor with all parameters");

        this.a = a;
        this.hello = hello;
    }

    private void writeObject(java.io.ObjectOutputStream out) throws IOException {
        System.out.println("Write object method");
        out.writeInt(a);
    }

    private void readObject(java.io.ObjectInputStream in) throws IOException, ClassNotFoundException {
        System.out.println("Read object method");
        a = in.readInt();
    }

    private void readObjectNoData() throws ObjectStreamException {
        System.out.println("Read object no data");
    }

    @Override
    public String toString() {
        return new StringJoiner(", ", Entity.class.getSimpleName() + "[", "]")
                .add("a=" + a)
                .add("hello='" + hello + "'")
                .toString();
    }
}
```

Результат будет:

```text
Constructor with all parameters
Before Serialization: Entity[a=14, hello='Hello']
Write object method
Read object method
After Deserialization: Entity[a=14, hello='null']
```

Данный пример мало чем отличается от примера с `java.io.Externalizable`. Здесь не используется механизм `java.io.Serializable`, так как им не воспользовались: мы вручную писали в поток.

Чтобы вызвать `default` механизм сериализации/десериализации, который предоставляет `java.io.Serializable`, существуют методы у `java.io.ObjectOutputStream`:

```java
public void defaultWriteObject() throws IOException
```

И у `java.io.ObjectInputStream` соответственно:

```java
public void defaultReadObject()  throws IOException, ClassNotFoundException
```

Эти методы могут быть вызваны только в `writeObject` и `readObject`, соответственно, иначе произойдет исключение `java.io.NotActiveException`.

Для примера попробуем сериализовать и десериализовать следующий класс:

```java
public class Entity implements Serializable {
    private int a;
    private String hello;

    public Entity(int a, String hello) {
        System.out.println("Constructor with all parameters");

        this.a = a;
        this.hello = hello;
    }

    private void writeObject(java.io.ObjectOutputStream out) throws IOException {
        System.out.println("Write object method");
        out.defaultWriteObject();

        out.writeInt(100);
    }

    private void readObject(java.io.ObjectInputStream in) throws IOException, ClassNotFoundException {
        System.out.println("Read object method");
        in.defaultReadObject();

        System.out.println("After default read object method" + this);

        a = in.readInt();
    }

    private void readObjectNoData() throws ObjectStreamException {
        System.out.println("Read object no data");
    }

    @Override
    public String toString() {
        return new StringJoiner(", ", Entity.class.getSimpleName() + "[", "]")
                .add("a=" + a)
                .add("hello='" + hello + "'")
                .toString();
    }
}
```

Результат будет:

```text
Constructor with all parameters
Before Serialization: Entity[a=14, hello='Hello']
Write object method
Read object method
After default read object methodEntity[a=14, hello='Hello']
After Deserialization: Entity[a=100, hello='Hello']
```

Важно отметить здесь, что после `in.defaultReadObject();` у нас был десериализованный класс и после этого, считав из потока последний записанный туда `int`, мы переприсвоили его полю `a`.

Эти методы бывают полезны, если, например, если вы хотите использовать стандартную сериализацию, но вам дополнительно надо добавить некоторые поля/значения в поток.

Напрмиер, так поступают в `java.io.File`, для сохранения разделителя файла:

```java
public class File
    implements Serializable, Comparable<File>
{

    /**
     * The system-dependent default name-separator character.  This field is
     * initialized to contain the first character of the value of the system
     * property {@code file.separator}.  On UNIX systems the value of this
     * field is {@code '/'}; on Microsoft Windows systems it is {@code '\\'}.
     *
     * @see     java.lang.System#getProperty(java.lang.String)
     */
    public static final char separatorChar = fs.getSeparator();

    // ...

    private synchronized void writeObject(java.io.ObjectOutputStream s)
        throws IOException
    {
        s.defaultWriteObject();
        s.writeChar(separatorChar); // Add the separator character
    }
}
```

Также это может быть полезно, если вы отнаследовались от сериализуемого класса, но по каким-то причинам вы не хотите, чтобы класс наследник был серализуемый - вы можете сделать вот так:

```java
private void writeObject(ObjectOutputStream out) throws IOException {
    throw new NotSerializableException("Uuups!");
}
private void readObject(ObjectInputStream in) throws IOException {
    throw new NotSerializableException("Uuups!");
}
```

Также использование этих методов может помочь в ситуациях, если вы поменяли структуру сериализуемого класса и не поменяли (или не хотите менять) `serialVersionUID`: чтобы поддержать обратную совместимость (по сути в ручном режиме).

## Работа с Singleton

Бывают ситуации, когда объект надо восстановить именно по состоянию, без ориентирования на метаданные и так далее.

Т.е. возвращать уже существующий экземпляр класса, соответствующий внутреннему состоянию десериализованного объекта.

Для этого существует метод:

```java
private Object readResolve() throws ObjectStreamException
```

Так как при десериализации мы получаем **другой** объект, но с тем же состоянием, что и сериализованный, то возникает некоторая проблема с [singleton](../../patterns/programming/creational/singleton.md): ведь поулчается, что создается еще один объект и это нарушает принцип паттерна.

Здесь нам как раз необходимо возвращать уже существующий экземпляр класса, соответствующий внутреннему состоянию десериализованного объекта (тот самый синглтон).

Воспользуемся методом `readResolve`:

```java
public class Answer implements Serializable {

    private static final String STR_YES = "Yes";
    private static final String STR_NO = "No";

    public static final Answer YES = new Answer(STR_YES);
    public static final Answer NO = new Answer(STR_NO);

    private String answer = null;

    private Answer(String answer) {
        this.answer = answer;
    }

    private Object readResolve() throws ObjectStreamException {
        if (STR_YES.equals(answer)) {
            return YES;
        }
        
        if (STR_NO.equals(answer)) {
            return NO;
        }
        
        throw new InvalidObjectException("Unknown value: " + answer);
    }
}
```

Пример я взял [отсюда](http://www.skipy.ru/technics/serialization.html#singleton).

Для `enum` в `Java` сделано нечто подобное, но внутри `JVM`, поэтому можно не переживать за сериализацию/десериализацию `enum`.

## The Serialization Proxy Pattern

Прокси сериализации предоставляют способ сериализации экземпляра класса путем создания "прокси", представляющего логическое состояние экземпляра. Такой подход позволяет избежать ловушек сериализации по умолчанию, не полагаясь на фактический поток байтов класса.

Т.е. сериализуется специальная "обертка", а при десериализации из обертки собирается объект.

Для этого, помимо `readResolve`, необходимо еще управлять потоком сериализации (другой класс же сериализуем - обертку!), существует метод:

```java
private Object writeReplace()
```

Этот метод позволяет объекту "назначить" замену себе перед тем, как писать в поток сериализации.

Посмотрим как это реализовать на нашем `Entity`:

```java
public class Entity implements Serializable {
    private String hello;

    public Entity(String hello) {
        System.out.println("Constructor with all parameters");

        this.hello = hello;
    }

    private Object writeReplace() {
        System.out.println("Write replace method");

        return new SerializationProxy(this);
    }

    private static class SerializationProxy implements Serializable {
        private final String data;

        SerializationProxy(Entity myClass) {
            this.data = myClass.getHello();
        }

        private Object readResolve() {
            System.out.println("Read resolve method");

            return new Entity(this.data);
        }
    }

    public String getHello() {
        return hello;
    }

    @Override
    public String toString() {
        return new StringJoiner(", ", Entity.class.getSimpleName() + "[", "]")
                .add("hello='" + hello + "'")
                .toString();
    }
}
```

Вывод:

```java
Constructor with all parameters
Before Serialization: Entity[hello='Hello']
Write replace method
Read resolve method
Constructor with all parameters
After Deserialization: Entity[hello='Hello']
```

Плюсы:

Каждое создание экземпляра проходит через конструктор(ы)/фабрики, и весь код, необходимый для правильной инициализации экземпляра, всегда выполняется.

Это также подразумевает, что такой код не должен явно вызываться во время десериализации, что предотвращает его дублирование.

Отсутствие ограничений на `final` поля
Поскольку десериализованный экземпляр инициализируется в своем конструкторе, этот подход не ограничивает, какие поля могут быть `final`.

Гибкое создание экземпляров
На самом деле `readResolve` прокси не обязательно должен возвращать экземпляр того же типа, который был сериализован. Он также может возвращать любой подкласс.

Минусы:

Потенциально, это может быть более дорогостоящее действие в плане производительности.

Тяжело использовать с наследованием.

Подробнее [тут](https://nipafx.dev/java-serialization-proxy-pattern/)

## java.io.Serial

Начиная с `Java 11+` была введена специальная аннотация `Serial`, чтобы помечать все, связанные с сериализацией действия (методы, поля).

Вот ее объявление:

```java
/**
 * Indicates that an annotated field or method is part of the {@linkplain
 * Serializable serialization mechanism} defined by the
 * <cite>Java Object Serialization Specification</cite>. This
 * annotation type is intended to allow compile-time checking of
 * serialization-related declarations, analogous to the checking
 * enabled by the {@link java.lang.Override} annotation type to
 * validate method overriding. {@code Serializable} classes are encouraged to
 * use {@code @Serial} annotations to help a compiler catch
 * mis-declared serialization-related fields and methods,
 * mis-declarations that may otherwise be difficult to detect.
 *
 * <p>Specifically, annotations of this type should be
 * applied to serialization-related methods and fields in classes
 * declared to be {@code Serializable}. The five serialization-related
 * methods are:
 *
 * <ul>
 * <li>{@code private void writeObject(java.io.ObjectOutputStream stream) throws IOException}
 * <li>{@code private void readObject(java.io.ObjectInputStream stream) throws IOException, ClassNotFoundException}
 * <li>{@code private void readObjectNoData() throws ObjectStreamException}
 * <li><i>ANY-ACCESS-MODIFIER</i> {@code Object writeReplace() throws ObjectStreamException}
 * <li><i>ANY-ACCESS-MODIFIER</i> {@code Object readResolve() throws ObjectStreamException}
 * </ul>
 *
 * The two serialization-related fields are:
 *
 * <ul>
 * <li>{@code private static final ObjectStreamField[] serialPersistentFields}
 * <li>{@code private static final long serialVersionUID}
 * </ul>
 *
 * Compilers are encouraged to validate that a method or field marked with a
 * {@code @Serial} annotation is one of the defined serialization-related
 * methods or fields declared in a meaningful context and issue a warning
 * if that is not the case.
 *
 * <p>It is a semantic error to apply this annotation to other fields or methods, including:
 * <ul>
 * <li>fields or methods in a class that is not {@code Serializable}
 *
 * <li>fields or methods of the proper structural declaration, but in
 * a type where they are ineffectual. For example, {@code enum} types
 * are defined to have a {@code serialVersionUID} of {@code 0L} so a
 * {@code serialVersionUID} field declared in an {@code enum} type is
 * ignored. The five serialization-related methods identified above
 * are likewise ignored for an {@code enum} type.
 *
 * <li>in a class that is {@code Externalizable}:
 * <ul>
 *   <li> method declarations of {@code writeObject}, {@code
 *   readObject}, and {@code readObjectNoData}
 *
 *  <li>a field declaration for {@code serialPersistentFields}
 * </ul>
 *
 * While the {@code Externalizable} interface extends {@code
 * Serializable}, the three methods and one field above are
 * <em>not</em> used for externalizable classes.
 *
 * </ul>
 *
 * Note that serialization mechanism accesses its designated fields
 * and methods reflectively and those fields and methods may appear
 * otherwise unused in a {@code Serializable} class.
 *
 * @see Serializable
 * @see Externalizable
 * @since 14
 */
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.SOURCE)
public @interface Serial {}
```

В таком случае наш пример с `Entity` должен выглядеть как:

```java
public class Entity implements Serializable {
    private String hello;

    public Entity(String hello) {
        System.out.println("Constructor with all parameters");

        this.hello = hello;
    }

    @Serial
    private Object writeReplace() {
        System.out.println("Write replace method");

        return new SerializationProxy(this);
    }

    private static class SerializationProxy implements Serializable {
        private final String data;

        SerializationProxy(Entity myClass) {
            this.data = myClass.getHello();
        }

        @Serial
        private Object readResolve() {
            System.out.println("Read resolve method");

            return new Entity(this.data);
        }
    }

    public String getHello() {
        return hello;
    }

    @Override
    public String toString() {
        return new StringJoiner(", ", Entity.class.getSimpleName() + "[", "]")
                .add("hello='" + hello + "'")
                .toString();
    }
}
```

## Безопасность

### Проблемы

У стандартного механизма есть ряд проблем:

Десериализация ненадежных данных: наиболее значительный риск возникает из-за десериализации данных из ненадежных источников. Это может привести к удаленному выполнению кода, поскольку метод `readObject` может в конечном итоге выполнить код без каких-либо явных инструкций от программиста. Особенно при халатном обращении с `serialVersionUID`.

Произвольное выполнение кода: во время десериализации классы часто загружаются динамически. Если злоумышленник может внедрить вредоносный байт-код в поток сериализации, он может выполнить произвольный код на `JVM`.

Отказ в обслуживании (DoS): объекты сериализации могут быть созданы для потребления чрезмерного объема памяти или ЦП при их десериализации, что приводит к DoS-атакам.

Раскрытие конфиденциальных данных: если конфиденциальные данные сериализованы и данные сериализации перехвачены, это может раскрыть конфиденциальную информацию злоумышленникам.

### Фильтры ObjectInputFilter

В `Java 9` появилась возможность добавить фильтр к потоку сериализации: `ObjectInputFilter`, который позволяет фильтровать классы, которые могут быть десериализованы.

Используйте его для указания разрешенных классов или для проверки различных атрибутов входящих сериализованных данных (например, размера массива, глубины и т.д.).

```java
ObjectInputFilter filter = ObjectInputFilter.Config.createFilter("java.base/*;!*");
ObjectInputStream ois = new ObjectInputStream(bais);
ois.setObjectInputFilter(filter);
```

### Security Managers

Используйте менеджер безопасности, который ограничивает действия, которые может выполнять `JVM`, особенно если вам необходимо выполнить десериализацию из ненадежных источников.

## Заключение

Если нет необходимости в кастомизации процесса сериализации/десериализации и производительность устраивает, то я не вижу смысла в использовании `java.io.Externalizable`.

Поэтому, наиболее часто встречаемым вариантом является стандартная сериализация через `java.io.Serializable`.

Необходимо помнить, что если родительский класс реализует интерфейс-маркер `java.io.Serializable`, то все дочерние классы также являются сериализуемыми.

Важным моментом является еще и то, что, если дочерний класс реализует `java.io.Serializable`, а родительский нет, то поля родительского конструктора будут инициализированы в конструкторе по умолчанию(у родительского класса, разумеется), если же конструктора по умолчанию нет, то будет выброшено исключение.

При необходимости можно также добавить кастомизированную логику по сериализации в стандартную реализацию `java.io.Serializable`, для этого следует добавить методы:

```java
 private void writeObject(java.io.ObjectOutputStream out) throws IOException
 private void readObject(java.io.ObjectInputStream in) throws IOException, ClassNotFoundException;
 private void readObjectNoData() throws ObjectStreamException;
```

Статические поля не сериализуются.

При сериализации стандартным способом в поток попадут метаданные класса и родительских классов, состояния объекта класса, который сериализуется и родительских.

Основным минусом может стандартной сериализации может являться не самая быстрая производительность, по сравнению с другими способами.

Подход с реализацией `java.io.Externalizable` более гибок и производителен, однако вся логика и ответственность по сериализации объекта теперь ложится на разработчика.

При грамотной реализации `java.io.Externalizable` можно получить ощутимый выигрыш в производительности: вот тут пишут про [выигрыш более чем в 10 раз](http://www.skipy.ru/technics/serialization.html#performance).

Однако учтите, что преждевременные оптимизации могут принести много горя, поэтому я еще раз повторю, если вас устраивает производительность стандартной сериализации, или вы вообще о ней не задумывались еще, то не стоит переходить на `java.io.Externalizable`.

Это может вызвать дополнительные трудности и сложности, а также ошибки, поэтому необходимо четко понимать, что вы получаете как плюсы подхода и что как минусы.

При этом, если класс реализует оба интерфейса - и `java.io.Externalizable`, и `java.io.Serializable`, то более высокий приоритет у `java.io.Externalizable`.

При `java.io.Externalizable` в поток не пишутся метаданные классов, также с его помощью можно сериализовать и статические поля, и поля, отмеченные как `transient`.

Однако, использование `final` полей будет невозможно при таком подходе, так как сериализовать их не составит труда, а вот десериализовать уже будет нельзя.

У стандартной сериализации есть ряд минусов с безопасностью и об этом надо помнить при работе с ней.

Необходимо избегать сериализации конфиденциальных данных.

На данный момент не рекомендуется использовать стандартную сериализацию `Java` для работы с публичными `API`, так как это потенциально несет в себе множество проблем как с производительностью, безопасностью так и с совместимостью. Лучше выбрать что-то более подходящее типа `json`, `protobuf`, `xml` и так далее.

С появлением новых методов и парадигм сериализации следует взвесить преимущества и недостатки традиционной сериализации `Java` по сравнению с современными альтернативами.

Однако, несмотря на все недостатки, она до сих пор достаточно широко применяется как в стандартной библиотеке `Java`, так и в популярных фреймворках, например, в `Spring` класс `org.springframework.security.core.context.SecurityContext`.

## Полезные ссылки

Рекомендую ознакомиться:

* [Сериализация как она есть](http://www.skipy.ru/technics/serialization.html#performance) - просто **must read**.
* [Guide to the Externalizable Interface in Java](https://www.baeldung.com/java-externalizable)
* [Java Object Serialization Specification: 2 - Object Output Classes](https://docs.oracle.com/en/java/javase/11/docs/specs/serialization/output.html#the-writereplace-method)
* [Interface Serializable](https://docs.oracle.com/javase/8/docs/api/java/io/Serializable.html)
* [Transient vs Static](https://javabeginnerstutorial.com/core-java-tutorial/transient-vs-static-variable-java/)
* [A Deep Dive into Java Serialization](https://medium.com/@AlexanderObregon/a-deep-dive-into-java-serialization-e514346ac2b2)
* [The Serialization Proxy Pattern](https://nipafx.dev/java-serialization-proxy-pattern/)
