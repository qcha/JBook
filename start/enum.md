# Enum

## Введение

Существует ряд задач, в которых требуется некоторый тип данных ограничить множеством допустимых значений.

Это могут быть дни недели(понедельник, вторник и т.д), времена года(весна, зима, лето, осень), типы животных(членистоногие, моллюски, иглокожие и т.д), [HTTP status code](https://ru.wikipedia.org/wiki/%D0%A1%D0%BF%D0%B8%D1%81%D0%BE%D0%BA_%D0%BA%D0%BE%D0%B4%D0%BE%D0%B2_%D1%81%D0%BE%D1%81%D1%82%D0%BE%D1%8F%D0%BD%D0%B8%D1%8F_HTTP) и прочее.

Т.е перечислить допустимые значения.

Так вот, в простейшем приближении можно сказать, что перечисление - это список именованных, логически связанных констант.

## Перечисления в Java

В `Java` перечисления появились с версии `1.5+`.

В `Java` перечисления создаются с помощью ключевого слова `enum`(enumeration - перечисление), затем идет список элементов перечисления через запятую:

```java
public enum NUMBER {
    ONE,
    TWO,
    THREE,
    FOUR
}
```

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

Элементы перечисления - экземпляры `enum`-класса, доступные статически.

Enum может реализовывать сколько угодно интерфейсов, но не может участвовать в наследовании. 

Это ограничение связано с тем, что `enum` **уже** неявно отнаследован от класса `java.lang.Enum`.

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

Еще одним ограничением является то, что для `enum` нельзя указывать модификаторы `final`  и `abstract`. Что в принципе логично, так как перечисления в наследовании участвовать не должны и абстрактными быть не могут - иначе что же это за перечисления?

Подробнее про [наследование](../oop/inheritance.md).

Также у перечисления есть возможность получить массив всех возможных значений этого перечисления в том порядке, в котором они объявлены. За это отвечает статический метод `values()`.

```java
for (Ex e : Ex.values())
    System.out.println(e);
```

Статический метод `valueOf(String name)`, вернет ссылку на константу перечисления по её имени.
Если перечисление не содержит элемента с именем name - выбрасывается `java.lang.IllegalArgumentException`:

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

---

**Вопрос**:

Можно ли перечисления сравнивать не через `equals`, а через `==`?

**Ответ**:

Можно!

Так как значения перечисления существуют в единственном экземпляре, поэтому их можно сравнивать их через `==`

```java
Season season = Season.SUMMER; 
if (season == Season.AUTUMN) season = Season.WINTER; 
```

---


Как уже было сказано выше, у перечисления могут быть конструкторы:

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
При этом, конструктор перечисления *НЕ* может использовать `super`!

---

**Вопрос**:

Могут ли у перечисления быть абстрактные методы? Ведь это же класс!

**Ответ**:

Могут!

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

---

## Заключение

Использование `enum` позволяет ограничить множество допустимых значений для некоторого типа данных.

Элементы enum - это статически доступные экземпляры enum-класса. 

Так как это класс, то `enum` может иметь констукрторы, методы и объявлять переменные.

## Полезные ссылки

[Enum in Java](http://www.quizful.net/post/java_enums)