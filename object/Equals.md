### Equals
Метод `equals()` содержится в `Object`, а значит есть у всех классов в `Java`.
Он является важнейшей частью работы с объектами.

Как выглядит по умолчанию:
```java
    public boolean equals(Object obj) {
        return (this == obj);
    }
```

Т.е по умолчанию он проверяет является ли объект obj тем же самым объектом, у которого вызвали `equals`.
Если сравнить два различных объекта с одними и теми же полями - получим `false`.

Так как же правильно работать с `equals`?

## Переопределяем equals
Основные правила:
* Проверяем на то, что obj является или нет `null`, чтобы обезопасить себя от `NPE`.
Если да - то `false`, если нет - `true`.
* Проверяем на то, что obj является или нет ссылкой на указанный объект.
Если да - то `true`, если нет - `false`.
* Проверяем на то, что объект имеет верный тип с помощью `instanceof`. Если нет - `false`.
* После этого приводим obj к правильному типу для дальнейшего сравнения.
* Проверяем необходимые поля объектов на равенство, если все равно - то `true`. Иначе  -`false`.

Советы:
* Для простых полей, кроме `double` и `float` используем обычное сравнение `==`, для `float` и `double` используем `Float.compare` и `Double.compare` соответственно. Так как существуют всякие `Float.NaN` и т.д.
* Для полей со ссылкой на объекты вызываем equals рекурсивно.
* Если поле у объекта *МОЖЕТ* принимать значение `null`, то используем:
```java
field == null ? o.field == null : field.equals(o.field)
```
* Не забываем также, что очередность сравнения влияет на производительность, поэтому сначала сравниваем поля, которые чаще других могут быть различны.

После этого проверим метод на транзитивность, симметричность и противоречивость, написав тесты. Обязательно переопределяйте `hashCode`.

**НЕ** пишите что-то типа:
```java
public boolean equals( MyClass obj){}
```
Этот метод не переопределяет `equals` у `Object`.

Пример реализации `equals`:
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
    public boolean equals(Object obj) {
        if (obj == null)
            return false;

        if (obj == this)
            return true;

        if (!(obj instanceof Person))
            return false;

        Person that = (Person) obj;

        if (that.age != this.age ||
                !that.name.equals(this.name) ||
                that.number != this.number ||  //need to check for NPE
                (Double.compare(that.salary, this.salary) != 0) ||
                that.carKey.equals(carKey)) //need to check for NPE
              {
            return false;
        }

        return true;
    }
  }
```

Все это можно записать проще, если мы используем `Java 7+` версии:
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
    public boolean equals(Object obj) {
        if (obj == null)
            return false;

        if (obj == this)
            return true;

        if (!(obj instanceof Person))
            return false;

        Person that = (Person) obj;

        return Objects.equals(age, that.age) && Objects.equals(number, that.number) &&
                Objects.equals(salary, that.salary) && Objects.equals(name, that.name) &&
                Objects.equals(carKey, that.carKey) && Objects.equals(carKey, that.carKey);
        }

        return true;
    }
  }
```
