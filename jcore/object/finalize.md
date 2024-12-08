# java.lang.Object#finalize

- [java.lang.Object#finalize](#javalangobjectfinalize)
    - [Введение](#введение)
    - [Принцип работы](#принцип-работы)
    - [Почему finalize не используется](#почему-finalize-не-используется)
    - [Неожиданные встречи с finalize](#неожиданные-встречи-с-finalize)
    - [Заключение](#заключение)
    - [Полезные ссылки](#полезные-ссылки)

## Введение

Из названия метода можем предположить его область ответственности. Итак, `finalize` - это завершить, заканчивать, придавать окончательную форму.

Что может завершать/заканчивать каждый объект? Во время работы объект может захватить какие-то ресурсы, например, файловый дескриптор. Так вот, когда объект уже будет признан не используемым и уничтожаться [Garbage Collector](../garbage_collector.md) такие ресурсы должны быть освобождены.

Это и есть та самая 'финализируемость': когда перед своим уничтожением объект освободит все захваченное.

Объявление метода выглядит как:

```java
    /**
     * Called by the garbage collector on an object when garbage collection
     * determines that there are no more references to the object.
     * A subclass overrides the {@code finalize} method to dispose of
     * system resources or to perform other cleanup.
     * <p>
     * The general contract of {@code finalize} is that it is invoked
     * if and when the Java virtual
     * machine has determined that there is no longer any
     * means by which this object can be accessed by any thread that has
     * not yet died, except as a result of an action taken by the
     * finalization of some other object or class which is ready to be
     * finalized. The {@code finalize} method may take any action, including
     * making this object available again to other threads; the usual purpose
     * of {@code finalize}, however, is to perform cleanup actions before
     * the object is irrevocably discarded. For example, the finalize method
     * for an object that represents an input/output connection might perform
     * explicit I/O transactions to break the connection before the object is
     * permanently discarded.
     * <p>
     * The {@code finalize} method of class {@code Object} performs no
     * special action; it simply returns normally. Subclasses of
     * {@code Object} may override this definition.
     * <p>
     * The Java programming language does not guarantee which thread will
     * invoke the {@code finalize} method for any given object. It is
     * guaranteed, however, that the thread that invokes finalize will not
     * be holding any user-visible synchronization locks when finalize is
     * invoked. If an uncaught exception is thrown by the finalize method,
     * the exception is ignored and finalization of that object terminates.
     * <p>
     * After the {@code finalize} method has been invoked for an object, no
     * further action is taken until the Java virtual machine has again
     * determined that there is no longer any means by which this object can
     * be accessed by any thread that has not yet died, including possible
     * actions by other objects or classes which are ready to be finalized,
     * at which point the object may be discarded.
     * <p>
     * The {@code finalize} method is never invoked more than once by a Java
     * virtual machine for any given object.
     * <p>
     * Any exception thrown by the {@code finalize} method causes
     * the finalization of this object to be halted, but is otherwise
     * ignored.
     *
     * @apiNote
     * Classes that embed non-heap resources have many options
     * for cleanup of those resources. The class must ensure that the
     * lifetime of each instance is longer than that of any resource it embeds.
     * {@link java.lang.ref.Reference#reachabilityFence} can be used to ensure that
     * objects remain reachable while resources embedded in the object are in use.
     * <p>
     * A subclass should avoid overriding the {@code finalize} method
     * unless the subclass embeds non-heap resources that must be cleaned up
     * before the instance is collected.
     * Finalizer invocations are not automatically chained, unlike constructors.
     * If a subclass overrides {@code finalize} it must invoke the superclass
     * finalizer explicitly.
     * To guard against exceptions prematurely terminating the finalize chain,
     * the subclass should use a {@code try-finally} block to ensure
     * {@code super.finalize()} is always invoked. For example,
     * <pre>{@code      @Override
     *     protected void finalize() throws Throwable {
     *         try {
     *             ... // cleanup subclass state
     *         } finally {
     *             super.finalize();
     *         }
     *     }
     * }</pre>
     *
     * @deprecated The finalization mechanism is inherently problematic.
     * Finalization can lead to performance issues, deadlocks, and hangs.
     * Errors in finalizers can lead to resource leaks; there is no way to cancel
     * finalization if it is no longer necessary; and no ordering is specified
     * among calls to {@code finalize} methods of different objects.
     * Furthermore, there are no guarantees regarding the timing of finalization.
     * The {@code finalize} method might be called on a finalizable object
     * only after an indefinite delay, if at all.
     *
     * Classes whose instances hold non-heap resources should provide a method
     * to enable explicit release of those resources, and they should also
     * implement {@link AutoCloseable} if appropriate.
     * The {@link java.lang.ref.Cleaner} and {@link java.lang.ref.PhantomReference}
     * provide more flexible and efficient ways to release resources when an object
     * becomes unreachable.
     *
     * @throws Throwable the {@code Exception} raised by this method
     * @see java.lang.ref.WeakReference
     * @see java.lang.ref.PhantomReference
     * @jls 12.6 Finalization of Class Instances
     */
    @Deprecated(since="9")
    protected void finalize() throws Throwable { }
```

Данный метод вызывается `Garbage Collector` (далее - просто `GC`) тогда, когда `GC` полагает, что объект более не нужен, никем не используется и его пора утилизировать.

На первый взгляд может показаться, что это отличный метод в который удобно поместить закрытие ресурсов и все, что нужно освобождать/удалять при уничтожении класса.

Однако, как мы можем видеть из описания, метод помечен как `@Deprecated(since="9")`, т.е устаревший и не рекомендуемый к использованию.

Чтобы понять почему так получилось и отчего данный метод не рекомендуется к использованию надо понять как он работает и кем, а главное в какой момент вызывается.

## Принцип работы

Итак, предположим, что существует некоторый объект, в котором переопределен метод `finalize`.

Метод `finalize` не вызывается `GC` напрямую, а добавляет объекты в список, вызывая `java.lang.ref.Finalizer.register(Object)`.

Объект класса `java.lang.ref.Finalizer` представляет собой ссылку на объект, для которого надо вызвать `finalize`, и хранит ссылки на следующий и предыдущий `Finalizer`, формируя двусвязный список.

Вызов `finalize` происходит в отдельном потоке `java.lang.ref.Finalizer.FinalizerThread`, вызовы `finalize` идут последовательно так, как добавлялись (но как будут вызваны методы для группы объектов не известно, так как кто в каком порядке из объектов добавился - неизвестно).

После завершения работы метода `java.lang.ref.Finalizer` на объекте будет удален из двусвязного списка финализаторов.

Почему так сделано?

Для защиты от некорректно реализованных методов `finalize`. Представьте что было бы, если `GC` напрямую вызывал бы такие методы? А если бы в таком методе было бы что-то подобное:

```java
while(true) {
    Thread.sleep(1000);
}
```

Именно поэтому если в `finalize` методе произойдет блокировка (например, deadlock), то `GC` не перестанет работать.

Если `finalize` зависает, то зависнет именно `FinalizerThread`, при этом, если класс имеет пустой метод `finalize`, то `JVM` данный метод легко оптимизирует (удаляет), поэтому объекты такого класса будут и дальше удаляться, если же метод не пуст, то добавляются в список и ждут, пока блокровка не снимется или приложение не упадет окончательно (или само приложение не завершит работу самостоятельно).

Из такого подхода следуют некоторые проблемы с производительностью работы: при первом вызове сборщика мусора объекты с `finalize` не удаляются, а помещаются в очередь на финализацию, после чего финализируются и только при втором вызове сборщика мусора уже очищаются/удаляются.

## Почему finalize не используется

Прежде всего потому, что у нас нет **НИКАКИХ** гарантий вызовется ли этот метод вообще.

Как это может произойти?

Во-первых, из-за человеческого фактора. У нас в коде может существовать некоторая забытая ссылка на объект: мы про ссылку ничего не знаем (забыли), но она существует и `GC` не будет удалять этот объект, а значит и ресурсы не будут освобождены. И если один 'потерянный' не используемый объект - это не критично, то проблема не освобожденных ресурсов может иметь куда более разрушительные последствия: от `deadlock`-ов до влияния на бизнес логику.

Во-вторых, метод также не будет вызван, если приложение будет остановлено или экстренно завершено. Например, будет произведен вызов `System.exit()` или приложение и вовсе закрашится.

Еще одним камнем в огород использования `finalize` является то, что сейчас алгоритмы сборки мусора изменились и она может происходить редко.

Что в свою очередь влияет на то, что большую часть времени ресурсы не будут освобождены, так как `finalize` будет вызываться редко.

Важно и то, что нет никаких гарантий о порядке вызова методов финализации для группы объектов, поэтому возможны ситуация с `deadlock`-ами.

Чрезмерное использование `finalize` влияет на производительность и даже быть причиной падения программы:

```text
Tardy finalization is not just a theoretical problem. Providing a finalizer for a class can, under rare conditions, arbitrarily delay reclamation of its instances.
A colleague debugged a long-running GUI application that was mysteriously dying with an OutOfMemoryError.

Analysis revealed that at the time of its death, the application had thousands of graphics objects on its finalizer queue just waiting to be finalized and reclaimed. Unfortunately, the finalizer thread was running at a lower priority than another application thread, so objects weren’t getting finalized at the rate they became eligible for finalization.

The language specification makes no guarantees as to which thread will execute finalizers, so there is no portable way to prevent this sort of problem other than to refrain from using finalizers.
```

При этом в `finalize` можно добавить ссылку на текущий объект и тем самым 'воскресить' его!

```java
public class Main {
    public static List<Object> objects = new ArrayList<>();

    public static void add(Object object) {
        objects.add(object);
    }
}

public class Point {
    @Override
    protected void finalize() throws Throwable {
        Main.add(this);
    }
}
```

Делать это, разумеется, это ни в коем случае **не следует**!

Исходя из всего вышеперечисленного, полагаться на работу `finalize` не стоит, это опасно и потому он помечен как `Depreacated` с версии `Java 9+`.

## Неожиданные встречи с finalize

Если вы считаете, что не работая у себя в коде с `finalize` вы с ним никогда не столкнетесь, то вот вам пример (из `Java 8`) из `java.io.FileInputStream`:

```java
/**
 * Ensures that the <code>close</code> method of this file input stream is
 * called when there are no more references to it.
 *
 * @exception  IOException  if an I/O error occurs.
 * @see        java.io.FileInputStream#close()
 */
protected void finalize() throws IOException {
    if ((fd != null) &&  (fd != FileDescriptor.in)) {
        /* if fd is shared, the references in FileDescriptor
         * will ensure that finalizer is only called when
         * safe to do so. All references using the fd have
         * become unreachable. We can call close()
         */
        close();
    }
}
```

Т.е. даже если вы явно вызываете `close` у себя в коде, то метод и финализация все равно будут вызваны, а значит это повлияет на производительность!

Аналогично сделано и в `java.io.FileOutputStream`, подробнее об [этом](https://issues.jenkins.io/browse/JENKINS-42934?page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel&showAll=true).

Решение:

* Использовать статические фабричные методы у `java.nio.file.Files`: `Files.newInputStream` и `Files.newOutputStream`
* Перейти на `Java 10` или выше, где данный метод у этих классов пустой и помечен как `deprecated` (а новых версиях уже удален).

Поэтому внимательно следите за классами, которые вы используете и тем, что внутри этих классов.

## Заключение

Никогда не используйте `finalize`, предпочитая ему явную очистку ресурсов: как через явные методы, так и с помощью `try-with-resources`.

Либо используйте `java.lang.ref.Cleaner`, но это уже тема для отдельного обсуждения!

## Полезные ссылки

1. [Effective Java, 2nd Edition. Item 7: Avoid finalizers](https://www.amazon.com/dp/0321356683)
2. [Тагир Валеев: finalize и Finalizer](https://habr.com/ru/post/144544/)
3. [Как я пытался понять смысл метода finalize](https://habr.com/ru/post/183344/)
4. [Why finalize is deprecated](https://stackoverflow.com/questions/56139760/why-is-the-finalize-method-deprecated-in-java-9)
5. [Вы всё ещё используете finalize()? Тогда мы идём к вам](https://www.youtube.com/watch?v=K5IctLPem0c)
