# Palindrome Linked List

Задача с [LeetCode](https://leetcode.com/problems/palindrome-linked-list/description/).

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

Напишите метод, определяющий, является ли этот список палиндромом.

Палиндром — это последовательность, которая одинаково читается слева направо и слева направо.

### Примеры

```text
Input: head = [1,2,2,1]
Output: true

Input: head = [1,2]
Output: false
```

## Решение

Основной проблемой в задаче является то, что у нас односвязанный список, который не умеет двигаться 'назад'. Что осложняет понимание палиндром ли это.

### Решение в лоб

Самым простым решением, что называется 'в лоб', является решение, где мы переводим наш список в структуру, у которой есть возможность двигаться по элементам как вперед, так и назад.

Например, список. Но можно применить любую другую структуру (очередь, массив и так далее).

```java
 public boolean isPalindrome(ListNode head) {
     List<Integer> elements = new ArrayList<>();
     ListNode curr = head;
     while (curr != null) {
         elements.add(curr.val);
         curr = curr.next;
     }

     int i = 0;
     int k = elements.size() - 1;

     while (i < k) {
         if (!elements.get(i).equals(elements.get(k))) {
             return false;
         }

         i++;
         k--;
     }

     return true;
 }
```

#### Асимптотика решения

Время: O(N), где N - это количество элементов в связном списке

Память: O(N)

### Два указателя

Решение, не требующее дополнительной памяти, основывается на алгоритме двух указателей.

Суть его в том, что в начале мы заведем два укуазателя, один будет двигаться с шагом 1 (медленный), а другой с шагом 2 (быстрый).
Таким образом, в тот момент, когда быстрый указатель достигнет конца списка, медленный будет указывать на середину списка.

Далее нам надо выделить два списка (первую половину и вторую) и 'развернуть' вторую половину (от медленного указателя и дальше).

Для этого пройдем по второй половине списка (от slow) и перекинем ссылки на next.
При этом ссылку на next у slow занулим, грубо говоря вторую половину явно выделим.

После первого шага:

```text
              slow
               |
[3] -> [2] -> [1] -> [2] -> [3]
```

Второй шаг даст нам:

```text
head
 |
[3] -> [2] -> null

               prev
                |
null <- [1] <- [2] <- [3]
```

В конце просто пройдемся по обеим половинам и сравним элементы:

```java
public static boolean isPalindrome(ListNode head) {
    // Шаг 1
    ListNode slow = head;
    ListNode fast = head;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }

    // Шаг 2
    ListNode prev = slow;
    ListNode reverseNode = slow.next;
    prev.next = null;

    while (reverseNode != null) {
        ListNode tmp = reverseNode.next;

        reverseNode.next = prev;
        prev = reverseNode;
        reverseNode = tmp;
    }

    // Шаг 3
    while (prev != null && head != null) {
        if (prev.val != head.val) {
            return false;
        }

        prev = prev.next;
        head = head.next;
    }

    return true;
}
```

#### Асимптотика решения

Время: O(N), где N - это количество элементов в связном списке

Память: O(1)
