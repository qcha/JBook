## Многопоточность в Java
### Введение
Многопоточность.
Ясно, что делать два дела одновременно и хорошо - это несомненный плюс.
Вы можете ехать в метро и слушать музыку, чистить зубы и смотреть фильм.

Зачем в Java?

* Используем CPU, несколько ядер
* многозадачность и производительность
* асинхронная обработка событий

В чем же разница между процессом и потоком?
* Процессы обладают собственным адресным пространством.
* Потоки работают с общей памятью.
* С потоками работает именно планировщик ОС.

### Создание и запуск
Создать поток просто - мы должны отнаследоваться от `Thread`:
```java
public class MyThread extends Thread {
    @Override
    public void run() {/*some work*/}
}
```

Запуск будет выглядеть как
```java
MyThread thread = new MyThread();
thread.start();
```

При этом, надо обязательно помнить, что вызывать надо именно `start`, который уже в отдельном потоке запустит то, что вы написали в `run`.
И да, мы **можем** переопределить start - но этого делать **не стоит**, так как там своя логика запуска вашего потока.

Пример кода:
```java
import java.util.concurrent.TimeUnit;

public class ThreadExample extends Thread {
    private final int COUNT = 10;
    private String name;
    private int delay;

    public ThreadExample(String name, int delay) {
        this.name = name;
        this.delay = delay;
    }

    @Override
    public void run() {
        System.out.println("Thread " + name + " started!");
        try {
            for (int i = 0; i < COUNT; i++) {
                System.out.println(name + " generate number: " + i);
                TimeUnit.SECONDS.sleep(delay);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Thread " + name + " finished!");
    }
}
```

Внимательный читатель сразу спросит, как так, а если я хочу, чтобы мой класс был потоком, но он уже наследует какой-то другой класс. Как быть?

Тут поможет второй вариант создания потока. Это разумеется реализация интерфейса! Наш интерфейс называется `Runnable`.
```java
class Task implements Runnable {
    @Override
    public void run() {/*some work*/}
}
```
Запуск:
```java
Thread thread = new Thread(new Task());
thread.start();
```

Тут все просто - мы реализуем интерфейс `Runnable` и объект такого класса передаем в конструктор `Thread`.
В принципе мы могли бы создать даже анонимный класс прямо в создании `Thread`.
Т.е выглядело бы это как-то так:
```java
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {/*some work*/}
});

thread.start();
```

Пример этого решения:
```java
import java.util.concurrent.TimeUnit;

public class RunnableExample implements Runnable {
    private final int COUNT = 10;
    private String name;
    private int delay;

    public RunnableExample(String name, int delay) {
        this.name = name;
        this.delay = delay;
    }

    @Override
    public void run() {
        System.out.println("RunnableThread " + name + " started!");
        try {
            for (int i = 0; i < COUNT; i++) {
                System.out.println(name + " generate number: " + i);
                TimeUnit.SECONDS.sleep(delay);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("RunnableThread " + name + " finished!");
    }
}
```

Есть еще третий вариант: это создать `Timer`.
```java
Timer timer = new Timer();
timer.schedule(new TimeTask {
    @Override
    public void run() {/*some work*/}
}, 60);
```

Выполнение задачи в отдельном потоке, но по таймеру.

```java
package samples.concurrency;

import java.util.TimerTask;
import java.util.concurrent.TimeUnit;

public class TimerTaskExample extends TimerTask {
    private final int COUNT = 10;
    private String taskName;
    private int delay;

    public TimerTaskExample(String taskName, int delay) {
        this.taskName = taskName;
        this.delay = delay;
    }

    @Override
    public void run() {
        System.out.println("Thread " + taskName + " started!");
        try {
            for (int i = 0; i < COUNT; i++) {
                System.out.println(taskName + " generate number: " + i);
                TimeUnit.SECONDS.sleep(delay);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Thread " + taskName + " finished!");
    }
}
```

