# Enum

## Введение

Существует ряд задач, в которых требуется некоторый тип данных ограничить множеством допустимых значений.

Это могут быть дни недели, времена года, [HTTP status code](https://ru.wikipedia.org/wiki/%D0%A1%D0%BF%D0%B8%D1%81%D0%BE%D0%BA_%D0%BA%D0%BE%D0%B4%D0%BE%D0%B2_%D1%81%D0%BE%D1%81%D1%82%D0%BE%D1%8F%D0%BD%D0%B8%D1%8F_HTTP) и т.д.

Т.е перечислить допустимые значения.

Так вот, в простейшем приближении можно сказать, что перечисление - это список именованных, логически связанных констант.

## Перечисления

В `Java` перечисления создаются с помощью ключевого слова `enum`, затем идет список элементов перечисления через запятую:

```java
public enum NUMBER {
    ONE,
    TWO,
    THREE,
    FOUR
}
```

В `Java` перечисления появились с версии `1.5+`.

Главное, что стоит помнить, это то, что `enum` - это класс. Это значит, что можно добавить конструкторы, методы, объявить переменные.

```java
public enum WeekDay {
   SUNDAY ("Воскресенье"),
   MONDAY ("Понедельник"),
   TUESDAY ("Вторник"),
   WEDNESDAY ("Среда"),
   THURSDAY ("Четверг"),
   FRIDAY ("Пятница"),
   SATURDAY ("Суббота");

   private String title;

   WeekDay(String title) {
       this.title = title;
   }

   public String getTitle() {
       return title;
   }

   @Override
   public String toString() {
       return "WeekDay{" +
               "title='" + title + '\'' +
               '}';
   }
}
```

Enum может реализовывать сколько угодно интерфейсов, но не может участвовать в наследовании. 

Почему? Потому что enum **уже** неявно отнаследован от класса `java.lang.Enum`.

Благодрая чему имеет методы:

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
