# Concurrent increment

Задание с собеседования в одну из крупных финтех компаний на технической секции.

## Условие

Дан следующий код:

```java
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Increment {
  private static int counter1 = 0;
  private static int counter2 = 0;

  public static void main(String[] args) throws InterruptedException {
    int tasksCount = 100_000;
    CountDownLatch latch = new CountDownLatch(tasksCount);
    ExecutorService executor = Executors.newFixedThreadPool(100);

    for (int i = 0; i < tasksCount; i++) {
      executor.submit(() -> {
        counter1++;
        counter2++;
        latch.countDown();
      });
    }

    latch.await();

    System.out.println(counter1);
    System.out.println(counter2);
    System.exit(0);
  }
}
```

Что выведет код при запуске? Объясните почему.

Что надо сделать, чтобы вывод был правильным? Объясните свой выбор.

## Решение

Вопрос с подвохом, так как результат работы кода не детерминирован: при каждом запуске мы будем получать разные значения.

Происходит это из-за конкурентного чтения и инкремента счетчиков в многопоточной среде.

Если коротко, то представьте ситуацию, что у вас есть переменная и два потока. Оба потока стартуют одновременно и считывают значение из переменной, после меняют его на единицу и записывают обратно. В момент чтения оба видят 0 и делают инкремент, после чего значение становится 1, каждый записывает эту единицу.

Далее происходит снова чтение, но в этот раз один из потоков читает быстрее и делает инкремент быстрее, а другой чуть медленнее. В такой момент первый читает 1, делает инкремент и записывает 2. Второй (который чуть опаздывает) читает уже 2 и делает инкремент, после этого пишет 3-ку. Далее снова оба читают одновременно 3-ку. Оба делают инкремент и пишут 4-ку. Это выливается в ситуацию, когда запись постоянно 'сбивается' из-за несогласованности чтения и записи.

Соответственно, чтобы вывод был правильный необходимо сделать синхронизацию. Это можно сделать несколькими способами.

### Atomic

Использовать `java.util.concurrent.atomic.AtomicInteger` или любой другой подходящий atomic.

 ```java
public class Main {
    private static final AtomicInteger counter1 = new AtomicInteger(0);
    private static final AtomicInteger counter2 = new AtomicInteger(0);
    public static void main(String[] args) throws InterruptedException {
        int tasksCount = 100_000;
        CountDownLatch latch = new CountDownLatch(tasksCount);
        ExecutorService executor = Executors.newFixedThreadPool(100);
        for (int i = 0; i < tasksCount; i++) {
            executor.submit(() -> {
                counter1.incrementAndGet();
                counter2.incrementAndGet();
                latch.countDown();
            });
        }
        latch.await();
        System.out.println(counter1);
        System.out.println(counter2);
        System.exit(0);
    }
}
```

Atomic классы как раз и призваны атомарно работать с такими операциями как инкремент.

### Synchronized

Вынести инкремент в синхронизованную секцию:

```java
public class Main {
    private static int counter1 = 0;
    private static int counter2 = 0;
    public static void main(String[] args) throws InterruptedException {
        int tasksCount = 100_000;
        CountDownLatch latch = new CountDownLatch(tasksCount);
        ExecutorService executor = Executors.newFixedThreadPool(100);
        for (int i = 0; i < tasksCount; i++) {
            executor.submit(() -> {
                incrementCounter1();
                incrementCounter2();
                latch.countDown();
            });
        }
        latch.await();
        System.out.println(counter1);
        System.out.println(counter2);
        System.exit(0);
    }
    public static synchronized void incrementCounter1() {
        counter1++;
    }
    public static synchronized void incrementCounter2() {
        counter2++;
    }
}
```

Здесь чтение, изменение и запись каждого счетчика вынесены в синхронизованные секциии. При этом, обратите внимание, что синхронизация происходит на объекте класса, так как `synchronized` стоит у метода.
