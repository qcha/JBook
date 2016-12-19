### HashCode
Прежде всего, надо понимать, что hashCode наряду с equals играет важную роль в сравнении объектов. По сути он показывает изменилась ли информация, которую мы используем в equals для сравнения объектов.

Спецификация гласит:
* Метод hashCode должен вернуть одно и то же целое значение, если мы обратимся к одному и тому же объекту, если информация в объекте не поменялась.
* Если equals показывает, что два объекта равны - у таких объектов одинаковые hashCode.
* Если equals показывает, что объекты не равны, то **НЕ обязательно** у них разные hashCode.

Главное запомнить, что одинаковые объекты имеют *одинаковый* hashCode.

У Object-а:
```java
public native int hashCode();
```

Именно поэтому, если вы переопределяете equals - вы *обязаны* переопределить hashCode.
### Переопределяем
Самый очевидный(и самый плохой) пример как переопределить hashCode - это всегда возвращать какое-то число.
```java
@Override
public int hashCode() {
return 14;
}
```

Для равных объектов такой метод вернет одно и то же число, что удовлетворяет спецификации. Но таким образом мы будем сводить на нет все полезности hashmap, превращая их в связные списки.

Да и надо помнить, что хорошая функция hashCode - это гарантия быстродействия при работе с hashmap-ами.

При переопределении важно помнить, что необходимо исключить избыточные поля(которые не участвуют в equals).
Также, если необходимо и ваш объект - неизменяемый, то hashCode можно закешировать.


Пример переопределения hashCode:
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
    public int hashCode() {
      int result = 31;
      result = result * 17 + age;
      result = result * 17 + number;
      long lnum = Double.doubleToLongBits(number);
      result = result * 17 + (int)(lnum ^ (lnum >>> 32));
      result = result * 17 + name.hashCode();
      result = result * 17 + carKey.hashCode();

      return result;
    }
}
```

Правила Блоха:
* Присваиваем переменной result ненулевое значение.
* Далее для каждого значимого поля в объекте вычисляем hashCode:

 1.Если поле `boolean` - `(f ? 1 : 0)`

 2.Если поле `byte`, `char`, `short` или `int` - `(int)f`

 3.Если поле `long` - `(int)(f ^ (f >>> 32))`

 4.Если поле `float`, то `Float.floatToIntBits(f);`

 5.Если поле `double`, то `Double.doubleToLongBits(f)`, а затем как с `long`.

 6.Если поле это ссылка на другой объект, то рекурсивно вызовите `hashCode()`

 7.Если поле `null` - то возвращаем 0.

 8.Если поле это массив - то обрабатываем так, как будто каждый элемент массива - это поле объекта.

* После каждого обработанного поля объединяем его hashCode с текущим значением:
```java
result = 31 * result + c; //c - это hashCode обработанного поля.
```
* Возвращаем результат.
