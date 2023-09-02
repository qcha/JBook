# java.lang.Object#clone

## Введение

Название метода подскажет его назначение.
Данный метод задумывался разработчиками как простой и понятный способ создать копию объекта, его клон - отсюда и название.

Объявление метода выглядит так:

```java
protected native Object clone() throws CloneNotSupportedException;
```

> Ключевое слово `native` означает, что метод реализован в платформенно-зависимом коде, чаще всего на `C/C++`, и скомпонован в виде динамической библиотеки.
>
> Эта реализация зависит от `JVM`.

> Возможно, вас сейчас это напугало, но на самом деле достаточно просто понимать, что 
> `native` означает лишь то, что вызываемый код, реализован не на `Java`.

Клонировать можно только те объекты, которые реализуют интерфейс `java.lang.Cloneable`.
Данный интерфейс является интерфейсом-марекром, как и `java.io.Serializable`.

Если объект не реализует интерфейс-маркер `java.lang.Cloneable`, то будет сгенерировано исключение `java.lang.CloneNotSupportedException`.

Это нетипичный пример использования интерфейсов, когда метод объявлен у одного класса, а право пользования этим методом подмешивается отдельным интерфейсом.
Я бы не рекомендовал подражать такому подходу.

> Подробнее про [интерфейсы](../oop/interface.md)


---

Исходя из `JavaDoc` документации вызов метода `clone` создает копию объекта без вызова конструктора. Что далеко не всегда удобно.
По умолчанию `clone` определяет **поверхностное** копирование, т.е копируются значения всех полей и **ссылок**.

Рассмотрим пример:

```java
public class CloneTest implements Cloneable {
    private String name;
    private int age;

    public void setAge(int age) {
        this.age = age;
    }
    public void setName(String name) {
        this.name = name;
    }

    public CloneTest(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public int getAge() {
        return age;
    }
    public String getName() {
        return name;
    }


    public static void main(String[] args) throws CloneNotSupportedException {
        CloneTest test = new CloneTest("Петя",20);
        CloneTest cloneTest = (CloneTest) test.clone();
        System.out.println("Original: " + test + ", name = " + test.getName() + ", age = " + test.getAge());
        System.out.println("Clone: " + cloneTest + ", name = " + cloneTest.getName() + ", age = " + cloneTest.getAge());

        cloneTest.setAge(30);
        cloneTest.setName("Вася");

        System.out.println("Original: " + test + ", name = " + test.getName() + ", age = " + test.getAge());
        System.out.println("Clone: " + cloneTest + ", name = " + cloneTest.getName() + ", age = " + cloneTest.getAge());
    }
}

```

Вывод:
 
```
Original: CloneTest@568db2f2, name = Петя, age = 20
Clone: CloneTest@312b1dae, name = Петя, age = 20
Original: CloneTest@568db2f2, name = Петя, age = 20
Clone: CloneTest@312b1dae, name = Вася, age = 30
```

Таким образом мы получили два разных объекта, где значения полей `age` и `name` были клонированы по значению.
Мы изменили поля у объекта-клона, при этом значения полей у оригинального объекта не изменились.

>Чтобы не делать постоянное приведение к нужному типу, можно переопределить наш метод и явно указать тип возвращаемого объекта.
Это считается хорошим тоном и застрахует нас от ошибок.

## Подводные камни

Рассмотрим чуть более сложный пример, содержащий ссылочный тип, и переопределим метод так, как мы говорили выше - укажем явно возвращаемый объект:

```java
class Hobby {
    private String name;

    public Hobby(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setHobbyName(String hobbyName) {
        this.name = hobbyName;
    }

    @Override
    public String toString() {
        return name;
    }
}

public class PersonToClone implements Cloneable {
    private String name;
    private int age;
    private Hobby hobby;

    public PersonToClone(String name, int age, Hobby hobby) {
        this.name = name;
        this.age = age;
        this.hobby = hobby;
    }

    @Override
    public String toString() {

        return String.format("PersonToClone@%d{ " +
                        "name=%s, age=%d, hobby=%s }",
                this.hashCode(), this.name, this.age, this.hobby);

    }

    @Override
    protected PersonToClone clone() throws CloneNotSupportedException {
        return (PersonToClone) super.clone();
    }

    public Hobby getHobby() {
        return hobby;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setHobby(Hobby hobby) {
        this.hobby = hobby;
    }
    public static void main(String[] args) throws CloneNotSupportedException {
       // уже скоро...
    }
}
```

