# Итератор с предикатом

Задача с собеседования в Яндекс.

## Условие

Напишите реализацию итератора, принимающего на вход другой итератор и предикат. Реализованный вами итератор должен отдавать только те значения, которые удовлетворяют предикату.

### Пример

Для примера возьмем итератор на последовательность из 6 чисел по возрастанию, предикат поставим на четность:

```java
PredicatedIterator<Integer> it = new PredicatedIterator<>(
    Stream.of(1, 2, 3, 4, 5, 6).iterator(),
    (e) -> e % 2 == 0
);
```

Проитерировавшсиь по нашему итератору должны получить:

```java
2, 4, 6
```

## Решение

Основной момент состоит в том, что надо понять место, где будет идти проверка с предикатом. Я выделил это в `hasNext`, в таком случае необходимо обработать ситуацию, когда пользователь вообще ни разу не вызовет `hasNext`, а сразу будет делать вызов `next`. Для этого вводится поле в классе `current`:

```java
class PredicatedIterator<T> implements Iterator<T> {
    private final Predicate<T> predicate;
    private final Iterator<T> iterator;
    private T current;

    public PredicatedIterator(Iterator<T> iterator, Predicate<T> predicate) {
        this.iterator = iterator;
        this.predicate = predicate;
    }


    @Override
    public boolean hasNext() {
        if (current != null) {
            return true;
        }

        while (iterator.hasNext()) {
            T elem = iterator.next();

            if (predicate.test(elem)) {
                current = elem;

                return true;
            }
        }

        return false;
    }

    @Override
    public T next() {
        if (current == null && !hasNext()) {
            throw new NoSuchElementException();
        }

        T next = current;
        current = null;

        return next;
    }
}
```
