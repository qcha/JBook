## JSON Serialization

JSON неплохой выбор из-за отличного синтаксиса, большого количества библиотек, маленького размера.
Для примера, возьмем библиотеку `GSON`.

####Почему GSON?
* Судя по тестам - отлично работает с большим объемом данных.
* Работаем без аннотаций и метаданных.По сути мы как бы просто говорим - хочу этот объект в JSON.
* Порядок записи контролируем.
* Работает с `null`.
* Довольно просто использовать.

####Как использовать?
Сначала создаем `GSON Builder`:
```java
Gson gsonBuilder = new GsonBuilder().setPrettyPrinting().create();
```
Когда мы создаем наш Gson парсер - можем также вызвать еще методы для дополнительных возможностей, например:
* excludeFieldsWithModifiers()
* setDateFormat()

и так далее.

Отличный выбор, простой и мощный, без метаданных и аннотаций.
Все собираем методами, отсюда - все прозрачно и ясно.

```java
public static void main(String[] args) {
    Gson parser = new Gson();
    //Simple DataObject
    DataObject dataObject = new DataObject();
    dataObject.setName("aarexer");
    dataObject.setCount(14);
    dataObject.addHobby("Football");
    dataObject.addHobby("Ping-Pong");
    dataObject.addHobby("Bicycle");

    //our json string
    String jsonString = parser.toJson(dataObject);
    System.out.println(jsonString);

    DataObject dataObjectFromJson = parser.fromJson(jsonString, DataObject.class);
    //we override toString!
    System.out.println(dataObjectFromJson);
}
```

// todo https://www.json.org/json-ru.html

## Полезные ссылки

[Введение в JSON](https://www.json.org/json-ru.html)