На первый взгляд может показаться, что все очень удобно и практично, однако здесь есть несколько подводных камней.

Во-первых, вспомним, что по умолчанию копируются значения всех полей и **ссылок**, а значит при клонировании будет скопирована именно ссылка.
Т.е и клон, и первоначальный объект будут ссылаться на один и тот же объект `Hobby`.

Что подводит нас к подводному камню, можно сказать айсбергу, потопившему титаник: если первоначальный объект изменит `Hobby`, то эти изменения окажутся и у клона!
В свою очередь это работает и в обратную сторону - если клон вдруг изменит `Hobby`, то и у первоначального объекта оно изменится.

>Вернемся к нашему коду, и посмотрим как это работает на практике.

```java

public static void main(String[] args) throws CloneNotSupportedException {
        PersonToClone original = new PersonToClone("Original", 20, new Hobby("Learn Java"));
        PersonToClone clone = original.clone();

        clone.setName("Clone");
        System.out.println(original);
        System.out.println(clone);
}

```

Здесь мы создали объект и его копию, вызвав наш переопределенный метод clone()

Вывод:

```
PersonToClone@931919113{ name=Original, age=20, hobby=Learn Java }
PersonToClone@455896770{ name=Clone, age=20, hobby=Learn Java }
```

Пока все идет отлично, добавим еще пару строк:

```java

public static void main(String[] args) throws CloneNotSupportedException {
        PersonToClone original = new PersonToClone("Original", 20, new Hobby("Learn Java"));
        PersonToClone clone = original.clone();

        clone.setName("Clone");
        System.out.println(original);
        System.out.println(clone);

        // новые строки
        clone.getHobby().setHobbyName("Learn C#");

        System.out.println(original);
        System.out.println(clone);
}

```

Вывод:

```
PersonToClone@931919113{ name=Original, age=20, hobby=Learn Java }
PersonToClone@455896770{ name=Clone, age=20, hobby=Learn Java }
PersonToClone@931919113{ name=Original, age=20, hobby=Learn C# } 
PersonToClone@455896770{ name=Clone, age=20, hobby=Learn C# }
```

Мы получили ссылку на объект типа `Hobby`, и изменили у него поле названия. Помним, что из-за поверхностного копирования,
у объекта-клона и объекта-оригинала одинаковая ссылка на один объект типа `Hobby`, поэтому наши действия привели к тому, что название хобби поменяется и у оригинального объекта,
хотя мы этого не планировали.

*Чувствуете пропасть под ногами? Холодок уже пробежал?*

## Что делать

Избежать подобного поведения можно двумя способами:

* Делать для таких 'опасных' объектов, как в примере выше, копии вручную и присваивать их через `setter`-ы.
* Конструктор копирования или специальные статические методы.

### Первый способ

Сначала вызываем `clone` у супер-класса, а после уже явно вручную работаем с 'опасными' полями.
Вызов `clone` у супер-класса даст нам копию объекта, но поля, которые содержат изменяемые объекты надо вручную обработать - создать копии.

Рассмотрим как это будет выглядеть в нашем случае:

```java
@Override
protected PersonToCloneBetter clone() throws CloneNotSupportedException {
    PersonToCloneBetter pClone = (PersonToCloneBetter) super.clone();
    pClone.setHobby(new Hobby(hobby.getHobbyName()));

    return pClone;
}
```

Т.е мы явно создаем новый объект `Hobby`, который уже и присваиваем через `setter` у клона. Таким образом, проблема с изменением объекта `Hobby` решена, это два разных объекта с одним состоянием.

Единственное 'но' - такой способ не сработает, если поле `hobby` объявлено у класса как `final`, т.е является неизменяемым.
Либо же у класса нет `setter` метода на это поле.

Вдобавок к этому, такой способ может породить трудно уловимые ошибки, ведь надо быть очень внимательным, так как можно просто забыть присвоить копию у клона.

