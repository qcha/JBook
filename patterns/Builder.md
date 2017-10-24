## Введение.
Пусть у нас есть некоторый класс. У класса - много параметров.

Пусть у нас для примера есть некая сущность телефона - Telephone, у него есть серийный номер, имя, марка и т.д

Какие возможные варианты есть для работы с таким классом в Java?

### Telescoping constructor pattern
С данным подходом мы пишем какое-то количество конструкторов, там заполняем обязательные и необязательные параметры.
```java
public class TelephoneTelescopingPattern {
    private int serialnumber;
    private String name;
    private String screenName;
    private String mark;

    public TelephoneTelescopingPattern(int serialnumber, String name, String screenName, String mark) {
        this.serialnumber = serialnumber;
        this.name = name;
        this.screenName = screenName;
        this.mark = mark;
    }
}
```

При создании класса мы вынуждены перечислить все в выбранном конструкторе. Если полей действительно много, то это - потенциальное место для ошибки.
Т.е этот подход хорош тогда, когда класс не имеет огромное количество параметров, так как иначе такой код сложно читать и использовать такой класс. Надо высчитывать позицию параметра, держать в уме что означает каждое число/строка/объект переданный и т.д. Если мы передаем много строк - то если мы перепутаем строки - наш код скомпилируется, будет ошибка, которую тяжело будет потом отловить.

Поэтому при работе с классами имеющими большое количество параметров лучше не использовать его.

### Java Beans pattern
Тут мы создаем объект, вызывая конструктор без параметров, а после уже устанавливаем значения set-рами.
Такой код проще читать, однако из-за разделения вызовов мы имеем объект, который некоторое время находится в неустойчивом состоянии(пока там эти все set-ры выполняются объект находится в полуподвешенном состоянии).
Также такой объект не может быть неизменным.

```java
public class TelephoneJavaBeansPattern {
    private int serialnumber;
    private String name;
    private String screenName;
    private String mark;

    public void setSerialnumber(int serialnumber) {
        this.serialnumber = serialnumber;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setScreenName(String screenName) {
        this.screenName = screenName;
    }

    public void setMark(String mark) {
        this.mark = mark;
    }
}
```

### Builder Pattern
Как же соединить два эти подхода, но так, чтобы мы обходили недостатки каждого, а использовали бы плюсы?
Суть заключается в том, что мы создаем внутренний класс, часто называемый Builder, через которого выставляем параметры, а после делаем `build()`. Т.е пока мы не сделали build - мы работаем с промежуточным объектом.

Благодаря этому подходу мы можем конструировать наш объект в нужном порядке, более тонкий контроль над процессом конструирования, изолирует код, реализующий конструирование и представление.

Можно по разному сделать Builder и поэтому внизу - два примера.

```java
public class TelephoneBuilderPattern {
    private int serialnumber;
    private String name;
    private String screenName;
    private String mark;


    private TelephoneBuilderPattern() {

    }

    public class Builder {

        private Builder() {
        }

        public Builder setName(String name) {
            TelephoneBuilderPattern.this.name = name;

            return this;
        }

        public Builder setScreenName(String screenName) {
            TelephoneBuilderPattern.this.screenName = screenName;

            return this;
        }

        public Builder setMark(String mark) {
            TelephoneBuilderPattern.this.mark = mark;
            return this;

        }

        public Builder setSerialnumber(int serialnumber) {
            TelephoneBuilderPattern.this.serialnumber = serialnumber;

            return this;
        }

        public TelephoneBuilderPattern build() {
            return TelephoneBuilderPattern.this;
        }
    }

    public static Builder getBuilder() {
        return new TelephoneBuilderPattern().new Builder();
    }
}
```
Либо вот так:
```java
public class TelephoneBuilderPattern2 {
    private final int serialnumber;
    private final String name;
    private final String screenName;
    private final String mark;

    private TelephoneBuilderPattern2(int serialnumber, String name, String screenName, String mark) {
        this.serialnumber = serialnumber;
        this.name = name;
        this.screenName = screenName;
        this.mark = mark;
    }

    public static class Builder {

        private int serialnumber;
        private String name;
        private String screenName;
        private String mark;

        private Builder() {
        }

        public Builder setName(String name) {
            this.name = name;

            return this;
        }

        public Builder setScreenName(String screenName) {
            this.screenName = screenName;

            return this;
        }

        public Builder setMark(String mark) {
            this.mark = mark;
            return this;

        }

        public Builder setSerialnumber(int serialnumber) {
            this.serialnumber = serialnumber;

            return this;
        }

        public TelephoneBuilderPattern2 build() {
            return new TelephoneBuilderPattern2(serialnumber, name, screenName, mark);
        }
    }

    public static Builder getBuilder() {
        return new Builder();
    }
}
```

Читать и работать с таким кодом гораздо проще, чем при использовании Telescoping, а также безопаснее, чем при использовании Java Beans.

Примеры использования каждого паттерна:
```java
public static void main(String[] args) {
    //example of telescoping pattern
    TelephoneTelescopingPattern telephone1 = new TelephoneTelescopingPattern(1, "Sony", "TFT", "X");

    //example of java beans pattern
    TelephoneJavaBeansPattern telephone2 = new TelephoneJavaBeansPattern();
    telephone2.setSerialnumber(2);
    telephone2.setName("Sony");
    telephone2.setScreenName("TFT");
    telephone2.setMark("X");

    //example of builder
    TelephoneBuilderPattern telephone3 = TelephoneBuilderPattern.getBuilder().setName("Sony")
                                                                             .setSerialnumber(3)
                                                                             .setMark("X")
                                                                             .setScreenName("TFT")
                                                                             .build();

    //example of builder2
    TelephoneBuilderPattern2 telephone4 = TelephoneBuilderPattern2.getBuilder().setMark("X")
                                                                               .setName("Sony")
                                                                               .setScreenName("TFT")
                                                                               .setSerialnumber(4)
                                                                               .build();
}
```
