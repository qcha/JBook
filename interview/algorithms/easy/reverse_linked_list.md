# Reverse Linked List

Задача с [LeetCode](https://leetcode.com/problems/reverse-linked-list/description/).

## Условие

Пусть определен односвязный список:

```java
public class ListNode {
    int val;
    ListNode next;

    ListNode() {
    }

    ListNode(int val) {
        this.val = val;
    }

    ListNode(int val, ListNode next) {
        this.val = val;
        this.next = next;
    }
}
```

Напишите метод, который вернет развернутый список.

### Примеры

```text
Input: head = [1, 2, 2, 1]
Output: [1, 2, 2,1]

Input: head = [1,2]
Output: [2, 1]
```

## Решение

Список у нас односвязный:

```text
head
 |
[1] -> [2] -> [3] -> null
```

Заведем указатель на 'предыдущий' элемент и занулим head.next (мы же разворачиваем список и null будет с другой стороны):

```text
        head
         |
null <- [1] -> [2] -> [3] -> null
                |
               prev
```

Теперь просто перекинем ссылки: prev.next - будет head, prev.next станет уже следующий элемент, а head станет prev:

```text
               head
                |
null <- [1] <- [2] -> [3] -> null
                       |
                      prev
```

Повторим это и получим развернутый список:

```text
                      head
                       |
null <- [1] <- [2] <- [3]    null
                              |
                             prev
```

Для того, чтобы 'перекинуть' ссылки друг на друга нужна будет tmp ссылка. По сути это как с swap-ом чисел.

```java
    public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }

        ListNode prev = head.next;
        head.next = null;

        while (prev != null) {
            ListNode tmp = prev.next;
            prev.next = head;
            head = prev;
            prev = tmp;
        }

        return head;
    }
```

### Асимптотика решения

Время работы: O(N), где N - это количество элементов в связном списке

Память: O(1)
