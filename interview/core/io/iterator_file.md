# Итератор на файл

Задача на то, как человек понимает паттерн итератор, умеет работать с исключениями, видеть краевые случаи и работать с ними.

Неплохой пример на то, чтобы посмотреть: понимает ли собеседуемый зачем и как применяется `AutoCloseable`, это может быть как продолжение ответа на вопрос про `try-with-resources` и использование в работе дать.

Это также может быть проверкой того, насколько человек читает контракт интерфейса: зачастую забывают про `java.util.NoSuchElementException`, когда реализуют `next`, хотя в контракте явно сказано:

```java
    /**
     * Returns the next element in the iteration.
     *
     * @return the next element in the iteration
     * @throws NoSuchElementException if the iteration has no more elements
     */
    E next();
```

## Условие

Напишите итератор по строкам на файл. Файл текстовый.

## Решение

```java
public class FileIterator implements Iterator<String>, AutoCloseable {
    private final BufferedReader br;
    private String nextLine;

    public FileIterator(String path, Charset charset) throws IOException {
        this(new File(path), charset);
    }

    public FileIterator(File file, Charset charset) throws IOException {
        this.br = new BufferedReader(new FileReader(file, charset));
        this.nextLine = br.readLine();
    }


    @Override
    public void close() throws IOException {
        br.close();
    }

    @Override
    public boolean hasNext() {
        return nextLine != null;
    }

    @Override
    public String next() {
        if (nextLine == null) {
            throw new NoSuchElementException("No more elements");
        }
        
        try {
            String line = nextLine;
            this.nextLine = br.readLine();

            return line;
        } catch (IOException e) {
            throw new UncheckedIOException(e);
        }
    }
}
```

Сразу делаем итератор закрываемым с помощью `try-with-resources` благодаря реализации `java.lang.AutoCloseable`.
Не надо забывать про кодировку, так как в задании явно сказано: файл текстовый.

Некоторые предлагают решение, когда сначала считывается все содержимое файла в список и далее у списка (как и у любой коллекции) берется итератор:

```java
Files.readAllLines(path, charset).iterator();
```

Здесь можно поговорить на тему минусов подобного решения, например, при работе с большими файлами.

Ну и нельзя забывать то, что `java.util.Scanner` и есть в своем роде итератор на файл! Поэтому это также хороший ответ на задачу.
