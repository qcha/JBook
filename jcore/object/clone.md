# java.lang.Object#clone

- [java.lang.Object#clone](#javalangobjectclone)
    - [Введение](#введение)
    - [Подводные камни](#подводные-камни)
        - [Работа с конструктором](#работа-с-конструктором)
        - [Protected](#protected)
        - [Размытый контракт](#размытый-контракт)
    - [Конструктор копирования как выход](#конструктор-копирования-как-выход)
    - [Заключение](#заключение)
    - [Полезные ссылки](#полезные-ссылки)

## Введение

Данный метод задумывался разработчиками как простой и понятный способ создать копию объекта, т.е. его клон - отсюда и название.

Объявление метода выглядит как:

```java
    /**
     * Creates and returns a copy of this object.  The precise meaning
     * of "copy" may depend on the class of the object. The general
     * intent is that, for any object {@code x}, the expression:
     * <blockquote>
     * <pre>
     * x.clone() != x</pre></blockquote>
     * will be true, and that the expression:
     * <blockquote>
     * <pre>
     * x.clone().getClass() == x.getClass()</pre></blockquote>
     * will be {@code true}, but these are not absolute requirements.
     * While it is typically the case that:
     * <blockquote>
     * <pre>
     * x.clone().equals(x)</pre></blockquote>
     * will be {@code true}, this is not an absolute requirement.
     * <p>
     * By convention, the returned object should be obtained by calling
     * {@code super.clone}.  If a class and all of its superclasses (except
     * {@code Object}) obey this convention, it will be the case that
     * {@code x.clone().getClass() == x.getClass()}.
     * <p>
     * By convention, the object returned by this method should be independent
     * of this object (which is being cloned).  To achieve this independence,
     * it may be necessary to modify one or more fields of the object returned
     * by {@code super.clone} before returning it.  Typically, this means
     * copying any mutable objects that comprise the internal "deep structure"
     * of the object being cloned and replacing the references to these
     * objects with references to the copies.  If a class contains only
     * primitive fields or references to immutable objects, then it is usually
     * the case that no fields in the object returned by {@code super.clone}
     * need to be modified.
     *
     * @implSpec
     * The method {@code clone} for class {@code Object} performs a
     * specific cloning operation. First, if the class of this object does
     * not implement the interface {@code Cloneable}, then a
     * {@code CloneNotSupportedException} is thrown. Note that all arrays
     * are considered to implement the interface {@code Cloneable} and that
     * the return type of the {@code clone} method of an array type {@code T[]}
     * is {@code T[]} where T is any reference or primitive type.
     * Otherwise, this method creates a new instance of the class of this
     * object and initializes all its fields with exactly the contents of
     * the corresponding fields of this object, as if by assignment; the
     * contents of the fields are not themselves cloned. Thus, this method
     * performs a "shallow copy" of this object, not a "deep copy" operation.
     * <p>
     * The class {@code Object} does not itself implement the interface
     * {@code Cloneable}, so calling the {@code clone} method on an object
     * whose class is {@code Object} will result in throwing an
     * exception at run time.
     *
     * @return     a clone of this instance.
     * @throws  CloneNotSupportedException  if the object's class does not
     *               support the {@code Cloneable} interface. Subclasses
     *               that override the {@code clone} method can also
     *               throw this exception to indicate that an instance cannot
     *               be cloned.
     * @see java.lang.Cloneable
     */
    @IntrinsicCandidate
    protected native Object clone() throws CloneNotSupportedException;
```

Ключевое слово `native` означает, что метод реализован в платформенно-зависимом коде, чаще всего на `C/C++`, и скомпонован в виде динамической библиотеки. Эта реализация зависит от `JVM`!
Возможно, сейчас это пугает, но достаточно просто понимать, что `native` означает лишь то, что вызываемый код реализован не на `Java`.

Как видно из описания, для клонирования классу необходимо реализовать интерфейс `java.lang.Cloneable`.
Данный интерфейс является интерфейсом-марекром, как, например, `java.io.Serializable`.

Что же делает интерфейс `java.lang.Cloneable`? Он определяет поведение закрытого метода `clone` в классе `java.lang.Object`: если какой-либо класс реализует интерфейс `Cloneable`, то метод `clone` возвратит его копию с копированием всех полей, в противном случае будет инициировано исключение `java.lang.CloneNotSupportedException`.

Это достаточно нетипичный пример работы с интерфейсом: когда метод объявлен у одного класса, а право пользования этим методом подмешивается отдельным интерфейсом.
Я бы не рекомендовал подражать такому подходу, на мой взгляд это вносит путаницу и противоречит концепции интерфейса о поведении.

Подробнее про [интерфейсы](../oop/interface.md)

В общем, казалось бы все просто (пусть и не очень красиво): реализуем интерфейс-маркер и пользуемся?
Но не все так просто, поээтому давайте для начала объявим класс, реализуем интерфейс-маркер `java.lang.Cloneable`, попробуем склонировать объект и разобраться с подводными камнями при использовании этого метода.

## Подводные камни

### Работа с конструктором

Давайте просто в качестве примера запустим такой код:

```java
public class Person implements Cloneable {
    private int age;

    public Person(int age) {
        this.age = age;
        System.out.println("Constructor");
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return new StringJoiner(", ", Person.class.getSimpleName() + "[", "]")
                .add("age=" + age)
                .toString();
    }

    public static void main(String[] args) throws CloneNotSupportedException {
        System.out.println("Initializing...");
        Person p = new Person(20);

        System.out.println("Before clone");
        System.out.println(p);

        System.out.println("Cloning...");
        Object o = p.clone();

        System.out.println("After clone");
        System.out.println(o);
    }
}
```

Вывод:

```java
Initializing...
Constructor
Before clone
Person[age=20]
Cloning...
After clone
Person[age=20]
```

Итак, первый подводный камень, что бросается в глаза - это то, что метод `clone` создает копию объекта **без** вызова конструктора.
Что зачастую достаточно не удобно, особенно, если у вас заложена логика в конструкторе.

В примере выше метод `main` объявлен прямо внутри класса, это искуственный пример, который к тому же скрыл очень важный момент.

### Protected

Давайте попробуем объект класса `Person` склонировать, но вызов сделать в другом месте, не внутри этого же класса. Для этого объявим класс `Main` и перенесем туда метод `main`:

```java
public class Main {
    public static void main(String[] args) throws CloneNotSupportedException {
        System.out.println("Initializing...");
        Person p = new Person(20);

        System.out.println("Before clone");
        System.out.println(p);

        System.out.println("Cloning...");
        Object o = p.clone(); // Ошибка компиляции!

        System.out.println("After clone");
        System.out.println(o);
    }
}
```

Получим ошибку компиляции:

```java
java: clone() has protected access in java.lang.Object
```

Дело в том, что метод `clone` объявлен с модификатором доступа `protected`, а это ограничивает применение метода: вызвать его можно только внутри класса или наследниках. Поэтому в прошлом примере у нас и не возникало никаких проблем.

Подробнее про [модификаторы доступа](../oop/encapsulation.md#модификаторы-доступа)

Для решения этой проблемы можно внутри класса создать свой метод с `public` доступом и вызов `clone` перенести туда:

```java
public class Person implements Cloneable {
    
    // ... Тело класса описано выше

    public Person doClone() throws CloneNotSupportedException {
        return (Person) super.clone();
    }
}
```

Теперь в `main` заменим клонирование на `doClone` и убедимся, что результат такой, какой мы получили до этого:

```java
Initializing...
Constructor
Before clone
Person[age=20]
Cloning...
After clone
Person[age=20]
```

В данном случае мы заранее сделали каст к `Person`, чтобы не делать это во вне.

Второй вариант заключается в том, что мы переопределим метод `clone`:

```java
public class Person implements Cloneable {
    
    // ... Тело класса описано выше

    @Override
    public Person clone() throws CloneNotSupportedException {
            Person clone = (Person) super.clone();
            return clone;
    }
}
```

Обратите внимание, что здесь мы не только изменили модификатор доступа на `public`, но и изменили тип возвращаемого значения с `java.lang.Object` на `Person`.

Результат будет таким же.

Второй подводный камень заключается в том, что для использования `clone` мало реализовать `java.lang.Cloneable`, надо еще переопределить метод (или создать метод-обертку)!

### Размытый контракт

Давайте теперь вчитаемся в то, что же говорит нам контракт и документация по методу `clone` (взято из `JDK 17`):

> Метод создает и возвращает копию объекта. Точное значение термина 'копия' может зависеть от класса этого объекта.
> Общая цель ставится так, чтобы для любого объекта х оба выражения:
>
> ```java
> x.clone() != x
> ```
>
> И:
>
> ```java
> x.clone().getClass() == x.getClass()
> ```
>
> Должны выполняться (быть `true`) и это не являются безусловными.
> По соглашению возвращаемый объект должен быть получен путем вызова `super.clone()`. Если класс и все его суперклассы (кроме Object) подчиняются этому соглашению, то будет так, что `x.clone().getClass() == x. getClass()`.
>
> По соглашению возвращаемый этим методом объект должен быть независим от этого объекта (который клонируется). Чтобы достичь этой независимости, может потребоваться изменить одно или несколько полей объекта, возвращаемого `super.clone()`, перед его возвратом.
>
> Обычно это означает копирование любых изменяемых объектов, которые составляют внутреннюю 'глубокую структуру' (deep structure) клонируемого объекта, и замену ссылок на эти объекты ссылками на копии. Если класс содержит только примитивные поля или ссылки на неизменяемые объекты, то обычно никакие поля в возвращаемом объекте, изменять не нужно.

Соглашения, указанные в начале, являются достаточно слабыми гарантиями для клона и оригинала: не дается никаких гарантий на копируемые поля.

Также, обратите внимание на слова про 'может потребоваться изменить одно или несколько полей объекта, возвращаемого `super.clone()`, перед его возвратом.' и явный совет на то, как работать с тем, если у вас изменяемые объекты в классе.

До этого в нашем классе-примере был только примитив, но что будет, если одним из полей будет являться изменяемый объект?

Давайте немного усложним наш класс и добавим ссылочных типов:

```java
public class Person implements Cloneable {
    private int age;
    private List<String> nicknames;

    public Person(int age) {
        this.age = age;
        this.nicknames = new ArrayList<>();

        System.out.println("Constructor");
    }

    public void addNickname(String nickname) {
        this.nicknames.add(nickname);
    }

    public void removeNickname(String nickname) {
        this.nicknames.remove(nickname);
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public Person clone() throws CloneNotSupportedException {
            Person clone = (Person) super.clone();
            return clone;
    }

    @Override
    public String toString() {
        return new StringJoiner(", ", Person.class.getSimpleName() + "[", "]")
                .add("age=" + age)
                .add("nicknames=" + nicknames)
                .toString();
    }
}
```

И запустим следующий код:

```java
public class Main {
    public static void main(String[] args) throws CloneNotSupportedException {
        System.out.println("Initializing...");
        Person p = new Person(20);
        p.addNickname("Name");
        p.addNickname("GigaName");

        System.out.println("Before clone");
        System.out.println(p);

        System.out.println("Cloning...");
        Object o = p.clone();

        System.out.println("After clone");
        System.out.println(o);

        System.out.println("Changing origin");
        p.addNickname("NEW");

        System.out.println("After changing nicknames in origin");
        System.out.println("Origin object:");
        System.out.println(p);
        System.out.println("Clone object:");
        System.out.println(o);
    }
}
```

Вывод:

```java
Initializing...
Constructor
Before clone
Person[age=20, nicknames=[Name, GigaName]]
Cloning...
After clone
Person[age=20, nicknames=[Name, GigaName]]
Changing origin
After changing nicknames in origin
Origin object:
Person[age=20, nicknames=[Name, GigaName, NEW]]
Clone object:
Person[age=20, nicknames=[Name, GigaName, NEW]]
```

Заметьте, что изменение списка никнеймов у оригинального объекта повлияло и на клон!

По умолчанию `clone` делает **поверхностное** копирование, т.е копируются значения всех полей и **ссылок**. Таким образом, и клон, и оригинальный объект имеют ссылки на один и тот же изменяемый список, а поэтому, любое его изменение, например в клоне, тут же влияет на оригинал.

Именно про эту ситуацию и было написано в документации метода: 'может потребоваться изменить одно или несколько полей объекта, возвращаемого `super.clone()`, перед его возвратом'.

Грубо говоря, необходимо сделать копию явно:

```java
@Override
public Person clone() throws CloneNotSupportedException {
    Person clone = (Person) super.clone();
    clone.nicknames = new ArrayList<>();
    clone.nicknames.addAll(this.nicknames);

    return clone;
}
```

Такой способ может породить трудно уловимые ошибки, ведь надо быть очень внимательным, так как можно просто забыть присвоить копию у клона (представьте, что у вас еще и наследники есть у класса, и полей не два).

И самое главное: такой способ не сработает, если поле объявлено у класса как `final`.

Т.е. архитектурно концепиця клонирования через `clone` не совместима с обычным использованием `final` полей, содержащих ссылки на изменяемые объек- ты.

## Конструктор копирования как выход

Из-за всех перечисленных сложностей на данный момент метод `clone` практически не используется и вместо него рекомендуется использовать конструкторы копирования или статические методы, создающие объект.

Конструктор копирования - это всего лишь конструктор, где единственный аргумент имеет тип, соответствующий классу, где находится этот конструктор, например:

```java
    public Person(Person p) {
        this.age = p.age;
        this.nicknames = new ArrayList<>(p.nicknames);
    }
```

Подобного можно также достичь с помощью статических методов, в которых вы также создадите новый объект и пропишите логику копирования:

```java
    public static Person copyOf(Person p) {    
        return new Person(p.age, new ArrayList<>(p.nicknames)); 
    }
```

Почему данный вариант лучше: он не связан с рискованным, плохо задокументированным механизмом клонирования объектов, не конфликтует с обычной схемой использования `final` полей, не требует работы с ненужными исключениями, возвращает объект строго определенного типа, что делает код более прозрачным и понятным.

Например, реализации коллекций имеют конструктор копий:

```java
List<String> strings = new ArrayList<>();
strings.add("Hello");

List<String> copy = new ArrayList<>(strings);
```

Выглядит он следующим образом:

```java
    public ArrayList(Collection<? extends E> c) {
        Object[] a = c.toArray();
        if ((size = a.length) != 0) {
            if (c.getClass() == ArrayList.class) {
                elementData = a;
            } else {
                elementData = Arrays.copyOf(a, size, Object[].class);
            }
        } else {
            // replace with empty array.
            elementData = EMPTY_ELEMENTDATA;
        }
    }
```

## Заключение

Метод `clone` задумывался как простой и понятный способ создать копию объекта. Однако, этого не вышло и в итоге получился сложный в использовании метод с большим количеством подводных камней:

* Метод `clone` не использует конструктор класса для создания объекта, что идет в разрез с обычным подходом работы с объектами в `Java` и усложняет работу.
* Для клонирования необходимо реализовать интерфейс-маркер `java.lang.Cloneable` и переопределить `clone()`. При этом интерфейс по сути просто меняет поведение некоего защищенного метода в суперклассе, что делает код неявным и подверженным ошибкам.
* При наличии у класса изменяемых полей или сложной структуры придется менять объект, возвращенный `clone`: в переопределенном методе необходимо сначала вызвать `super.clone()`, после чего начать работать с полями, значения которых могут изменяться, т.е надо менять все ссылки на объекты соответствующими копиями. Помните также, что это не будет работать с `final` полями.
* Учтите, что, если в классе, предназначенном для наследования, вы не создадите правильно работающий метод `clone`, то реализация интерфейса `Cloneable` и работа с клонированием в подклассах станет невозможной.

Чтобы иметь возможность клонирования класс и все его суперклассы должны следовать сложному, трудновыполнимому и слабо описанному протоколу, где велика вероятность совершить ошибку.

Из-за множества недостатков клонирование через метод `clone` просто предпочитают никогда использовать (за исключением, быть может, простого копирования массивов с примитивами), предпочитая конструкторы копирования или статические методы копирования.

## Полезные ссылки

1. [Clone() vs Copy constructor- which is recommended in java](https://stackoverflow.com/questions/2427883/clone-vs-copy-constructor-which-is-recommended-in-java)
2. [Effective Java 2nd Edition, Item 11: Override clone judiciously](https://www.amazon.com/dp/0321356683)
