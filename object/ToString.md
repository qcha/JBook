### toString
Данный метод вызывается всегда, когда вы передаете объект в `System.out.prinln()` и подобные методы.
Метод рекомендуется переопределять, так как ожидается, что возвращаемая строка будет легко читаемая и информативная.
Когда вы вызываете `toString()` на объекте Person, то желательно, чтобы возвращалось что-то типа: `Person name: Aleksandr, phone: -----`.

Хорошо реализованный toString помогает также при отладке кода, так как вы в логе видите что за объект был, какие были поля и какие стали.

Однако, если вы переопределяете метод `toString`, то возвращаемая строка должна содержать *всю* значимую информацию объекта.

Также надо понимать, что переопределив метод `toString` и возвращая значение полей объекта в нем, вы по=хорошему должны дать доступ к этим полям на чтение(сделать get-ы), так не сделав этого вы обрекаете других на единственный возможный путь получения информации из этих полей - парсинг вашей строки.

Ну и действительно, смысл убирать `getName()` у `Person`, если `toString` возвращает имя вашего `Person` объекта?

Заметим в примере ниже, что без переопределения toString у CarKey получается не совсем понятно, это было оставлено специально, для примера.

Пример переопределения:
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
    public String toString() {
      return String
            .format("Person name: %s, age: %d, number: %s, salary: %4.2f, carKey: %s",
                    name, age, number, salary, carKey);
    }
}
```
