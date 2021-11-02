# java.lang.Object#finalize

## Введение

Данный метод вызывается `Garbage Collector` (далее - просто `GC`) тогда, когда `GC` полагает, что объект более не нужен, никем не используется и его пора утилизировать.

> Подробнее про то, как работает [GC](../other/garbage_collector.md).

Объявление метода в `Java 8` выглядит следующим образом:

```java
/**
 * Called by the virtual machine when there are no longer any (non-weak)
 * references to the receiver. Subclasses can use this facility to
 * guarantee that any associated resources are cleaned up before
 * the receiver is garbage collected. Uncaught exceptions which are
 * thrown during the running of the method cause it to terminate
 * immediately, but are otherwise ignored.
 * <p>
 * Note: The virtual machine assumes that the implementation
 * in class Object is empty.
 *
 * @exception	Throwable
 *					The virtual machine ignores any exceptions
 *					which are thrown during finalization.
 *
 */
protected void finalize () throws Throwable {
}
```

На первый взгляд, может показаться, что это отличный метод, в который удобно поместить закрытие ресурсов и все, что нужно освобождать/удалять при уничтожении класса.
Но это впечатление обманчиво. Более того, использование `finalize` подвержено ошибкам, а в версиях `Java 9+` метод и вовсе помечен как `@Deprecated(since="9")`, т.е устаревший и не рекомендуемый к использованию. 

Почему?

## Почему finalize не используется

Прежде всего потому что у нас нет **НИКАКИХ** гарантий, что этот метод вообще вызовется.
Это легко может произойти, если у нас где-то в коде есть забытая ссылка на объект: мы про ссылку ничего не знаем, но она существует и `GC` не будет удалять этот объект.
Данный метод также вызван не будет, например, если приложение будет остановлено или экстренно завершено.

Еще одним камнем в огород использования `finalize` является то, что сейчас алгоритмы сборки мусора изменились и она может происходить редко.
А это значит, что большую часть времени ресурсы не будут освобождены.
А представьте, что эти ресурсы - это подключения к серверу базы данных?
В таком случае это будет негативно влиять на производительность не только вашего приложения, но и сервера базы данных.

Добавим еще то, что нет никаких гарантий в каком порядке будут вызваны методы финализации для группы объектов, поэтому возможны ситуация с `deadlock`-ом.

Также, чрезмерное использование `finalize` влияет на производительность и даже быть причиной падения программы:

```
Tardy finalization is not just a theoretical problem. Providing a finalizer for a class can, under rare conditions, arbitrarily delay reclamation of its instances.
A colleague debugged a long-running GUI application that was mysteriously dying with an OutOfMemoryError.

Analysis revealed that at the time of its death, the application had thousands of graphics objects on its finalizer queue just waiting to be finalized and reclaimed. Unfortunately, the finalizer thread was running at a lower priority than another application thread, so objects weren’t getting finalized at the rate they became eligible for finalization.

The language specification makes no guarantees as to which thread will execute finalizers, so there is no portable way to prevent this sort of problem other than to refrain from using finalizers.
```

## Как работает

Метод `finalize` не вызывается `GC` напрямую, а добавляет объекты в список, вызывая `java.lang.ref.Finalizer.register(Object)`.

Объект класса `java.lang.ref.Finalizer` представляет собой ссылку на объект, для которого надо вызвать `finalize`, и хранит ссылки на следующий и предыдущий `Finalizer`, формируя двусвязный список.

Вызов `finalize` происходит в отдельном потоке `java.lang.ref.Finalizer.FinalizerThread`, вызовы `finalize` идут последовательно так, как добавлялись (но как будут вызваны методы для группы объектов не известно, так как кто в каком порядке из объектов добавился - неизвестно). 
После вызова метода и его отработки, `java.lang.ref.Finalizer` на объекте будет удален из двусвязного списка финализаторов.

Поэтому, если `finalize` зависает, то зависнет именно `FinalizerThread`, если класс имеет пустой метод `finalize`, то `JVM` данный метод легко оптимизирует (удаляет), поэтому объекты такого класса будут и дальше удаляться, если же метод не пуст, то добавляются в список и ждут, пока все не отвиснет или не упадет окончательно, либо мы вообще не завершим наше приложение.

Именно поэтому, если в `finalize` методе произойдет блокировка (например, deadlock), то `GC` не перестанет работать.

Отсюда же и проблемы с производительностью: при первом вызове сборщика мусора, объекты с `finalize` не удаляются, а помещаются в очередь на финализацию, после чего финализируются и только при втором вызове сборщика мусора уже очищаются.

Представим, что в `finalize` мы написали какую-то долгую и большую операцию и получим ту самую проблему с `OutOfMemoryError`, описанную выше!

При этом мы в `finalize` можем и добавить ссылку на текущий объект куда-то и тем самым 'воскресить' его! Делать это, разумеется, ни в коем случае не следует!

## Опасности

При этом, если вы считаете, что если вы не пользуетесь `finalize` и вы с этим никогда не столкнетесь, то вот вам пример из `java.io.FileInputStream`:

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

И даже, если вы явно вызываете `close`, то метод и финализация все равно будут вызваны, а значит и повлияет на производительность!

Аналогично сделано и в `java.io.FileOutputStream`, подробнее об [этом](https://issues.jenkins.io/browse/JENKINS-42934?page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel&showAll=true).

Решение: 

* Использовать статические фабричные методы у `java.nio.file.Files`: `Files.newInputStream` и `Files.newOutputStream`
* Перейти на `Java 10+`, где данный метод у этих классов пустой и помечен как `deprecated`.

Поэтому внимательно следите за классами, которые вы используете и тем, что внутри этих классов.

## Заключение

Никогда не используйте `finalize`, предпочитая ему явную очистку ресурсов, либо `java.lang.ref.Cleaner`.

## Полезные ссылки

1. [Effective Java, 2nd Edition. Item 7: Avoid finalizers](https://www.informit.com/articles/article.aspx?p=1216151&seqNum=7)
2. [Тагир Валеев: finalize и Finalizer](https://habr.com/ru/post/144544/)
3. [Как я пытался понять смысл метода finalize](https://habr.com/ru/post/183344/)
4. [Why finalize is deprecated](https://stackoverflow.com/questions/56139760/why-is-the-finalize-method-deprecated-in-java-9)
5. [Вы всё ещё используете finalize()? Тогда мы идём к вам](https://www.youtube.com/watch?v=K5IctLPem0c)