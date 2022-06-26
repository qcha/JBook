# Java Concurrency

## Введение

Человек давно привык делать несколько дел одновременно.
Мы едем в метро, слушаем музыку и думаем о работе, чистим зубы и смотрим фильм.

При этом каждая задача борется с другими за 'процессорное время' - за мозг человека.
Мозг, ловко переключается между задачами, давая каждой чуть-чуть своего внимания, и создается впечатление, что мы делаем задачи параллельно.

Зачем нам это? Для производительности!
Всегда ли это хорошо и помогает нам? Не всегда, сами понимаете, что некоторые задачи выполнять 'параллельно' не получится.

Точно то же самое можно сказать и про программирование!
Только в `Java` задачи выполняют `Thread`-ы или потоки.

> В разных источниках по разному переводится слово `Thread`: поток, нить и прочее.
> Мы будем просто говорить `thread` или поток.

Где мы можем получить выгоду от многопоточности?

* У нас много `cpu` ядер и наша задача хорошо разбивается на подзадачи.
    Например, вычислительные задачи.

* Нам надо быстро отреагировать на запрос.
    Например, пользователь запросил данные, для их получения требуется время, поэтому мы параллельно запускаем задачу сбора данных и при этом не прекращаем диалог с пользователем, не блокируем ему интерфейс.

* Многопользовательский сервис.
    Сервис, где каждый запрос обрабатывается параллельно с другими.

Где выгоды от увеличения многопоточности не будет?

* Задача плохо параллелизуется.
    Например, упирается в неразделяемый ресурс.

* У нас мало `cpu` ядер.
    Если у нас всего два ядра, то от большого увеличения количества потоков пользы не будет.
    Ведь потоки будут 'драться' за процессорное время, по сути за эти два ядра.
    И увеличение количества потоков только негативно скажется на производительности.

Еще одним ограничением является закон Амдала.

### Закон Амдала

Иллюстрирует ограничение роста производительности вычислительной системы с увеличением количества вычислителей:

