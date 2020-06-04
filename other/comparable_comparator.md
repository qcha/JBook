# java.lang.Comparable и java.util.Comparator

## Введение

Как уже было сказано в статье про [интерфейсы](../oop/interface.md), интерфейс - это определение поведения. Умение объектов сравнивать себя с друг с другом - это точно такое же поведение.

Сравнение используется различными алгоритмами от сортировки и двоичного поиска до поддержания порядка в сортированных коллекциях вроде `java.util.TreeMap`. 

Это повдение можно добавить как в непосредственно классы, с которыми мы работаем, так и объявить специальный класс, умеющий сравнивать два переданных объекта и решать кто из них больше, меньше или они вовсе равны.

Такие классы-судьи называются компараторы.

## java.lang.Comparable

Для добавления поведения сравнения непосредственно классам, с которыми мы работаем, существует интерфейс `java.lang.Comparable` (от англ. сравнимый, сопоставимый):

```java
public interface Comparable<T> {
    /**
     * Compares this object with the specified object for order.  Returns a
     * negative integer, zero, or a positive integer as this object is less
     * than, equal to, or greater than the specified object.
     *
     * <p>The implementor must ensure <tt>sgn(x.compareTo(y)) ==
     * -sgn(y.compareTo(x))</tt> for all <tt>x</tt> and <tt>y</tt>.  (This
     * implies that <tt>x.compareTo(y)</tt> must throw an exception iff
     * <tt>y.compareTo(x)</tt> throws an exception.)
     *
     * <p>The implementor must also ensure that the relation is transitive:
     * <tt>(x.compareTo(y)&gt;0 &amp;&amp; y.compareTo(z)&gt;0)</tt> implies
     * <tt>x.compareTo(z)&gt;0</tt>.
     *
     * <p>Finally, the implementor must ensure that <tt>x.compareTo(y)==0</tt>
     * implies that <tt>sgn(x.compareTo(z)) == sgn(y.compareTo(z))</tt>, for
     * all <tt>z</tt>.
     *
     * <p>It is strongly recommended, but <i>not</i> strictly required that
     * <tt>(x.compareTo(y)==0) == (x.equals(y))</tt>.  Generally speaking, any
     * class that implements the <tt>Comparable</tt> interface and violates
     * this condition should clearly indicate this fact.  The recommended
     * language is "Note: this class has a natural ordering that is
     * inconsistent with equals."
     *
     * <p>In the foregoing description, the notation
     * <tt>sgn(</tt><i>expression</i><tt>)</tt> designates the mathematical
     * <i>signum</i> function, which is defined to return one of <tt>-1</tt>,
     * <tt>0</tt>, or <tt>1</tt> according to whether the value of
     * <i>expression</i> is negative, zero or positive.
     *
     * @param   o the object to be compared.
     * @return  a negative integer, zero, or a positive integer as this object
     *          is less than, equal to, or greater than the specified object.
     *
     * @throws NullPointerException if the specified object is null
     * @throws ClassCastException if the specified object's type prevents it
     *         from being compared to this object.
     */
    public int compareTo(T o);
}
```

Интерфейс добавляет только один метод, который принимает объект и возвращает результат сравнения:

* 0, если два объекта равны.
* отрицательное число, если объект меньше того, с которым его сравнивают. Обычно возвращают значение `-1`.
* положительное число, если объект больше того, с которым его сравнивают. Обычно возвращают значение `1`.

По контракту метода при работе с `null` будет выброшено исключение `java.lang.NullPointerException`.

Во многих классах интерфейс `java.lang.Comparable` уже реализован, например, в классах `java.lang.Integer`, `java.lang.String` и т.д.

Имеет смысл реализовывать интерфейс в классах, где определен естественный порядок у объектов. Например, в числах.

Для примера научим объекты класса `Person`, этой рабочей лошадке всех примеров в интернете, сравниваться между собой:

```java
class Person implements Comparable<Person>{
     
    private String name;
    private int age;
    
    Person(String name, int age){
        this.age = age; 
        this.name = name;
    }

    String getName(){
        return name;
    }

    String getName(){
        return name;
    }
     
    public int compareTo(Person p){ 
        return name.compareTo(p.getName());
    }
}
```

Однако, возникают ситуации, когда разработчик не предусмотрел реализацию `java.lang.Comparable` в своем классе, а вам надо как-то добавить это поведение, уметь сравнивать объекты. Либо реализовал, но вас не устраивает эта реализация, и вы хотите ее переопределить, или вам необходимо реализовать несколько видов сравнений. На такой случай существуют `компараторы`.

### Компаратор

Интерфейс `java.util.Comparator` содержит ряд методов, ключевым из которых является метод `compare`:

