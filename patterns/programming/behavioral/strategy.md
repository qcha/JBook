# Паттерн Strategy (Стратегия)

## Введение

Паттерн `Strategy` - это поведенческий паттерн проектирования, который определяет семейство алгоритмов, инкапсулирует каждый из них и делает их взаимозаменяемыми. Стратегия позволяет изменять алгоритмы независимо от клиентов, которые их используют.

Основная идея паттерна заключается в том, чтобы выделить изменяющееся поведение (алгоритм) в отдельную иерархию классов и сделать его независимым от контекста, в котором это поведение используется.

## Структура паттерна

* **Context** (Контекст) - класс, который использует стратегию. Содержит ссылку на объект стратегии и может предоставлять интерфейс, через который стратегия может получать доступ к данным контекста.
* **Strategy** (Стратегия) - общий интерфейс для всех поддерживаемых алгоритмов.
* **ConcreteStrategy** (Конкретная стратегия) - конкретная реализация алгоритма, соответствующая интерфейсу стратегии.

## Реализация

Рассмотрим пример реализации паттерна Strategy на Java:

```java
// Интерфейс стратегии
interface PaymentStrategy {
    void pay(int amount);
}

// Конкретные стратегии
class CreditCardStrategy implements PaymentStrategy {
    private String name;
    private String cardNumber;
    private String cvv;
    private String dateOfExpiry;

    public CreditCardStrategy(String name, String cardNumber, String cvv, String dateOfExpiry) {
        this.name = name;
        this.cardNumber = cardNumber;
        this.cvv = cvv;
        this.dateOfExpiry = dateOfExpiry;
    }

    @Override
    public void pay(int amount) {
        System.out.println(amount + " оплачено с помощью кредитной карты");
    }
}

class PayPalStrategy implements PaymentStrategy {
    private String emailId;
    private String password;

    public PayPalStrategy(String emailId, String password) {
        this.emailId = emailId;
        this.password = password;
    }

    @Override
    public void pay(int amount) {
        System.out.println(amount + " оплачено через PayPal");
    }
}

// Контекст
class ShoppingCart {
    private PaymentStrategy paymentStrategy;

    public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void checkout(int amount) {
        paymentStrategy.pay(amount);
    }
}

// Пример использования
public class StrategyDemo {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();

        // Оплата кредитной картой
        cart.setPaymentStrategy(new CreditCardStrategy("John Doe", "1234567890123456", "123", "12/25"));
        cart.checkout(100);

        // Оплата через PayPal
        cart.setPaymentStrategy(new PayPalStrategy("johndoe@example.com", "password"));
        cart.checkout(200);
    }
}
```

## Преимущества

1. **Избавление от условных операторов** - вместо множества условных операторов, выбирающих поведение, мы используем полиморфизм.
2. **Изоляция реализации алгоритмов** - детали реализации алгоритмов скрыты от клиентского кода.
3. **Замена наследования композицией** - вместо наследования для изменения поведения используется композиция.
4. **Возможность замены алгоритмов на лету** - можно менять стратегию во время выполнения программы.

## Недостатки

1. **Увеличение числа объектов** - для каждого алгоритма создается отдельный класс.
2. **Клиент должен знать о стратегиях** - клиент должен понимать, чем отличаются стратегии, чтобы выбрать подходящую.
3. **Накладные расходы на коммуникацию** - стратегия и контекст могут обмениваться данными, которые не всегда нужны всем стратегиям.

## Применение

Паттерн Strategy следует использовать, когда:

1. Вам нужно использовать разные варианты алгоритма внутри одного объекта.
2. У вас есть множество похожих классов, отличающихся только поведением.
3. Вам нужно скрыть от клиента сложные детали реализации алгоритма.
4. В классе определено много поведений, что приводит к созданию сложных условных операторов.

## Примеры из реальной жизни

1. **Алгоритмы сортировки** - можно реализовать разные алгоритмы сортировки (пузырьком, быстрая, слиянием) как стратегии.
2. **Стратегии сжатия** - разные алгоритмы сжатия данных (ZIP, GZIP, RAR).
3. **Стратегии валидации** - разные способы проверки ввода пользователя.
4. **Стратегии маршрутизации** - разные алгоритмы поиска маршрута в навигационных системах.

## Связь с другими паттернами

* **Factory Method** может использоваться для создания объектов стратегии.
* **Flyweight** может использоваться для совместного использования стратегий.
* **State** похож на Strategy, но имеет другое назначение - State позволяет объекту изменять свое поведение при изменении внутреннего состояния.

## Заключение

Паттерн Strategy позволяет определить семейство алгоритмов, инкапсулировать каждый из них и сделать их взаимозаменяемыми. Это позволяет изменять алгоритмы независимо от клиентов, которые их используют, что делает код более гибким и поддерживаемым.

## Полезные ссылки

1. [Refactoring.guru - Паттерн Стратегия](https://refactoring.guru/ru/design-patterns/strategy)
2. [Head First Design Patterns](https://www.oreilly.com/library/view/head-first-design/0596007124/)
3. [Design Patterns: Elements of Reusable Object-Oriented Software](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612)
