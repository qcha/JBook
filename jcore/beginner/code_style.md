# Правила оформления кода

- [Правила оформления кода](#правила-оформления-кода)
  - [1.1. Введение](#11-введение)
  - [1.2. Наименования классов и интерфейсов](#12-наименования-классов-и-интерфейсов)
    - [1.2.1. Стиль](#121-стиль)
    - [1.2.2. Рекомендации](#122-рекомендации)
  - [1.3. Наименования переменных](#13-наименования-переменных)
    - [1.3.1. Стиль](#131-стиль)
    - [1.3.2. Рекомендации](#132-рекомендации)
      - [1.3.2.1. Свойства класса](#1321-свойства-класса)
    - [1.3.3. Место объявления переменной](#133-место-объявления-переменной)
  - [1.4. Наименования методов](#14-наименования-методов)
    - [1.4.1. Стиль](#141-стиль)
    - [1.4.2. Рекомендации](#142-рекомендации)
  - [1.5. Boolean](#15-boolean)
  - [1.6. Наименования пакетов](#16-наименования-пакетов)
  - [1.7. Общие рекомендации](#17-общие-рекомендации)
    - [1.7.1. Обрамление логических выражений фигурными скобками](#171-обрамление-логических-выражений-фигурными-скобками)
    - [1.7.2. Выделение логических блоков кода](#172-выделение-логических-блоков-кода)
    - [1.7.3. Дробление кода](#173-дробление-кода)
  - [1.8. Комментарии](#18-комментарии)
  - [1.9. JavaDoc](#19-javadoc)
    - [1.9.1. JavaDoc на поля](#191-javadoc-на-поля)
    - [1.9.2. JavaDoc на методы](#192-javadoc-на-методы)
    - [1.9.3. JavaDoc на классы](#193-javadoc-на-классы)
  - [Заключение](#заключение)

## 1.1. Введение

Программист большую часть времени занимается не написанием кода, а чтением и изучением уже написанной кодовой базы.
Поэтому оформление кода это не менее важная часть процесса его написания.

Хорошо написанный и оформленный код может рассказать о программе большую часть еще до ее запуска.

Зачастую (но, очевидно, не всегда), даже если есть выбор: написать **чуть** более производительный код или гораздо более читаемый, то стоит сделать выбор в сторону более читаемого.

> Преждевременные оптимизации - корень всех зол.
>
> (c) Дональд Кнут.

Также можно столкнуться с работами, где люди, пришедшие в `Java` с других языков, тянут свои привычки.
Но надо понимать, что и в `Java` есть свои правила, которым надо следовать.

Поэтому следование некоторым простым правилам является обязательным требованием к разработчикам.

В команде или на проекте может быть свой `code style`, но, даже если так, на данный момент основой является [Google Style Guide](https://google.github.io/styleguide/javaguide.html).

Можно посмотреть и на [Oracle Code Conventions](https://www.oracle.com/java/technologies/javase/codeconventions-contents.html), однако, это довольно старый документ и, на мой взгляд, Google Style Guide - это нынешний стандарт.

## 1.2. Наименования классов и интерфейсов

### 1.2.1. Стиль

Классы принято называть именами существительными, начинающимися с прописной буквы.
При наименовании классов и интерфейсов в `Java` придерживаются стиля написания [CamelCase](https://ru.wikipedia.org/wiki/CamelCase).

Это стиль написания составных слов, при которому несколько слов пишутся слитно без пробелов, при этом каждое слово внутри фразы пишется с прописной буквы.

Например:

```java
public class UserService {}
public class Hasher {}
public class FileUtils {}
```

Запомните, что имя класса, перечисления или интерфейса **всегда** пишется с заглавной буквы!

Правила наименования интерфейсов идентичны классам:

```java
public interface List {}
```

При этом, если интерфейс добавляет только конкретное действие, то имеет смысл в название добавить окончание `able`:

```java

public interface Closable {}
public interface Clonable {}
```

Смысл такой, что вы показываете в названии что именно за действие добавляется: ваш класс становится закрываемым, т.е. приобретает возможность закрывать какие-то ресурсы, например.

### 1.2.2. Рекомендации

Помните, что имя класса - это то, с чего начинается чтение любого кода в `Java`.
Имя класса не должно быть слишком длинным, но при этом обязано отражать задачу, которую этот класс призван решать.

Основой названия класса, его корнем, должно быть слово наиболее подходящее под описание сути класса, при этом не обязательно, чтобы это было одно слово.

Например, как бы вы назвали неизменяемый список?

```java
class ImmutableList {}
```

Старайтесь избегать множественного числа в названиях классов, так как это размывает ответственность класса и понимание зачем он нужен. Сравните:

```java
class User
class Users
```

Из названия `User` уже понятно, что этот класс скорее всего модель, внутри будет информация о пользователе, например имя или номер телефона (в зависимости от предметной области).

Что можно сказать о том, что ждет нас в пользователях? Это хранилище пользователей? Тогда было бы лучше назвать `UserStorage`, `UserRepository` или подобным образом. Если это не хранилище, то это группировка пользователей? Но по какому признаку? Название уже не отражает то, что можно ждать - явно не хватает деталей.

Однако иногда бывает так, что есть несколько утилитных статических методов, относящихся к одной предметной области. Например, к работе с файлами.
У вас есть статические методы для чтения, записи в файл, проверки прав доступа и так далее. Как его лучше назвать?

Выделяете суть: `File`, далее задумайтесь: все эти статические методы - это что? Это вспомогательные методы, как утилиты!
Вот нам и название: `FileUtils`.

Здесь мы можем использовать множественное число - так как говорим об утилитах для файла. Контекст операций отражается в названии.

Иногда используют еще приставку `Helper`, но мне `Utils` нравится больше.

Обычно такие утилитные классы не должны участвовать в наследовании, поэтому имеют еще модификатор `final`.

Если класс участвует в наследовании или реализует интерфейс, то задумайтесь о том, чтобы прибавить к имени вашего класса какой-то корень имени класса-родителя или интерфейса:

```java
public interface Parser {
    // some code
}

public class JsonParser implements Parser {}

public class Criteria {
    // some code
}

public class SelectCriteria extends Criteria {}
```

Исключениями из этого правила могут быть классы, название которых и так говорит о том, что данный класс наследует супер-класс:

```java
public class Figure {}
public class Triangle extends Figure {}
```

Имена классов-исключений должны заканчиваться на `Exception`, чтобы выделять сразу из названия, что это исключение:

```java
public class InvalidUserException extends Excepton {}
public class UserException extends RuntimeExcepton {}
```

## 1.3. Наименования переменных

### 1.3.1. Стиль

При наименовании переменных в `Java` придерживаются стиля написания [lowerCamelCase](https://ru.wikipedia.org/wiki/CamelCase).

Данный стиль написания говорит о том, что имена переменных (если это не константы) пишутся слитно, где все слова, кроме первого, начинаются с прописной буквы.

Например:

```java
private int maxSize = 10;
private List<Person> persons;
```

Имена констант же принято писать заглавными буквами, разделяя слова знаком нижнего подчеркивания `_`:

```java
public static final int MAX_VALUE = 64; // Хорошо!
private static final String name = "NAME_1"; // Плохо!
```

### 1.3.2. Рекомендации

Переменные предназначены для хранения состояния.
Они могут быть локальными, принадлежать объекту класса или быть статическими, т.е принадлежать непосредственно классу.

По сути это очень напоминает области видимости: если переменная объявлена в методе, то ее область видимости - это этот метод. При объявлении полем класса, то в зависимости от модификатора доступа переменной, область видимости может быть как этот класс, так и внешние сущности.

В зависимости от того, к чему, к какой области видимости, переменная принадлежит, будет требоваться более подробное, многословное название и более высокие требования к неймингу. Хотя бы потому, что имея широкую область видимости переменная должна в любой момент явно своим названием ответить разработчику на вопрос: что переменная хранит и зачем нужна?

Это значит, что у вас не должно быть кода, похожего на подобный:

```java
private int a = 1;
```

Так как что такое `a` и почему оно `int` знаете только вы в ближайшие сорок минут. Дальше вы не вспомните что такое `a`, потом понятия не будете иметь почему вдруг `a` - это `int`.

Однако из этого **не следует** то, что каждую переменную вы должны называть и описывать максимально подробно. Тут действует основное правило: чем меньше(уже) область использования - тем короче имя переменной.

Если переменая объявляется в пределах метода или цикла, то давать подробное имя не имеет смысла.

Давайте продемонстрируем эту мысль в коде:

```java
for(int indexOfElementInArray = 0; indexOfElementInArray < array.len; indexOfElementInArray++) {
    System.out.println(array[indexOfElementInArray]);
    //some logic
}
```

Понятнее не стало, более того, это вносит путаницу, хаос и катастрофически съедает все место на экране.

Гораздо более лучше сократить это до:

```java
for(int index = 0; index < array.len; index++) {
    System.out.println(array[index]);
    //some logic
}
```

Либо вообще заменить `index` на `i`:

```java
for(int i = 0; index < array.len; i++) {
    System.out.println(array[i]);
    //some logic
}
```

Учитывая, что `Java` в целом довольно многословный язык, добавлять еще свои пять копеек в эту копилку не стоит, тем более, что использовать эту переменную вы будете только в области цикла.

Помните: чем короче область видимости (использования) переменной - тем короче ее имя.

#### 1.3.2.1. Свойства класса

Для улучшения читаемости лучше разбить свойства класса на логические блоки: то, что относится к классу (константы и статика) и то, что относится к объектам класса:

```java
public class ClusterHealthCheckWorker {
    private static final Logger LOGGER = LoggerFactory.getLogger(ClusterHealthCheckWorker.class);
    private static final String WORKER_TYPE = "Health check worker";
    private static final String WORKER_ID = "HealthCheckWorker";

    private final ClusterInfoService clusterInfoService;
    private final KubernetesClientService kubernetesClientService;
    private final ClusterType clusterType;
    private final String podLabelName;
    private final String podLabelValue;
```

Выделяем сразу блок для `static` и переносом строки отделяем от того, что относится к объектам.

Также надо помнить еще и то, что частично о смысле переменной, являющейся свойством класса, говорит и название самого класса.

Например:

```java
public class Сonnection {
    private int port;
    private String host;
}
```

Тут совершенно излишне уже писать `connectionPort`, например, так как из названия класса ясно, что свойство относится именно к `connection`.

Обязательно **задумывайтесь о названии переменных** - это если не половина, то треть обеспечения будущей поддержки, развития и переиспользования вашего кода.

Теперь, когда мы разобрали как выбрать имя для переменной, пришло время поговорить о том, как влияет место объявления на читаемость кода.

### 1.3.3. Место объявления переменной

То, где вы объявили переменную также может влиять на читаемость вашего кода:

```java
public void print(int[] array) {
    int i = 0;
    // some code
    // and another code
    // still code

    while(i != array.size) {
        System.out.println(array[i]);
        i++;
    }
}
```

Если между объявлением переменной и непосредственным местом ее использования содержится много логики, кода и т.д, то это ***существенно понижает** читаемость кода.

Все дело в том, что человеку очень тяжело держать в уме весь контекст, а чем дальше место объявление переменной - тем больше нужно помнить.

Поэтому старайтесь придерживаться правила: локальные переменные желательно объявлять ближе к месту использования.
Согласитесь, что если мы в начале метода объявим весь список переменных с которыми нам придется столкнуться и напишем объемный кусок кода, то такое объявление только запутает.

Возвращаясь к нашему примеру:

```java
for(int i = 0; index < array.len; i++) {
    System.out.println(array[i]);
    //some logic
}
```

Объявление переменной, хранящей индекс массива происходит непосредственно в месте использования, все сгруппировано и читаемо.

## 1.4. Наименования методов

### 1.4.1. Стиль

Названия методов должны быть глаголами, первая буква должна быть строчной, первые буквы внутренних слов — заглавные.
При наименовании методов в `Java` придерживаются стиля написания [lowerCamelCase](https://ru.wikipedia.org/wiki/CamelCase).

Например:

```java
public int getPort();
public String toLowerCase();
```

### 1.4.2. Рекомендации

Из того, что метод - это поведение объекта, следует, что в названии метода предпочтительнее использовать глаголы, которые как можно более точно и полно описывают то, что делает метод.

Помните, что метод обязан передавать суть своей работы в названии.

Худшее, что можно сделать при задании имени метода - это дать имя, которое не соответствует действию.

Например:

```java
//название метода не отображает его суть - плохо
public void getHost() {}
```

Из названия мы ожидаем получить некоторый хост, однако возвращаемый тип метода `void`.
Это вводит в ступор и требует дополнительных усилий, чтобы понять почему `getHost` ничего не вернул.

Также помните, что если ваш метод возвращает коллекцию объектов, то и в имени должно содержаться отсылка к этому:

```java
List<String> getHosts() {}
Person findByName(String name) {}
```

Плохим стилем считается использование символов подчеркивания, цифр и знаков пунктуации:

```java
//подчеркивание лишнее - плохо
public void send_request() {}

//начинается с большой буквы - плохо
public void SendRequest() {}

//название метода не отображает его суть (непонятно что это) - плохо
public void methodOne() {}
```

Хорошим вариантом будет, например:

```java
public boolean sendRequest() {}
```

Имена методов, выполняющих чтение/изменение значений полей класса, должны начинаться со слов `get` и `set`.

> Эти методы называются еще `get`-ы и `set`-ы, и тесно связаны с [инкапсуляцией](../oop/encapsulation.md).

Имена методов, выполняющих преобразование к другому типу данных, желательно должны начинаться на `to`:

```java
public String toString() {}
```

Имена методов, которые создают и возвращают созданный объект, желательно должны начинаться с `create`, `build` и т.д.:

```java
public Record createRecord() {}
```

Помните, что метод неразрывно связан с классом и имя этого класса тоже участвует в понимании действия метода:

```java
public class RequestValidator {
    public void validate(String request) throws ValidationException {}
}
```

Здесь имя класса подскажет вам, что метод производит валидацию именно запроса.

Отдельной строкой стоит сказать про именование методов и переменных, возвращаемых `boolean` тип.

## 1.5. Boolean

Переменные типа `boolean`имеют только два состояния: `true` или `false`.

Чтобы подчеркнуть это при именовании таких переменных и методов, их возвращающих, имеет смысл использовать префиксы `is` или `has`.

```java
boolean isInitialized;
boolean hasNext();
```

Благодаря этому, использование таких имен выглядит в коде довольно органично и понятно:

```java
while(hasNext()) {
    // some code
}
```

Это улучшает читабельность вашего кода, при этом, если вы встречаете переменную с таким префиксом, то в 99% случаев это будет именно `boolean` переменная.

## 1.6. Наименования пакетов

Имена пакетов дают дополнительную информацию о том, что за классы лежат внутри и какие задачи они призваны решать.

Например:

```java
package org.apache.kafka.streams.errors;
```

Из названия пакета `errors` можно заключить, что там содержатся все кастомные классы-исключения, а из названия пакетов `org.apache.kafka.streams` можно понять, что эти ошибки относятся к работе с kafka-стримами.

Лично я в своей работе стараюсь не использовать названия в множественном числе (а вот коллеги из `Apache Kafka`, как видите, нет). Но вопросы такого рода решаются уже на уровне команды/проекта, каких-то требований по `code style` на сей счет нет.

Но помните, что имена пакетов и подпакетов должны быть существительными в нижнем регистре! Это требование.

У начинающих разработчиков часто наблюдается желание сделать название пакета составным, состоящим из нескольких слов.
Для достижения этой цели слова часто разделяют символом нижнего подчёркивания или даже пишутся в `camelCase`.

Например: `domain_tables` или `domainNames`.

Однако, на мой взгляд, это верный признак того, что над именем пакета не подумали. В ситуации, когда имя пакета состоит из нескольких слов, лучшим выходом будет разбить этот пакет на подпакеты.

Пример: `domain.tables`

Рассуждать здесь следует следующим образом: domain_tables - это таблицы домена(ов), значит можно выделить пакет для домена(ов) и явно группировать там то, что к нему непосредственно относится. Раз относятся таблицы - значит для них свой подпакет.
И если возникнет необходимость добавить к доменам еще что-то, например, регионы, то в `domain` можно будет спокойно добавить новый подпакет.

Очень похоже на то, как группируют по папкам документы в бухгалтерии!

В [Google Style Guide](https://google.github.io/styleguide/javaguide.html#s5.2.1-package-names) явно говорят, что разделители в именах пакетов запрещены:

> 5.2.1 Package names
>
> Package names use only lowercase letters and digits (no underscores). Consecutive words are simply concatenated together. For example, com.example.> deepspace, not com.example.deepSpace or com.example.deep_space.

Помните, что пакеты - это дополнительная возможность сгруппировать ваш код, поэтому пакеты должны содержать только те классы, которые логически могут быть там.

Если у вас существует пакет `parser`, то объявлять там класс `Person`, отвечающий за некоторую модель в вашем приложении не логично. Модель принадлежит проекту, а значит не может быть в пакете, где классы отвечают за парсинг.

## 1.7. Общие рекомендации

### 1.7.1. Обрамление логических выражений фигурными скобками

Очень важным моментом стоит выделить еще и оформление логических выражений `if/else`.

Несмотря на то, что язык позволяет писать в виде:

```java
if(num == 1) System.out.println("Hello");
else System.out.println("Hi");
```

Писать так я **крайне** не рекомендую.
Лично я предпочитаю вариант более явный:

```java
if(num == 1) {
    System.out.println("Hello");
} else {
    System.out.println("Hi");
}
```

На мой взгляд, это понятнее.
Но плюсы такой записи логических выражений на моих вкусовых предпочтениях не заканчиваются.
В такой записи логического ветвления **сложнее** сделать ошибку по невнимательности.

В качестве иллюстрации приведу такой пример:

```java
if(num == 1) System.out.println("Hello");
else System.out.println("Hi");
System.out.println("Else case!")
```

Мы добавляем к прошлом примеру одну строчку, в ожидании, что это также войдет в `else` ветку. Но ждет нас лишь горькое разочарование. При такой записи легче пропустить, что ваш новый код не попал в работу с логическим выражением. В варианте с обрамлением фигурными скобками вы защищены от такого рода ошибок.

Во избежание таких сюрпризов обрамляйте `if`-ы фигурными скобками `{}`.

### 1.7.2. Выделение логических блоков кода

Еще одним важным аспектом, который многие игнорируют является логическое выделение кода переносами строк. Зачастую можно встретить проекты, где пишут огромную лапшу, которая идет непрерывным водопадом от начала и до конца монитора, без каких-то логических резделений.

Как например тут:

```java
        Assert.assertEquals(0, postRepository.findAll().size());
        Assert.assertEquals(0, commentRepository.findAll().size());
        Assert.assertEquals(0, tagRepository.findAll().size());
        Post post = new Post("Title", "Description", "Content");
        Comment comment1 = new Comment("Content of Comment1");
        Comment comment2 = new Comment("Content of Comment2");
        Comment comment3 = new Comment("Content of Comment3");
        post.getComments().add(comment1);
        post.getComments().add(comment2);
        post.getComments().add(comment3);
        postRepository.save(post);
        Assert.assertEquals(1, postRepository.findAll().size());
        Assert.assertEquals(3, commentRepository.findAll().size());
        Assert.assertEquals(0, tagRepository.findAll().size());

        Optional<Post> persist = postRepository.getById(1L);
        Assert.assertTrue(persist.isPresent());
        Post savedPost = persist.get();
        Assert.assertEquals(3, savedPost.getComments().size());
        Assert.assertEquals(Sets.newHashSet(comment1, comment2, comment3), savedPost.getComments());
```

Весь код рабочий, но читается с трудом.

Большей читаемости можно достичь простыми переносами строк.
Явно выделяются блоки проверки репозиториев, инициализации тестовых данных, сохранения и новой проверки:

```java
        Assert.assertEquals(0, postRepository.findAll().size());
        Assert.assertEquals(0, commentRepository.findAll().size());
        Assert.assertEquals(0, tagRepository.findAll().size());

        Post post = new Post("Title", "Description", "Content");
        Comment comment1 = new Comment("Content of Comment1");
        Comment comment2 = new Comment("Content of Comment2");
        Comment comment3 = new Comment("Content of Comment3");

        post.getComments().add(comment1);
        post.getComments().add(comment2);
        post.getComments().add(comment3);

        postRepository.save(post);

        Assert.assertEquals(1, postRepository.findAll().size());
        Assert.assertEquals(3, commentRepository.findAll().size());
        Assert.assertEquals(0, tagRepository.findAll().size());

        Optional<Post> persist = postRepository.getById(1L);
        Assert.assertTrue(persist.isPresent());

        Post savedPost = persist.get();

        Assert.assertEquals(3, savedPost.getComments().size());
        Assert.assertEquals(Sets.newHashSet(comment1, comment2, comment3), savedPost.getComments());
```

Всего несколько разделителей строк, а получается гораздо читабельнее.

### 1.7.3. Дробление кода

В попытке повысить читаемость кода не надо уходить в крайности.

Иногда можно встретить классы, в которых объявлены методы в 1-2 строки, использующиеся только в одном месте и закрытые модификатором `private`, т.е предназначенные для использования только внутри этого класса.

Чаще всего это неоправданно и выглядит нелогично:

```java
class Example {
    
    public List<String> forUpdate(List<Node> nodes) {
        return nodes.stream().filter(node -> isForUpdate(node)).map(Node::getName).collect(Collectors.toList());
    }

    // Метод больше нигде не используется, кроме как в forUpdate и закрыт как private
    private boolean isForUpdate(Node node) {
        return !node.getName().isEmpty() && node.getValue().equals("Up!"); 
    }
}
```

А теперь представьте, что `forUpdate` и `isForUpdate` описаны не рядом и между ними еще есть много кода, добавьте сюда еще пару-тройку подобных методов-спутников в других частях класса и получим тяжелый и трудно читаемый код, так как при разборе поведения `forUpdate` вы будете обязаны искать метод-спутник `isForUpdate`, смотреть что там и возвращаться обратно.

Ведь согласитесь, что если `isEmptyNode` используется только в одном месте и сам по себе довольно компактный, то смысла для выделения специального метода под это нет - гораздо проще и читабельнее все поместить в предикат фильтра:

```java
class NodeParser {
    public List<String> parse(List<Node> nodes) {
        return nodes.stream()
                .filter(node -> !node.getName().isEmpty() && node.getValue().equals("Up!"))
                .map(Node::getName)
                .collect(Collectors.toList());
    }
}
```

Поэтому старайтесь не дробить излишне свой код. Не плодите сущности сверх необходимого!

## 1.8. Комментарии

Комментарии — это пояснения к исходному тексту программы, находящиеся непосредственно внутри комментируемого кода.

В `Java` существует две возможности добавить комменатрии к коду.

1. Строчный комментарий, начинающийся с `//`.

    ```java
    // Строчный
    ```

2. Блочный комментарий.

    ```java
    /*
    * Блочный комментарий
    */
    ```

В идеале необходимо стремиться к такому коду, который не нуждается в комментировании.

Идеал достигается не всегда, в дополнении к этому часто бывает так, что код выполняет запутанную бизнес-логику, либо вы написали временное решение.

И такие места, я считаю, нужно комментировать и объяснять.

Самой большой ошибкой будет злоупотребление комментариями в коде.
Так как это серьезно захламляет код, при этом не добавляя ничего нового, например:

```java
// url and password
private String url;
private String password;
```

Или:

```java
// если url не null
if (url == null) {
    httpClient.send(url, request);
}
```

Код не нуждается в комментировании, более того, это только усложняет его.

Существует даже совет:

> Если у вас есть время на комментирование кода - потратьте его на рефакторинг.

Однако из этого не следует, что комментирование вредно или не нужно, просто не стоит им злоупотрелять.

Например, с помощью комментариев можно объяснить группировку объявлений полей:

```java
    // left view for sources
    private VBox leftPane;
    private CheckBox selectAll;
    private ListView<SearchViewModel.SearchSource> lstCompanies;
    private Button initConfBtn;

    // right view for results
    private WebView resultView;
```

Вот, для примера, комментарий из `java.util.ArrayList`:

```java
    transient Object[] elementData; // non-private to simplify nested class access
```

### 1.9 JavaDoc

`Javadoc` — это стандарт для документирования классов, методов и переменных. Это **не** комментарии, это именно документация.
Документация бывает на поля класса, методы и сам класс.

Начало `JavaDoc` - это всегда `/**` и конец всегда `*/`, где каждая новая строка начинается с `*`.

Примеров `JavaDoc` много прямо в `JDK`, поэтому не стесняйтесь смотреть и перенимать лучшие практики.

Пример `JavaDoc` на метод:

```java
/**
 * Returns an Image object that can then be painted on the screen. 
 * The url argument must specify an absolute {@link URL}. The name
 * argument is a specifier that is relative to the url argument. 
 * <p>
 * This method always returns immediately, whether or not the 
 * image exists. When this applet attempts to draw the image on
 * the screen, the data will be loaded. The graphics primitives 
 * that draw the image will incrementally paint on the screen. 
 *
 * @param  url  an absolute URL giving the base location of the image
 * @param  name the location of the image, relative to the url argument
 * @return      the image at the specified URL
 * @see         Image
 */
 public Image getImage(URL url, String name) {
        try {
            return getImage(new URL(url, name));
        } catch (MalformedURLException e) {
            return null;
        }
 }
 ```

Все, что начинается с `@` называется `тегом`.
Например, `@see` тег - это ссылка на другое место в документации.

Список тегов и сущностей, к которым их можно применять:

| Тег               | Описание                                                           | Применение                    |
|-------------------|--------------------------------------------------------------------|-------------------------------|
| @author           | Автор                                                              | класс, интерфейс              |
| @version          | Версия. Не более одного тега на класс                              | класс, интерфейс              |
| @since            | Указывает, с какой версии `JDK` доступно                           | класс, интерфейс, поле, метод |
| @see              | Ссылка на другое место в документации                              | класс, интерфейс, поле, метод |
| @param            | Входной параметр метода                                            | метод                         |
| @return           | Описание возвращаемого значения                                    | метод                         |
| @throws           | Описание исключения, которое может быть послано из метода          | метод                         |
| @deprecated       | Описание устаревшего кода                                          | класс, интерфейс, поле, метод |
| {@link reference} | Ссылка                                                             | класс, интерфейс, поле, метод |
| {@value}          | Описание значения переменной                                       | статичное поле                |
| {@code}           | Для вставки фрагментов кода                                        | класс, интерфейс, поле, метод |
| {@inheritDoc}     | Документация будет унаследована от ближайшего родительского класса | класс, интерфейс, поле, метод |

#### 1.9.1 JavaDoc на поля

Документиация полей проста, главное, что необходимо помнить - это не скупитесь на примеры.
Чем больше написано - тем лучше.

Например, здесь:

```java
    /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;
```

Согласитесь, было бы гораздо лучше, если бы добавили информации почему именно 10 выбрано как `DEFAULT_CAPACITY`? Возможно, что через @link могла бы быть ссылка на исследование, может быть просто более подробные слова, но нужно больше деталей.

#### 1.9.2 JavaDoc на методы

Документация на методы начинается с описания и должна содержать теги: `@param`, `@return`, `@throws`.
В зависимости от возвращаемого типа, принимаемых параметров и бросаемых исключений.

Для примера рассмотрим метод `get` из `java.util.ArrayList`:

```java
    /**
     * Returns the element at the specified position in this list.
     *
     * @param  index index of the element to return
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E get(int index) {}
```

В начале идет непосредственно описание, далее тегами вы описываете что за парамтеры и какого типа принимаются, каково возвращаемое значение и бросаемые исключения.

Если метод не принимает аргументов тег @param не указывается, если метод принимает более одного параметра, то каждый параметр описывается отдельно со своим тегом `@param`:

```java
    /**
     * Replaces the element at the specified position in this list with
     * the specified element.
     *
     * @param index index of the element to replace
     * @param element element to be stored at the specified position
     * @return the element previously at the specified position
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E set(int index, E element) {}
```

Если возвращаемого значения или бросаемых исключений нет, то соответствующие теги не указываются.

При нескольких типах исключений вы также каждое исключение описываете отдельно, как и с `@param`.
В `@throws` необходимо указывать все исключения, которые выбрасывает метод: и `checked`, и `unchecked`.

#### 1.9.3 JavaDoc на классы

Правила те же, разве что можно добавить `@author`, `@since` и так далее. Но это не является обязательным требованием.

Главное, что надо понять - это то, что документация на класс должна быть подробной и развернутой.

Для примера рассмотрим снова `java.util.ArrayList`:

```java
/**
 * Resizable-array implementation of the {@code List} interface.  Implements
 * all optional list operations, and permits all elements, including
 * {@code null}.  In addition to implementing the {@code List} interface,
 * this class provides methods to manipulate the size of the array that is
 * used internally to store the list.  (This class is roughly equivalent to
 * {@code Vector}, except that it is unsynchronized.)
 *
 * <p>The {@code size}, {@code isEmpty}, {@code get}, {@code set},
 * {@code iterator}, and {@code listIterator} operations run in constant
 * time.  The {@code add} operation runs in <i>amortized constant time</i>,
 * that is, adding n elements requires O(n) time.  All of the other operations
 * run in linear time (roughly speaking).  The constant factor is low compared
 * to that for the {@code LinkedList} implementation.
 *
 * <p>Each {@code ArrayList} instance has a <i>capacity</i>.  The capacity is
 * the size of the array used to store the elements in the list.  It is always
 * at least as large as the list size.  As elements are added to an ArrayList,
 * its capacity grows automatically.  The details of the growth policy are not
 * specified beyond the fact that adding an element has constant amortized
 * time cost.
 *
 * <p>An application can increase the capacity of an {@code ArrayList} instance
 * before adding a large number of elements using the {@code ensureCapacity}
 * operation.  This may reduce the amount of incremental reallocation.
 *
 * <p><strong>Note that this implementation is not synchronized.</strong>
 * If multiple threads access an {@code ArrayList} instance concurrently,
 * and at least one of the threads modifies the list structurally, it
 * <i>must</i> be synchronized externally.  (A structural modification is
 * any operation that adds or deletes one or more elements, or explicitly
 * resizes the backing array; merely setting the value of an element is not
 * a structural modification.)  This is typically accomplished by
 * synchronizing on some object that naturally encapsulates the list.
 *
 * If no such object exists, the list should be "wrapped" using the
 * {@link Collections#synchronizedList Collections.synchronizedList}
 * method.  This is best done at creation time, to prevent accidental
 * unsynchronized access to the list:<pre>
 *   List list = Collections.synchronizedList(new ArrayList(...));</pre>
 *
 * <p id="fail-fast">
 * The iterators returned by this class's {@link #iterator() iterator} and
 * {@link #listIterator(int) listIterator} methods are <em>fail-fast</em>:
 * if the list is structurally modified at any time after the iterator is
 * created, in any way except through the iterator's own
 * {@link ListIterator#remove() remove} or
 * {@link ListIterator#add(Object) add} methods, the iterator will throw a
 * {@link ConcurrentModificationException}.  Thus, in the face of
 * concurrent modification, the iterator fails quickly and cleanly, rather
 * than risking arbitrary, non-deterministic behavior at an undetermined
 * time in the future.
 *
 * <p>Note that the fail-fast behavior of an iterator cannot be guaranteed
 * as it is, generally speaking, impossible to make any hard guarantees in the
 * presence of unsynchronized concurrent modification.  Fail-fast iterators
 * throw {@code ConcurrentModificationException} on a best-effort basis.
 * Therefore, it would be wrong to write a program that depended on this
 * exception for its correctness:  <i>the fail-fast behavior of iterators
 * should be used only to detect bugs.</i>
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/java.base/java/util/package-summary.html#CollectionsFramework">
 * Java Collections Framework</a>.
 *
 * @param <E> the type of elements in this list
 *
 * @author  Josh Bloch
 * @author  Neal Gafter
 * @see     Collection
 * @see     List
 * @see     LinkedList
 * @see     Vector
 * @since   1.2
 */
```

Не скупитесь на слова, чем более подробно документируете код - тем лучше.
Для некоторых классов (например, dto-объектов) можно даже указать пример представления. Т.е. если у вас заведены `dto`-объекты для представления в `json` формат, то не ленитесь и добавьте это представление (вид) объекта в `json` формате как пример!

## Заключение

Помните, что от правильного наименования выигрывают все: и вы, и разработчики, использующие ваш код.
Поэтому старайтесь максимально ответственно подходить к этому моменту, думайте над названиями классов, методов, пакетов и переменных.

Всегда помните, что имена полей и локальных переменных не должны вводить в заблуждение относительно их типов.
Не забывайте про префиксы `is`, `has` при работе с `boolean` переменными и методами!

Если вы автор какого-то метода, то именно вы в ответе за поведение метода и за то, что его контракт выполняется верно.
Иначе вы просто вводите в заблуждение, а это требует дополнительное время на изучение вашего кода и силы на то, чтобы держать в голове, что вот тот метод ведет себя не так, как написано.

Такие ситуации вызывают только раздражение по отношению к автору кода.

Помните, что чем шире модификатор доступа к вашему коду (чем он доуступнее для использования из других частей) - тем большее влияние оказывает наименование на его применение.

## Полезные ссылки

- [Google Style Guide](https://google.github.io/styleguide/javaguide.html)
- [JavaDoc](https://www.oracle.com/technical-resources/articles/java/javadoc-tool.html)
- [A Guide to Formatting Code Snippets in Javadoc](https://reflectoring.io/howto-format-code-snippets-in-javadoc/)
- [Javadoc Wiki](https://ru.wikipedia.org/wiki/Javadoc)
