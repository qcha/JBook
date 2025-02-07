# Пользователь с наибольшим количеством заказов

Задача с [LeetCode](https://leetcode.com/problems/customer-placing-the-largest-number-of-orders/description/).

## Условие

Имеется следующая таблица:

```sql
create table orders(
  order_number integer primary key,
  customer_number integer
);

insert into orders(order_number, customer_number) values(1, 1);
insert into orders(order_number, customer_number) values(2, 2);
insert into orders(order_number, customer_number) values(3, 3);
insert into orders(order_number, customer_number) values(4, 3);

```

Необходимо найти клиента, который разместил наибольшее количество заказов.

### Примеры

Input:

| order_number | customer_number |
|--------------|-----------------|
| 1            | 1               |
| 2            | 2               |
| 3            | 3               |
| 4            | 3               |

Output:

| customer_number |
|-----------------|
| 3               |

Объяснение: наибольшее количество заказов (2) сделал пользователь с номером 3. Его и выводим.

## Решение

Если сгруппировать по `customer_number` и применить `count` к `order_number`, то получим таблицу, где будет количество заказов каждого пользователя.

Если отфильтровать по убыванию такую таблицу и взять первую строку, то получим искомого пользователя.

```sql
select
    customer_number
from
    orders
group by
    customer_number
order by
    count(order_number) desc
limit
    1
```
