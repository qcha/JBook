### TODO List
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
* class A extends B, and both classes have static method, which method we invoke?
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
* Продюсер фактори в проект в утилы
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
