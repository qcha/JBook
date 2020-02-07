## CSV Serialization

### CSV формат

Наряду с записью в xml, json и существует еще запись в csv-файлы.
На мой взгляд - это простейший способ хранения информации в файлах.

По сути csv-файл - это файл, где значения разделяются с помощью так
называемого разделителя - часто этот разделитель - это запятая.
Но надо отметить, что это не обязательно должна быть запятая,
подойдет любой разделитель, например, `|`).

Иногда в начале такого файла идет так называемый `Header`, где описываются поля.

Пример такого файла:
```java
Id, name, surname, age
1, Aleksandr, Kuchuk, 25
2, Aleksandr, Luchuk, 22
```

Видим, что в начале идет `Header`, а после идут уже значения. Однако, `Header` не является обязательным.

Несмотря на то, что использование csv кажется устаревшим он до сих пор применяется.

 Например для логов биллинга платежей, также эти файлы - это основа документов
 excel.

### Как подступиться?
Разумеется, мы можем записывать и считывать данные из таких файлов несколькими
способами.

1. Самописный код

  Да, для чего-то простого, например, пару раз записать в файлик значения, такой
  подход подойдет.

  Тут можно посоветовать лишь использовать `BufferedReader` и `BufferedWriter`,
  что добавит вашему приложению производительности. Также, не забывайте про
  `try-with-resources`, если вы пишите на Java 7+.

2. Использование сторонних библиотек.

  Данный способ наиболее подходит для случаев, когда вам необходимо работать с
  файлами большого размера, файлов, которые используют `Header`-ы, вы собираетесь
  часто обращаться к ним на запись/чтение.

  Насколько я смог разобраться - самые популярные библиотеки это:
  * `OpenCSV`
  * `Apache Commons CSV`
  * `SuperCSV`


  Это три наиболее популярные библиотеки, самая новая из них - это `SuperCSV`.

### Пример использования
Класс, объект которого будем сериализовать:
```java
public class Student {
    private long id;
    private String name;
    private String surname;
    private int age;

    public Student(long id, String name, String surname, int age) {
        this.id = id;
        this.name = name;
        this.surname = surname;
        this.age = age;
    }
    /** getters **/
}
```

Далее напишем простой класс, которй записывает студента в файл.
```java
public class CSVFileWriter {
    private final BufferedWriter writer;
    private final String delimiter;

    public CSVFileWriter(File file, String delimiter) throws IOException {
        this.writer = new BufferedWriter(new FileWriter(file));
        this.delimiter = delimiter;
    }

    public void writeStudent(Student student) throws IOException {
        final StringBuilder s = new StringBuilder();

        s.append(student.getId());
        s.append(delimiter);
        s.append(student.getName());
        s.append(delimiter);
        s.append(student.getSurname());
        s.append(delimiter);
        s.append(student.getAge());

        final String line = s.toString();
        writer.write(line);
    }

    public void close() throws IOException {
      writer.close();
    }
}
```
Здесь мы не использовали `Header`, а просто сделали из объекта класса `Student`
 строку, разделив поля с помощью `delimiter`.

Отметим, что мы можем также сделать еще и `flush()` в конце, чтобы гарантировано
протолкнуть нашего студента в файл.
