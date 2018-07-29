## java.lang.Object#clone
### Введение
Название метода подскажет его назначение.
Данный метод задумывался разработчиками как простой и понятный способ создать копию объекта, его клон - отсюда и название.

Объявление метода выглядит так:
```java
protected native Object clone() throws CloneNotSupportedException;
```

Клонировать можно только те объекты, которые реализуют интерфейс `java.lang.Cloneable`.
Данный интерфейс является интерфейсом-марекром, как и `Serializable`.

Если объект не реализует интерфейс-маркер `java.lang.Cloneable`, то выбросится исключение `CloneNotSupportedException`.

По умолчанию `clone` определяет *поверхностное* копирование - копируются значения всех полей и **ссылок**.

Рассмотрим пример:
```java
public class CloneTest implements Cloneable {
    private final int i;

    public CloneTest(int i) {
        this.i = i;
    }

    public int getI() {
        return i;
    }

    public static void main(String[] args) throws CloneNotSupportedException {
        CloneTest test = new CloneTest(2);
        CloneTest cloneTest = (CloneTest) test.clone();
        System.out.println("Original : " + test + ", i = " + test.getI());
        System.out.println("Clone : " + cloneTest + ", i = " + cloneTest.getI());
    }
}
```

Чтобы не делать постоянное привделение к нужному типу можно переопределить наш метод и явно указать тип возвращаемого объекта.
Это считается хорошим тоном и застрахует нас от некоторых ошибок.

### Подводные камни
Рассмотрим чуть более сложный пример, содержащий ссылочный тип, и переопределим метод так, как мы говорили выше - укажем явно возвращаемый объект:
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

На первый взгляд может показаться, что все очень удобно и практично, однако использование `clone` также часто не рекомендуется, так как несет за собой большое количество узких мест и подводных камней.

Во-первых, вспомним, что по умолчанию копируются значения всех полей и **ссылок**, а значит при клонировании будет скопирована именно ссылка.
Т.е и клон, и первоначальный объект будут ссылаться на один и тот же объект `Hobby`.

Что подводит нас к подводному камню, можно скзаать айсбергу, потопившему титаник: если первоначальный объект изменит `Hobby`, то эти изменения окажутся и у клона!
В свою очередь, это работает и в обратную сторону - если клон вдруг изменит `Hobby`, то и у первоначального объекта оно изменится.


Чувствуете пропасть под ногами?

### Что делать?
Избежать подобного поведения можно вот как: воспользоваться конструктором копирования.
#### Конструктор копирования и clone
Т.е сначала вызвать `clone` супер класса, а после уже явно вручную работать с такими опасными полями.

Рассмотрим как это будет выглядеть в нашем случае:
```java
@Override
protected PersonToCloneBetter clone() throws CloneNotSupportedException {
    PersonToCloneBetter pClone = (PersonToCloneBetter) super.clone();
    pClone.setHobby(new Hobby(hobby.getName()));

    return pClone;
}
```

Единственное но - такой способ не сработает, если поле `hobby` объявлено как `final`, т.е неизменяемое.
В таком случае мы просто не сможем вызвать `setHobby`.

#### Конструктор копирования или статический метод генерации новых объектов
Лучшим вариантом, на мой взгляд, является определение конструктора копирования или реализация статического метода генерации новых объектов.
Плюсы:
* Проще реализуется
* Не работаем с исключениями клонирования, как например, `CloneNotSupportedException`
* Поддерживает работу с `final`-полями
* Код получается более явным - мы просто создаем новый объект с помощью конструктора или специального метода.

Этот вариант является гораздо более предпочтительным, на мой взгляд.

Пример:
```java
public static PersonToClone newInstance(PersonToClone personToClone) {
    return new PersonToClone(
      personToClone.getName(),
      personToClone.getAge(),
      new Hobby(personToClone.hobby.getName())
    );
}
```
### Заключение
* Все методы, реализующие `Cloneable` должны переопределять `clone()` и делать его открытым.
* В переопределенном методе необходимо сначала вызвать `super.clone()`, после чего начать работать с полями, значения которых могут изменяться, т.е надо заменять все ссылки на объекты соответствующими копиями.
* Лучше всего - рассмотреть альтернативы использования метода `clone` в виде создания конструктора копирования.
