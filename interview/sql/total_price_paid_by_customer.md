# Количество покупок покупателя

Задачи подобного плана показывают то, как человек умеет работать с `sql`, понимает ли как работают агрегатные функции .

## Условие

Имеется таблица customers, в которой есть 3 столбца:
* customer_id - int
* item_id - int
* quantity - int

Схема:
```postgresql
create table customers(
    customer_id int not null,
    item_id int not null,
    quantity int not null
);
```

Primary key отсутствует.
В строках может повторяться id, то есть один покупатель мог приобрести разные item_id.

Требуется для каждого покупателя определить суммарное количество его покупок. Нужно вывести тех покупателей, у которых количество приобретенных товаров больше 300.

## Решение

```postgresql
select customer_id, sum(quantity) 
from customers
group by customer_id
having sum(quantity) > 300
```