Давайте представим, что мы хотим получить копию очень сложного объекта, то есть у объекта клонирования есть ссылочные поля на объекты, которые тоже имеют ссылочные поля на другие объекты и т.д.
 В таком случае нам надо убедиться, что метод `clone` определен у каждого типа, или быть предельно внимательным, распутывая вереницу ссылочных типов.

На помощь нам может прийти альтернативный метод глубокого копирования - **копирование через сериализацию**.
Сразу стоит отметить, что это затратный по времени способ, так как создание сокета, сериализация объекта, передача его через сокет, а затем его десериализация требуют больше времени, по сравнению с вызовом методов у существующих объектов.

Реализация:

(не забываем про интерфейс `Serializable` для классов)

```java
import java.io.*;

class Music implements Serializable {
    public String getName() {
        return name;
    }

    private String name;

    public void setName(String name) {
        this.name = name;
    }

    public Music(String name) {
        this.name = name;
    }
}

class Person implements Serializable {
    private String name;
    private Music music;

    public Music getMusic() {
        return music;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public Person(String name, Music music) {
        this.name = name;
        this.music = music;
    }


    @Override
    public String toString() {
        return "name: " + name + ", music: " + music.getName();
    }
}

public class SerializationClone {

    public static Object clone(Object source) {
        Object cloneVal = null;
        try {
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(baos);
            oos.writeObject(source);
            oos.flush();
            oos.close();

            ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(baos.toByteArray()));
            cloneVal = in.readObject();
        } catch (IOException ex) {

        } catch (ClassNotFoundException ex) {

        }

        return cloneVal;
    }

    public static void main(String[] args) {
        Person personOriginal = new Person("person1", new Music("song1"));
        Person personClone = (Person) clone(personOriginal);

        personClone.getMusic().setName("song2");
        personClone.setName("person2");

        System.out.println(personOriginal);
        System.out.println(personClone);
    }
}
```

Вывод:

```
name: person1, music: song1
name: person2, music: song2
```

Таким образом, мы получили копию объекта `personOriginal`, изменяя поля у которой, мы не меняем объект-оригинал.

К слову о времени работы. Добавим реализацию метода `clone` классу `Person`

```java

class Person implements Serializable, Cloneable {
    // same
    // ...
 
    @Override
    public Person clone() throws CloneNotSupportedException {
     Person clone = (Person) super.clone();
     clone.music = new Music(clone.music.getName());
     return clone;
    }
}
```

И изменим метод `main` в классе `SerializationClone` следующим образом:

```java
public static void main(String[] args) throws CloneNotSupportedException {
        Person personOriginal = new Person("person1", new Music("song1"));

        int iterations = 10000;

        long start1 = System.currentTimeMillis();
        for (int i = 0; i < iterations; i++) {
            clone(personOriginal);;
        }
        long time1 = (System.currentTimeMillis() - start1);

        System.out.println("Serialization time: " + time1);

        long start2 = System.currentTimeMillis();
        for (int i = 0; i < iterations; i++) {
            personOriginal.clone();
        }
        long time2 = (System.currentTimeMillis() - start2);

        System.out.println("Clone-method time: " + time2);

    }
```

