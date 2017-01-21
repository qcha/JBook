### Введение
Иногда мы работаем с неизменяемыми объектами, тогда, при работе с таким объектом часто бывает полезно создать его копию и работать с ней. Также часто бывает создать копию какого-либо объекта и работать с ней, при этом не изменяя первоначальный объект.

Одним из способов клонировать объект является переопределение метода `clone()` и реализация интерфейса `Cloneable`.

### Подробнее.

У объекта `Object` есть метод `clone`, как он объявлен?
```java
protected Object clone() throws CloneNotSupportedException
```

По умолчанию этот метод определяет 'поверхностное' копирование, копируются значения всех полей и ссылок. Так как он `protected`, то у объектов, не переопределивших его, этот метод не получится вызвать.
Заметим также, что с версии 1.5 хорошим тоном при переопределении этого метода является явное указание возвращаемого объекта, а не `Object`.

В java есть интерфейс-маркер `Cloneable`, который показывает, может ли объект быть клонирован. Классы, открывающие метод `clone` обязаны реализовывать этот интерфейс.
Сам `Object` - не реализует этот интерфейс.
Почему? Потому, что не всем классам это надо. Не все классы обязательно должны уметь клонироваться. А реализовав интерфейс в `Object` мы бы потащили эту возможность везде.

Для примера возьмем некий класс у которого будет три поля:
```java
public class PersonToClone implements Cloneable {
    private String name;
    private int age;
    private Hobby hobby;

    public PersonToClone(String name, int age, Hobby hobby) {
        this.name = name;
        this.age = age;
        this.hobby = hobby;
    }

    @Override
    public String toString() {
        return "PersonToClone{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", hobby=" + hobby +
                '}';
    }

    @Override
    protected PersonToClone clone() throws CloneNotSupportedException {
        return (PersonToClone) super.clone();
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public void setHobby(Hobby hobby) {
        this.hobby = hobby;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    public Hobby getHobby() {
        return hobby;
    }
}
```

Однако, как мы помним, если поля класса содержат ссылки на изменяемые объекты, то при клонировании будет скопирована именно ссылка, а значит, изменив такое поле в клоне объекта - мы измени его и в оригинальном!

А это может иметь печальные последствия.

Если мы создадим объект класса `PersonToClone`, склонируем его и изменим у копии хобби, то хобби измениться и у
родительского объекта!

### Как такого избежать?
Избежать можно вот как: воспользоваться конструктором копирования.
Т.е сначала вызвать `clone` супер класса, а после уже работать с такими опасными полями.
Реализуем такой метод:

```java
@Override
protected PersonToCloneBetter clone() throws CloneNotSupportedException {
    PersonToCloneBetter pClone = (PersonToCloneBetter) super.clone();
    pClone.setHobby(new Hobby(hobby.getName()));

    return pClone;
}
```

Отметим, что клонирование таким способом **НЕ** работает, если поле помечено как `final`.

### Что в итоге?
Все методы, реализующие `Cloneable` должны переопределять `clone()` и делать его открытым. В переопределенном методе необходимо сначала вызвать `super.clone()`, после чего начать работать с полями, значения которых могут изменяться, т.е надо заменять все ссылки на объекты соответствующими копиями.

Лучшим вариантом является определение конструктора-копий или статический метод генерации.
Плюсы таких вариантов заключается в том, что это проще, клиент получает объект определенного типа, не отлавливает исключения, можно работать с `final` полями.

Этот вариант является гораздо более предпочтительным.

```java
public static PersonToClone newInstance(PersonToClone personToClone) {
    return new PersonToClone(personToClone.getName(), personToClone.getAge(), new Hobby(personToClone.hobby.getName()));
}
```