Теперь посмотрим что будет, если я вместо start() буду вызывать именно run() метод.
```java
    void wrongRun() {
        ThreadExample te1 = new ThreadExample("My thread", 1);
        ThreadExample te2 = new ThreadExample("My thread 2", 2);
        te1.run();
        te2.run();
    }
```
И увидим мы то, что все будет выполняться последовательно, без многопоточности. Сначала отработает te1, а уже после него начнет работу te2. Поэтому **НЕ** вызывайте run() напрямую - это *бесполезно*.

### Жизнь потока
Итак, как создать и запустить - разобрались.
Теперь - о том, что вообще с ним происходит после start.

Не вдаваясь в подробности, у потока есть состояния: `RUNNABLE`, `WAITING`, `BLOCKED`, `TERMINATED`.

Сразу после start - поток в состоянии `RUNNABLE`. Он выполняется. Далее, если мы вызываем `wait()` - мы переводим его в состояние `WAITING`, откуда он может стать снова `RUNNABLE`, если вызвать `notify()` и может перейти в состояние `TERMINATED`.

Также из состояния `RUNNABLE` мы можем попасть в `BLOCKED`, если получим lock по монитору(lock on monitor), соответственно как только монитор освободится - мы снова перейдем в `RUNNABLE`. Также можно попасть в `TERMINATED`.

И последний вариант из `RUNNABLE` мы можем попасть в `TERMINATED`, если вызовем метод `interrupt()`.

На схеме это выглядит как-то так:


![](../images/thread-life.png)


Как видно по схеме - мы из `TERMINATED` не выйдем обратно, т.е если поток завершился - второй раз start() уже не вызовем. Т.е надо создать объект новый потока и запустить его заново, если нам надо это.

Почему так? А потому, что наши потоки - это объекты. А значит, запустив поток мы можем состояние нашего потока как-то изменить, например, изменить ему имя, если у нашего объекта есть поле имя. Или что-то еще, но главное, что мы можем изменить состояние объекта. После завершения потока наш объект может быть совсем не в том виде, что при создании объекта конструктором.  И если бы мы могли заново запускать отработанный поток - мы бы запускали уже объект с неизвестными данными. Поэтому - так нельзя.

### Поток демон
Мы видим, что у потоков есть метод: `setDaemon()`.
Зачем?

Приложение на Java работает до тех пор, пока существует и работает хотя бы один поток не демон.
Так вот, если у нас поток-демон, то его состояние не учитывается в решении завершать или нет приложение.
Пример - запись в лог.

### Thread join()
Join заставляет дождаться завершения потока у которого он вызван.
Т.е:
```java
    static void joinExample() throws InterruptedException {
        ThreadExample te1 = new ThreadExample("My thread", 1);
        ThreadExample te2 = new ThreadExample("My thread 2", 2);
        te1.start();
        te1.join();
        System.out.println("Main thread!");
        te2.start();
    }
```

Видим, что пока не завершится поток te1 - дальше ничего не происходит, ни в Main-thread, ни в te2.

### Остановка по требованию
А теперь рассмотрим как же остановить поток.

Да, есть метод `stop()` - но он помечен как **depreacated** и настоятельно рекомендуется его не использовать.

Почему?
Да потому, что так делать опасно. Пусть мы в потоке пишем в БД какие-то данные. Или же работаем с каким-нибудь файлом или сетью. И тут - резкий стоп. Т.е поток сразу все бросает и оставляет какие-то ресурсы, состояния объектов, с которыми работал такой поток может быть не валидным. Поэтому - **не используйте stop**.

Тогда как быть?

Случай 1 - работаем со своим флагом.
Заводим флажок, оборачиваем все в цикл и работаем, пока флаг - false.
```java
class MyThread extends Thread {
private volatile boolean  stopFlag = false;
    @Override
    public void run() {
        while(!stopFlag) {
            /*work here*/
        }
    }
}

//bla bla
MyThrad thread = new MyThread();
thread.setStopFlag(true);
```

Пример:
```java
import java.util.concurrent.TimeUnit;

public class ThreadWithFlag extends Thread {
    private volatile boolean stopFlag = false;

    @Override
    public void run() {
        while (!stopFlag) {
            System.out.println("i still alive");
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        System.out.println("That's all...");
    }

    public void setStopFlag(boolean stopFlag) {
        this.stopFlag = stopFlag;
    }
}
```

