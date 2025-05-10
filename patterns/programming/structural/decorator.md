# Паттерн Decorator (Декоратор)

## Введение

Паттерн `Decorator` - это структурный паттерн проектирования, который позволяет динамически добавлять объектам новую функциональность, оборачивая их в полезные «обёртки». Декоратор предоставляет гибкую альтернативу наследованию для расширения функциональности.

Основная идея паттерна заключается в том, чтобы обернуть исходный объект в декоратор, который реализует тот же интерфейс и делегирует работу обёрнутому объекту, но может добавлять своё поведение до или после вызова методов оригинала.

## Структура паттерна

* **Component** (Компонент) - определяет интерфейс для объектов, на которые могут быть динамически возложены дополнительные обязанности.
* **ConcreteComponent** (Конкретный компонент) - определяет объект, на который возлагаются дополнительные обязанности.
* **Decorator** (Декоратор) - поддерживает ссылку на компонент и определяет интерфейс, соответствующий интерфейсу компонента.
* **ConcreteDecorator** (Конкретный декоратор) - добавляет обязанности к компоненту.

## Реализация

Рассмотрим пример реализации паттерна Decorator на Java:

```java
// Компонент
interface Coffee {
    String getDescription();
    double getCost();
}

// Конкретный компонент
class SimpleCoffee implements Coffee {
    @Override
    public String getDescription() {
        return "Простой кофе";
    }

    @Override
    public double getCost() {
        return 1.0;
    }
}

// Базовый декоратор
abstract class CoffeeDecorator implements Coffee {
    protected final Coffee decoratedCoffee;

    public CoffeeDecorator(Coffee coffee) {
        this.decoratedCoffee = coffee;
    }

    @Override
    public String getDescription() {
        return decoratedCoffee.getDescription();
    }

    @Override
    public double getCost() {
        return decoratedCoffee.getCost();
    }
}

// Конкретные декораторы
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return decoratedCoffee.getDescription() + ", с молоком";
    }

    @Override
    public double getCost() {
        return decoratedCoffee.getCost() + 0.5;
    }
}

class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return decoratedCoffee.getDescription() + ", с сахаром";
    }

    @Override
    public double getCost() {
        return decoratedCoffee.getCost() + 0.2;
    }
}

class WhipDecorator extends CoffeeDecorator {
    public WhipDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public String getDescription() {
        return decoratedCoffee.getDescription() + ", со взбитыми сливками";
    }

    @Override
    public double getCost() {
        return decoratedCoffee.getCost() + 0.7;
    }
}

// Пример использования
public class DecoratorDemo {
    public static void main(String[] args) {
        // Простой кофе
        Coffee coffee = new SimpleCoffee();
        System.out.println(coffee.getDescription() + ": $" + coffee.getCost());

        // Кофе с молоком
        Coffee milkCoffee = new MilkDecorator(coffee);
        System.out.println(milkCoffee.getDescription() + ": $" + milkCoffee.getCost());

        // Кофе с молоком и сахаром
        Coffee sweetMilkCoffee = new SugarDecorator(milkCoffee);
        System.out.println(sweetMilkCoffee.getDescription() + ": $" + sweetMilkCoffee.getCost());

        // Кофе со всеми добавками
        Coffee ultimateCoffee = new WhipDecorator(new SugarDecorator(new MilkDecorator(new SimpleCoffee())));
        System.out.println(ultimateCoffee.getDescription() + ": $" + ultimateCoffee.getCost());
    }
}
```

## Преимущества

1. **Большая гибкость, чем наследование** - можно добавлять и удалять обязанности во время выполнения программы.
2. **Принцип единственной обязанности** - можно разделить монолитный класс, реализующий множество возможных вариантов поведения, на несколько классов.
3. **Комбинирование поведений** - можно комбинировать несколько дополнительных поведений, оборачивая объект в несколько декораторов.
4. **Соблюдение принципа открытости/закрытости** - можно добавлять новые виды декораторов, не изменяя существующий код.

## Недостатки

1. **Трудность удаления конкретного обёртки** - если нужно удалить конкретную обёртку из цепочки, это может быть сложно.
2. **Сложность реализации декоратора** - декоратор должен быть прозрачным для клиента, что может быть сложно обеспечить.
3. **Проблемы с идентичностью объекта** - декорированный компонент не идентичен исходному компоненту.
4. **Большое количество маленьких объектов** - использование декораторов может привести к системам, где задействовано много мелких объектов, похожих друг на друга.

## Применение

Паттерн Decorator следует использовать, когда:

1. Вам нужно добавлять обязанности объектам динамически и прозрачно, не затрагивая другие объекты.
2. Вам нужно добавлять обязанности, которые могут быть сняты с объекта.
3. Расширение с помощью наследования нецелесообразно или невозможно.
4. Вам нужно комбинировать несколько дополнительных поведений.

## Примеры из реальной жизни

1. **Потоки ввода-вывода в Java** - классы `java.io.InputStream`, `java.io.OutputStream`, `java.io.Reader` и `java.io.Writer` имеют декораторы, которые добавляют функциональность, такую как буферизация, фильтрация и т.д.
2. **Графические интерфейсы** - добавление рамок, полос прокрутки и других элементов к компонентам пользовательского интерфейса.
3. **Веб-сервисы** - добавление функциональности, такой как логирование, безопасность, кэширование, к базовым веб-сервисам.
4. **Кофе в кофейне** - базовый кофе может быть дополнен различными ингредиентами, как в нашем примере.

## Связь с другими паттернами

* **Adapter** изменяет интерфейс объекта, а **Decorator** изменяет его поведение, не меняя интерфейс.
* **Composite** и **Decorator** имеют похожие структуры, но с разными целями: **Composite** для создания древовидных структур, **Decorator** для добавления функциональности.
* **Strategy** позволяет изменять внутренности объекта, а **Decorator** изменяет его внешнюю оболочку.
* **Chain of Responsibility** и **Decorator** имеют похожие структуры, но с разными целями: **Chain of Responsibility** для передачи запроса по цепочке, **Decorator** для добавления функциональности.

## Заключение

Паттерн Decorator позволяет динамически добавлять объектам новую функциональность, оборачивая их в полезные «обёртки». Это обеспечивает гибкую альтернативу наследованию для расширения функциональности и соответствует принципу открытости/закрытости.

## Полезные ссылки

1. [Refactoring.guru - Паттерн Декоратор](https://refactoring.guru/ru/design-patterns/decorator)
2. [Head First Design Patterns](https://www.oreilly.com/library/view/head-first-design/0596007124/)
3. [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612)
