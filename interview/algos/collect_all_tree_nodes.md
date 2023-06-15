# Обход дерева и сбор всех нод в коллекцию

## Условие

Дано дерево, каждая нода которого это:

```java
class TreeNode<T> {
    T value;
    List<TreeNode<T>> nodes;
}
```

Требуется написать метод, принимающий на вход корень дерева, и выдающий коллекцию всех значений нод дерева (порядок не важен).

## Решение

### Рекурсивное решение с помощью обхода в глубину

Будем считать, что терминальной нодой считается нода, внутренний список нод которой пуст.\
Создадим вспомогательный метод `fillListWithNodes`, который будет производить рекурсивное заполнение списка обходом в глубину.\
Основным методом будет `getAllNodes`, который принимает корень дерева, создает пустой список, а затем вызывает вспомогательный метод.

```java
public static <E> void fillListWithNodes(TreeNode<E> node, List<E> nodeList) {
    if (node.nodes.isEmpty()) {
        return;
    }

    nodeList.add(node.value);
    for (var childrenNode : node.nodes) {
        fillListWithNodes(childrenNode, nodeList);
    }
}

public static <E> List<E> getAllNodes(TreeNode<E> node) {
    var nodeList = new ArrayList<E>();
    fillListWithNodes(node, nodeList);
    return nodeList;
}
```
