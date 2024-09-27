# Симметричность набора точек относительно прямой

## Условие

Дан массив точек. Требуется определить, существует ли такая вертикальная линия на плоскости, относительно которой можно разделить набор точек так, что точки будут симметричны. Под симметричностью подразумевается, что у каждой точки есть пара, которая находится на таком же удалении от вертикальной прямой (по оси Х) и в другом наборе.

### Примеры

```java
[(2, 2), (4, 2)]

Ответ: True
```

```java
[(2, 2), (3, 3), (4, 4)]

Ответ: False
```

## Решение

Задача требует того, чтобы мы сделали вывод о том, где должна находиться такая вертикальная прямая, если она существует.

Предполагается, что прямая должна находиться посередине между точками с минимальной и максимальными координатами x. А далее, нам нужно просто просмотреть все точки, и убедиться что у них есть симметричная пара.

Создадим класс, который будет представлять точку, а также можно использовать как элемент множества.

```java
public class Point2d {
    private int x;
    private int y;

    public Point2d(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() {
        return x;
    }

    public int getY() {
        return y;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }

        if (o == null || getClass() != o.getClass()) {
            return false;
        }

        Point2d point2d = (Point2d) o;
        if (x != point2d.x) {
            return false;
        }

        return y == point2d.y;
    }

    @Override
    public int hashCode() {
        int result = x;
        result = 31 * result + y;
        return result;
    }
}
```

Теперь создадим класс со статическим методом, который будет непосредственно решать нашу задачу.

```java
public class Solution {
    public static boolean lineExists(List<Point2d> points) {
        // Задаем точки с MAX_VALUE и MIN_VALUE, чтобы найти точку с минимальной и максимальной координатами по х.
        Point2d minimumPoint = new Point2d(Integer.MAX_VALUE, 0);
        Point2d maximumPoint = new Point2d(Integer.MIN_VALUE, 0);

        // Создаем множество, в которое будем добавлять точки
        var pointSet = new HashSet<Point2d>();
        for (var point : points) {
            pointSet.add(point);

            // Нашли точку с координатой по х больше текущей максимальной и обновили
            if (point.getX() > maximumPoint.getX()) {
                maximumPoint = point;
            }

            // Нашли точку с координатой по х меньше текущей минимальной и обновили
            if (point.getX() < minimumPoint.getX()) {
                minimumPoint = point;
            }
        }

        // Вертикальная прямая должна быть посередине между минимальной и максимальными координатами по х
        int linePoint = (minimumPoint.getX() + maximumPoint.getX()) / 2;
        for (var point : points) {
            // Модуль не нужен, поскольку есть две ситуации
            // Если linePoint > point.getX, значит симметричная точка должна быть справа от прямой, разность будет положительна, добавим с плюсом
            // Если linePoint < point.getX, значит симметричная точка должна быть слева от прямой, разность будет отрицательно, добавим с минусом
            var distance = linePoint - point.getX();

            // Создаем симметричную точку
            var symmetricPoint = new Point2d(linePoint + distance, point.getY());

            // Проверяем наличие симметричной точки в множестве
            // Если хоть одной такой точки нет, значит такая прямая не существует
            if (!pointSet.contains(symmetricPoint)) {
                return false;
            }
        }

        return true;
    }
}
```

Время работы: O(N)

Память: O(N)
