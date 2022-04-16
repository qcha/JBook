# Enum

- [Enum](#enum)
    - [Введение](#введение)
    - [Перечисления в Java](#перечисления-в-java)
    - [Устройство enum](#устройство-enum)
    - [Внутренности](#внутренности)
    - [Вклад java.lang.Enum](#вклад-javalangenum)
        - [ordinal и name](#ordinal-и-name)
        - [values](#values)
        - [valueOf](#valueOf)
        - [compareTo](#compareto)
        - [Serializable](#serializable)
    - [Советы по работе с enum](#советы-по-работе-с-enum)
    - [Секция вопросов](#секция-вопросов)
    - [Заключение](#заключение)
    - [Полезные ссылки](#полезные-ссылки)

## Введение

Существует ряд задач, в которых требуется некоторый тип данных ограничить множеством допустимых значений.

Это могут быть дни недели (понедельник, вторник и т.д), времена года (весна, зима, лето, осень), типы животных (членистоногие, моллюски, иглокожие и т.д), [HTTP-коды](https://ru.wikipedia.org/wiki/%D0%A1%D0%BF%D0%B8%D1%81%D0%BE%D0%BA_%D0%BA%D0%BE%D0%B4%D0%BE%D0%B2_%D1%81%D0%BE%D1%81%D1%82%D0%BE%D1%8F%D0%BD%D0%B8%D1%8F_HTTP) и прочее.

В простейшем приближении можно сказать, что перечисление - это список именованных, логически связанных констант.

В принципе, ничто не мешает нам использовать для этой задачи простые строки или числа.
Допустим, мы работаем над задачей документооборта, документы могут быть разных типов, поэтому мы создаем класс документа навроде:

```java
class Document {
    private String data;
    private String type;

    // code
}
```

Ну и документы у нас получаются наподобие (опустим здесь заполнение данных документа):

```java
new Document("Кредит");
new Document("Оплата");
```

Подобный подход встречается в разработке, но у него есть серьезные минусы:

* Разработчик не может узнать все возможные значения.

    Строка как тип документа ничего не говорит нам о том, какие значения допустимы, а какие нет.
    В лучшем случае эти значения будут указаны в `JavaDoc` над полем, в худшем - это нигде не будет указано и вам придется гадать.

* Компилятор также не может помочь в проверке валидности значений.

    Для компилятора все обстоит еще хуже: для него нет никакой логической связи между "Кредит" и "Оплата".
    И, несмотря на то, что для нас это значения типов документов, для компилятора они вообще никак не связаны.

* Рано или поздно вам потребуется `util`-класс для вспомогательных методов работы с типами документов: валидацией, преобразованиями и т.д.

Добавим сюда еще то, что у строк есть регистр и "Кредит" при сравнении будет совсем не то же самое, что "кредит".
Хотя по сути это один и тот же тип документа.

Т.е проблемы с регистром, нечитаемыми символами (например, пробелы в начале или в конце строки) явно не добавляют удобства работы.

Из этих минусов вытекают и требования:

* Компилятор должен понимать, что эти значения связаны логически - это некоторые перечисления возможных значений.
* Разработчик не должен бегать по всему проекту в поисках допустимых значений или надеяться на то, что в `JavaDoc` не забыли указать какой-то тип.
* Должна быть возможность добавления нового поведения и состояния (при необходимости): какие-то свои методы или значения.

И тут нас спасают перечисления или `enum`.

## Перечисления в Java

В `Java` перечисления появились с версии `1.5+`.

Перечисления создаются с помощью ключевого слова `enum` (enumeration - перечисление), затем идет список элементов перечисления через запятую:

```java
public enum DocumentType {
    CREDIT,
    PAYMENT
}
```

В таком случае, наш код с типами документа будет уже:

```java
class Document {
    private String data;
    private DocumentType type;

    // code
}
```

Этим мы закрываем большую часть проблем, которые получили от строкового типа: теперь компилятор гарантирует, что в поле `type` попадут только значений, относящиеся к типам документов `DocumentType`, все возможные типы у нас собраны в одном месте и связаны логически.

Но что если мы хотим к нашему типу добавить какую-то информацию или поведение?
Это возможно.

Главное, что стоит помнить, это то, что `enum` - это почти точно такой же класс, как и остальные. Это значит, что можно добавить конструкторы, методы, объявить необходимые переменные.

```java
public enum DocumentType {
    CREDIT ("Кредитный документ"),
    PAYMENT ("Оплата");

   private String description;

   DocumentType(String description) {
       this.description = description;
   }

   public String getDescription() {
       return description;
   }

   @Override
   public String toString() {
       return "DocumentType{" +
               "description='" + description + '\'' +
               '}';
   }
}
```

Здесь мы добавили поле `description`, как некоторое описание типа документа, добавили конструктор, чтобы это поле инициализировать на этапе создания экзмепляра `enum`-а, а также сгенерировали `toString` с более подробным описанием.

## Устройство enum

Раз `enum` - это класс, то здесь поддерживается ровно та же работа с интерфейсами, как и у обычных классов:

```java
public enum DocumentType implements Runnable, AutoCloseable {
    CREDIT,
    PAYMENT;

    public static void main(String[] args) {
        System.out.println(DocumentType.CREDIT.ordinal());
    }

    @Override
    public void run() {
        // а вот и метод из интерфейса Runnable!
    }

    @Override
    public void close() throws Exception {
        // а вот и метод из интерфейса AutoCloseable!
    }
}
```

Однако не все так радужно с наследованием:

```java
// compile error
// No extends clause allowed for enum
public enum DocumentType extends Object {
    CREDIT,
    PAYMENT;
}
```

Причина этого поведения кроется в том, что как видно из объявления нашего перечисления, это не совсем обычный класс.
Как минимум, потому что используется специальное ключевое слово `enum`, а некоторые могут забежать чуть вперед и увидеть, что наш `DocumentType` уже имеет дополнительные методы, которые мы не объявляли: `valueOf`, `values`, `ordinal` и т.д.

Например:

```java
    public static void main(String[] args) {
        System.out.println(DocumentType.CREDIT.ordinal()); // 0
    }
```

Это связано с тем, что в `Java` все `enum`-ы **уже** неявно отнаследованы от класса `java.lang.Enum`.
В свою очередь, это же является причиной того, что `enum`-ы не могут участвовать в наследовании.

Теперь давайте разберем внутренне устройство `enum` в `Java` чуть подробнее.

## Внутренности

Давайте посмотрим на байткод нашего `enum`:

```java
public enum DocumentType {
    CREDIT,
    PAYMENT;
}
```

Байткод скомпилированного класса можно посмотреть несколькими способами:

* Через консольные команды:

    `javap -c DocumentType`
* Через `bytecode viewer` встроенный в `Intellij Idea`:
    Выделяем скомпилированный класс, далее вкладка `view`, после чего в ней `Show Bytecode`.

В итоге получаем:

```java
// class version 61.0 (61)
// access flags 0x4031
// signature Ljava/lang/Enum<LDocumentType;>;
// declaration: DocumentType extends java.lang.Enum<DocumentType>
public final enum DocumentType extends java/lang/Enum {

  // compiled from: DocumentType.java

  // access flags 0x4019
  public final static enum LDocumentType; CREDIT

  // access flags 0x4019
  public final static enum LDocumentType; PAYMENT

  // access flags 0x101A
  private final static synthetic [LDocumentType; $VALUES

  // access flags 0x9
  public static values()[LDocumentType;
   L0
    LINENUMBER 1 L0
    GETSTATIC DocumentType.$VALUES : [LDocumentType;
    INVOKEVIRTUAL [LDocumentType;.clone ()Ljava/lang/Object;
    CHECKCAST [LDocumentType;
    ARETURN
    MAXSTACK = 1
    MAXLOCALS = 0

  // access flags 0x9
  public static valueOf(Ljava/lang/String;)LDocumentType;
   L0
    LINENUMBER 1 L0
    LDC LDocumentType;.class
    ALOAD 0
    INVOKESTATIC java/lang/Enum.valueOf (Ljava/lang/Class;Ljava/lang/String;)Ljava/lang/Enum;
    CHECKCAST DocumentType
    ARETURN
   L1
    LOCALVARIABLE name Ljava/lang/String; L0 L1 0
    MAXSTACK = 2
    MAXLOCALS = 1

  // access flags 0x2
  // signature ()V
  // declaration: void <init>()
  private <init>(Ljava/lang/String;I)V
   L0
    LINENUMBER 1 L0
    ALOAD 0
    ALOAD 1
    ILOAD 2
    INVOKESPECIAL java/lang/Enum.<init> (Ljava/lang/String;I)V
    RETURN
   L1
    LOCALVARIABLE this LDocumentType; L0 L1 0
    MAXSTACK = 3
    MAXLOCALS = 3

  // access flags 0x100A
  private static synthetic $values()[LDocumentType;
   L0
    LINENUMBER 1 L0
    ICONST_2
    ANEWARRAY DocumentType
    DUP
    ICONST_0
    GETSTATIC DocumentType.CREDIT : LDocumentType;
    AASTORE
    DUP
    ICONST_1
    GETSTATIC DocumentType.PAYMENT : LDocumentType;
    AASTORE
    ARETURN
    MAXSTACK = 4
    MAXLOCALS = 0

  // access flags 0x8
  static <clinit>()V
   L0
    LINENUMBER 2 L0
    NEW DocumentType
    DUP
    LDC "CREDIT"
    ICONST_0
    INVOKESPECIAL DocumentType.<init> (Ljava/lang/String;I)V
    PUTSTATIC DocumentType.CREDIT : LDocumentType;
   L1
    LINENUMBER 3 L1
    NEW DocumentType
    DUP
    LDC "PAYMENT"
    ICONST_1
    INVOKESPECIAL DocumentType.<init> (Ljava/lang/String;I)V
    PUTSTATIC DocumentType.PAYMENT : LDocumentType;
   L2
    LINENUMBER 1 L2
    INVOKESTATIC DocumentType.$values ()[LDocumentType;
    PUTSTATIC DocumentType.$VALUES : [LDocumentType;
    RETURN
    MAXSTACK = 4
    MAXLOCALS = 0
}
```

Если попробовать перевести это в `Java`-код обратно, опустив детали, то получим:

```java
public final enum DocumentType extends java.lang.Enum<LDocumentType> {
  public final static LDocumentType CREDIT;
  public final static LDocumentType PAYMENT;

  // code
}
```

Помимо подтверждения уже сказанного, что `enum` неявно наследуется от `java.lang.Enum`, мы также можем увидеть: наши перечисления превратились в статические финальные поля. Отсюда следует и то, что значения `enum` существуют в единственном экземпляре!

---

**Вопрос**:

Можно ли перечисления сравнивать не через `equals`, а через `==`?

**Ответ**:

Можно!

Так как значения перечисления существуют в единственном экземпляре (ведь они статические и финальные), поэтому их спокойно можно сравнивать через `==`:

```java
DocumentType type = DocumentType.PAYMENT;

if (DocumentType.CREDIT == type) {
    System.out.println("Кредит!");
} else {
    System.out.println("Не кредит");
}
```

Более того, такой способ сравнения позволит избежать `java.lang.NullPointerException` и работает быстрее!

```java
enum Shape { RECTANGLE, SQUARE, CIRCLE, TRIANGLE; }

// ...
Shape unknown = null;
Shape circle = Shape.CIRCLE;

System.out.println(unknown == circle); // false
System.out.println(unknown.equals(circle)); // кидает NullPointerException
```

---

Теперь проведем следующий эксперимент, запустим вот такой код:

```java
public enum DocumentType {
    CREDIT("Кредит"),
    PAYMENT("Оплата");

    private String description;

    static {
        System.out.println("Статический блок кода");
    }

    {
        System.out.println("Блок кода");
    }

    DocumentType(String description) {
        System.out.println("Запуск конструктора со значением");
        this.description = description;
    }
}
```

Запускаем:

```java
public class Main {
    public static void main(String[] args) {
        System.out.println(DocumentType.CREDIT);
    }
}
```

И что же мы видим?

```java
Блок кода
Запуск конструктора со значением
Блок кода
Запуск конструктора со значением
Статический блок кода
CREDIT
```

Можно сделать вывод: значения, объявленные в перечислении — это статические финальные поля, инициализация которых происходит в статическом блоке до всех остальных статических выражений.

В этом также можно убедиться, если посмотреть снова байткод нашего класса.

Теперь обратим свой взор на дополнительные методы, которые дает нам `enum`.

## Вклад java.lang.Enum

Раз мы выяснили, что `enum` неявно наследники `java.lang.Enum`, то давайте взглянем, что там внутри?

Выделим часть с объявлением класса, конструктором и полями, а остальное пока опустим:

```java
/**
 * This is the common base class of all Java language enumeration classes.
 *
 * More information about enums, including descriptions of the
 * implicitly declared methods synthesized by the compiler, can be
 * found in section {@jls 8.9} of <cite>The Java Language
 * Specification</cite>.
 *
 * Enumeration classes are all serializable and receive special handling
 * by the serialization mechanism. The serialized representation used
 * for enum constants cannot be customized. Declarations of methods
 * and fields that would otherwise interact with serialization are
 * ignored, including {@code serialVersionUID}; see the <cite>Java
 * Object Serialization Specification</cite> for details.
 *
 * <p> Note that when using an enumeration type as the type of a set
 * or as the type of the keys in a map, specialized and efficient
 * {@linkplain java.util.EnumSet set} and {@linkplain
 * java.util.EnumMap map} implementations are available.
 *
 * @param <E> The type of the enum subclass
 * @serial exclude
 * @author  Josh Bloch
 * @author  Neal Gafter
 * @see     Class#getEnumConstants()
 * @see     java.util.EnumSet
 * @see     java.util.EnumMap
 * @jls 8.9 Enum Classes
 * @jls 8.9.3 Enum Members
 * @since   1.5
 */
@SuppressWarnings("serial") // No serialVersionUID needed due to
                            // special-casing of enum classes.
public abstract class Enum<E extends Enum<E>>
        implements Constable, Comparable<E>, Serializable {
    /**
     * The name of this enum constant, as declared in the enum declaration.
     * Most programmers should use the {@link #toString} method rather than
     * accessing this field.
     */
    private final String name;

    /**
     * Returns the name of this enum constant, exactly as declared in its
     * enum declaration.
     *
     * <b>Most programmers should use the {@link #toString} method in
     * preference to this one, as the toString method may return
     * a more user-friendly name.</b>  This method is designed primarily for
     * use in specialized situations where correctness depends on getting the
     * exact name, which will not vary from release to release.
     *
     * @return the name of this enum constant
     */
    public final String name() {
        return name;
    }

    /**
     * The ordinal of this enumeration constant (its position
     * in the enum declaration, where the initial constant is assigned
     * an ordinal of zero).
     *
     * Most programmers will have no use for this field.  It is designed
     * for use by sophisticated enum-based data structures, such as
     * {@link java.util.EnumSet} and {@link java.util.EnumMap}.
     */
    private final int ordinal;

    /**
     * Returns the ordinal of this enumeration constant (its position
     * in its enum declaration, where the initial constant is assigned
     * an ordinal of zero).
     *
     * Most programmers will have no use for this method.  It is
     * designed for use by sophisticated enum-based data structures, such
     * as {@link java.util.EnumSet} and {@link java.util.EnumMap}.
     *
     * @return the ordinal of this enumeration constant
     */
    public final int ordinal() {
        return ordinal;
    }

    /**
     * Sole constructor.  Programmers cannot invoke this constructor.
     * It is for use by code emitted by the compiler in response to
     * enum class declarations.
     *
     * @param name - The name of this enum constant, which is the identifier
     *               used to declare it.
     * @param ordinal - The ordinal of this enumeration constant (its position
     *         in the enum declaration, where the initial constant is assigned
     *         an ordinal of zero).
     */
    protected Enum(String name, int ordinal) {
        this.name = name;
        this.ordinal = ordinal;
    }

    // продолжение следует
```

### ordinal и name

Обратите внимание на конструктор и поля `ordinal`, `name`:

```java
    protected Enum(String name, int ordinal) {
        this.name = name;
        this.ordinal = ordinal;
    }
```

Т.е. все значения `enum` имеют имя, а также еще и числовое значение, порядковый номер.
Для получения этого номера также есть метод `ordinal`, который возвращает порядковый номер константы в перечислении, считая от 0:

```java
    /**
     * Returns the ordinal of this enumeration constant (its position
     * in its enum declaration, where the initial constant is assigned
     * an ordinal of zero).
     *
     * Most programmers will have no use for this method.  It is
     * designed for use by sophisticated enum-based data structures, such
     * as {@link java.util.EnumSet} and {@link java.util.EnumMap}.
     *
     * @return the ordinal of this enumeration constant
     */
    public final int ordinal() {
        return ordinal;
    }
```

Для получения имени:

```java
    /**
     * Returns the name of this enum constant, exactly as declared in its
     * enum declaration.
     *
     * <b>Most programmers should use the {@link #toString} method in
     * preference to this one, as the toString method may return
     * a more user-friendly name.</b>  This method is designed primarily for
     * use in specialized situations where correctness depends on getting the
     * exact name, which will not vary from release to release.
     *
     * @return the name of this enum constant
     */
    public final String name() {
        return name;
    }
```

Простой пример:

```java
enum Shape { RECTANGLE, SQUARE, CIRCLE, TRIANGLE; }

public class Main {
    public static void main(String[] args) {
        System.out.println(Shape.RECTANGLE.name()); // RECTANGLE
        System.out.println(Shape.RECTANGLE.ordinal()); // 0

        System.out.println(Shape.CIRCLE.name()); // CIRCLE
        System.out.println(Shape.CIRCLE.ordinal()); // 2
    }
}
```

---

**Вопрос**:

Как видно из `JavaDoc`, большинство программистов не использует `ordinal` для построения логики своих программ.
Почему?

**Ответ**:

Это ненадажно и неявно. Любое изменение `enum`: удаление элемента, добавление нового изменяет номера всех.

```java
enum Shape { RECTANGLE, SQUARE, CIRCLE, TRIANGLE; }

public class Main {
    public static void main(String[] args) {
        System.out.println(Shape.RECTANGLE.name()); // RECTANGLE
        System.out.println(Shape.RECTANGLE.ordinal()); // 0

        System.out.println(Shape.CIRCLE.name()); // CIRCLE
        System.out.println(Shape.CIRCLE.ordinal()); // 2
    }
}
```

А теперь представьте, что вы построили логику на том, что `Shape.CIRCLE.ordinal()` - это всегда 2.
Спустя какое-то время вы понимаете, что `SQUARE` не используется и удаляете его.

И что же мы видим?

```java
enum Shape { RECTANGLE, CIRCLE, TRIANGLE; }

public class Main {
    public static void main(String[] args) {
        System.out.println(Shape.RECTANGLE.name()); // RECTANGLE
        System.out.println(Shape.RECTANGLE.ordinal()); // 0

        System.out.println(Shape.CIRCLE.name()); // CIRCLE
        System.out.println(Shape.CIRCLE.ordinal()); // 1
    }
}
```

Значение `ordinal` изменилось! А мы об этом даже не узнали. Все компилируется, запускается, но номер перечисления поменялся.
Поэтому закладывать логику на `ordinal` не стоит.

---

### values

Также у `enum` существует возможность получить массив всех возможных значений перечисления в том порядке, в котором они объявлены. За это отвечает статический метод `values()`:

```java
for (Shape e : Shape.values())
    System.out.println(e);
```

Выполнив получаем:

```java
RECTANGLE
SQUARE
CIRCLE
TRIANGLE
```

### valueOf

Также существует возможность получить из значение перечисления, за это отвечает статический метод `valueOf(String name)`, который вернет ссылку на константу перечисления по её имени, либо выкинет исключение `java.lang.IllegalArgumentException`:

```java
public class EnumExample {
    public static void main(String[] args) {
        System.out.println(Shape.RECTANGLE);
        System.out.println(Shape.valueOf("RECTANGLE"));
        System.out.println(Shape.valueOf("TWO_ERROR")); // java.lang.IllegalArgumentException
    }
}

enum Shape { RECTANGLE, SQUARE, CIRCLE, TRIANGLE; }
```

При этом **важно** понимать, что поиск константы ведется по имени `name`!
Т.е. при попытке `Shape.valueOf("rectangle")` будет также выброшено исключение! Перечисления **чувствительны** к регистру!

Из этого следует, что если вы хотите получить значение `enum` из строки, то обязательно приводите ее к тому виду, как у вас записано значение.

На самом деле, внимательный читатель байткода уже мог с самого начала обратить внимание на присутствие этих методов, так как если перевести из байткода наш `enum` в `Java`, то мы бы видели нечто подобное:

```java
public class LShape extends Enum<LShape> {

    public static final LShape RECTANGLE;
    public static final LShape SQUARE;
    public static final LShape CIRCLE;
    public static final LShape TRIANGLE;
    private static final LShape[] VALUES;

    protected LShape(String name, int ordinal) {
        super(name, ordinal);
    }

    static {
        RECTANGLE = new PineIsNotEnum("RECTANGLE", 0);
        SQUARE = new PineIsNotEnum("SQUARE", 1);
        CIRCLE = new PineIsNotEnum("CIRCLE", 2);
        TRIANGLE = new PineIsNotEnum("TRIANGLE", 3);
        VALUES = new LShape[] {RECTANGLE, SQUARE, CIRCLE, TRIANGLE};
    }

    public static LShape[] values() {
        return VALUES.clone();
    }

    public static LShape valueOf(String name) {
        return Enum.valueOf(LShape.class, name);
    }
}
```

Именно поэтому и регистр в использовании `valueOf` так важен, и `ordinal` проставляется так и `values` - это статический массив значений перечисления.

### compareTo

Обратим внимание на интерфейс `Comparable<E>`. Интерфейс добавляет поведение сравнения элементов, добавляя метод `compareTo`:

```java
    /**
     * Compares this enum with the specified object for order.  Returns a
     * negative integer, zero, or a positive integer as this object is less
     * than, equal to, or greater than the specified object.
     *
     * Enum constants are only comparable to other enum constants of the
     * same enum type.  The natural order implemented by this
     * method is the order in which the constants are declared.
     */
    public final int compareTo(E o) {
        Enum<?> other = (Enum<?>)o;
        Enum<E> self = this;
        if (self.getClass() != other.getClass() && // optimization
            self.getDeclaringClass() != other.getDeclaringClass())
            throw new ClassCastException();
        return self.ordinal - other.ordinal;
    }
```

Благодаря этому `enum` могут быть использованы с `TreeMap`, `TreeSet` и другими структурами данных, которым для работы необходимо сравнение элементов.

### Serializable

Стандартная сериализация с помощью `java.io.Serializable` у `enum` обстоит несколько иначе, чем у обычных объектов.

Сериализованная форма константы перечисления состоит исключительно из ее имени, значения поля константы отсутствуют.

При сериализации `enum`  записывается значение, возвращаемое уже занкомым нам методом имени константы перечисления. Чтобы десериализовать константу перечисления, считывается имя константы из потока, а затем десериализованная константа получается путем вызова метода `java.lang.Enum.valueOf`.

## Советы по работе с enum

Так как значения `enum` - это статические финальные поля, по сути константы, объединенные логически с некоторым поведением, то и называют их обычно также как и [константы](./code_style.md).

Отсюда и в примерах выше встречались `RECTANGLE`, `CREDIT` и т.д.

Так как `valueOf` - это крайне капризная вещь, `name` из-за наименований возвращает всегда значения в `uppercase`, то пользоваться этим не всегда удобно.

Обычно я оформляю `enum` следующим образом (приведу пример из реального проекта):

```java
public enum MarkingCodeExtension {
    LP("lp", "Легкая промышленность (одежда)"),
    SHOES("shoes", "Обувные товары"),
    PHARMA("pharma", "Лекарственные препараты"),
    TOBACCO("tobacco", "Табачная продукция"),
    TIRES("tires", "Шины и покрышки"),
    PHOTO("photo", "Фототехника"),
    PERFUM("perfum", "Духи и туалетная вода"),
    MILK("milk", "Молочная продукция"),
    BICYCLE("bicycle", "Велосипеды"),
    WHEELCHAIRS("wheelchairs", "Кресла-коляски"),
    OTP("otp", "Альтернативная табачная продукция"),
    WATER("water", "Упакованная вода");

    private final String code;

    private final String description;

    private static final Map<String, MarkingCodeExtension> MAP = Collections.unmodifiableMap(Arrays.stream(MarkingCodeExtension.values())
            .collect(Collectors.toMap(
                    MarkingCodeExtension::getCode,
                    x -> x
            ))
    );

    public static Optional<MarkingCodeExtension> findByCode(String code) {
        return Optional.ofNullable(MAP.get(code.toLowerCase()));
    }

    public static MarkingCodeExtension getByCode(String code) {
        return findByCode(code).orElseThrow(() -> new RuntimeException("Failed to find enum by code: " + code));
    }

    public String getDescription() {
        return description;
    }

    MarkingCodeExtension(String code, String description) {
        this.code = code;
        this.description = description;
    }

    public String getCode() {
        return code;
    }
}
```

Обязательно вводится поле `code`, которое по сути является идентификатором для перечисления. По нему будет поиск, его удобно записывать в БД, отдавать на фронт, в целом работать с ним.

Это необязательно текстовое поле, вполне можно и числовое значение использовать, при необходимости.

Это же поле участвует и в преобразовании строки в значение `enum`, благодаря методам: `findByCode` и `getByCode`. Один из них возвращает `Optional<MarkingCodeExtension>`, другой `MarkingCodeExtension` или исключение (можно сделать более конкретное свое исключение при необходимости), для более гибкого использования.

Поле `description` необязательно, но для удобства бывает полезно. Например, для какого-то короткого замещения `toString`, человекочитаемого значения в лог.

А вот пример того, как оформляются `enum`-ы в [Apache Kafka](https://kafka.apache.org/), листинг взят из их зеркала на [github](https://github.com/apache/kafka/blob/trunk/clients/src/main/java/org/apache/kafka/common/resource/ResourceType.java):

```java
/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements. See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License. You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.apache.kafka.common.resource;

import org.apache.kafka.common.annotation.InterfaceStability;

import java.util.HashMap;
import java.util.Locale;

/**
 * Represents a type of resource which an ACL can be applied to.
 *
 * The API for this class is still evolving and we may break compatibility in minor releases, if necessary.
 */
@InterfaceStability.Evolving
public enum ResourceType {
    /**
     * Represents any ResourceType which this client cannot understand,
     * perhaps because this client is too old.
     */
    UNKNOWN((byte) 0),

    /**
     * In a filter, matches any ResourceType.
     */
    ANY((byte) 1),

    /**
     * A Kafka topic.
     */
    TOPIC((byte) 2),

    /**
     * A consumer group.
     */
    GROUP((byte) 3),

    /**
     * The cluster as a whole.
     */
    CLUSTER((byte) 4),

    /**
     * A transactional ID.
     */
    TRANSACTIONAL_ID((byte) 5),

    /**
     * A token ID.
     */
    DELEGATION_TOKEN((byte) 6);

    private final static HashMap<Byte, ResourceType> CODE_TO_VALUE = new HashMap<>();

    static {
        for (ResourceType resourceType : ResourceType.values()) {
            CODE_TO_VALUE.put(resourceType.code, resourceType);
        }
    }

    /**
     * Parse the given string as an ACL resource type.
     *
     * @param str    The string to parse.
     *
     * @return       The ResourceType, or UNKNOWN if the string could not be matched.
     */
    public static ResourceType fromString(String str) throws IllegalArgumentException {
        try {
            return ResourceType.valueOf(str.toUpperCase(Locale.ROOT));
        } catch (IllegalArgumentException e) {
            return UNKNOWN;
        }
    }

    /**
     * Return the ResourceType with the provided code or `ResourceType.UNKNOWN` if one cannot be found.
     */
    public static ResourceType fromCode(byte code) {
        ResourceType resourceType = CODE_TO_VALUE.get(code);
        if (resourceType == null) {
            return UNKNOWN;
        }
        return resourceType;
    }

    private final byte code;

    ResourceType(byte code) {
        this.code = code;
    }

    /**
     * Return the code of this resource.
     */
    public byte code() {
        return code;
    }

    /**
     * Return whether this resource type is UNKNOWN.
     */
    public boolean isUnknown() {
        return this == UNKNOWN;
    }
}
```

Как видите, подход схожий.
Идея одна и та же: не надеемся на номер перечисления, вводим свой код, вспомогательные методы и реакцию на то, если невозможно через `fromCode` получить значение `enum`, все подробно описываем `JavaDoc`-ом.

## Секция вопросов

**Вопрос**:

А могут ли быть `enum` **без** ничего? Без экземпляров, но с методами?

**Ответ**:

Могут!

Но вам потребуется добавить `;` для того, чтобы усмирить компилятор.

```java
public enum Example {
    ;
}
```

---

**Вопрос**:

Может ли быть абстрактный `enum`?

```java
public abstract enum DocumentType {
    CREDIT,
    PAYMENT;
}
```

**Ответ**:

Нет!
Что в целом логично, ведь абстракция подразумевает незавершенность, которая будет дополнена потомками. А потомков у `enum`-ов быть не может.

**Вопрос**:

Могут ли у `enum` быть абстрактные методы? Ведь это же класс!

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

**Вопрос**:

Могут ли с `enum` использоваться ключевые слова `final`?

```java
public final enum DocumentType {
    CREDIT,
    PAYMENT;
}
```

**Ответ**:

Нет!

Что тоже логично, так как `final` у класса подразумевает его полную завершенность, запрет на использование в наследовании. А для `enum`-ов это **излишне**, они и без этого не могут участвовать в наследовании.

При этом поля у `enum` могут использовать `final` модификатор.

```java
public enum DocumentType {
    CREDIT ("Кредитный документ"),
    PAYMENT ("Оплата");

   private final String description;

   DocumentType(String description) {
       this.description = description;
   }
}
```

**Вопрос**:

Что будет в результате выполнения кода:

```java
enum Outcome {
    SUCCESS, FAILURE;
    {System.out.print("IO ");}
    Outcome() {System.out.print("CO ");}
    static {System.out.print("SO ");}
}

class TestCase {  
    Outcome result;
    public TestCase() {
        System.out.print("CTC ");
    }
    { result = Outcome.SUCCESS; System.out.print("ITC ");}
}

// выполняем в psvm
TestCase tc1 = new TestCase();
TestCase tc2 = new TestCase();
```

**Ответ**:

Результат выполнения:

```java
IO CO IO CO SO ITC CTC ITC CTC
```

Больше вопросов [тут](https://blogs.oracle.com/javamagazine/post/quiz-yourself-initializing-enums-in-java-code)

---

## Заключение

Использование `enum` позволяет ограничить множество допустимых значений для некоторого типа данных.

Элементы `enum` - это статически доступные экземпляры.
При этом все экземпляры этого класса уже объявлены - перечислены.

Грубо говоря, `enum` - это точно такой же класс, у которого может быть состояние (свойства класса), поведение (методы), конструкторы, он может реализовывать интерфейсы, но лишен права участвовать в наследовании.

Также нельзя создавать экземпляры `enum` вне границ `enum`, поскольку у `enum` нет `public` конструктора.

Помимо всего прочего, у `enum` присутствуют вспомогательные методы, наподобие `ordinal`, `name`, `values`.

Все `enum` можно использовать в `TreeMap`, `TreeSet` и т.д. так как они реализуют интерфейс `java.lang.Comparable`.

## Полезные ссылки

- [Enum in Java](http://www.quizful.net/post/java_enums)
- [Сравнение enum](https://javarevisited.blogspot.com/2013/04/how-to-compare-two-enum-in-java-equals.html)
- [Загадки Enum'ов](https://habr.com/ru/post/575208/)
- [Why can't a Java enum be final?](https://stackoverflow.com/questions/9891613/why-cant-a-java-enum-be-final#:~:text=Java%20does%20not%20allow%20you,inside%20of%20the%20enum%20descriptor)
- [Ограничения на создание enum](https://docs.oracle.com/javase/specs/jls/se11/html/jls-8.html#jls-8.9.2)
- [Сериализация enum](https://docs.oracle.com/javase/6/docs/platform/serialization/spec/serial-arch.html#6469)
- [Про сериализацию enum](https://stackoverflow.com/questions/15521309/is-custom-enum-serializable-too)
- [JLS о enum](https://docs.oracle.com/javase/specs/jls/se11/html/jls-8.html#jls-8.9)
- [Execution order of Enum in Java](https://stackoverflow.com/questions/28296547/execution-order-of-enum-in-java/50184535#50184535)
