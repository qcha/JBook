# Binary Serialization in Java

## Введение

Одним из серьезных преимуществ `Java` является поддержка сериализации объекта 'из коробки'.

Все необходимое уже есть в `JDK`.

Так как умение объекта сериализовываться - это некоторое поведение, а за поведение в `Java` отвечают интерфейсы, то в `Java` существует целых два интерфейса, связанных с сериализацией:

* `java.io.Serializable`
* `java.io.Externalizable`

### java.io.Serializable

Универсальный способ сделать объект сериализуемым - это реализовать интерфейс-маркер `java.io.Serializable`.

Это наиболее часто встречаемый и используемый вариант, можно сказать, что это стандартный способ сериализации.

В `Java` он работает через `Reflection`: класс раскладывается на поля и метаданные, после чего уже пишется в выходной поток.

За эту универсальность, разумеется, надо платить и плата как раз заключается в использовании `Reflection`, который не дает максимальную производительность.

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

> Работа с ресурсами и их закрытие специально сделана наиболее просто(и не совсем верно), чтобы не пересоложнять пример.

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

---

И вот это поведение надо помнить.

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

При этом существует возможность кастомизировать даже стандартную сериализацию. Для этого в класс, реализующий  `java.io.Serializable` добавляются методы, отвечающие за сериализацию/десериализацияю класса:

```java
 private void writeObject(java.io.ObjectOutputStream out) throws IOException
 private void readObject(java.io.ObjectInputStream in) throws IOException, ClassNotFoundException;
 private void readObjectNoData() throws ObjectStreamException;
```

Этим методы будут вызваны при сериализации/десериализации и в поток данные попадут уже с помощью них, поэтому использовать эти методы надо только в случае, если нам необходимо четко и гибко контроллировать весь процесс сериализации/дессериализации. 

При этом обязательно вызывайте default-методы, которые обеспечат стандартную сериализацию, а потом уже вносите какие-то свои изменения.

В writeObject, например:
```java
private void writeObject(java.io.ObjectOutputStream stream) throws IOException {
stream.defaultWriteObject(); // default serialization
//our special serialization
}
```
Аналогично и тут:

В readObject, например:
```java
private void readObject(java.io.ObjectInputStream stream) throws IOException, ClassNotFoundException {
stream.defaultReadObject(); // default deserialization
//our special deserialization and using setters
}
```

Заметим, что все эти методы - private, что вполне логично, так как мы уже пишем сериализаццию только для этого нашего класса специально.
Также если мы отнаследовались от класса, который сериализуемый, но по каким-то причинам не хотим, чтобы наш класс был серализуемый - мы можем сделать вот так в нашем классе:

```java
private void writeObject(ObjectOutputStream out) throws IOException {
    throw new NotSerializableException("Uuups!");
}
private void readObject(ObjectInputStream in) throws IOException {
    throw new NotSerializableException("Uuups!");
}
```

Также стоит отметить еще и то, что поля помеченные `static` - не сериализуются.
Поля помеченные `transient` также не сериализуются.

Выводы и советы:
* Если родительский класс -  Serializable - все дочерние классы - serializable.
* Если дочерний класс Serializable, но родительский нет - поля родительского класса заполнятся как в дефолтном конструкторе, при этом он обязан быть!
* Процессом serialization and deserialization можно управлять добавив специальные методы в класс.
* При Serialization порядок записи: метаданные класса, метаданные родителя, данные родителя , данные класса.
* При Deserialization порядок записи: метаданные родителя, метаданные класса, данные класса, данные родителя.
* Если у нас есть поле-ссылка на не Serializable класс и это поле не `null` - мы словим ошибку.
* Поля помеченные `static` - не сериализуются.
* Прост в использовании - `java.io.Serializable` у всех сериализуемых объектов и все.
* Не такой уж и быстрый способ.

#### Externalizable interface
Как мы и говорили в начале - второй подход к сериализации.

В отличии от `java.io.Serializable` в данном интерфейсе у нас два метода – `writeExternal(ObjectOutput)` и`readExternal(ObjectInput)`. Именно этими методами мы и сериализуем/десериализуем объект. В данном случае мы не пишем никаких метаданных! 

Мы __только__ вызываем наши методы - и сериализуем наш класс как нам хочется. Это гибкий путь. Мы сами полностью описываем как мы будем сериализовать класс, при этом не пишем никаких метаданных. По сути - у нас свой протокол сериализации тут мы создаем!

Как это работает?
Прежде всего тут мы вызываем конструктор по умолчанию. Именно поэтому мы **обязаны** его иметь. Все дочерние классы также обязаны его иметь.
__После__ этого на новом объекте мы вызываем методы(readExternal) и все поля заполняются значениями.
Если поле `transient` - оно обязано иметь значение по умолчанию.

Вывод и советы:
* Если мы реализуем оба интерфейса - Externalizable и Serializable - то Externalizable имеет больший приоритет.
* Externalizable более гибок, более быстрый.
* Externalizable не пишем метаданные.
* Можем сериализовывать `Static` поля, хотя это плохой подход.
* Мы не можем десериализовать `final` поля класса в Externalizable - так как мы используем конструктор, хотя сериализуются они как обычно.

Также:
* После десериализации мы  __обязаны__ проверить объект на корректность - и кинуть `java.io.InvalidObjectException.` Если что-то не так.
* Будем испытывать проблемы с serialzie/deserialize Singleton объекта.

#### Serial Version UID
Хорошо бы еще иметь private static final long serialVersionUID поле.
В этом поле хранится уникальный id version сериализованного класса. Вычисляется из полей класса, порядком их объявления, методами и т.д. Если мы меняем класс - мы меняем и id.
Это поле записывается в поток сериализации, при десериализации объекта мы сравниваем id и если что-то не так - кидаем exception. Java _строго_ рекомендует объявлять это поле. Если его не объявлять, то, по-моему это поле будет браться из hash code.
Это гарантия того, что все прошло правильно.

#### Вывод:
В целом, я бы советовал использовать стандартный подход, если не требуется какой-то специальной гибкости и супер-быстродействия.