> Пусть необходимо решить некоторую вычислительную задачу.
>
> Предположим, что её алгоритм таков, что доля `a` от общего объёма вычислений может быть получена только последовательными расчётами, а, соответственно, доля `1- a`  может быть распараллелена идеально (то есть время вычисления будет обратно пропорционально числу задействованных узлов `p`.
>
> Тогда ускорение, которое может быть получено на вычислительной системе из `p` процессоров, по сравнению с однопроцессорным решением не будет превышать величины:
>
> ![Amdahl's law](../images/concurrency/amdahl_law.png)

Графически это выглядит как:

![Amdahl's law graphics](../images/concurrency/amdahl_law_graphics.png)

Теперь представим, что у нас есть работа и 80% ее можно выполнить раздельно, но 20% должно быть выполнено последовательно.

Вопрос: какое максимальное ускорение мы можем получить при распараллеливании?

И в таком случае, ответ на вопрос будет: не более, чем в пять раз.

Таблица показывает, во сколько раз быстрее выполнится программа с долей последовательных вычислений `a` при использовании `p` процессоров:

| a \ p  | 10     | 100    | 1 000  |  
|--------|--------|--------|--------|
|  10 %  | 5,263  | 9,174  | 9,910  |
|  25 %  | 3,077  | 3,883  | 3,988  |
|  40 %  | 2,174  | 2,463  | 2,496  |

На самом деле все еще хуже, потому что есть еще межпоточная координация, которая тоже влияет на производительность.

Это как раз учитывается в `Universal Scalability Law` или `USL`, который является расширением закона Амдала:

![USL](../images/concurrency/USL.png)

Здесь, k - это параметр, определяющий штраф на межпоточную координацию (cohesion).

И графики сравнения будут:

![USL](../images/concurrency/usl_vs_amdahls.png)

Видно, что после какого-то значения увеличение количества потоков только ухудшает производительность. Где-то есть лимит, после которого нет смысла уже увеличивать количество потоков.

Поэтому, прежде чем использовать многопоточность у себя необходимо ответить на вопросы:

* Хорошо ли задача паралелизуется?
* Насколько многопоточным должно быть решение?

## Concurrency vs Parallelism

Еще важно понимать, что многопоточная программа не обязательно выполняется параллельно.

Как же так?

У нас два автомата по выдаче напитков, соответственно, два человека могут получить напиток сразу - это параллельное выполнение программы.

![Concurrency vs Parallelism](../images/concurrency/concurrency_vs_parallelism.png)

Многопоточная программа не обязательно выполняет действия параллельно, она потенциально может стать параллельной, если добавить больше ресурсов.

Потоков может быть больше (чаще всего их и так больше), нежели ядер процессора, на которых потоки выполняются.
Например, мы вполне можем создать несколько потоков на одноядерном процессоре, соответственно, наша программа будет многопоточной, но не параллельной.

Остюда же неявно следует, что если бесконтрольно плодить потоки, то это негативно скажется на производительности.
Просто представьте, что у вас 4 автомата с колой, а вы сделали 1000 очередей!

Добавим сюда умные формулы, графики из `USL` и получим следующую картину:

![Theory vs Actual](../images/concurrency/theory_vs_actual.png)

Теперь, когда мы поняли все риски, приступим к рассмотрению многопоточности в `Java`.

## Создание

### Наследник java.lang.Thread

Самый простой (и не самый лучший) способ создать поток - это отнаследоваться от `java.lang.Thread` и переопределить метод `void run()`, куда поместить свою логику:

```java
public class MyThread extends Thread {
    @Override
    public void run() {
        // наша логика здесь
    }
}
```

Внимательный читатель увидит, что у класса `java.lang.Thread` есть еще один метод: `start()`

Запуск поток **всегда** происходит через метод `start()`:

```java
MyThread thread = new MyThread();
thread.start();
```

---

**Вопрос**:

Зачем нам два метода? Почему `start` - это запуск потока, а `run` - это наша логика работы, которая будет запущена в потоке?

**Ответ**:

Для ответа на этот вопрос рассмотрим пример:

```java
class ThreadExample extends Thread {
    private final String greeting;
    private final int count;

    public ThreadExample(String greeting, int count) {
        this.greeting = greeting;
        this.count = count;
    }

    @Override
    public void run() {
        System.out.println(greeting);
        
        int sum = 0;
        for(int i = 0; i < count; i ++) {
            sum += i;
        }

        System.out.println("Sum is:" + sum);
    }
}

public class Example {
    public static void main(String[] args) {
        ThreadExample te1 = new ThreadExample("Hello from 1", 1_000_000);
        te1.run();
        
        ThreadExample te2 = new ThreadExample("Hello from 2", 2);
        te2.run();
    }
}
```

Запустим и увидим, что все будет выполняться последовательно.
Сначала отработает `te1`, а уже после него начнет работу `te2`.

Поэтому никогда **НЕ** вызывайте метод `run` напрямую!

И отсюда же следует ответ почему существует метод `start`!

Дело в том, что кто-то должен породить поток. Да, мы написали логику в `run`.
Но далее кто-то должен породить в операционной системе поток и запустить эту логику в нем.

Этим и занимается `start`, прося выделить поток в `ОС` (через `native` метод `private native void startImpl();`), а после запуская внутри `run`.

---

**Вопрос**:

Можем ли мы переопределить `start`? Что в таком случае будет?

**Ответ**:

Как и любой не `final` публичный метод в `Java`, разумеется `start` можно переопределить.

Но этого делать **не стоит**!.

Ведь переопределнный метод `start` уже не сможет запросить создание потока у `ОС`, соответственно дальнейшая работа уже будет бессмысленна:

---

### java.lang.Runnable

Создавать наследника `java.lang.Thread`, выделяя под это целый класс зачастую неудобно (и неправильно).

Во-первых, ваш класс уже может быть чьим-то наследником, поэтому просто отнаследоваться от `java.lang.Thread` не получится.

Во-вторых, иногда просто неудобно создавать целый класс, чтобы переопределить `run`.
Неудобно, громоздко, плодит лишние классы, добавляя проблем с выдумыванием им названий!

В-третьих, как уже было сказано, основная логика всегда в `run`, остальное состояние `Thread`-а вам не нужно.
Поэтому было бы логично, если бы существовал еще и интерфейс для выделения логики в `run`.

Классы не могут, интерфейсы помогут и решение нашей проблемы называется `java.lang.Runnable`:

```java
/**
 * The <code>Runnable</code> interface should be implemented by any
 * class whose instances are intended to be executed by a thread. The
 * class must define a method of no arguments called <code>run</code>.
 * <p>
 * This interface is designed to provide a common protocol for objects that
 * wish to execute code while they are active. For example,
 * <code>Runnable</code> is implemented by class <code>Thread</code>.
 * Being active simply means that a thread has been started and has not
 * yet been stopped.
 * <p>
 * In addition, <code>Runnable</code> provides the means for a class to be
 * active while not subclassing <code>Thread</code>. A class that implements
 * <code>Runnable</code> can run without subclassing <code>Thread</code>
 * by instantiating a <code>Thread</code> instance and passing itself in
 * as the target.  In most cases, the <code>Runnable</code> interface should
 * be used if you are only planning to override the <code>run()</code>
 * method and no other <code>Thread</code> methods.
 * This is important because classes should not be subclassed
 * unless the programmer intends on modifying or enhancing the fundamental
 * behavior of the class.
 *
 * @author  Arthur van Hoff
 * @see     java.lang.Thread
 * @see     java.util.concurrent.Callable
 * @since   JDK1.0
 */
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}
```

Реализовав интерфейс и передав его в конструктор `java.lang.Thread`, мы точно также через `start` запустим отдельный поток выполнения:

```java
class Task implements Runnable {
    @Override
    public void run() {
        // логика
    }
}

Thread thread = new Thread(new Task());
thread.start();
```

В принципе, можно сделать то же самое через анонимный класс:

```java
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        /*some work*/
    }
});

thread.start();

// или через лямбду
Thread thread2 = new Thread(() -> {
    // работа тут
}));

thread2.start();
```

---

**Вопрос**:

Как вы думаете, что будет выведено на экран, если запустить следующий код?

```java
class ThreadExample extends Thread {
    private final int num;

    public ThreadExample(int num) {
        this.num = num;
    }

    @Override
    public void run() {
        System.out.println("Thread : " + num);
    }
}

public class Example {
    public static void main(String[] args) {
        new ThreadExample(1).start();
        new ThreadExample(2).start();
        new ThreadExample(3).start();
        new ThreadExample(4).start();
    }
}
```

**Ответ**:

Запустим пример выше:

```java
Thread : 2
Thread : MAIN
Thread : 1
Thread : 3
Thread : 4
```

Сделаем это еще раз:

```java
Thread : 2
Thread : 3
Thread : 4
Thread : 1
Thread : MAIN
```

Порядок выполнения потоков **недетерминирован**!

---

С запуском разобрались. Теперь посмотрим, что еще мы можем сделать с потоком?

## Остановка

Примеры ранее представляли поток как последовательный набор операций. После выполнения последней операции завершался и поток.

Но зачастую поток должен постоянно делать какую-то работу, пока его явно не попросят остановиться. Это может быть как опрос сокета на новые данные, мониторинг появления новых файлов в директории, да что угодно!

И как в таком случае правильно останавливать поток?

У `java.lang.Thread` есть метод `stop`:

```java
   /**
     * Forces the thread to stop executing.
     * <p>
     * If there is a security manager installed, its {@code checkAccess}
     * method is called with {@code this}
     * as its argument. This may result in a
     * {@code SecurityException} being raised (in the current thread).
     * <p>
     * If this thread is different from the current thread (that is, the current
     * thread is trying to stop a thread other than itself), the
     * security manager's {@code checkPermission} method (with a
     * {@code RuntimePermission("stopThread")} argument) is called in
     * addition.
     * Again, this may result in throwing a
     * {@code SecurityException} (in the current thread).
     * <p>
     * The thread represented by this thread is forced to stop whatever
     * it is doing abnormally and to throw a newly created
     * {@code ThreadDeath} object as an exception.
     * <p>
     * It is permitted to stop a thread that has not yet been started.
     * If the thread is eventually started, it immediately terminates.
     * <p>
     * An application should not normally try to catch
     * {@code ThreadDeath} unless it must do some extraordinary
     * cleanup operation (note that the throwing of
     * {@code ThreadDeath} causes {@code finally} clauses of
     * {@code try} statements to be executed before the thread
     * officially dies).  If a {@code catch} clause catches a
     * {@code ThreadDeath} object, it is important to rethrow the
     * object so that the thread actually dies.
     * <p>
     * The top-level error handler that reacts to otherwise uncaught
     * exceptions does not print out a message or otherwise notify the
     * application if the uncaught exception is an instance of
     * {@code ThreadDeath}.
     *
     * @throws     SecurityException  if the current thread cannot
     *             modify this thread.
     * @see        #interrupt()
     * @see        #checkAccess()
     * @see        #run()
     * @see        #start()
     * @see        ThreadDeath
     * @see        ThreadGroup#uncaughtException(Thread,Throwable)
     * @see        SecurityManager#checkAccess(Thread)
     * @see        SecurityManager#checkPermission
     * @deprecated This method is inherently unsafe.  Stopping a thread with
     *       Thread.stop causes it to unlock all of the monitors that it
     *       has locked (as a natural consequence of the unchecked
     *       {@code ThreadDeath} exception propagating up the stack).  If
     *       any of the objects previously protected by these monitors were in
     *       an inconsistent state, the damaged objects become visible to
     *       other threads, potentially resulting in arbitrary behavior.  Many
     *       uses of {@code stop} should be replaced by code that simply
     *       modifies some variable to indicate that the target thread should
     *       stop running.  The target thread should check this variable
     *       regularly, and return from its run method in an orderly fashion
     *       if the variable indicates that it is to stop running.  If the
     *       target thread waits for long periods (on a condition variable,
     *       for example), the {@code interrupt} method should be used to
     *       interrupt the wait.
     *       For more information, see
     *       <a href="{@docRoot}/java.base/java/lang/doc-files/threadPrimitiveDeprecation.html">Why
     *       are Thread.stop, Thread.suspend and Thread.resume Deprecated?</a>.
     */
    @Deprecated(since="1.2")
    public final void stop()
```

Однако, как можно заметить по `javadoc`, он `deprecated` и крайне не рекомендован к использованию.

Почему?

Обратимся к Oracle [за поясненимями](https://docs.oracle.com/javase/1.5.0/docs/guide/misc/threadPrimitiveDeprecation.html):

> Why is Thread.stop deprecated?
>
> Because it is inherently unsafe. Stopping a thread causes it to unlock all the monitors that it has locked. (The monitors are unlocked as the ThreadDeath exception propagates up the stack.)
> If any of the objects previously protected by these monitors were in an inconsistent state, other threads may now view these objects in an inconsistent state.
> Such objects are said to be damaged. When threads operate on damaged objects, arbitrary behavior can result.
> This behavior may be subtle and difficult to detect, or it may be pronounced.
> Unlike other unchecked exceptions, ThreadDeath kills threads silently; thus, the user has no warning that his program may be corrupted.
> The corruption can manifest itself at any time after the actual damage occurs, even hours or days in the future.

Что это значит?

А это значит, что завершать поток на полном ходу, через `stop`, словно коня на скаку, плохая идея.
Например, потому что у нас нет гарантии, что в таком случае поток не заврешается посередине атомарной операции!
Это чревато тем, что поток при завершении может оставить какие-то объекты, ресурсы в незаконченном/поврежденном состоянии.

Соответственно, нам нужен механизм, который **сообщит** потоку, что **пора** завершаться.

### Флаг

Как мы можем это сделать? С помощью флага!

```java
class MyThread extends Thread {
    private volatile boolean interrupted = false;

    public void setInterrupted(boolean flag) {
        stopFlag = flag;
    }

    @Override
    public void run() {
        while(!interrupted) {
            /* here we're doing our work */
            System.out.println("I'm still alive");
        }

        /* here we're shutting down and releasing resources */
        System.out.println("That's all...");
    }
}

// bla bla
Thread thread = new MyThread();

// some work

thread.setInterrupted(true);
```

> О магическом слове `volatile` будет сказано позднее и вы позже поймете, почему оно здесь и зачем.

Сам алгоритм прост, потому красив: пока потоку не сказали через флаг, что пора завершаться - он выполняется, выставляем флаг и поток (при итерации следующей) выйдет из цикла.

При использовании флага можно освободить ресурсы, подготовить объекты, с которыми работали к завершению работы и т.д.

### interrupted

Давайте взглянем на внутреннее устройство `java.lang.Thread` и обнаружим там похожие на то, что мы писали выше:

```java
    /* Interrupt state of the thread - read/written directly by JVM */
    private volatile boolean interrupted;

    /**
     * Interrupts this thread.
     *
     * <p> Unless the current thread is interrupting itself, which is
     * always permitted, the {@link #checkAccess() checkAccess} method
     * of this thread is invoked, which may cause a {@link
     * SecurityException} to be thrown.
     *
     * <p> If this thread is blocked in an invocation of the {@link
     * Object#wait() wait()}, {@link Object#wait(long) wait(long)}, or {@link
     * Object#wait(long, int) wait(long, int)} methods of the {@link Object}
     * class, or of the {@link #join()}, {@link #join(long)}, {@link
     * #join(long, int)}, {@link #sleep(long)}, or {@link #sleep(long, int)}
     * methods of this class, then its interrupt status will be cleared and it
     * will receive an {@link InterruptedException}.
     *
     * <p> If this thread is blocked in an I/O operation upon an {@link
     * java.nio.channels.InterruptibleChannel InterruptibleChannel}
     * then the channel will be closed, the thread's interrupt
     * status will be set, and the thread will receive a {@link
     * java.nio.channels.ClosedByInterruptException}.
     *
     * <p> If this thread is blocked in a {@link java.nio.channels.Selector}
     * then the thread's interrupt status will be set and it will return
     * immediately from the selection operation, possibly with a non-zero
     * value, just as if the selector's {@link
     * java.nio.channels.Selector#wakeup wakeup} method were invoked.
     *
     * <p> If none of the previous conditions hold then this thread's interrupt
     * status will be set. </p>
     *
     * <p> Interrupting a thread that is not alive need not have any effect.
     *
     * @implNote In the JDK Reference Implementation, interruption of a thread
     * that is not alive still records that the interrupt request was made and
     * will report it via {@link #interrupted} and {@link #isInterrupted()}.
     *
     * @throws  SecurityException
     *          if the current thread cannot modify this thread
     *
     * @revised 6.0, 14
     */
    public void interrupt()

    /**
     * Tests whether this thread has been interrupted.  The <i>interrupted
     * status</i> of the thread is unaffected by this method.
     *
     * @return  {@code true} if this thread has been interrupted;
     *          {@code false} otherwise.
     * @see     #interrupted()
     * @revised 6.0, 14
     */
    public boolean isInterrupted() {
        ...
    }
```

Т.е. у поток **уже** есть этот флаг, а также методы работы с ним!

В таком случае метод `run` потока можно написать уже так:

```java
class MyThread extends Thread {

    @Override
    public void run() {
        while(!isInterrupted()) {
            /* here we're doing our work */
            System.out.println("I'm still alive");
        }

        /* here we're shutting down and releasing resources */
        System.out.println("That's all...");
    }
}

// bla bla
Thread thread = new MyThread();

// some work

thread.interrupt();
```

Заметьте, что метод `interrupt` не ждет завершения потока, он лишь **сообщает** потоку, что его попросили прерваться.
Поэтому флаг `interrupted` надо явно проверять во время выполнения, никакой обработки по умолчанию у него нет.
В примере выше мы явно проверяем состояние флага с помощью метода `isInterrupted`.

Здесь возникает вопрос: а что если мы не наследовались от `java.lang.Thread`, а реализовали интерфейс `java.lang.Runnable`?
Как проверять флаг в таком случае?

```java
class RunnableExample implements Runnable {
    @Override
    public void run() {
        while (/* как проверять флаг? */) {
            System.out.println("I'm still alive");
        }

        System.out.println("That's all...");
    }
}
```

В таком случае, у `java.lang.Thread` существует статический метод, возвращающий ссылку на текущий поток:

```java
    /**
     * Returns a reference to the currently executing thread object.
     *
     * @return  the currently executing thread.
     */
    @IntrinsicCandidate
    public static native Thread currentThread();
```

И наш код с проверкой флага можно переписать в виде:

```java
class RunnableExample implements Runnable {
    @Override
    public void run() {
        while (!Thread.currentThread().isInterrupted()) {
            System.out.println("I'm still alive");
        }

        System.out.println("That's all...");
    }
}
```

### interrupted vs isInterrupted

Внимательный читатель уже заметил, что у класса `java.lang.Thread` целых два метода, возвращающих `boolean` о состоянии флага прервывания, при этом один из них, `interrupted`, статический:

```java
    /**
     * Tests whether the current thread has been interrupted.  The
     * <i>interrupted status</i> of the thread is cleared by this method.  In
     * other words, if this method were to be called twice in succession, the
     * second call would return false (unless the current thread were
     * interrupted again, after the first call had cleared its interrupted
     * status and before the second call had examined it).
     *
     * @return  {@code true} if the current thread has been interrupted;
     *          {@code false} otherwise.
     * @see #isInterrupted()
     * @revised 6.0, 14
     */
    public static boolean interrupted() {
        Thread t = currentThread();
        boolean interrupted = t.interrupted;
        // We may have been interrupted the moment after we read the field,
        // so only clear the field if we saw that it was set and will return
        // true; otherwise we could lose an interrupt.
        if (interrupted) {
            t.interrupted = false;
            clearInterruptEvent();
        }
        return interrupted;
    }
```

В чем отличия `isInterrupted` и `interrupted`?

Метод `isInterrupted` - это просто проверка на прерванный статус.

В то время как `interrupted`, как видно из кода, это не просто проверка, это ещё и сброс флага прерванности потока в `false`.
Т.е. вызов **повторный** вызов метода `interrupted` вернет `false`, даже если поток был прерван! Ведь метод сбросит флаг.

Обратите внимание еще на то, что это статический метод, действующий ***только*** на текущий поток: он имеет смысл только в контексте проверки того, должен ли быть остановлен окружающий код.

Итак, зачем же сбрасывается флаг?

Для более коррек


Вопр




Завершенный поток нельзя запустить.

## Volatile

В целях повышения эффективности работы, спецификации языка `Java` позволяет сохранять локальную копию переменной в каждом потоке, который ссылается на нее.
Можно считать эти 'внутрипоточные' копии переменных похожими на кэш, помогающий избежать проверки главной памяти каждый раз, когда требуется доступ к значению переменной.

Теперь пердставим, что 

```java

```

### join

Иногда бывает необходимо дождаться в одном потоке завершения другого и только после этого продолжить выполнение логики.

Снова рассмотрим пример с `ThreadExample`:

```java
class ThreadExample extends Thread {
    private final String greeting;
    private final int count;

    public ThreadExample(String greeting, int count) {
        this.greeting = greeting;
        this.count = count;
    }

    @Override
    public void run() {
        System.out.println(greeting);

        int sum = 0;
        for (int i = 0; i < count; i++) {
            sum += i;
        }

        System.out.println("Sum from " + greeting + " is:" + sum);
    }
}

public class Example {
    public static void main(String[] args) {
        new ThreadExample("Hello from 1", 1_000_000).start();
        new ThreadExample("Hello from 2", 2).start();
    }
}
```

Но что если нам надо сначала дождаться выполнения `Hello from 1`, а уже после `"Hello from 2"`?

В этом нам поможет метод `join`:

```java
public class Example {
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new ThreadExample("Hello from 1", 1_000_000);
        Thread t2 = new ThreadExample("Hello from 2", 1_000_000);
        t1.start();
        t2.start();

        t1.join();
        t2.join();

        System.out.println("Hello from MAIN");
    }
}
```

Запустим и увидим:

```java
Hello from 1
Hello from 2
Sum from Hello from 2 is:1783293664
Sum from Hello from 1 is:1783293664
Hello from MAIN
```

Видим, что оба потока стартовали и выполнялись параллельно, но **главный** поток, наш `main`, прежде чем выполнить печать `Hello from MAIN`, дождался выполнения обоих потоков.

Т.е `join` заставляет дождаться завершения потока у которого он вызван, при этом из какого потока мы делаем вызываем `join` - тот и ждет.

### sleep





## Поток-демон

Мы видим, что у потоков есть метод `setDaemon`.
Зачем?

Приложение на Java работает до тех пор, пока существует и работает хотя бы один поток не-демон.
Так вот, если у нас поток-демон, то его состояние не учитывается в решении завершать или нет приложение.
Пример &mdash; запись в лог.

## Остановка по требованию


Однако не все так просто.

Ведь есть блокирующие операции:
* `sleep`
* `join`
* `wait`

**Замечание**

При этом чтение чего-либо большого, например, из сети &mdash; это тоже
блокирующая операция, но `InterruptedException` не выкидывается при `interrupt`.
Но мы можем перед тем, как вызывать read, вызывать проверку &mdash; есть ли данные.

В этим моменты (когда, например, поток спит) &mdash; мы не проверяем флаг.
А значит, и не останавливаем поток. Пусть поток спит. В это время мы вызываем
`interrupt`. Что произойдет?

А произойдет вот что: в нашем потоке после `interrupt` возникнет исключение
(помните, что мы оборачиваем все блокирующие вызовы в `try/catch`?), мы его
перехватим (или поднимем выше), и будем уже думать, что делать (закрывать ресурсы,
выходить из потока и т.д). *Обратите внимание на `try/catch` блок!*

И такое будет работать **только** при использовании Java `interrupt`, со своим
флагом это уже не сработает &mdash; дело в реализации `interrupt`.

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

Подытоживая:

* `interrupt` &mdash; выставляет флаг прерывания потока, если он был в состоянии `BLOCKED` или `WAITING` &mdash; кинется `InterruptedException`
* `isInterrupted` &mdash; проверит флаг
* `static interrupted` &mdash; проверит состояние флага у потока и сбросит флаг. Это если мы вызываем `interrupted` дважды подряд, тогда второй вызов вернет `false`.
* обязательно проверяем состояние потока в нем, иначе не остановим его.
* не использовать `stop`.
* не использовать какие-то свои флаги, так как в состояниях `BLOCKED` или `WAITING` не дойдем до его проверки.
* поймав `InterruptedException` &mdash; думаем, что делать, хорошей идеей будет вызывать `interrupt` у текущего потока, так как мы его разбудили InterruptedException, а теперь &mdash; выставляем флаг.


## Жизненный цикл потока

Поток в `Java` может находиться в следующих состояниях: `RUNNABLE`, `WAITING`, `BLOCKED` и `TERMINATED`.

![Thread Lifecycle](../images/thread-life.png)

Сразу после старта поток находится в состоянии `RUNNABLE`, выполняется.

Поток может завершиться (может мы его прервали, может его работа была сделана), в таком случае он перейдет в состояние `TERMINATED`. Это конечное состояние, из него в другое состояние не перейти.

Помимо этого поток может из `RUNNABLE` перейти в состоянии `WAITING`, т.е. ожидания, либо может быть заблокирован, т.е. в состоянии `BLOCKED`.

В чем отли

Далее, если мы вызываем `wait` &mdash; мы переводим его в состояние `WAITING`,
откуда он может стать снова `RUNNABLE`, если вызвать `notify`, и может перейти в
состояние `TERMINATED`.

Также из состояния `RUNNABLE` мы можем попасть в `BLOCKED`, если получим lock по
монитору (lock on monitor), соответственно как только монитор освободится &mdash;
снова перейдем в `RUNNABLE`. Также можно попасть в `TERMINATED`.

И последний вариант из `RUNNABLE` мы можем попасть в `TERMINATED`, если вызовем
метод `interrupt`.


Как видно по схеме, мы из `TERMINATED` не выйдем обратно, т.е если поток
завершился &mdash; второй раз `start` уже не вызовем. Т.е надо создать объект
нового потока и запустить его заново, если нам это нужно.

Почему так? А потому, что наши потоки &mdash; это объекты. А значит, запустив
поток, мы можем состояние нашего потока как-то изменить, например, изменить ему
имя, если у нашего объекта есть поле имя. Или что-то еще, но главное &mdash; мы
можем изменить состояние объекта. После завершения потока наш объект может быть
совсем не в том виде, что при создании объекта конструктором. И если бы мы могли
заново запускать отработанный поток &mdash; мы бы запускали уже объект с
неизвестными данными. Поэтому так нельзя.

## Пул потоков

Окей, с одиночными потоками разобрались.
Теперь рассмотрим такую штуку, как пул потоков.

Пул потоков может ограничивать количество создаваемых потоков, например, у нас
есть сервис и мы хотим, чтобы с клиентами общалось не более 10 потоков, а больше
уже не надо. Тогда мы создаем пул потоков для этого, как только задача потоком
выполнилась &mdash; мы возвращаем поток обратно в пул. А после его же
переиспользуем. Т.е. пул копит некоторые задачи на исполнение, и как только
появляется свободный поток &mdash; выдаёт ему задачу.

```java
public class ThreadPoolExample {
    private ExecutorService threadPool = Executors.newFixedThreadPool(2);
    private List<Future> futureList = new ArrayList<>();

    static class Task implements Runnable {
        private String taskName;
        private int delay;

        public Task(String taskName, int delay) {
            this.delay = delay;
            this.taskName = taskName;
        }

        @Override
        public void run() throws Exception {
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

    public void exampleOfTasks() {
        for (int i = 0; i < 5; i++) {
            threadPool.submit(new Task("Thread#" + i, 5-i));
        }

        threadPool.shutdown();
    }
}
```

Также есть еще механизм, называемый Future.


## Future

Помимо интерфейса `Runnable`, который дает нам метод `public void run()`, есть
еще один интерфейс &mdash; `Callable`, он параметризован и имеет метод
`public K call()`, где `K` &mdash; тип возвращаемого значения.
Поток, созданный с `Callable`, умеет возвращать значение после завершения.
Если не параметризовать интерфейс, то `call` будет возвращать `Object`.
А после, мы можем у `Future` методом `get` получить это значение, когда поток
уже сделал всю работу. При этом надо понимать, что `get` дождется окончания потока.

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


## Race condition

Пусть у нас есть некий класс Counter, а у него поле `counter`. И метод, который
умеет увеличивать значение счетчика. И пусть мы в двух потоках увеличиваем этот
счетчик в цикле до 100.

Что мы ожидаем? Что будет 200.
Но на деле мы каждый раз при запуске будем получать разное значение. Почему?
Из-за Race condition.

Это &mdash; Критическая секция.
Это код, в котором мы обращаемся к общему ресурсу, который не должен быть
использован более чем одним потоком. Мы обращаемся к общему ресурсу из разных
потоков и можем попасть в состояние, когда он изменяется одним потоком из второго.

По сути это из-за того, что когда мы делаем инкремент, мы на уровне процессора
делаем три операции (Read-Modify-Write). Эта операция не атомарна.

Атомарные операции &mdash; это операции выполняющиеся как единое целое или вообще не выполняющиеся.

Атомарные:
* Чтение/запись примитивов (**но без поддержки long/double**)
* Чтение/запись ссылок
* Чтение/запись volatile примитивов
* Атомарные операции из java пакета (Atomic integer и т.д)

Так вот, в Java есть такое понятие как монитор &mdash; средство контроля к ресурсу.
У каждого объекта есть свой монитор. У монитора в каждый момент времени &mdash; только один поток.
Монитор по сути &mdash; это id исполняющегося потока, если у него стоит 0 &mdash; объект свободен.

Ниже мы делаем лок на мониторе текущего объекта, т.е `this`.
`Synchronized` можно передать объект для лока тоже.

```java
public class Example {
    // synchronized на this
    public synchronized void test() {
        /* work here */
    }

    // synchronized на объекте
    public void test(Object obj) {
        synchronized(obj) {
             /* work here */
        }
    }
}
```

`Static` блокируется на `.class` объекте.

Разницы между локом на объекте и на `this` почти нет, разве что мы при локе на
объекте можем не весь метод в критическую секцию отдать, а только часть.

Одна из возможных проблем &mdash; deadlock.
Пусть есть два ресурса (A и B) и два потока. Один поток захватывает ресурс A,
второй &mdash; ресурс B, теперь если первый поток обратится к B, а второй поток
&mdash; к A, они останутся в вечном ожидании, так как каждый будет ждать
освобождения ресурса, занятого другим потоком.
Для предотвращения можно использовать `tryLock`, `lock` по таймеру и т.д

Есть еще семафор &mdash; это лок, который допускает не один поток к ресурсу,
а несколько, сколько мы зададим. Как только поток ресурс освобождает &mdash;
семафор впускает следующий поток.




В чем же разница между процессом и потоком?

* Процессы обладают собственным адресным пространством.
* Потоки работают с общей памятью.
* С потоками работает именно планировщик ОС.


## Полезные ссылки

1. [Олег Шелаев — Обзор возможностей многопоточности в Java](https://www.youtube.com/watch?v=O2QwNjkBXNk)
2. [Иван Пономарёв. Лекторий ФПМИ. Java #10. Concurrency](https://www.youtube.com/watch?v=CBhy6ZTgUvY)
3. [Why Are Thread.stop, Thread.suspend, Thread.resume and Runtime.runFinalizersOnExit Deprecated?](https://docs.oracle.com/javase/1.5.0/docs/guide/misc/threadPrimitiveDeprecation.html)
4. [Amdahl's law](https://ru.wikipedia.org/wiki/%D0%97%D0%B0%D0%BA%D0%BE%D0%BD_%D0%90%D0%BC%D0%B4%D0%B0%D0%BB%D0%B0)
5. [Закон Амдала](https://medium.com/german-gorelkin/amdahls-law-79a8edb040e2)
5. [USL](https://tangowhisky37.github.io/PracticalPerformanceAnalyst/pages/spe_fundamentals/what_is_universal_scalability_law/)
6. [Concurrency vs. Parallelism — A brief view](https://medium.com/@itIsMadhavan/concurrency-vs-parallelism-a-brief-review-b337c8dac350)
7. [What Do You Do With InterruptedException?](https://www.yegor256.com/2015/10/20/interrupted-exception.html)
8. [Java. Многопоточность. Остановка потока. Обработка InterruptedException.](https://www.youtube.com/watch?v=K2QOWJp_IQU)



Устарело и лучше scheduled fixed thread pool

Есть еще третий вариант &mdash; создать `Timer`:

```java
Timer timer = new Timer();
timer.schedule(new TimeTask {
    @Override
    public void run() {/*some work*/}
}, 60);
```

Выполнение задачи в отдельном потоке, но по таймеру:

```java
package samples.concurrency;

import java.util.TimerTask;
import java.util.concurrent.TimeUnit;

public class TimerTaskExample extends TimerTask {
    private static final int COUNT = 10;
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