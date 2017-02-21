### Google Guava
`Google Guava` - это библиотека, в которой собраны многие решения рутинных задач, которые так или иначе встречаются в работе.

С выходом `Java 8` некоторые из проблем, решаемых данной библиотекой, переехали в JDK.

Однако, Guava все еще не теряет актуальности. Кратко перечислю где что искать.

#### Joiner/Splitter
`Joiner` используется для того, чтобы заменить рутинные задачи, суть которых состоит в том, чтобы получить строку из элементов, разделенных разделителем.
Т.е:
```java
public String buildStringByGuava(List<String> stringList, String delimiter) {
    return Joiner.on(delimiter).skipNulls().join(stringList);
}
```

Напротив, `Splitter` - класс, призванный разбить подобные строки на элементы.
```java
    String testString = "Monday,Tuesday,,Thursday,Friday,,";
    System.out.println(Splitter.on(',').split(testString));
    //prints [Monday, Tuesday, , Thursday, Friday, , ]
```

#### Collections
##### Lists
Класс, имеющий большое количество статических методов для создания экземпляров `List`-а.
```java
List<Person> personList = Lists.newArrayList();
```

Partition:

Также, имеет довольно полезный метод, для разбивания списка на более мелкие списки:
```java
List<Person> personList = Lists.newArrayList(person1, person2, person3, person4);
List<List<Person>> subList = Lists.partition(personList, 2);
//Gets [[person1, person2], [person3, person4]]
```

##### Sets
Как и `Lists` - нужен для создания экзмепляров `Set`-ов, а также имеет несколько полезных методов для работы.

Difference:
```java
Set<String> s1 = Sets.newHashSet("1", "2", "3");
Set<String> s2 = Sets.newHashSet("2", "3", "4");
Sets.difference(s1, s2);
```

Intersection:
```java
Set<String> s1 = Sets.newHashSet("1", "2", "3");
Set<String> s2 = Sets.newHashSet("3", "2", "4");
Sets.SetView<String> sv = Sets.intersection(s1, s2);
```

Union:
```java
Set<String> s1 = Sets.newHashSet("1", "2", "3");
Set<String> s2 = Sets.newHashSet("3", "2", "4");
Sets.SetView<String> sv = Sets.union(s1, s2);
```

##### Maps
Также содержит статические методы для работы с `Map`.
//todo

//todo Table

//todo Range

//todo Immutable collections
