# Бинарное дерево поиска

## Определение 

**Бинарное дерево поиска** (от англ. _binary search tree_) - тип дерева, для которого выполняется следующие свойства:
* У каждой вершины не более двух детей
* Все вершины обладают ключами, у которых определена операция сравнения (целые числа, строки и т.п.)
* У всех вершин левого поддерева вершины `node` ключи не больше, чем ключ вершины `node`
* У всех вершин правого поддерева вершниы `node` ключи больше, чем ключ вершниы `node`
* Оба поддерева (от корня - левое и правое) являются двоичными деревьями поиска

Также существуют более общие деревья поиска, в которых количество детей может быть больше двух (см. [B-tree](https://en.wikipedia.org/wiki/B-tree)).

## Зачем

Такая структура данных позволяет осуществлять быстрый поиск, вставку, удаление элементов. Если в случае с динамическим массивом поиск, вставка, удаление имеют асимптотическую сложность O(n), то у бинарного дерева - O(logn).

Важно заметить, что такое дерево может превратиться в "бамбук", то есть выродиться в постоянно возрастающую или убывающую последовательность вершин. В таком случае, поиск и вставка будут иметь сложность O(n). В подобных случаях применяют правила балансировки. 

Сбалансированные деревья на основе бинарного дерева поиска:
* [AVL-tree](https://en.wikipedia.org/wiki/AVL_tree)
* [RB-tree (красно-черное дерево)](https://en.wikipedia.org/wiki/Red–black_tree)

## Реализация

Обычно такие деревья представляют с помощью класса, описывающего вершину со следующими полями:
* value - ключ вершины
* left - левый ребенок
* right - правый ребенок

```java
public class TreeNode<T> { 
    T value;
    TreeNode<T> left;
    TreeNode<T> right;

    public TreeNode(T value) {
        this.value = value;
        this.left = null;
        this.right = null;
    }

    public TreeNode(T value, TreeNode<T> left, TreeNode<T> right) {
        this.value = value;
        this.left = left;
        this.right = right;
    }
}
```

Напоминаем, что тип `T` должен иметь корректно определенную операцию сравнения.

На этой основе можно легко реализовать рекурсивные функции поиска и вставки элементов.

```java
public static <T extends Comparable<T>> boolean find(TreeNode<T> root, T valueToFind) {
    // Мы пришли в терминальную вершину (лист), то есть не нашли необходимое значение в дереве
    if (root == null) {
        return false;
    }

    // Нашли необходимое значение
    if (root.value.compareTo(valueToFind) == 0) {
        return true;
    }

    // Если значение, которое мы ищем, больше ключа в текущей вершине, то мы должны пойти в правое поддерево
    if (root.value.compareTo(valueToFind) < 0) {
        return find(root.right, valueToFind);
    }

    // Оставшийся случай, это то, что значение которое мы ищем, меньше ключа в текущей вершине
    // Значит нужно пойти в левое поддерево
    return find(root.left, valueToFind);
}

public static <T extends Comparable<T>> TreeNode<T> insert(TreeNode<T> root, T valueToInsert) {
    // Пришли в терминальную вершину (лист), значит нужно создать новую вершину
    if (root == null) {
        return new TreeNode<>(valueToInsert);
    }
    
    // Если значение, которое мы хотим вставить, больше ключа в текущей вершине, 
    // нужно рекурсивно просмотреть вершины в правом поддереве

    // Если значение, которое мы хотим вставить, меньше ключа в текущей вершине, 
    // нужно рекурсивно просмотреть вершины в левом поддереве
    
    // Если значение уже есть в дереве, то мы ничего не будем добавлять, поскольку в дереве не должно быть дупликатов ключей
    if (root.value.compareTo(valueToInsert) < 0) {
        root.right = insert(root.right, valueToInsert);
    } else if (root.value.compareTo(valueToInsert) > 0) {
        root.left = insert(root.left, valueToInsert);
    }
    
    return root;
}
```

