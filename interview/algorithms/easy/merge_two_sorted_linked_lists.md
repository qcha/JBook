# Объединить два отсортированных связных списка

Задача с [LeetCode](https://leetcode.com/problems/merge-two-sorted-lists/description/).

## Условие

Вам даны заголовки двух отсортированных односвязанных списков.

Связанный список состоит из нод вида:

```java
public class ListNode {
    int val;
    ListNode next;
    ListNode() {}
    ListNode(int val) { this.val = val; }
    ListNode(int val, ListNode next) { this.val = val; this.next = next; }
}
```

Необходимо объеденить два списка в один отсортированный список. Список должен быть создан путем сращивания узлов первых двух списков.

Верните заголовок объединенного связанного списка.

### Примеры

```text
Input: list1 = [1,2,4], list2 = [1,3,4]
Output: [1,1,2,3,4,4]
```

Или:

```text
Input: list1 = [], list2 = [0]
Output: [0]
```

## Решение

### Рекурсия

Рекурсивное решение строится на том, что в решении мы будем 'погружаться' все глубже в рекурсию, пока не получим условие выхода из рекурсии, базис.

Выходом из рекурсии послужит тот факт, что в одном из списков закончились элементы.

Шаг рекурсии же будет переход к следующему элементу.

Итак:
Если в левом списке (list1) закончились элементы - вернем правый (list2).
Если в правом закончились - вернем, соответственно левый.

Если это не так (оба списка не пусты), то смотрим какой из текущих элементов меньше: от этого зависит куда мы 'провалимся' дальше.

Если элемент list1 меньше, то следующий 'нырок' будет в `mergeTwoLists(list1.next, list2)`. Иначе: `mergeTwoLists(list2.next, list1)`.

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        if (list1 == null) {
            return list2;
        } 

        if (list2 == null) {
            return list1;
        }

        if (list1.val < list2.val) {
            list1.next = mergeTwoLists(list1.next, list2);
            
            return list1;
        } else {
            list2.next = mergeTwoLists(list2.next, list1);
            
            return list2;
        }
    }
}
```

Время работы: O(N + M), где N - размер первого списка, M - размер второго списка

Память: O(1)

### Без рекурсии

Без рекурсии необходимо завести результирующую ноду (которую и вернем): result.

Первым шагом понять какая из голов двух списков будет головой result.

После этого достаточно пройтись в цикле (пока хотя бы один из списков не станет пустым) и выстроить связный список.
В конце проверить какой из списков остался еще с элементами (после выхода из цикла) и привязать его остатки:

```java
class Solution {
    public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        if (list1 == null) {
            return list2;
        }

        if (list2 == null) {
            return list1;
        }

        ListNode result;
        if (list1.val < list2.val) {
            result = list1;
            list1 = list1.next;
        } else {
            result = list2;
            list2 = list2.next;
        }  

        ListNode current = result;
        while (list1 != null && list2 != null) {
            if (list1.val < list2.val) {
                current.next = list1;
                current = list1;
                list1 = list1.next;
            } else {
                current.next = list2;
                current = list2;
                list2 = list2.next;
            }
        }

        if (list1 != null) {
            current.next = list1;
        } else {
            current.next = list2;
        }

        return result;
    }
}
```

Время работы: O(N + M), где N - размер первого списка, M - размер второго списка

Память: O(1)
