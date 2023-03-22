# Generics

## Введение
Зачем нужны Generics?
`Generics` -  это механизм, который позволяет писать безопасный для типов код. С помощью Generics в Java можно создавать классы, интерфейсы и методы, которые могут работать с различными типами данных, без необходимости создавать отдельные версии этих элементов для каждого типа.

## Generics
Для указания параметризованного типа используют угловые скобки `(<>)`. Например, можно создать обобщенный класс, который будет хранить массив элементов любого типа:
```java
public class Test<T> {
  private T[] elements;
  
  public Test(T elements) {
    this.elements = elements;
  }
  
  public void printElems() {
    for (T element : elements) {
      System.out.print(element+" ");
    }
  }
  public T getFirstElem() {
    return elements[0];
  }
}
```
Здесь <T> означает, что класс Test параметризован типом T. При создании экземпляра класса Test, тип T будет заменен на конкретный тип данных. Например:
```java
Test<String> test = new Test<>(new String[]{"Hello", "World"});
test.printElems(); //Hello World

Test<Integer> test = new Test<>(new Integer[]{1, 2, 3});
test.printElems(); //1 2 3

```
Также, если вы заметили, в нашем классе есть метод, который возвращает первый элемент массива. Но ведь мы не знаем, какой тип данных будет у массива. Поэтому, чтобы избежать ошибок, мы можем указать, что метод должен возвращать объект типа T:
```java
test.getFirstElem(); //Hello
```
Но стоит отметить, что при указании типа T, мы не можем использовать примитивные типы. Например, следующий код не скомпилируется:
```java
Test<int> test = new Test<>(new int[]{1, 2, 3});
```
Поэтому, если вы хотите использовать примитивные типы, то вам нужно использовать их обертки:
```java
Test<Integer> test = new Test<>(new Integer[]{1, 2, 3});
```
Мы можем использовать несколько параметров типа. Например, можно создать класс, который будет хранить пару элементов любого типа:
```java
public class Pair<T, V> {
  private T first;
  private V second;
  
  public Pair(T first, V second) {
    this.first = first;
    this.second = second;
  }
  
  public T getFirst() {
    return first;
  }
  
  public V getSecond() {
    return second;
  }
}
```
Теперь, при создании экземпляра класса Pair, мы можем указать типы данных для параметров T и V:
```java
Pair<String, Integer> pair = new Pair<>("Hello", 1);
System.out.println(pair.getFirst()); //Hello
```
#### Рассмотрим пример принимаемых параметров, которые являются наследниками определенного класса

 Так как мы не можем указать несколько классов в качестве параметров типа, то нам нужно использовать ограничения. Для этого используется ключевое слово `extends`. Например, мы можем создать класс, который будет принимать в качестве параметров типа `Cat` и `Dog`, которые наследуются от класса `Animal`:
```java
// Родительский класс Animal
public class Animal {
  public void say() {
    System.out.println("Hello");
  }
}

// Классы Cat и Dog, которые наследуются от класса Animal
public class Cat extends Animal {
  @Override
  public void say() {
    System.out.println("Meow");
  }
}

public class Dog extends Animal {
  @Override
  public void say() {
    System.out.println("Woof");
  }
}

// Класс Test, который принимает в качестве параметров типа Cat и Dog
public class Test<T extends Animal> {
  private T[] elements;
  
  public Test(T elements) {
    this.elements = elements;
  }
  
  public void printElems() {
    for (T element : elements) {
      element.say();
    }
  }
}
```
Но, если мы попытаемся создать экземпляр класса Test, указав в качестве параметра типа класс `String`, то код не скомпилируется
## Wildcards
* Инвариантность

    Это когда можно подставлять только определенный тип.
* Ковариантность

    Это когда можно подставлять более конкретный тип, вместо более обобщенного.

    Это у нас `extends`.
* Контрвариантность

    Это когда можно подставлять более общий тип, вместо более конкретного.

    Это у нас `super`.

#### А теперь подробнее:

`List` в `Java` инвариантен, т.е если у меня есть два класса, один из которых наследник другого, например,  `List<String>` и `List<Object>`, то эти коллекции - не являются наследниками друг друга и подставить одну вместо другой мы не можем. Они инвариантны.
```java
List<String> strings = new ArrayList<>();
List<Object> objects = strings; //wrong!
```

Так делать запрещено! Почему? Потому что если бы такое было разрешено, то мы бы получали ошибки в рантайме, которые сложно отследить. Когда я бы у коллекции `objects` какой-нибудь элемент кастовал бы в `String`, а он был бы не `String`.

Чтобы это работало - надо использовать ограничения.
Так как у нас тут `list` - это `producer` данных, то использовать надо `extends`.
```java
List<String> strings = new ArrayList<>();
List<? extends String> objects = strings; //right!
```

Есть еще ограничения `super`. Если у нас коллекция - это `consumer`.
Коллекция потребляет данные, т.е мы туда что-то записываем.
Тогда можно написать так:
```java
    static void putAnimalToCollection(List<? super Animal> list) {
        list.add(new Cat("Kitty"));
        list.add(new Dog("Doggerman"));
    }
```

Мы можем только писать туда, но не забирать оттуда данные, так как мы не знаем, что конкретно к нам придет из такой коллекции. Т.е компилятор считает, что там `Object`, поэтому мы не знаем к чему кастовать.

Это еще называется `PECS` - **Producer extends Consumer super**.
Еще раз: `Producer` - может работать с типом `T` и его наследниками, `Consumer` - может принимать `T` и его предков.
