### Splitter
#### Зачем нужно?
Бывают случаи, когда у нас есть строка(например, полученная из файла), содержащая значения, разделенные
некоторым разделителем.

Один из вариантов решения - это воспользоваться методом `split` у `String`.
```java
public class Example {
    public static void main(String[] args) {
        String testString = "Monday,Tuesday,,Thursday,Friday,,";
        String[] parts = testString.split(",");

        System.out.println(Arrays.toString(parts));
        //print [Monday, Tuesday, , Thursday, Friday]
    }
}
```

Видно, использование `String.split` в большинстве случаев не подойдет  - последние значения, пусть и пустые, были просто опущены.
Т.е ожидаемый размер массива: 7
Ожидаемые значения: `[Monday, Tuesday, , Thursday, Friday, , ]`.

Вместо этого `split` просто выкинул последние два значения ничего не сообщив нам.

В случае, когда такое поведение вам подходит - использование метода оправдано, иначе - необходимо написание своего метода `split`.

В таких случаях можно воспользоваться классом `Splitter`.

#### Пример использования.
Решим данную задачу с помощью `Splitter`.
```java
public class Example {
  public static void main(String[] args) {
      String testString = "Monday,Tuesday,,Thursday,Friday,,";
      System.out.println(Splitter.on(',').split(testString));
      //print [Monday, Tuesday, , Thursday, Friday, , ]
  }
}
```
Разделителем могут являться как символ(`char`) или строка так и регулярное выржаение, или `CharMatcher` - еще один класс из библиотеки `Guava`.

Для работы с `Map` используется `withKeyValueSeparator`.
