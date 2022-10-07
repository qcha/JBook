# Чтение из файла

Задачи подобного плана показывают на то, как человек умеет работать с исключениями, видеть краевые случаи задачи и работать с ними.

## Условие

Ваш коллега пишет метод для получения строк из файла, он сделал описание метода и предоставил несколько решений:

```java
class FileUtils {
    /**
     * Чтение всех строк из файла, путь до которого передан в аргументах.
     * @param path путь до файла
     * @return список строк-содержимого файла
     */
    public List<String> readAll(String path) {
        // ...
    }
}
```

Вариант 1.

```java
    public List<String> readAll(String path) throws IOException {
        List<String> result = new ArrayList<>();
        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            String line;
            while ((line = br.readLine()) != null) {
                result.add(line);
            }
            
            return result;
        } catch (IOException e) {
            return result;
        }
    }
```

Вариант 2.

```java
    public List<String> readAll(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            List<String> result = new ArrayList<>();

            String line;
            while ((line = br.readLine()) != null) {
                result.add(line);
            }
            
            return result;
        } catch (IOException e) {
            return null;
        }
    }
```

Объясните минусы каждого решения и предоставьте свое.

## Решение

### Разбор варианта 1

Итак, разберем вариант 1:

```java
    public List<String> readAll(String path) throws IOException {
        List<String> result = new ArrayList<>();
        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            String line;
            while ((line = br.readLine()) != null) {
                result.add(line);
            }
            
            return result;
        } catch (IOException e) {
            return result;
        }
    }
```

Чем опасен этот вариант?

Если путь до файла был указан неверно, то выполнение кода вернет пустой список.
И в таком случае никак не отличить ситуацию, когда файл существует по пути и он просто пуст, от ситуации, когда файла по заданному пути нет вообще.

Точно также, если ошибка случится во время чтения файла. Невозможно будет понять: возвращенный список - это полное содержимое файла или же просто где-то произошло исключение и потому список содержит строки до состояния возникновения ошибки.

При этом, если возникла ошибка - вызывающий метод код никогда об этом не узнает и не поймет.

### Разбор варианта 2

Итак, разберем вариант 2:

```java
    public List<String> readAll(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            List<String> result = new ArrayList<>();

            String line;
            while ((line = br.readLine()) != null) {
                result.add(line);
            }
            
            return result;
        } catch (IOException e) {
            return null;
        }
    }
```

Чем опасен этот вариант?

Проблема в том, что здесь вы вынуждаете ставить проверки на `null` абсолютно везде, где вызываете метод.

Мало того, что одна забытая проверка и вы можете получить `java.lang.NullPointerException`, так еще вы довольно сильно захламляете кодовую базу.

Добавим сюда то, что теперь при возникновении ошибки чтения (или отсутствия) файла невозможно определить что являлось причиной того, что возвращен был `null`.

Вы вынуждены постоянно проверять на `null`, а при получении `null` не сможете ответить на вопрос: почему метод вернул `null`, что именно было не так.

### Свое решение

Какие краевые случаи мы здесь должны рассмотреть и обработать? Я намеренно возьму наиболее распространенные, остальные подводные камни этой задачи будут перечислены в конце.

1. Отсутствие файла по переданному пути
2. Ошибка работы с файлом (чтения, доступа)
3. Закрыть ресурс после работы

В данном решении ресурсы будут закрыты через `try-with-resources`, а ситуации 1, 2 будут делегированы вызывающему коду.

```java
    /**
     * Чтение всех строк из файла, путь до которого передан в аргументах.
     * @param path путь до файла
     * @return список строк-содержимого файла
     */
    public static List<String> readAll(String path) throws IOException {
        try (
                FileReader fr = new FileReader(path);
                BufferedReader br = new BufferedReader(fr)
        ) {
            List<String> result = new ArrayList<>();

            String line;
            while ((line = br.readLine()) != null) {
                result.add(line);
            }

            return result;
        }
    }
```

Почему делегирование исключения здесь - наиболее верное решение?

Во-первых, исключение будет содержать всю информацию об ошибке: причину, тип.
Таким образом вызывающий код сможет правильно отреагировать на вылетевшее исключение.

Во-вторых, внутри метода здесь вы не знаете какую реакцию на ошибку хочет видеть вызвающий код.

Что хочет делать вызывающий код при отсутствии файла? При ошибке во время чтения?
Вы внутри метода `readAll` не сможете ответить на этот вопрос, поэтому вы делегируете проблему до той поры, когда кто-то сможет.

Вызывающий код на каждую из ситуаций может отреагировать по разному:

```java
try {
    List<String> lines = FileUtils.readAll("file.txt");
} catch (FileNotFoundException e) {
    // реакция на то, когда файла нет 
} catch (IOException e) {
    // реакция на то, когда ошибка произошла во время чтения 
}
```

При этом, в идеале, надо было бы добавить информацию о кодировке файла: `charset`.

#### Другие решения

##### Java Core

Начиная с версии `Java 8+` в `JDK` можно воспользоваться классом `java.nio.file.Files`:

```java
    public List<String> readAll(String path) throws IOException {
        return Files.readAllLines(Paths.get(path));
    }
```

##### Сторонние библиотеки

Для чтения строк из файла существует большое количество сторонних библиотек: [Apache Commons IO](https://mvnrepository.com/artifact/commons-io/commons-io) или [Guava: Google Core Libraries For Java](https://mvnrepository.com/artifact/com.google.guava/guava).

### Что дальше

Эта задача может стать хорошим трамплином для разговора о более сложных темах: что если файл огромный (десятки гигабайт)? А в какой кодировке данные файла?
Что за метод `flush` у потока и почему мы его не вызываем? Что такое буффер у `java.io.BufferedReader` и на что он влияет?

Это позволит посмотреть на кругозор собеседуемого человека.
