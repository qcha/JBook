# Enum

## Введение

Необходимостью ограничить множество допустимых значений для некоторого типа данных требует от нас использования некоторого специального типа.

Использование перечисления оправдано всегда, когда вам нужен набор предопределённых связанных между собой данных.

Например, времена года, дни недели и т.д

Объявляется очень просто:

```java
public enum NUMBER {
    ONE,
    TWO,
    THREE,
    FOUR
}
```

Стоит помнить, что enum - это класс.

Он может реализовывать сколько угодно интерфейсов, но не может наследоваться. Почему? Потому что enum *уже* неявно отнаследован от класса `java.lang.Enum.`

Благодрая этому наследованию каждый enum имеет методы:

* `public final int ordinal()` - возвращает порядковый номер константы в перечислении, считая от 0.
* `public final String name()` - возвращает имя константы так, как оно было объявлено в перечислении.

```java
public class EnumExample {
    public static void main(String[] args) {
        for (Ex e : Ex.values())
            System.out.println(e.ordinal() + " has name " + e.name());
    }
}

enum Ex {
    ONE, TWO, THREE
}
```

Нельзя указывать модификаторы `final`  и `abstract`  для перечислений.

К каждому перечислению компилятор добавляет статический метод `values()`, который возвращает массив возможных значений для перечисления в том порядке, в котором они объявлены, и статический метод valueOf(String name) , который возвращает ссылку на константу перечисления по её имени.

Если перечисление не содержит элемента с именем name - выбрасывается IllegalArgumentException.

```java
public class EnumExample {
    public static void main(String[] args) {
        System.out.println(Ex.ONE);
        System.out.println(Ex.valueOf("TWO"));
        System.out.println(Ex.valueOf("TWO_ERROR")); //exception
    }
}

enum Ex {
    ONE, TWO, THREE
}
```

Пример `values()`, печатаем все элементы перечисления:

```java
        for (Ex e : Ex.values())
            System.out.println(e);
```

Значения перечисления существуют в единственном экземпляре, поэтому можно сравнивать их через `==`

У перечисления могут быть конструкторы:

```java
public enum EnumExample {
    ONE("ONE_ONE") {
        @Override
        public void method() {
            System.out.println("Method of " + this.name);
        }
    }, TWO("TWO_TWO") {
        @Override
        public void method() {
            System.out.println("Method 2 of " + this.name);
        }
    };

    private String name;

    EnumExample(String name) {
        this.name = name;
    }

    public abstract void method();
}
```

Если в перечислении нет ни одного объявления конструктора, то автоматически добавляется конструктор по умолчанию без параметров, с модификатором доступа `private`.
Конструктор перечисления *НЕ* может использовать `super`!

У перечисления могут быть абстрактные-методы, тогда каждое элемент должен определить такой метод:

```java
public enum EnumExample {
    ONE("ONE_ONE") {
        @Override
        public void method() {
            System.out.println("Method of " + this.name);
        }
    }, TWO("TWO_TWO") {
        @Override
        public void method() {
            System.out.println("Method 2 of " + this.name);
        }
    };

    private String name;

    EnumExample(String name) {
        this.name = name;
    }

    public abstract void method();
}
```
