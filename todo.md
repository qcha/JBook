### TODO List
* JUnit4 and Rule, testing
* Article about properties
* Article about Collections
* Article about charsets
* Apache Commons IOUtils
* NIO(code in code-examples)
* Split Serialization article and rewrite it
* Wrapping checked exception into RE
* Article about logging
* Overriding/overloading/hiding in java methods
* Diff between constructor and method(return type of constructor)
* interfaces in java 8 - static and default, two interfaces with same method as default - is it normal?
* Future and etc in concurrency
* Runtime.getRuntime.addShutdownHook(...)
* classpath
* Objects.equals Objects.hash and Objects class from java 7.
* System.exit(res ? 0 : 1);
* order of class initializing
* в блоке инициализации можно пользоваться не только публичным API, но и protected
```
        Map<String, String> a = new HashMap<String, String>(){{

            put("a", "A");

            put("b", "B");

        }};
```
* article enum
* в тредах и лямбдах можно использовать только final, замыкания
```
final int[] i = {0};
            csvParser.iterator().forEachRemaining(record -> {
                    final int numOfRecord = i[0]++;
                    Optional<ProducerRecord<K, SpecificRecord>> prec = createProducerRecord(record, numOfRecord);
                    if (prec.isPresent()) {
                        logger.trace("Sending record: {}", prec.get());
                        producer.send(prec.get());
                    }
                    if (i[0] % 100000 == 1) {
                        logger.info("File {} processed {} rows by {} milliseconds.", filename, i[0], System.currentTimeMillis() - start);
                    }
            });
```
* I/O, Steam-ы и BufferedReader-ы, Reader-ы
* `@Override
           public int hashCode() {
               return Objects.hash(fileName);
           }`

* stax xml dom
* Работа с json
* https://habrahabr.ru/post/260773/
* lombok
* Patterns - Порождающие
 * Abstract factory (Абстрактная фабрика)
 * Builder (Строитель)
 * Factory method (Фабричный метод)
 * Prototype (Прототип)
 * Singleton (Одиночка)
* Структурные
 * Adapter (Адаптер);
 * Bridge (Мост);
 * Composite (Компоновщик);
 * Decorator (Декоратор);
 * Facade (Фасад);
 * Flyweight (Приспособленец);
 * Proxy (Заместитель).
* Поведенческие шаблоны
 * Command (Команда);
 * Interpreter (Интерпретатор);
 * Iterator (Итератор);
 * Mediator (Посредник);
 * Memento (Хранитель);
 * Observer (Наблюдатель);
 * State (Состояние);
 * Strategy (Стратегия);
 * Template method (Шаблонный метод);
 * Visitor (Посетитель).
* Deflater
* Guava
* org.apache.commons.pool2
* Exceptions
```
первое действие происходит во время компиляции
при компиляции javac в тело метода “зашивает” кроме байт-кода самого метода таблицу с catch-блоками (edited)
то есть, в любой метод который в себе содержит try-catch конструкции будет зашита эта таблица
в каждой записи этой таблице содержится информация о каком-то catch-блоке в этом методе
примерно так
first-line-of-try-statement, last-line-of-try-statement, exception-class, catch-block-start
вот, это в байткоде
дальше - само исключение это просто объект
если вы посмотрите в байт-код - оно создается точно так же как любой другой объект (edited)
класс, вызов конструктора, передача параметров конструктору и т.д.
а вот после это исключение можно “выбросить"
если хочется
это же объект, никто не мешает вам не бросать исключение, а что-то там с ним делать как с любым другим объектом
для того чтобы “выбросить” исключение в байт-коде есть отдельная команда, throw/athrow, как-то так называется
ей в качестве “аргумента" передается объект исключения
слово “аргумент” в кавычках потому что Oracle JVM это стековая виртуальная машина, в ней нет аргументов у команд в привычном смысле
но не суть
так вот, когда команда athrow бросает исключение JVM начинает поиск в таблице catch-блоков этого метода
с учетом номера строки где было брошено исключение и типа
у вас ведь в методе может быть много try-catch блоков и каждый может обрабатывать одновременно несколько типов исключений
JVM находит в этой таблице самый подходящий catch-блок и передает туда управление
“туда” - это значит на первую инструкцию этого catch-блока
если JVM не нашла исключение в catch-таблице этого метода - она пробрасывает это исключение в метод выше по стеку
то есть, в вызывающий метод
и там делает то же самое - смотрит на таблицу catch-методов уже этого, родительского метода
нашла подходящий catch-обработчик - передает ему управление
не нашла - еще выше по стеку (edited)
и так до тех пор пока обработчик не будет найден (edited)
вот как-то так
дополнительно хочу отметить такие две вещи:
1) в байткоде НЕТ инструкции (и вообще понятия) finally-блока. Это существует только в исходниках, передачу обработчика в finally-блок после try или catch-блока делает компилятор javac
2) в байткоде нет разницы между checked/unchecked исключениями и всем похеру на то, какие исключения декларирует метод (void method() throws BLA-BLA-BLA)
Все проверки что мы обернули в try-catch вызов метода, который декларирует какое-то исключение, производятся на этапе компиляции (javac) (edited)
то есть, при вызове метода который бросает IOException (одно из самых частых, видимо), компилятор просто смотрит что в коде есть одно из двух:
- либо вызывающий метод сам помечен как throws IOException
- либо мы обернули вызов в try-catch и ловим либо IOException либо его супер-классы
```