Тут кинется исключения, но об этом - ниже.

Примерно также работает и второй случай - это через `Thread.currentThread.isInterrupted())`:
```java
class MyThread extends Thread {
    @Override
    public void run() {
        while(!Thread.currentThread.isInterrupted())) {
            /*work here*/
        }
    }
}

//bla bla
MyThrad thread = new MyThread();
thread.interrupt();
```

То же самое, что и первый способ, но используя реализацию в Java этого.

Пример:
```java
import java.util.concurrent.TimeUnit;

public class ThreadInterruptExample extends Thread {
    @Override
    public void run() {
        while (!Thread.currentThread().isInterrupted()) {
            System.out.println("i still alive");
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        System.out.println("That's all...");
    }
}
```

Но надо помнить - вы должны где-то проверять поток на остановку!
Если сделать как:
```java
class MyThread extends Thread {
    @Override
    public void run() {
        /*work here*/
    }
}

//bla bla
MyThrad thread = new MyThread();
thread.interrupt();
```
**Поток не остановится!**

Пример:
```java
public class ThreadWithoutInterruptCheck extends Thread {
    @Override
    public void run() {
        while (true) {
            System.out.println("i still alive");
        }
    }
}
```

Ну вроде понятно как с этим работать.
Однако не все так просто.

Ведь есть блокирующие операции:
* sleep()
* join()
* wait()

*Замечание*
При этом чтение чего-либо большого, например, из сети - это тоже блокирующая операция, но `InterruptedException` не выкидывается при `interrupt()`. Но мы можем перед тем, как вызывать read вызывать проверку - есть ли данные там.

В этим моменты(когда, например, поток спит) - мы не проверяем флаг. А значит и не останавливаем поток.
Пусть поток спит. В этом время мы делаем `interrupt()`. Что произойдет?
А произойдет вот что: в нашем потоке после interrupt() кинется исключение(помните, что мы оборачиваем все блокирующие вызовы try-catch?), мы его перехватим(или поднимем выше) и будем уже думать, что делать(закрывать ресурсы, выходить из потока и т.д). *Обратите внимание на try/catch блок!*

И такое будет работать **только** при использовании Java `interrupt()`, со своим флагом такое уже не сработает - дело в реализации `interrupt()`.

Пример:
```java
import java.util.concurrent.TimeUnit;

public class ThreadInterruptWhileSleep extends Thread {
    @Override
    public void run() {

        while (!Thread.currentThread().isInterrupted()) {
            System.out.println("i still alive");
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                //catch exception while sleep
                e.printStackTrace();
                //and after print exception - interrupt our thread
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

To sum up:
* `interrupt()` выставляет флаг прерывания потока, если он был в состоянии `BLOCKED` или `WAITING` - то кинется `InterruptedException`
* `isINterrupted()` - проверит флаг
* `static interrupted()` - проверит состояние флага у потока и сбросит флаг. Это если мы вызываем `interrupted()` дважды подряд, тогда второй вызов вернет false.
* обязательно проверяем состояние потока в нем, иначе не остановим его.
* Не использовать `stop()`.
* Не использовать какие-то свои флаги, так как в состояниях `BLOCKED` или `WAITING` не дойдем до его проверки.
* Поймав `InterruptedException` - думаем, что делать, хорошей идеей будет вызывать `interrupt()` у текущего потока, так как мы его разбудили InterruptedException, а теперь - выставляем флаг.

### Пул потоков
Окей, с одиночными потоками разобрались.
Теперь рассмотрим такую штуку, как пул потоков.
Пул потоков может ограничивать количество создаваемых потоков, например, у нас есть сервис и мы хотим, чтобы с клиентами общалось не более 10 потоков, а больше уже не надо. Тогда мы создаем пул потоков для этого, как только задача потоком выполнилась - мы возвращаем поток обратно в пул. А после уже и переиспользовать. Т.е он копит некоторые задачи на исполнение и как только появляется свободный поток - он дает ему задачу.

```java
public class ThreadPoolExample {
    private ExecutorService threadPool = Executors.newFixedThreadPool(2);
    private List<Future> futureList = new ArrayList<>();

    static class Task implements Callable<Integer> {

