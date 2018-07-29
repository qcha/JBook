## java.lang.Object и все-все-все
### Введение
`Java` - это объектно-ориентированный язык программирования.
В этом ЯП мы оперируем классами и объектами.

И класс `java.lang.Object` является корнем иерархии классов в `Java`.
Каждый класс, **включая массивы**, является потомком `java.lang.Object` - прямым или косвенным.


Это значит, что все классы в `Java` так или иначе наследуют методы `java.lang.Object`.
Давайте посмотрим, какие методы есть, благодаря этому, в каждом классе.

### Основные методы
Основные методы, на момент `Java 8`:
* [toString](./ToString.md)
* [clone](./Clone.md)
* [hashCode](./Hashcode.md)
* [equals](./Equals.md)
* [finalize](./finalize.md)
* [getClass](./getClass.md)
* `wait(long timeout)`
* `notify()`
* `notifyAll()`
