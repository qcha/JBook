# Количество покупок покупателя

## Условие

Имеется таблица customers, в которой есть 3 столбца:

```sql
create table customers(
    customer_id int not null,
    item_id int not null,
    quantity int not null
);
```

В таблице отсутствует `primary key`.
В строках может повторяться id, то есть один покупатель мог приобрести разные `item_id`.

Нужно вывести тех покупателей, у которых количество приобретенных товаров больше 300.

## Решение

Сгруппировав по пользователям надо проссумировать количество приобретенных товаров, для этого можем воспользоваться агрегатной фукнцией `sum`.

```sql
select 
    customer_id,
    sum(quantity) 
from
    customers
group by
    customer_id
having
    sum(quantity) > 300
```
