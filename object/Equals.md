### Equals
Метод `equals()` содержится в Object, а значит есть у всех классов в Java.
Он является важнейшей частью работы с объектами.

Как выглядит по умолчанию:
```java
    public boolean equals(Object obj) {
        return (this == obj);
    }
```

Т.е по умолчанию он проверяет является ли объект obj тем же самым объектом, у которого вызвали equals.
Если сравнить два различных объекта с одними и теми же полями - получим `false`.

Так как же правильно работать с equals?

## Переопределяем equals
Основные правила:
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
Этот метод не переопределяет equals у Object.

Пример реализации equals:
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

        Person tPerson = (Person) obj;

        if (tPerson.age != this.age ||
                !tPerson.name.equals(this.name) ||
                tPerson.number != this.number ||
                (Double.compare(tPerson.salary, this.salary) != 0) ||
                tPerson.carKey.equals(carKey)) {
            return false;
        }

        return true;
    }
  }
```
