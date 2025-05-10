# Паттерн Iterator (Итератор)

## Введение

Паттерн `Iterator` - это поведенческий паттерн проектирования, который предоставляет способ последовательного доступа ко всем элементам составного объекта, не раскрывая его внутреннего представления. Итератор позволяет обходить коллекцию элементов, не зная деталей внутреннего устройства этой коллекции.

Основная идея паттерна заключается в том, чтобы вынести поведение обхода коллекции из самой коллекции в отдельный класс. Это упрощает интерфейс коллекции и позволяет иметь несколько способов обхода одной и той же коллекции.

## Структура паттерна

* **Iterator** (Итератор) - определяет интерфейс для доступа и обхода элементов.
* **ConcreteIterator** (Конкретный итератор) - реализует интерфейс итератора. Отслеживает текущую позицию при обходе коллекции.
* **Aggregate** (Агрегат) - определяет интерфейс для создания объекта-итератора.
* **ConcreteAggregate** (Конкретный агрегат) - реализует интерфейс создания итератора, возвращая экземпляр конкретного итератора.

## Реализация

Рассмотрим пример реализации паттерна Iterator на Java:

```java
import java.util.ArrayList;
import java.util.List;

// Интерфейс итератора
interface Iterator<T> {
    boolean hasNext();
    T next();
}

// Интерфейс коллекции
interface Collection<T> {
    Iterator<T> createIterator();
}

// Конкретная коллекция
class BookCollection implements Collection<String> {
    private List<String> books;

    public BookCollection() {
        books = new ArrayList<>();
    }

    public void addBook(String book) {
        books.add(book);
    }

    public List<String> getBooks() {
        return books;
    }

    @Override
    public Iterator<String> createIterator() {
        return new BookIterator(this);
    }
}

// Конкретный итератор
class BookIterator implements Iterator<String> {
    private BookCollection bookCollection;
    private int position;

    public BookIterator(BookCollection bookCollection) {
        this.bookCollection = bookCollection;
        this.position = 0;
    }

    @Override
    public boolean hasNext() {
        return position < bookCollection.getBooks().size();
    }

    @Override
    public String next() {
        if (!hasNext()) {
            return null;
        }
        String book = bookCollection.getBooks().get(position);
        position++;
        return book;
    }
}

// Пример использования
public class IteratorDemo {
    public static void main(String[] args) {
        BookCollection bookCollection = new BookCollection();
        bookCollection.addBook("Design Patterns");
        bookCollection.addBook("Clean Code");
        bookCollection.addBook("Effective Java");
        bookCollection.addBook("Refactoring");

        Iterator<String> iterator = bookCollection.createIterator();
        while (iterator.hasNext()) {
            String book = iterator.next();
            System.out.println("Book: " + book);
        }
    }
}
```

## Преимущества

1. **Принцип единственной обязанности** - выделение алгоритма обхода в отдельный класс упрощает код коллекции.
2. **Поддержка различных способов обхода** - можно создавать разные итераторы для одной коллекции.
3. **Параллельный обход** - можно иметь несколько итераторов, обходящих одну коллекцию одновременно.
4. **Отложенные вычисления** - итератор вычисляет следующий элемент только при необходимости.

## Недостатки

1. **Избыточность** - применение паттерна может быть избыточным, если вы работаете с простой коллекцией.
2. **Эффективность** - использование итераторов может быть менее эффективным, чем прямой доступ к элементам коллекции.
3. **Сложность** - добавление дополнительных классов увеличивает сложность кода.

## Применение

Паттерн Iterator следует использовать, когда:

1. Вам нужно обеспечить доступ к элементам составного объекта, не раскрывая его внутреннего представления.
2. Вам нужно поддерживать несколько способов обхода составного объекта.
3. Вам нужно предоставить единый интерфейс для обхода различных структур данных.
4. Вам нужно отделить алгоритм обхода от структуры данных.

## Примеры из реальной жизни

1. **Java Collections Framework** - все коллекции в Java реализуют интерфейс `Iterable`, который возвращает объект `Iterator`.
2. **Базы данных** - курсоры в базах данных работают как итераторы, позволяя обходить результаты запроса.
3. **Файловые системы** - итераторы используются для обхода файлов и директорий.
4. **Веб-браузеры** - история посещений в браузере может быть реализована с использованием итератора.

## Связь с другими паттернами

* **Composite** часто используется вместе с **Iterator** для обхода древовидных структур.
* **Factory Method** может использоваться для создания итераторов.
* **Memento** может использоваться с **Iterator** для сохранения состояния итерации.
* **Visitor** может использоваться вместе с **Iterator** для применения операции ко всем элементам коллекции.

## Заключение

Паттерн Iterator предоставляет способ последовательного доступа ко всем элементам составного объекта, не раскрывая его внутреннего представления. Это позволяет отделить алгоритм обхода от структуры данных, что делает код более модульным и поддерживаемым.

## Полезные ссылки

1. [Refactoring.guru - Паттерн Итератор](https://refactoring.guru/ru/design-patterns/iterator)
2. [Head First Design Patterns](https://www.oreilly.com/library/view/head-first-design/0596007124/)
3. [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612)