```java
public interface Comparator<E> {
 
    /**
     * Compares its two arguments for order.  Returns a negative integer,
     * zero, or a positive integer as the first argument is less than, equal
     * to, or greater than the second.<p>
     *
     * In the foregoing description, the notation
     * <tt>sgn(</tt><i>expression</i><tt>)</tt> designates the mathematical
     * <i>signum</i> function, which is defined to return one of <tt>-1</tt>,
     * <tt>0</tt>, or <tt>1</tt> according to whether the value of
     * <i>expression</i> is negative, zero or positive.<p>
     *
     * The implementor must ensure that <tt>sgn(compare(x, y)) ==
     * -sgn(compare(y, x))</tt> for all <tt>x</tt> and <tt>y</tt>.  (This
     * implies that <tt>compare(x, y)</tt> must throw an exception if and only
     * if <tt>compare(y, x)</tt> throws an exception.)<p>
     *
     * The implementor must also ensure that the relation is transitive:
     * <tt>((compare(x, y)&gt;0) &amp;&amp; (compare(y, z)&gt;0))</tt> implies
     * <tt>compare(x, z)&gt;0</tt>.<p>
     *
     * Finally, the implementor must ensure that <tt>compare(x, y)==0</tt>
     * implies that <tt>sgn(compare(x, z))==sgn(compare(y, z))</tt> for all
     * <tt>z</tt>.<p>
     *
     * It is generally the case, but <i>not</i> strictly required that
     * <tt>(compare(x, y)==0) == (x.equals(y))</tt>.  Generally speaking,
     * any comparator that violates this condition should clearly indicate
     * this fact.  The recommended language is "Note: this comparator
     * imposes orderings that are inconsistent with equals."
     *
     * @param o1 the first object to be compared.
     * @param o2 the second object to be compared.
     * @return a negative integer, zero, or a positive integer as the
     *         first argument is less than, equal to, or greater than the
     *         second.
     * @throws NullPointerException if an argument is null and this
     *         comparator does not permit null arguments
     * @throws ClassCastException if the arguments' types prevent them from
     *         being compared by this comparator.
     */
    int compare(T o1, T o2);
    
    // остальные методы
}
```

Метод также возвращает числовое значение - если оно отрицательное, то объект `o1` считается меньше, предшествует объекту `o2`, иначе - наоборот. А если метод возвращает ноль, то объекты равны.

Компаратор на все тот же `Person`, но сравнивающий по именам:

```java
class PersonComparator implements Comparator<Person>{
    public int compare(Person a, Person b){
        return a.getName().compareTo(b.getName());
    }
}
```

## Требования

Помните три важных свойства сравнивающей функции: `рефлексивность` (сравнение элемента с самим собой всегда даёт `0`), `антисимметричность` (сравнение A с B и B с A должны дать разный знак) и `транзитивность` (если сравнение A с B и B с C выдаёт одинаковый результат, то и сравнение A с C должно выдать такой же).

При несоблюдении этих свойств результат работы алгоритмов сортировки и структур данных, основанных на сравнении, как например, `java.util.TreeMap`, будет неверным.

## Применение

### Collection и Array

Классы `java.util.Collections` и `java.util.Arrays` предоставляют метод `sort` для сортировки с помощью компаратора списков и массивов соответственно:

```java
    public static <T> void sort(List<T> list, Comparator<? super T> c) {
        // ...
    }
```

и 

```java
public static <T> void sort(T[] a, Comparator<? super T> c) {
    // ...
}
```

### Структуры данных

Структуры данных, типа `java.util.TreeMap` или `java.util.TreeSet` работают только с элементами, реализующими `java.lang.Comparable`, или требуют в конструкторе компаратор. 

```java
Set<Message> messages = new TreeSet(comparator);
```

### Stream

В `Java 8` появились `Stream`-ы, позволяющие использовать компараторы прямо в цепочке вычислений:

```java
stream
    .sorted(Comparator.naturalOrder())
    .forEach(e -> System.out.println(e));
```

---

**Вопрос**:

Я написал свой компаратор, работающий с целыми числами:

```java
class IntegerComparator implements Comparator<Integer> {
    public int compare(Integer a, Integer b) {
        return a - b;
    }
}
```

Видите ли вы ошибку?

**Ответ**:

Компаратор реализован неправильно, так как не учитывает возможности переполнения (overflow) у `java.lang.Integer`.

```java
int a = -2000000000;
int b =  2000000000;
System.out.println(a - b); // prints "294967296"
```

Помните об этом, когда работаете с числами в `Java`!

Также помните о существовании `NaN` у `Double` и `Float`. Оно не больше, не меньше и не равно никакому другому числу.

---

## Полезные ссылки

1. [Тагир Валеев Пишите компараторы правильно](https://habr.com/ru/post/247015/)
2. [Timur Batyrshinov Compare и Comparator в Java](https://www.youtube.com/watch?v=EIiHdEOqf2k)
3. [Subtraction comparator](https://stackoverflow.com/questions/2728793/java-integer-compareto-why-use-comparison-vs-subtraction)
4. [Overflow and Underflow in Java](https://www.baeldung.com/java-overflow-underflow)