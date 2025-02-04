# Print in Order

Задача с [LeetCode](https://leetcode.com/problems/print-in-order/description/).

В том или ином варианте интерпретации иногда встречается на собеседованиях.

## Условие

Дан класс:

```java
class Foo {
    public Foo() {
        
    }

    public void first(Runnable printFirst) throws InterruptedException {
        // printFirst.run() outputs "first". Do not change or remove this line.
        printFirst.run();
    }

    public void second(Runnable printSecond) throws InterruptedException {
        // printSecond.run() outputs "second". Do not change or remove this line.
        printSecond.run();
    }

    public void third(Runnable printThird) throws InterruptedException {
        // printThird.run() outputs "third". Do not change or remove this line.
        printThird.run();
    }
}
```

Один и тот же экземпляр класса `Foo` будет передан трем разным потокам. Поток A вызовет first(), поток B вызовет second(), а поток C вызовет third().

Измените код так, чтобы гарантировать, что second() выполняется после first(), а third() выполняется после second().

## Решение

Решение этой задачи - это использование механизмов синхронизации.

### Semaphore

Синхронизатор `Semaphore` реализует шаблон синхронизации Семафор.
Чаще всего, семафоры необходимы, когда нужно ограничить доступ к некоторому общему ресурсу.

Семафоры действуют как ворота, которые открываются после того, как предыдущая операция подала сигнал о том, что можно безопасно начать следующую операцию.

В конструктор этого класса (Semaphore(int permits) или Semaphore(int permits, boolean fair)) обязательно передается количество потоков, которому семафор будет разрешать одновременно использовать заданный ресурс.

Доступ управляется с помощью счётчика: изначально значение счётчика равно int permits, когда поток заходит в заданный блок кода, то значение счётчика уменьшается на единицу, когда поток его покидает (release), то увеличивается. Если значение счётчика равно нулю, то текущий поток блокируется, пока кто-нибудь не выйдет из блока (в качестве примера из жизни с permits = 1, можно привести очередь в кабинет в поликлинике: когда пациент покидает кабинет, мигает лампа, и заходит следующий пациент).

В нашей задаче у нас есть три кабинета-метода, в которые будут заходить пациенты, но по очереди.

Мы не можем гарантировать, что вызовы потоков будут упорядочены или синхронизованы, а потому нам необходимо заблокировать каждый кабинет своим семафором.

При этом, чтобы не дать кому-то 'заскочить' в второй или третий кабинет-метод без посещения первого, семафоры для second и third по умолчанию создадим с permits = 0, т.е. по-умолчанию поток заблокируется на них. Ну а для first установим permits в 1, чтобы дать возможность зайти в секуию и напечатать "first".

Далее, после выхода из first 'разрешаем' заходить в second, т.е. делаем release на соответствующем семафоре. Аналогично с third.

```java
class Foo {
    private Semaphore first = new Semaphore(1);
    private Semaphore second = new Semaphore(0);
    private Semaphore third = new Semaphore(0);

    public Foo() {
        
    }

    public void first(Runnable printFirst) throws InterruptedException {
        first.acquire();
        // printFirst.run() outputs "first". Do not change or remove this line.
        printFirst.run();
        second.release();
    }

    public void second(Runnable printSecond) throws InterruptedException {
        second.acquire();
        // printSecond.run() outputs "second". Do not change or remove this line.
        printSecond.run();
        third.release();
    }

    public void third(Runnable printThird) throws InterruptedException {
        third.acquire();
        // printThird.run() outputs "third". Do not change or remove this line.
        printThird.run();
        first.release();
    }
}
```

В конце можно освободить семафор у first, чтобы была возможность снова воспроизвести сценарий.

## Полезные ссылки

1. [Справочник по синхронизаторам java.util.concurrent.*](https://habr.com/ru/articles/277669/)