        private int number;

        public Task(int number) {
            this.number = number;
        }

        @Override
        public Integer call() throws Exception {
            int result = 0;
            for(int i = 1; i <= number; i++)
                result += i;

            return result;
        }
    }

    public void exampleOfTasks() {
        for (int i = 0; i < 5; i++) {
            threadPool.submit(new ThreadExample("Thread#" + i, 2));
        }

        threadPool.shutdown();
    }
}
```

Также есть еще механизм, называемый Future.
### Future
Помимо интерфейса `Runnable`, который дает нам метод `public void run()`, есть еще один интерфейс - `Callable`, он параметризован и имеет метод `public K call()`, где K - тип возвращаемого значения.
Поток, созданный с Callable умеет возвращать значение, после завершения.
Если не параметризовать интерфейс - то `call()` возвращать будет Object.
А после уже, мы можем у Future методом `get()` получить это значение, когда поток уже сделал всю работу. При этом надо понимать, что `get()` дождется окончания потока.

```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

public class ThreadPoolExample {
    private ExecutorService threadPool = Executors.newFixedThreadPool(2);
    private List<Future> futureList = new ArrayList<>();

    static class Task implements Callable<Integer> {

        private int number;

        public Task(int number) {
            this.number = number;
        }

        @Override
        public Integer call() throws Exception {
            int result = 0;
            for(int i = 1; i <= number; i++)
                result += i;

            return result;
        }
    }

    public void exampleOfFuture() throws Exception {
        for (int i = 0; i < 5; i++) {
            Future future = threadPool.submit(new Task(i));
            futureList.add(future);
        }

        for (Future future : futureList) {
            System.out.println("Result: " + future.get());
        }

        threadPool.shutdown();
    }
}
```

### Race condition
Пусть у нас есть некий класс Counter, у него поле - counter. И метод, который умеет увеличивать значение счетчика.
И пусть мы в двух потоках увеличиваем этот счетчик в цикле до 100.

Что мы ожидаем?Что будет 200.
Но на деле мы каждый раз при запуске будем получать разное значение. Почему? Из-за Race condition.
Это - Критическая секция.
Это код, в котором мы обращаемся к общему ресурсу, который не должен быть использован более чем одним потоком.
Мы обращаемся к общему ресурсу из разных потоков и можем попасть в состояние, когда он изменяется одним потоком из второго.

По сути это из-за того, что когда мы делаем инкремент - мы на уровне процессора делаем три операции(Read-Modify-Write). Эта операция не атомарна.

Атомарные операции - это операции выполняющиеся как единое целое или вообще не выполняющиеся.

Атомарные:
* Чтение/запись примитивов(**но без поддержки long/double**)
* Чтение/запись ссылок
* Чтение/запись volatile примитивов
* Атомарные операции из java пакета(Atomic integer и т.д)

Так вот, в Java есть такое понятие как монитор - средство контроля к ресурсу. У каждого объекта есть свой монитор. У монитора в каждый момент времени - только один поток.
Монитор по сути - это id исполняющегося потока, если у него стоит 0 - объект свободен.

Ниже - мы делаем лок на мониторе текущего объекта, т.е this.
Synchronized можно передать объект для лока тоже.
```java
public class Example {
    //synchronized на this
    public synchronized void test() {
        /* work here */
    }

    //synchronized на объекте
    public  void test() {
        synchronized(obj) {
             /* work here */
        }
    }
}
```

`Static` блокируется на `.class` объекте.

Разницы между локом на объекте и на `this` почти нет - разве что мы при локе на объекте можем не весь метод в критическую секцию отдать, а только часть.

Одна из возможных проблем - deadlock.
Пусть есть два ресурса(A и B) и два потока. Один поток захватывает ресурс A, второй - ресурс B, теперь если первый поток обратится к B, а второй поток к A - они останутся в вечном ожидании, так как каждый будет ждать освобождения ресурса от другого.
Для предотвращения можно использовать tryLock, lock по таймеру и т.д

Есть еще семафор - это лок, который допускает не один поток к ресурсу, а несколько, сколько мы зададим. Как только поток ресурс освобождает - семафор впускает следующий поток.
