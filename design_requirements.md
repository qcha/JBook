# Требования к оформлению

Абсолютно запрещено употребление ругательств, мат.
Запрещены упоминания политических тем, а также оскорбления человека и животных по любым признакам.

## Название файла

Название файла, содержащего заметку, пишется на английском языке. В имени должна быть информация о том, что это за заметка и про какую тему будет идти речь.

Если в названии более одного слова, то вместо пробела слова связываются символом нижнего подчеркивания `_`.

Например:

`abstract_class.md`

Заметки логически группируются по директориям.
Принцип группирования тот же, что и в группировке классов по пакетам.

Сначала выбираете область, к которой относится ваша статья, например, `web` или `jcore`.

Возвращаясь к нашему примеру с `abstract_class.md`: так как эта заметка относится к абстрактным классам в Java, то логично положить ее будет в `jcore`. Однако, можно более точно описать область, к которой она относится - это ООП в Java. Значит, логично будет поместить ее в `jcore/oop`.

Памятка:

| Директория        | Описание                                                 |
|-------------------|----------------------------------------------------------|
| algorithms        | Алгоритмы, структуры данных                              |
| build             | Все, что касается сборки и ci/cd                         |
| course            | Курсы (HSE, MIPT, Zero to Hero)                          |
| db                | Секция баз данных                                        |
| frameworks        | Фреймворки                                               |
| images            | Хранилище картинок и ресурсов для статей                 |
| interview         | Задачи с собеседований                                   |
| jcore             | Основные темы Java                                       |
| logging           | Логирование                                              |
| patterns          | Паттерны: архитектурные, программирования (GoF), проекта |
| testing           | Секция про тестирование                                  |
| web               | Веб и все, что с ним связано                             |
| other             | Все, что не вошло в другие разделы                       |
| kafka             | Статьи про кафку                                         |
| zookeeper         | Статьи про zookeeper                                     |
| serialization     | Статьи про сериализацию/десериализацию                   |

Обращайте внимание на подпапки в директориях: это поможет детальнее подойти к выбору места для заметки.

## Оформление заметки

Шаблон для оформления находится здесь: [Pull Request Template](./PULL_REQUEST_TEMPLATE.md)

Ниже даны пояснения по разделам.

### Заголовок

Каждая заметка должна иметь заголовок, оформленный в виде `h1`.
Заголовок должен коротко передавать суть заметки и содержать не более 3-4 слов.

Крайне нежелательно в заголовок выносить какие-то цифры, менять регистр как вздумается и использование табуляции.

После `h1` заголовка статьи следует блок `Введение`, чья задача - это ввести читателя в рассматриваемую тему или проблему. Либо добавить деталей о том, что за материал будет дальше изложен.

До введения следует расположить блок `ToC`: Table of Contents. Это блок оглавления и навигации.

### Основная часть

После заголовка начинается основная часть статьи, каждый логический блок выделяется через `h2`-заголовок.

Каждый блок может также дробиться, выделяя новые блоки через `h3` и так далее.

Каждый блок отделется переносом строки до и после.

Значимые и ключевые слова выделяются с помощью ``.
Если в статье используется какой-то класс из `JDK`, либо из библиотеки, то имя класса используется полное, с указанием имени пакета и библиотеки.

Желательно избегать сокращений, либо в начале явно читателю показать, что вы сокращаете некоторое имя и далее будет использоваться сокращенная форма.

Примером может служить `java.lang.NullPointerException`, где вы можете указать, что здесь и далее будет использовано сокращение `NPE`.

Если рассматриваемый материал пересекается или ссылается на отдельную заметку будет не лишним дать на неё ссылку.

Необходимо делать скидку на то, что уровень читателя может разниться. Поэтому избегайте переосложнений и сленга.

Старайтесь давать примеры кода и оформлять их в виде:

\```java
// some code
\```.

### Заключение

В качестве заключения пишется краткий обзор рассмотренного материала, с выделением наиболее занчимых мест.

### Полезные ссылки

После заключения обязательно необходимо указать блок полезных ссылок, которые помогут либо в дальнейшем, более глубоком изучении материала, либо были использованы для основы статьи.

### Отдельная благодарность

В самом конце можно выделить блок для вынесения благодарностей для тех, кто помог в создании статьи: делал ревью, консультировал, предлагал материалы и т.д.

### Изображения

Изображения хранятся в директории `images` репозитория. Под каждую статью выделяется своя директория. Структура должна повторять то, как расположена статья.

Например, если ваша статья - это `jcore/oop/abstract_class.md`, то изображения для нее должны располагаться в директории `images/jcore/oop/abstract_class`.

Изображения, используемые в заметках и статьях, хранятся в формате `png` или `jpg`.

Я использую [draw.io](https://www.draw.io/) в качестве редактора.

### В качестве примера

В качесте ориентира того, как я вижу правильно оформленную статью, можно использовать статью по [http 1.1](./web/http/http_11.md)

### Шаблон

```text
# Название

## Введение

## Основная часть

## Заключение

## Полезные ссылки

## Отдельная благодарность
```

## Оформление interview заметки

Это заметки, описывающие вопросы и [задачи с интервью](./interview/).

Раздел interview состоит из:

| Директория        | Описание                                                 |
|-------------------|----------------------------------------------------------|
| algorithms        | Задачи на алгоритмы и leetcode                           |
| code_review       | Задачи на код ревью                                      |
| design_interview  | Задачи по System Design                                  |
| live_coding       | Задачи на написание кода                                 |
| questions         | Вопросы с собеседований                                  |

Вопросы и задачи в каждом разделе делятся по сложности.

| Сложность         | Описание                                                                                              |
|-------------------|-------------------------------------------------------------------------------------------------------|
| beginner          | Задачи и вопросы для самых начинающих, стажеров                                                       |
| easy              | Задачи и вопросы не требующие глубоких знаний, базовый уровень                                        |
| medium            | Задачи и вопросы, требующие более глубокого обдумывания, проработки краевых случаев                   |
| hard              | Сложные и нестандартные задачи и вопросы                                                              |

В каждом разделе есть intro.md файл с описанием раздела и списком задач, там же описывается принцип разделения по сложности.

### Шаблон interview задачи

Шаблон:

```text
# Заголовок

## Условие

### Примеры

## Решение

### Асимптотика решения (если это задача на алгоритмы)

Время: O(_)

Память: O(_)

```

Пример: [move_zeroes.md](./interview/algorithms/beginner/move_zeroes.md)

Не забудьте добавить заметку в intro.md раздела, в случае с примером выше - это [intro.md](./interview/algorithms/intro.md).
