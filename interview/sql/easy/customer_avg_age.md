# Средний возраст покупателей часов smartwatch

Задача с [Youtube](https://youtu.be/6WI_rsiD8Jw?t=2453) как задача с собеседования.

## Условие

Имеются следующие таблицы:

```sql
create table customer(
  customer_key int primary key,
  name text,
  age int
);
  
create table purchase(
    purchase_key primary key int,
    customer_key int,
    product_key  int,
    quantity     int,
    date         timestamp
);
    
create table product(
    product_key  int primary key,
    category_key int,
    name         text,
    price        int
);

create table product_category(
    category_key int primary key,
    category     text,
);

insert into 
    product_category (category_key, category)
values
    (1, 'electronics'),
    (2, 'clothing'),
    (3, 'sports'); 

insert into
    product (product_key, category_key, name, price) 
values
    (1, 1, 'smartwatch', 15000),
    (2, 1, 'smartphone', 50000),
    (3, 2, 'jacket', 8000),
    (4, 3, 'yoga mat', 2000);  

insert into 
    customer (customer_key, name, age)
values
    (1, 'Алексей', 28),
    (2, 'Мария', 34),
    (3, 'Дмитрий', 45),
    (4, 'Ольга', 22);  

insert into 
    purchase (purchase_key, customer_key, product_key, quantity, date) 
values
    (1, 1, 1, 1, '2024-03-15'),
    (2, 2, 1, 1, '2024-07-20'),
    (3, 3, 2, 1, '2024-05-10'),
    (4, 4, 1, 1, '2023-11-01'),
    (5, 1, 3, 2, '2024-01-05');
```

Необходимо посчитать средний возраст клиентов, купивших smartwatch (использовать наименование товара в product.name) в 2024 году.

## Решение

Здесь достаточно просто объединить таблицы, отфильтровать необходимые нам продукты и применить фильтр на даты.

```sql
select
    avg(c.age) as avg_age
from
    purchase p
inner join
    customer c
on
    p.customer_key = c.customer_key
inner join
    product pr
on
    p.product_key = pr.product_key
where
    pr.name = 'smartwatch'
and
    p.date >= '2024-01-01'
and
    p.date < '2025-01-01';
```

Отмечу, что можно было бы использовать и extract(year from p.date) = 2024, и date_part('year', p.date) = 2024, но фильтрация в нашем варианте, как по мне, нагляднее, и при наличии индексов на p.date даст производительность выше.
