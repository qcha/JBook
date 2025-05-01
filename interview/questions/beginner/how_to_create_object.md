# Как можно создать объект в Java?

Вопрос с реального собеседования в секции 'платформа'.

## Условие

Как можно создать объект в Java? Перечислите и продемонстрируйте все способы.

Иногда его можно услышать в формулировке: "Можно ли создать объект без использования оператора new в Java?"

## Решение

В `Java` существует 4 способа создать объект:

* Через оператор `new`
* Десериализация (например, из потока байт)
* С помощью рефлексии
* Метод `clone`

### Ключевое слово new

Наиболее распространенный способ - это вызов оператора `new`.

```java
Person p = new Person();
```

### Десериализация

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

Продемонстрируем работу:

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

Здесь мы продемонстрировали бинарную десериализацию, более подробнее про нее можно почитать в [статье](../../../serialization/binary/binary.md).

Заметим также, что возможна десериализация и из другого формата, например, json.

### Рефлексия

С помощью рефлексии можно получить конструкторы класса, выбрать нужный и вызвать его.

В качестве примера рассмотрим следующий код:

```java
package example;

import java.lang.reflect.InvocationTargetException;

public class Main {
    public static void main(String[] args) throws ClassNotFoundException, InvocationTargetException, InstantiationException, IllegalAccessException {
        Class<?> clazz = Class.forName("example.Person");
        Person person = (Person) clazz.getDeclaredConstructors()[0].newInstance();

        System.out.println(person);
    }
}

class Person {

}
```

Более подробно про [Reflection](../../../jcore/reflection/)

### Метод clone

Ну и последний способ - это с помощью метода `clone`:

```java
public class CloneTest implements Cloneable {
    private final int i;

    public CloneTest(int i) {
        this.i = i;
    }

    public int getI() {
        return i;
    }

    public static void main(String[] args) throws CloneNotSupportedException {
        CloneTest test = new CloneTest(2);
        CloneTest cloneTest = (CloneTest) test.clone();
        System.out.println("Original : " + test + ", i = " + test.getI());
        System.out.println("Clone : " + cloneTest + ", i = " + cloneTest.getI());
    }
}
```

Данный способ в современном мире используется редко, так как имеет множество подводных камней (как, например, deep clone и т.д.).

Более подробно про [clone](../../../jcore/object/clone.md)

### Unsafe

С помощью `sun.misc.Unsafe` (или `jdk.internal.misc.Unsafe` в более новых версиях) можно создать объект в `Java` без вызова конструктора.

```java
import sun.misc.Unsafe;
import java.lang.reflect.Field;

public class Main {
    static class MyClass {
        private MyClass() {
            System.out.println("Constructor called!");
        }
    }

    public static void main(String[] args) throws Exception {
        Field f = Unsafe.class.getDeclaredField("theUnsafe");
        f.setAccessible(true);
        Unsafe unsafe = (Unsafe) f.get(null);

        MyClass obj = (MyClass) unsafe.allocateInstance(MyClass.class);
        System.out.println("Object created without constructor: " + obj);
    }
}
```

Но надо помнить, что в последних версиях возможности Unsafe все больше ограничиваются.
