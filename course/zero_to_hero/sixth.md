# Java Базовый Курс

## Занятие 6

Шестое занятие в рамках `Java` курса.

В [прошлый раз](./fifth.md) мы затронули следующие темы:

* Коллекции в `Java`
* Реализации `java.util.List`
* Реализации `java.util.Map`

В рамках этого занятия мы обсудим такую тему как // TODO .

## Введение

В начале проведем некоторую проверку знаний по прошлому материалу.

---

**Вопрос**:

Разработчик написал класс, который будет использоваться в качестве ключа `java.util.HashMap`.
Класс выглядит следующим образом:

```java
class Key {
    private String value;

    public Key(String value) {
        this.value = value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    public String getValue() {
        return value;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;

        Key other = (Key) obj;

        return value.equals(other.value);
    }

    @Override
    public int hashCode() {
        return 32;
    }

    @Override
    public String toString() {
        return value;
    }
}
```

Видите ли вы недочеты в подобном классе?

**Ответ**:

Метод `hashCode`, всегда возвращающий одно и то же значение, это гарантия того, что все добавляемые элементы будут попадать в одну и ту же корзину, а значит `java.util.HashMap` будет деградировать в производительности.

При этом в худшем случае это будет деградация до `O(N)`!

В целом, даже если этот класс и не был бы ключом так делать нельзя, потому что `equals` и `hashCode` должны переопределяться вместе и рассчитываться по одним полям класса.

---

**Вопрос**:

Как происходит удаление элементов из `java.util.ArrayList`? Как меняется в этом случае размер `java.util.ArrayList`? А `java.util.LinkedList`?

**Ответ**:

При удалении произвольного элемента из списка, все элементы находящиеся 'правее' смещаются на одну ячейку влево и реальный размер массива (его емкость, `capacity`) не изменяется никак. Для изменения надо явно вызвать метод `trimToSize`.

---

**Вопрос**:

Напишите код, который удалит все повторяющиеся элементы из `java.util.ArrayList`.

**Ответ**:

Вариантов выполнить задание масса, можно, например, добавить все элементы в `java.util.HashSet`, убрав дубли, а после в очищенную коллекцию добавить все оставшиеся значения из множества.

```java
        ArrayList<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(3);
        list.add(2);

        Set<Integer> set = new HashSet<>(list);
        list.clear();
        list.addAll(set);
```

---