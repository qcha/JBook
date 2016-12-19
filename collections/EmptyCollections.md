### Введение
Мы часто создаем в наших классах ссылки на какие-то коллекции. Т.е:
```java
public class Hello {
private List<String> names;
/*Some code and logic*/
}
```

При таком объявлении вы должны в конструкторе создать объект List-а и присвоить его ссылке. Если этого не сделать - names будет равен `null`, что чревато NPE.
Именно для такого случая в Java, в классе `Collections` есть специальные неизменяемые пустые коллекции.

#### Empty Collections
Для примера рассмотрим вот такой код:
```java
public class PersonNames {
    private List<String> personNames;

    public PersonNames() {
        personNames = Collections.emptyList();
    }

    public PersonNames(String... personNames) {
        this.personNames = new ArrayList<>();
        Collections.addAll(this.personNames, personNames);
    }

    @Override
    public String toString() {
        return personNames.toString();
    }

    public Iterator<String> iterator() {
        return personNames.iterator();
    }
}
```

Пользуясь таким способом мы можем избежать нежелательных NPE.

При этом надо понимать, что вы можете создать объект и присвоить ссылке прямо при объявлении, но данный способ дает вам гибкость в выборе реализации(в разных конструкторах можно использовать разные реализации List), позволяет, при необходимости, рассчитать `capacity` для коллекции, в зависимости от количества добавляемых элементов. Плюс вы можете вернуть всегда безбоязненно итератор на такую коллекцию.

#### Вывод
Подобный подход будет полезен, если вы хотите как-то более гибко работать с полями класса,
выбирая реализацию от того, какой конструктор вы используете.
С помощью `Collections` мы можем писать safety code, избегая лишних проверок на `null`.