По итогу прогона 10\`000 итераций получаем, что для способа с сериализацией требуется ~400 мс (у вас может получиться другой ответ, в зависимости от машины, на которой запускаете код), 
в то время как для клонирования через метод `clone` требуется меньше 1 мс.

---

### Конструктор копирования

Вы спросите: "Что еще за конструктор копирования?"

Отвечаю: вы создаете конструктор, который принимает объект, копию которого вы хотите сделать. Можно вместо конструктора объявить статический метод, который будет делать то же самое.

Для примера рассмотрим следующий код:

```java
public static PersonToClone newInstance(PersonToClone personToClone) {
    return new PersonToClone(
      personToClone.getName(),
      personToClone.getAge(),
      new Hobby(personToClone.hobby.getHobbyName())
    );
}
```

Т.е мы просто получили объект, взяли всю необходимую информацию и создали новый объект, положив туда все из старого. При этом вы не используете метод `clone`, вы сами написали копирование вашего объекта.

Подобного можно также достичь с помощью статических методов, в которых вы также создадите новый объект и пропишите логику копирования.

Этот вариант является более предпочтительным, на мой взгляд.

Плюсы:

* Проще реализуется.
* Не работаем с исключениями клонирования, как например, `java.lang.CloneNotSupportedException`.
* Поддерживает работу с `final`-полями.
* Код получается более явным, мы просто создаем новый объект с помощью конструктора или специального метода.

---
### Еще немного сложностей

Помимо выше перечисленных проблем, при использовании метода `clone` можно столкнуться со следующей сложностью:
*у интерфейса `Cloneable` нет открытого (`public`) метода `clone`*. При этом в самом классе `Object` этот метод является защищенным (`protected`).
Это приводит к тому, что вы не можете, не обращаясь к механизму рефлексии, вызывать для объекта метод `clone` лишь на том основании, что он
реализует интерфейс `Cloneable`.

Рассмотрим такую файловую структуру: есть пакет `src`, в нем лежит исполняемый файл `CloneTest.java` и еще один пакет `models` с файлом `Person.java`

```
- src
   |--- models
   |      |--- Person.java
   |
   |--- ClonePerson.java
```

```java
public class ClonePerson {

    public static void main(String[] args) {
        var originalPerson = new Person("Person-1");

        Person clonePerson = (Person) originalPerson.clone();
    }
}
```

Данный код выполнить не получиться, так как без переопределения открытого метода `clone`, используется защищенный метод у класса `Object`
(`'clone()' has protected access in 'java.lang.Object'`)


Также **важно** напомнить, что метод clone **не должен** вызывать каких-либо переопределяемых
методов, взятых из создаваемого клона! Это может произойти, например, в случае, если переопределяемый метод обращается или
изменяет копию глубокого состояния объекта в клоне, но это состояние ещё не было установлено.

Посмотрим на пример:

```java
class MyText {
    private String text;

    public String getText() {
        return text;
    }
    public void setText(String text) {
        this.text = text;
    }
}

class ParentClass implements Cloneable {
    protected void overrideMethod() {
        System.out.println("Parent class");
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        ParentClass parentClass = (ParentClass) super.clone();
        overrideMethod();
        return parentClass;
    }
}

class SubClass extends ParentClass implements Cloneable {
    private MyText text;

    @Override
    protected void overrideMethod() {
        System.out.println("Sub class");
        this.text.setText("text1");
    }

    @Override
    protected SubClass clone() throws CloneNotSupportedException {
        var clone = (SubClass) super.clone();
        this.text = new MyText();
        return clone;
    }

    public static void main(String[] args) throws CloneNotSupportedException {
        SubClass subClass = new SubClass();
        SubClass clone = subClass.clone();
    }
}

```

**Тут нам важно помнить два момента:**

1. В методе clone родительского класса (`ParentClass`) будет вызван переопределенный метод из класса-наследника (`SubClass`)
2. При вызове этого метода будет вызвана ошибка (`NullPointerException`), так как мы пытаемся изменить состояние поля `text`, но само состояние еще не установлено.


Еще один момент, который стоит упомянуть: если вы решите сделать клонируемым
потокобезопасный класс, то метод `clone` нужно будет корректно синхронизировать, так как метод `Object.clone`
не синхронизирован.

---

## Заключение

Метод `clone` задумывался разработчиками как простой и понятный способ создать копию объекта. Помните, что классы, реализующие `java.lang.Cloneable` должны переопределять `clone()` и делать его открытым. Также старайтесь придерживаться правила, когда другие интрефейсы не расширяют `java.lang.Cloneable`.
 Реализуйте этот интерфейс явно только тогда, когда хотите использовать метод `clone`.

В переопределенном методе необходимо сначала вызвать `super.clone()`, после чего начать работать с полями, значения которых могут изменяться, т.е надо заменять все ссылки на объекты соответствующими копиями.

В целом, чаще гораздо более удобно копию объекта получать через конструктор копирования или специального метода, в котором явно создается новый объект и присваиваются значения из копируемого.

Чаще всего этот метод стараются не использовать.

---

### Полезные ссылки

* [Альтернатива глубокому копированию](https://www.infoworld.com/article/2077578/java-tip-76--an-alternative-to-the-deep-copy-technique.html)
