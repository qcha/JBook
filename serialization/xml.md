## XML Serialization

### Введение

Сериализовать можно также и в файл, например, в xml.
Как можно это сделать? Есть стандартная библиотека.

### JAXB

Java Architecture for XML Binding.

Здесь мы уже можем пользоваться XML-schema(XSD)и сравнивать Java объект с XML-документом.
Также это известно как marshalling и demarshalling процесс.
JAXB может сам сгенерить XSD. Конфигурируется все с помощью annotations.

JAXB, как и говорилось выше, входит в стандартную библиотеку Java - и это еще один плюс!

* Сначала кидаем аннотацию `@XmlRootElement` на главный элемент. У нас может быть только один root элемент.
* После чего задаем accessor type - `@XmlAccessorType(XmlAccessType.____)` - тут заменить надо ___ на то, что надо, например - `XmlAccessType.NONE`.

О том, что можно подставить:

* XmlAccessType.NONE - Сохраняем только те поля, которые аннотированы.
* XmlAccessType.FILED - Сохраняем все, кроме transient и static.
* XmlAccessType.PROPERTY - Все getters/setters пары и аннотированные поля.
* XmlAccessType.PUBLIC - Все public поля и аннотированные поля.

Вроде все понятно. Двигаемся дальше.

* Кидаем аннтоацию`@XmlElement` на все поля, которые нам нужны.
Необязательно, чтобы эти поля имели getters and setters, ведь мы снова используем Reflection.

Если у нас collections - используем XmlElementWrapper аннотацию.
Смотрим пример кода. //todo
JAXB включает в себя еще кучу аннотаций, которые помогут сохранить объект как нам хочется, например, задать порядок записи в xml и т.д.
