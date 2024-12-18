# java.lang.Object#getClass

- [java.lang.Object#getClass](#javalangobjectgetclass)
    - [Введение](#введение)
    - [О методе](#о-методе)
    - [Заключение](#заключение)

## Введение

В `Java` мы оперируем классами и объектами.

Объявление класса нам привычно:

```java
public class Person {
    private int age;
}
```

Каждый объект является объектом какого-то класса.

Но что такое класс `Person` с точки зрения `Java`?

Ровно как `Person` описывает некую абстракцию над человеком, также и класс можно представить как набор набор полей, методов, конструкторов и так далее.

Для простоты мы можем объявить что-то в виде:

```java
class Class {
    private String name;
    private String packageName;
    private Field[] fields;
}
```

На самом деле все сложнее, но нам сейчас эта сложность будет мешать и потому мы ее опустим

Так вот, если описать таким образом наш `Person`, то получим абстракцию, которая содержит данные об объявленном классе. И каждый наш объявленный класс будет новым объектом класса `Class`.

В таком случае, у каждого объекта есть возможность получить информацию о своем классе, методах и полях, ведь вся эта мета-информация доступна у объекта `Class`, связанного, например, с `Person`. При этом получить эту информацию можно будет прямо во время выполенения программы - точно также как мы обращаемся к объекту `person` и его полям.

В `Java` уже существует класс, который является абстракцией для работы с классами - это `java.lang.Class`. Понятно, что его объявление гораздо сложнее, чем то, что мы предположили выше, но суть одна.

И для получения класса объекта в `java.lang.Object` объявлен метод `getClass`.

Объявление метода выглядит как:

```java
    /**
     * Returns the runtime class of this {@code Object}. The returned
     * {@code Class} object is the object that is locked by {@code
     * static synchronized} methods of the represented class.
     *
     * <p><b>The actual result type is {@code Class<? extends |X|>}
     * where {@code |X|} is the erasure of the static type of the
     * expression on which {@code getClass} is called.</b> For
     * example, no cast is required in this code fragment:</p>
     *
     * <p>
     * {@code Number n = 0;                             }<br>
     * {@code Class<? extends Number> c = n.getClass(); }
     * </p>
     *
     * @return The {@code Class} object that represents the runtime
     *         class of this object.
     * @jls 15.8.2 Class Literals
     */
    @IntrinsicCandidate
    public final native Class<?> getClass();
```

Ключевое слово `native` означает, что метод реализован в платформенно-зависимом коде, чаще всего на `C/C++`, и скомпонован в виде динамической библиотеки.

Эта реализация зависит от `JVM`.

Возможно, вас сейчас это напугало, но на самом деле достаточно просто понимать, что `native` означает лишь то, что вызываемый код, реализован не на `Java`.

## О методе

Данный метод является финальным и переопределить его нельзя.

Давайте взглянем на описание метода:

```text
     * Returns the runtime class of this {@code Object}. The returned
     * {@code Class} object is the object that is locked by {@code
     * static synchronized} methods of the represented class.
```

Благодаря этому, метод часто используется как проверка перед безопасным кастом объекта:

```java
        if (getClass() != o.getClass()) {
            return false;
        }

        Person that = (Person) obj;  // тот самый 'каст' obj к Person
        // Работаем с that как с Person
```

Важно отметить, что `getClass` вернет именно класс объекта, не взирая на ссылку:

```java
public class Main {
    public static void main(String[] args) {
        Person p = new Employee();
        System.out.println(p.getClass());
    }
}

class Person {

}

class Employee extends Person {

}
```

В `Java`существует специальный механизм, который называется `Reflection API` или **рефлексия**, который позволяет получить и совершить некоторые манипуляции с полями, методами и конструкторами класса прямо во время выполнения программы - в рантайме.

И метод `getClass` - это часть этого механизма.

Например, выведем имя класса:

```java
void printClassName(Object obj) {
    System.out.println("The object's" + " class is " +
        obj.getClass().getSimpleName());
}
```

Или вызовем явно метод:

```java
import java.lang.reflect.InvocationTargetException;

public class Main {
    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        Object p = new Person();
        p.getClass().getMethod("print").invoke(p);
    }
}

class Person {
    public void print() {
        System.out.println("Print!");
    }
}
```

Все это дает богатый простор для фантазии и возможностей работы с объектами прямо во время выполнения вашей программы (или на этапе старта).

## Заключение

Метод `getClass` служит для того, чтобы в рантайме получить информацию о классе объекта.

Применяется как для безопасного каста к объекту определенного типа, так и для работы с объектом в рантайме.
