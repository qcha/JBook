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

Является ли родительский класс сериализуемым, если наследник реализует интерфейс `java.io.Serializable`, т.е:

```java
class Parent {
    // some code
}

class Child extends Parent implements Serializable {
    // some code
}
```

**Ответ**:

Нет, не будет! Поля родителя в поток не попадут.

// todo пример

---

У нас есть класс у которого также есть супер-класс родитель. Наш класс - serializable, будет ли родитель serializable? 
Тогда как будет работать при дессериализации?
Когда мы попытаемся дессериализовать наш класс - Мы вызовем конструктор супер класса(родительского) без параметров!
Сам же конструктор дессериализуемого класса не вызовется, ведь мы используем Serializable.
При это если у супер класса нет такого конструктора - мы поймаем ошибку.

Еще один интересный момент:
Если мы реализуем Serializable интерфейс и в класс добавим методы:
```java
 private void readObject(java.io.ObjectInputStream stream) throws IOException, ClassNotFoundException;
 private void writeObject(java.io.ObjectOutputStream stream) throws IOException;
 private void readObjectNoData() throws ObjectStreamException;
```

И когда мы попытаемся сериазовать объект - java вызовет эти методы! И будет писать/читать в поток уже с помощью них, поэтому использовать эти методы надо только в случае, если нам необходимо четко и гибко контроллировать весь процесс сериализации/дессериализации. При этом обязательно вызывайте default-методы, которые обеспечат стандартную сериализацию, а потом уже вносите какие-то свои изменения.

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

####Вывод:
В целом, я бы советовал использовать стандартный подход, если не требуется какой-то специальной гибкости и супер-быстродействия.
