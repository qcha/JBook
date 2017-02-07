## Основные утилиты
### Joiner
Часто возникают ситуации, когда нам необходимо собрать строку из элементов, разделенных некоторым
разделителем.
Без использования посторонних стредств это можно сделать так:
```java
public String buildString(List<String> stringList, String delimiter) {
    final StringBuilder builder = new StringBuilder();
    for (String s : stringList) {
        if (s != null) {
            builder.append(s).append(delimiter);
        }
    }
    builder.setLength(builder.length() - delimiter.length());
    return builder.toString();
}
```
В данном коде нет ничего сложного, мы стандартно соединяем строки с помощью
`StringBuilder`-а и перед `return`-ом убираем последний символ разделителя.

Так как данная задача встречается довольно часто, то логично было бы вынести
подобный кусок кода в класс утилит, а затем его использовать.

Поэтому и появился класс `Joiner`:

То же самое, но через `Joiner`:
```java
public String buildStringByGuava(List<String> stringList, String delimiter) {
    return Joiner.on(delimiter).skipNulls().join(stringList);
}
```

Без игнорирования `null`:
```java
private static String buildStringByGuavaWithNull(List<String> stringList, String delimiter) {
    return Joiner.on(delimiter).useForNull("None").join(stringList);
}
```

Здесь мы можем явно указать, что хотим видеть вместо `null`.

Стоит отметить:
* Результат работы `Joiner` получается благодаря вызову метода `toString()` у объекта.
Это значит, что если мы явно не обговорим что делать с `null` значениями - будет выброшено `NPE`.

  **Решение** : использовать `useForNull` или `skipNulls`.

* `Joiner` является неизменяемым, `immutable` и потокобезопасным.
* Для работы с `Map` используется `withKeyValueSeparator`.
