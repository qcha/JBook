## Что такое pom.xml
Основной файл настройки вашего билда - это pom.xml - POM - Project Object Model.
Все pom.xml переопределяют некий 'super pom' - некоторый набор действий по умолчанию.

При внесении в наш pom.xml каких-то настроек - мы по сути переопределяем поведение в родительском pom.
Это позволяет делать достаточно компактыне pom.xml для проектов.

### Что содержится в pom.xml?
* Описание проекта
* Зависимости
* Плагины и их конфигурацияё
* Профили
* и т.д

### Разбираем pom.xml
Корневой элемент pom.xml - это <project>, схема и версия.

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
</project>
```
Здесь можно не заострять внимание, это стандартные строки для maven, необходимые для его работы.

Следующий важный момент - в pom.xml содержится описание проекта.
#### Описание проекта
```xml
    <groupId>com.github.aarexer.java-ex</groupId>
    <artifactId>java-ex</artifactId>
    <version>1.0</version>
```
Данные поля являются обязательными, каждый проект идентифицируется парой groupId и artifactId.
Если корокто, то groupId - это наименование организации, а artifactId - название проекта.

В поле для версии мы указываем текущую версию проекта, если состояние кода не зафиксировано, то принято
 добавлять в конец версии еще '-SNAPSHOT', что ознаечает, что версия еще в разработке.

Там же может быть добавлено еще строка:
```xml
<packaging>jar</packaging> 
```
Вместо jar вы можете выбрать war или ear, в зависимости от того какой тип файла нужен нам после сборки.

Иногда можно встретить еще информацию для самих разработчкиов, некоторое описание проекта для людей:
```xml
<name>java examples</name> 
<description>Java Examples And Codes</description> 
<url>http://www.mysite.com</url>
```

Где в тегах указывается название проекта для человека, описание проекта и сайт проекта соответственно.

Также может быть еще и различная метаинформация о проекте, как например, список разработчиков, ссылки на issue tracker,
ci manager и прочее.
````xml
    <issueManager>
        <system>My issue system</system>
        <url>url</url>
    </issueManager>
    
    <ciManager>
        <system>My ci system</system>
        <url>url</url>
    </ciManager>

    <developers>
        <developer>
            <id>aarexer</id>
            <name>Aleksandr Kuchuk</name>
            <email>email</email>
            <organization>QCH</organization>
        </developer>
    </developers>
````
#### Зависимости
Как мы уже говорили выше - пара из groupId и artifactId уникально описывает проект или библиотеку.
А значит, для добавления в проект зависимостей достаточно указать данную пару вместе с версией - GAV - 
groupId, artifactId, version.

Все зависимости перечисляются в теге `<dependencies>.`
```xml
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```

Каждую зависимость мы описываем в теге `<dependency>`.
Поиск зависимостей осуществляется с помощью ресурсов типа maven-central, но об этом позже.

Также у зависимостей может быть еще добавлен тег `<scope>` - этот тег отвечает за то 
,где будет использоваться зависимость.

В приведенном выше примере видно, что библиотека с GAV junit:junit:4.12 нужна только для выполнения тестов.

##### Какие scope есть?
* compile - нужно при компиляции
* test - нужно при тестах
* runtime - необходимо в runtime, т.е при запуске
* provided - необходимо для запуска, но поставляется кем-то другим //todo 
* system - то же, что и выше, но с указанием пути до библиотеки

##### Дерево зависимостей
Для того, чтобы посмотреть дерево зависимостей проекта можно выполнить:
```
mvn dependency:tree
```

Бдует выведено псевдографическое изображение дерева зависимостей в проекте.

При конфликте или при желании можно исключить библиотеку:
```xml
<exclusions>
    <exclusion>
        <groupId></groupId>
        <artifactId></artifactId>
    </exclusion>
</exclusions>
```

Данный скоуп можно поместить для любой зависимости, чтобы явно указать, чтобы она не тащила за собой то, что нам не надо.

Например: мы используем логгер версии 3, а какая-то библиотека использует второй версии.

##### Необязательные зависимости
Мы можем пометить зависимость как 
```xml
<optional>true</optional>
``` 
В таком случае все проекты, которые зависят от нашего, такую зависимость скачивать не будут.

#### Модули
Если наш проект многомодульный, то список всех модулей будет перечислен в корневом pom.xml:
```xml
    <modules>
        <module>utils</module>
        <module>patterns</module>
        <module>algoritms</module>
        <module>code-examples</module>
        <module>solved-tasks</module>
    </modules>
```

Каждый модуль также будет содержать строки, связывающие его с корневым pom.xml и только потом
будет описание модуля:
```xml
    <parent>
        <groupId>com.github.aarexer.java-ex</groupId>
        <artifactId>java-ex</artifactId>
        <version>1.0</version>
    </parent>
    
    
    <artifactId>algoritms</artifactId>
    <version>1.0</version>
```

Заметим, что в случае для модуля нам нет необходимости явно прописывать groupId для модуля - так как понятно, что данный 
 модуль будет с тем же groupId, что и родитель.

#### Константы

Также удобно бывает вынести что-то в константы, например, используемый порт или версию Java на которой мы пишем.
```xml
<properties>
    <jetty.port>9990</jetty.port>
    <compiler.version>1.8</compiler.version>
</properties>
```
В таком случае мы сможем обращаться к нашим константам как:
```xml
<port>${jetty.port}</port>
```
